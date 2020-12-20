---
title: 오픈소스의 컨트리뷰터가 되는 법
date: 2018.03.12 12:00:00
categories:
- Opensource
tags:
- GIT
---


지난 3월 10일 토요일, 모두가 만나고 싶어하는 `김영근` 님과 함께하는 `pandas` docstring 스프린트가 있었습니다. 크게 엄청난 뜻이 있어서 참여했다기 보다는 평소에 데이터 분석을 하는 데에 자주 쓰던 판다스에 직접 기여할 수 있는 기회라는 것과 김영근님을 뵐 수 있다는 점에 끌려서 참여신청을 하게 되었습니다.

>  이 글을 쓰고 있는 시점에서는 아직 머지가 되지 않았지만 (아직 고쳐야될 점이 많은 커밋이지만), 완료되는 대로 커밋로그를 자랑해 보겠습니다...



먼저 오픈소스에 컨트리뷰트하는 것이 무엇인지 모르시는 분들을 위해 간단히 설명하자면, `Contribute` 한다는 것은 오픈소스에 일정 부분 기여를 한다는 것입니다. 그것이 소스코드에 있는 버그를 수정한다거나 저 같은 경우처럼 docstring 을 수정하는 것과 같이 다양한 형태로 기여할 수 있지만, 크게 이 오픈소스 프로젝트가 발전하는 것에 기여한다는 점에서 그 궤를 같이 합니다.

이와 같은 이해를 바탕으로 컨트리뷰트 하려면 다음과 같은 것들이 필요합니다.

- 기여하고 싶은 오픈소스 프로젝트
- 깃에 대한 이해
- 깃헙 아이디

> 이에 관련한 자세한 내용들은 [sprint guideline](https://python-sprints.github.io/pandas/guide/pandas_setup.html) 을 참조하였습니다. 또한, 로컬에 깃은

### 1. 깃헙 아이디 만들기

이 과정은 여느 사이트에 회원하는 것과 다르지 않습니다. [깃허브]( https://github.com/join)

### 2. 오픈소스 프로젝트 레포 포크하기  

오픈소스 프로젝트에 컨트리뷰트 하기 위해서는 그 프로젝트의 원본 레포에 어떤 변형을 가하는 것이 아니라 그것의 사본을 가져와서 바꾸고 싶은 부분을 바꾼다음 *Pull Request* 라는 과정을 통해 진행해야 합니다. 그 사본을 가져오는 것을 `Fork` 라고 하는데 아주 간단합니다.

![포크하는 방법](/images/pandas-example.png)

> 이제부터 프로젝트를 판다스라고 생각하고 설명을 진행하겠습니다.

위와 같이 판다스 프로젝트의 오른쪽 부분에 보면 `Fork` 라는 부분이 있습니다. 이 버튼을 클릭하면 본인의 깃헙에 프로젝트의 사본이 만들어집니다.

그런 다음 코드나 docstring을 추가하기 위해서는 이 사본을 로컬로 가지고 와야하는데요. 다음과 같이 진행하면 됩니다.

```bash
$ git clone https://github.com/<your-github-username>/pandas
```

그런 뒤에 로컬으로 가져온 폴더로 이동한 뒤,

```bash
$ cd <pandas-dir>
```

원래의 판다스 레포를 `upstream` 으로 지정해 줍니다.

```bash
$ git remote add upstream https://github.com/pandas-dev/pandas
```

이후에 기존의 판다스 레포와의 Sync를 맞추기 위해서는 다음과 같은 명령어들을 실행해 주면 됩니다.

```bash
$ git fetch upstream
$ git checkout master
$ git merge upstream/master
```