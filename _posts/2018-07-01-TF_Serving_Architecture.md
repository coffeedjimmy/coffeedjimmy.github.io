---
title: TensorFlow Serving Architecture
date: 2018.07.01 12:00:00
categories:
- TensorFlow Serving
tags:
- TensorFlow
---

.

# Architecture Overview

Tensorflow Serving은 실제 배포환경을 위해 디자인되었고, 머신러닝 모델을 서빙하는데 있어서 유연하고, 높은 성능을 보입니다. Tensorflow Serving은 서버 아키텍쳐와 API들을 유지하면서도 새로운 알고리즘과 시도들을 손쉽게 적용할 수 있습니다. Tensorflow Serving 모델들과의 통합에 있어서 어떠한 설치나 추가적인 환경설정이 필요하지 않고, 다양한 타입의 모델들을 서빙하도록 범위를 확장할 수도 있습니다.

> An **out-of-the-box feature** or **functionality** (also called **OOTB** or **off the shelf**), particularly in software, is a feature or functionality of a product that works immediately after or even without any special installation without any configuration or modification.

## Key Concepts

Tensorflow Serving의 구조를 이해하기 위해서는 다음과 같은 주요 컨셉들을 이해해야 합니다.

### Servables

`Servables`는 Tensorflow Serving에 있는 중심적인 추상체입니다. `Servables`는 클라이언트가 연산을 수행하기 위해 사용하는 기저에 있는 객체입니다.

Servable의 크기와 세분화 정도(granularity)는 유연하게 구성할 수 있습니다. 하나의 Servable은 하나의 lookup 테이블의 일부를 가지는 것부터 하나의 모델 또는 여러 inference 모델들을 가지는 것까지 어떤 것이든 포함할 수 있습니다. Servables는 어떠한 타입이나 인터페이스가 될수 있으며 이와 함께 유연성과 다음과 같은 향후 개선가능성 또한 가집니다.

- streaming results
- experimental APIs
- asynchronous modes of operation

**Servables는 라이프 사이클을 스스로 관리하지 않습니다.**

그리고 보통의 servables는 다음과 같은 것들을 포함합니다.

- Tensorflow `SavedModelBundle` (tensorflow::Session)
- 임베딩 lookup 테이블이나 vocab lookup 테이블

**Servable Versions**

Tensorflow Serving은 한 서버 인스턴스의 라이프 사이클 동안 하나 이상의 servable을 다룰 수 있습니다. 이는 시간이 지남에 따라 로드되는 새로운 알고리즘에 대한 환경설정, 가중치 그리고 다른 데이터들의 사용을 가능하게 합니다. 이런 환경은 하나 이상의 servable들이 동시에 로드되는 것이 가능하게 해주고 또한, 점진적인 `rollout` 과 실험을 가능하게 해줍니다. 서빙 상황에서 클라이언트들은 최신버전을 요구할 수도 있고, 특정 모델에 대한 특정 버전 id를 요구할 수도 있습니다.

**Servable Streams**

`Servable Stream` 은 여러 버전의 servable들이 오름차순으로 정렬된 sequence입니다.

**Models**

Tensorflow Serving은 모델을 하나 이상의 servable들로 표현합니다. 그리고 그 각각의 모델은 아마도 하나 이상의 알고리즘들과 lookup 또는 embedding 테이블들을 포함하고 있습니다.

그렇기 때문에 `composite model`을 표현하는 방법에는 다음과 같은 것들이 있습니다.

- multiple independent servables
- single composite servable

Servable은 모델의 일부와 대응될 수도 있습니다. 예를 들어서 매우 큰 lookup 테이블은 많은 Tensorflow Serving 인스턴스들로 샤딩되어 서빙될 수 있습니다.

### Loaders

Loader는 servable의 라이프 사이클을 관리합니다. Loader API는 특정 알고리즘이나 데이터 또는 유즈케이스들이 포함된 서비스들 속에서 API가 일반적이고 인프라 독립적으로 존재할 수 있도록 해줍니다. 특히 Loader는 servable을 로드하는 것과 언로드하는 것에 대한 API들을 규격화해줍니다.

