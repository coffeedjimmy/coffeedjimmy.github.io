---
title: 머신러닝에 필요한 선형대수 1
date: 2019.09.01 12:00:00
categories:
- Linear Algebra
tags:
- Linear Algebra
---

머신러닝에 필요한 선형대수 1편

머신러닝/딥러닝에 대한 관심이 늘어나면서 최근에는 학부과정에서도 머신러닝/딥러닝과 같은 교과과정이 들어가거나 아예 과 이름을 변경하는 경우가 국/내외에서 눈에 띄게 보이기 시작했다. 이러한 학과들에서 어느정도 체계화된 커리큘럼에 따라 교육을 받은 분들은 어느 정도의 기초는 닦였다고 생각한다. 다만, 머신러닝 자체의 붐이 인지 크게 오래 되지 않은 상황에서 일단 응용하는 측면부터 시작한 분들에게는 어느 순간 이 학문의 기초 지식에 대한 니즈가 커지는 시점이 오는 것 같다.

나 또한, 그런 찜찜함?을 마음 한 켠에 담아두고 있었기에 이번 기회에 한번 정리를 해보려는 목적에서 이번 시리즈를 시작하고자 마음 먹었다. 전반적인 공부에 있어서는 `프로그래머를 위한 선형대수`를 바탕으로 진행했다는 점을 참고해주길 바란다.

시리즈 이름에 있어서 좀 더 fancy한 이름을 붙이고 싶었지만, 매우 솔직담백하게 `머신러닝에 필요한 선형대수` 로 이름 붙였다.

## 1. 우리는 왜 선형대수를 배워야 할까

### 1. 공간이라고 생각하면 직관을 적용할 수 있다.

데이터를 다루다 보면 단일 수치가 아닌, 다수의 수치를 조합한 데이터를 다루고 싶은 경우가 분명 생긴다. 사실 이 몇 가지의 데이터 조합을 굳이 `공간` 이라는 개념과 매핑시키지 않고 다룰 수도 있지만, 만약 이를 `고차원 공간 내의 점` 의 측면에서 바라보게 되면 우리가 직관적으로 공간에 대해서 가지고 있는 생각들을 적용해볼 수 있다. 그래서 공간이라는 시각에서 하나 이상의 데이터 조합을 바라보게 되면 우리는 3개의 데이터로 이루어진 공간의 한 점을 상상할 수 있다. 그렇다면 우리가 상상할 수 없는 4차원, 5차원, N차원 등은 어떨까?

물론 모든 경우에 성립하는 말은 아니지만 우리가 이해할 수 있는 3차원 공간으로부터 유추하여 N차원에도 적용시킬 수 있는 현상이 많다. 우리는 이런 부분에 좀 더 집중해서 선형대수를 배우면 좋지 않을까 생각한다.

### 2. 근사 수단으로 사용하기 쉽다.

선형 대수가 다루는 것은 바로 `선형`으로 이루어진 것들이다. 선형대수를 간단히라도 접한 사람들은 2차 함수와 같이 휘어진 곡선에도 선형대수를 들이미는 것을 본 적이 있을 것이다. 그렇다면 당연하게도 이런 의문이 들 수 있는데, 선형만 다룬다면서 왜 곡선에도 적용하는거지? 와 같은 의문이다.

이는 의외로 쉽게 받아들여지는 부분이다. 곡선도 확대해보면, 그리고 범위를 매우 축소하면 그 범위 안에서는 결국 선형이기 때문이다. 그리서 곡선 또한 작은 평면의 조립으로 근사 표현할 수 있다. 어쩌면 이런 무책임한 접근이 어디있나 생각할 수도 있지만, 특정한 가설에 대한 식을 세우기 힘들 때, 일단 선형으로 근사해보자와 같은 접근은 의외로 자주 쓰이는 방법이다. 우리의 목적은 이런 부분들이 수학적으로 엄밀히 성립하느냐라기보다는 머신러닝 이해에 필요한 부분을 채우는 것이기 때문에 이 부분은 살포시 넘어가도록 하겠다.

대학생들의 개강 시점이 다가오는데 보통 개강 후 첫 수업도 OT 개념으로 간단히 소개만 하고 끝내지 않는가. 시리즈 1번 째 글에 구구절절 길게 내용을 작성하면 다들 아 안읽어 이럴까봐 첫 번째 글은 매우 심플하게 이쯤에서 마무리하고자 한다. 이번 시리즈는 부디 뚝심있게 마무리할 수 있기를 바라며.