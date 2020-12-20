---
layout: post
title: "[TF-Serving] TF-Serving 쉽게 사용하기"
tags: [tensorflow serving, tensorflow]
comments: true
---

> 이 글은 도커 환경에서 Tensorflow-Serving을 설치하고 모델을 서빙하는 과정을 다룹니다. SavedModel을 이용한 모델 저장부분은 공식 홈페이지를 참고해주세요.

> 이 글에서 사용하는 Tensorflow-Serving version은 **r1.11**입니다. 또한, 텐서플로 공식 이미지로 컨테이너를 만드는 것을 기준으로 합니다.

도커 사용법은 [이 곳](http://pyrasis.com/Docker/Docker-HOWTO#commit)을 참고하시고, 제가 처음 TF-Serving을 접할 때 참고한 자료는 [이 곳](http://xoit.tistory.com/34)입니다.

먼저 텐서플로 서빙의 버전과 텐서플로 버전이 명시적으로 맞지는 않아도 됩니다. `docker pull`을 이용해 텐서플로 이미지를 다운받고 컨테이너를 생성해 주세요. 앞서 언급했지만, 이 글에서는 TF-serving r1.11버전을 사용할 것입니다.

다른 버전이 아닌 r1.11을 사용한 데에는 여러가지 이유가 있지만, 주로 사용할 rest api 쪽이 기존에 사용하던 r1.9에서는 아직 제대로 구현되지 않은 부분들이 많아서 결국 버전업을 결정하게 되었습니다.

### Prerequisite

```bash
$ apt-get update

$ apt-get install git pkg-config zip g++ zlib1g-dev unzip wget automake libtool

$ apt-get autoremove

$ apt-get clean

```

위와 같은 커맨드들로 필요한 것들을 설치해 줍니다.

### Bazel 설치

텐서플로 서빙을 빌드하기 위해서는 Bazel이 필요합니다. 기존에 제가 사용했던 bazel 버전은 0.10.0이었지만(TF-r1.9 기준 동작), r1.11로 버전업을 하면서 빌드해보니 오류가 나서 0.18.0버전을 사용했습니다.

```bash
$ wget https://github.com/bazelbuild/bazel/releases/download/0.18.0/bazel-0.18.0-installer-linux-x86_64.sh

$ chmod +x bazel-0.18.0-installer-linux-x86_64.sh

$ ./bazel-0.18.0-installer-linux-x86_64.sh
```

위의 커맨드들을 이용해서 bazel을 설치할 수 있습니다. 이제 bazel이 제대로 설치되었는지 확인해보겠습니다.

```bash
$ bazel version
```

필요없는 설치 파일들을 지워주는 과정은 다음과 같습니다.(~~굳이 진행하지 않아도 됩니다.~~)

```bash
$ rm -rf bazel-0.18.0-installer-linux-x86_64.sh

$ rm -rf tensorflow_gpu-1.9.0-cp27-cp27mu-manylinux1_x86_64.whl

```

### Tensorflow-Serving 설치

이제 본격적으로 텐서플로 서빙을 설치하려고 합니다.

깃에서 텐서플로 서빙을 클론해옵니다.

```bash
$ git clone https://github.com/tensorflow/serving.git
```

그리고선 빌드하고 싶은 텐서플로 서빙 버전으로 checkout합니다.

```bash
$ git checkout r1.11
```

그 뒤에 `/serving` 폴더로 들어가서 `bazel clean`을 해줍니다.

```bash
$ cd serving

$ bazel clean
```

여기서 두 가지로 나뉩니다. 텐서플로 서빙 레포에 들어있는 모든 것을 빌드하면 위의 커맨드, 모델 서버만 빌드하려면 아래의 커맨드를 실행하면 됩니다. **위의 커맨드를 실행하면 시간도 오래걸리고 모든 테스트 코드까지 다 빌드 되어서 용량도 커집니다. 궁금해서 하는 것이 아니라면 모델 서버만 빌드하시는 것을 추천합니다.**

```bash
$ bazel build -c opt tensorflow_serving/...  // 전체 빌드
```

```bash
$ bazel build //tensorflow_serving/model_servers:tensorflow_model_server  // 모델 서버만 빌드
```

이제 제대로 모델 서빙이 되는지 체크하려면 다음과 같이 모델 서버를 동작시키면 됩니다.

```bash
$ /serving/bazel-bin/tensorflow_serving/model_servers/tensorflow_model_server
--port=9000  --rest_api_port={원하는 REST API port} --model_name={모델이름}
--model_base_path={모델이 저장된 폴더} &
```

이렇게 하면 저장했던 모델을 서빙할 수 있습니다. 이 서버에 inference call을 날리는 방법은 [다음 글](https://coffeedjimmy.github.io/2018/07/01/TF-Serving-RESTful/)을 참고해 주세요.

Tensorflow-Serving은 모델을 숫자로 저장하면 지정된 폴더에 새로 모델이 추가될 때마다 버전업을 자동으로 해준다는 점과 여러 다른 장점들을 가지고 있습니다. 기존에 Flask나 직접 서버를 띄워서 모델을 서빙하는 것에 부담을 느끼시던 분들은 한번 시도해 보셔도 좋을 것 같습니다.

.