### Sources

Source는 servable들을 찾고 제공하는 플러그인 모듈입니다. 각각의 소스는 servable stream을 제공하지 않을 수도 많이 제공할 수도 있습니다. 각각의 servable stream에 대하여 Source는 각각의 버전이 로드될 수 있도록 버전마다 하나의 Loader 인스턴스를 제공합니다. (Source는 실제로 0개 이상의 SourceAdapter들과 연결되어 있고, 체인에 있는 마지막 아이템이 Loader를 `emit` 합니다.)

Source와 관련된 Tensorflow Serving의 인터페이스는 임의의 저장시스템으로부터 servable들을 찾아낼 수 있습니다. Tensorflow Serving은 Source 구현에 대한 공통된 레퍼런스를 포함하고 있습니다. 예를 들어 Source는 RPC와 같은 매커니즘에 접근하거나 파일시스템을 폴링할 수 있습니다.

Source는 다수의 servable이나 버전들에 걸쳐 샤딩되어 있는 상태를 저장할 수 있습니다. 이는 servable에게 있어 버전들 사이의 delta(차이)만 업데이트하면 된다는 점에서 유용합니다.

**Aspired Versions**

이는 준비되고 로드되어야 하는 servable 버전들의 모음을 나타냅니다. Source는 이 하나의 servable stream에 대한 servable 버전들의 모음과 커뮤니케이션합니다. 따라서 Source가 Manager에게 새로운 aspired version들의 리스트를 줬을 때, 이것은 servable stream에 대한 그 전 버전의 리스트를 대체(supercede)합니다. Manager는 새 리스트에 존재하지 않는 지난 버전들을 언로드 시킵니다.

### Managers

Manager는 다음과 같은 servable의 라이프 사이클을 관리합니다.

- Loading Servables
- Serving Servables
- Unloading Servables

Manager는 Source들을 `listen`하고 있고 모든 버전들을 추적합니다. Manager는 Source들의 요청을 해결해 주려고 시도하지만, 요청된 자원이 사용불가할 때는 `aspired version` 로드하는 것을 거부합니다. Manager는 또한 "unload"하는 것을 연기하기도 합니다. 예를 들어 적어도 하나의 버전이 항상 로드되어 있어야 한다는 정책이 있을 경우에 Manager는 새로운 버전이 로드될 때까지 "unload"하는 것을 지연시킵니다.

Tensorflow Serving Manager는 클라이언트가 로드된 servable 인스턴스에 접근할 수 있도록 `GetServableHandle()`라는 간단한 인터페이스를 제공합니다.

### Core

**Tensorflow Serving Core**는 규격화된 Tensorflow Serving API를 통해 다음과 같은 servable의 측면들을 관리합니다.

- lifecycle
- metrics

Tensorflow Serving Core는 servable과 loader을 `opaque obejcts`로 다룹니다.

## Life of a Servable

![image](https://user-images.githubusercontent.com/18420539/42131410-61cc9480-7d3c-11e8-9673-6c681408284b.png)

대략적으로 말해서...

1. Source는 Servable 버전들에 대해서 Loader들을 생성한다
2. Loader는 `Aspired Versions`을 클라이언트의 요청에 대해 이것들을 로드하고 서빙하는 주체인 Manager에게 보낸다.

좀 더 자세하게 말해서...

1. Source Plugin이 특정 버전에 대해서 Loader를 생성한다. Loader는 Servable을 로드하기 위해 필요한 metadata를 포함한다.
2. Source는 `Aspired Version`을 Manager에게 알려주기 위해 callback을 사용한다.
3. Manager는 이미 로드된 버전을 언로드할 지 또는 새 버전을 로드할 지에 대한 결정의 기준이 되는 `Version Policy`를 적용한다.
4. 만약 Manager가 이 모든 과정이 정상적이라고 판단했다면 Loader에게 요청받았던 리소스를 할당하며 새 버전을 로드하게 한다.
5. 클라이언트는 Manager에게 특정버전을 명시하거나 단순히 최신버전의 Servable을 요청한다. 이에 대해 Manager는 클라이언트에게 Servable에 대한 통제권을 반환한다.

예를 들어, Source가 빈번하게 모델 가중치가 없데이트 되는 Tensorflow graph를 표현한다고 생각해보자. 가중치들은 디스크에 있는 파일에 저장되어 있다.

1. Source가 새로운 버전의 모델 가중치를 감지한다. 이는 디스크에 있는 모델 데이터를 가리키는 포인터가 포함된 Loader를 생성한다.
2. Source는 `Aspired Version`의 Dynamic Manager에게 알린다.
3. Dynamic Manager는 Version Policy를 적용하고 새 버전을 로드할 지를 결정한다.
4. Dynamic Manager는 Loader에게 충분한 메모리가 있음을 말해주고 Loader는 Tensorflow graph를 새 가중치와 함께 초기화한다.
5. 클라이언트는 최신 버전 모델에 대한 통제권을 요청하고 Dynamic Manager는 새 버전의 Servable에 대한 통제권을 반환한다.

## Extensibility

Tensorflow Serving은 개발자가 새로운 기능을 추가할 수 있는 몇 가지 포인트들을 제공한다.

### Version Policy

Version Policy는 하나의 servable stream 안에서 로드/언로드될 버전들의 sequence들을 명세합니다.

Tensorflow Serving은 많이 알려진 유즈케이스들을 수용하고 있는 두 개의 정책을 포함하고 있습니다. 첫 번째 `Availability Preserving Policy` 는 적어도 한 개의 버전이 로드되어 있기를 원하는 정책입니다. 그래서 보통 이 정책에 따라 새로운 버전으로 교체하려는 경우 새 버전을 구 버전을 언로드하기 전에 먼저 로드하는 방식을 택합니다. 다른 하나는 `Resource Preserving Policy`입니다. 이는 동시에 두 버전이 로드되는 것을 피하는 것으로 자원이 두 배로 요청되는 것을 막는 것에 초점을 둡니다. 이 경우에는 구 버전을 먼저 언로드하고 새 버전을 로드합니다. 단순한 유즈케이스의 경우에 Tensorflow Serving에 있어서 serving availability가 더 중시되고 자원의 값은 상대적으로 낮게 고려됩니다. 따라서 이 경우에는 `Availability Preserving Policy`를 적용합니다. 다만, 다수의 서버 인스턴스들에 걸쳐 버전들을 관리하는 경우와 같이 좀 더 상세화된 사용을 위해서는 `Resource Preserving Policy`를 사용합니다.

### Source

새로운 Source들은 새로운 파일시스템, 클라우드, 알고리즘 백엔드를 지원합니다. Tensorflow Serving은 새로운 소스를 빠르고 쉽게 만들 수 있는 building block을 제공합니다. 예를 들어 Tensorflow Serving은 polling 을 래핑한 유틸리티를 제공합니다. Source는 특정 알고리즘과 데이터 호스팅 servable을 위한 Loader와 밀접하게 연관이 되어 있습니다.

### Loader

Loader에 있어서는 알고리즘 추가와 데이터 백엔드에 있어 성능 개선점이 존재합니다. Tensorflow는 그러한 알고리즘 백엔드 중 하나입니다. 예를 들어, 개발자는 새로운 다입의 servable을 위해 로드와 엑세스 그리고 언로드를 수행하는 Loader를 만들 수 있습니다.

### Batcher

다수의 요청을 배치로 처리하는 것은 inference를 수행하는 cost를 상당히 줄일 수 있고 특히 GPU가 있을 경우 그 효과가 더욱 뛰어납니다. Tensorflow Serving은 `request batching widget`을 포함하고 있고, 이는 클라이언트가 손쉽게 요청에 따라 타입이 다양한 inferences들을 배치 요청으로 만들 수 있게 해줍니다.
