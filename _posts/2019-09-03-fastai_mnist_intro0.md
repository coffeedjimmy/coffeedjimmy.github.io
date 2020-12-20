---
layout: post
title: "Fast AI로 MNIST 학습시키기"
tags: [fastai, 머신러닝, MNIST]
comments: true
---

.

머신러닝을 접해본 사람이라면 누구나 Keras, Tensorflow, Pytorch 등의 프레임워크에 대해 들어보거나 써봤을 것이다. 이런 프레임워크들은 딥러닝 모델의 학습 기반이 되는 Backpropagation들을 모델러들이 굳이 한땀한땀 짜지 않고 손쉽게 모델을 짤 수 있도록 돕는다. 

이렇게 다양한 프레임워크들이 있는데, Fast AI란 또 무엇인가 싶을 수 있다. Fast AI는 제레미 하워드라는 사람이 만든 Pytorch의 상위 wrapper라고 생각하면 이해하기 쉽다. Pytorch를 한번 더 감싸서 많은 것들을 자동화 시켜주고 모델러들이 핵심(데이터)에 집중할 수 있도록 하는 것을 목표로 하는 듯 보인다.

사실 말로만 들으면 잘 안 와닿지 않는가? 그래서 머신러닝/딥러닝의 Hello, World와 같은 MNIST 데이터셋 분류를 학습시키면서 Fast AI에 대한 소개를 하고자 한다. 다만, 아직 신생(이제서야 v1)인 프레임워크인 탓에 메서드의 이름 변화 등이 잦고, 아마 인터넷에는 지난 버전을 기반으로 한 튜토리얼들이 많을 것으로 생각해 최신 버전의 Fastai에 맞춰 튜토리얼을 작성하고자 한다.

## 1. 데이터셋 가져오기

    %reload_ext autoreload  # Reload an IPython extension by its module name.
    %autoreload 2  # 파이썬 코드를 실행하기 전에 항상 모든 모듈을 Reload
    %matplotlib inline  # plot을 노트북 자체에 보이도록

아마도 많은 모델러들이 Jupyter Notebook을 기반으로 작업을 할 것이라고 생각하므로 위와 같은 옵션들을 추가해준다. 캐글을 접해본 독자라면 위의 magic command들은 익숙할 것이다. 

이젠 실제 fastai를 불러와야할 시간이다.

    from fastai.vision import *

fastai는 자연어처리, 추천 관련 모듈도 제공하는데 우리는 이미지를 다룰 것이니 [fastai.vision](http://fastai.vision) 을 임포트하고, 튜토리얼인만큼 편의를 위해 모든 것을 임포트 한다.

    path = untar_data(URLs.MNIST)  # MNIST 데이터 다운로드 + untar
    # URLs.MNIST : 'https://s3.amazonaws.com/fast-ai-imageclas/mnist_png'

[untar_data](https://s3.amazonaws.com/fast-ai-imageclas/mnist_png) 는 넘겨받은 URL에 있는 데이터를 다운 받은 후 압축을 풀어준다. 그 후 파이썬 Path 객체를 리턴하게 된다. (pathlib 모듈은 경로를 문자열이 아닌 객체로 다룸)

    path.ls()
    
    [PosixPath('/home/deploy/.fastai/data/mnist_png/testing'),
     PosixPath('/home/deploy/.fastai/data/mnist_png/training')]

이렇게 하면 자동으로 training/testing 셋으로 데이터가 나뉘어져있는 것을 볼 수 있다. 이제 이 이미지들을 다루기 위해서 fastai는 [ImageList](https://docs.fast.ai/vision.data.html#ImageList) 라는 클래스를 활용한다. (이미 fastai에 대해 아는 분은 이 클래스가 ItemList의 서브클래스라고 생각하면 된다.). 그리고 우리는 데이터가 이미 정돈된 폴더로 구성되어 있다는 점을 확인했는데, 이 경우 손쉽게 데이터를 구성할 수 있는 `from_folder` 메서드가 있다.

    image_list = ImageList.from_folder(path, convert_mode='L')

`ImageList` 를 활용해서 이미지를 열 때, `PIL.Image.convert` 옵션을 같이 가지고 열게 되는데, 위에서 나온 convert_mode는 이 PIL의 convert에 전달되게 된다. 그러니 다음과 같이 설정해도 된다.

    image_list = ImageList.from_folder(path)
    image_list.convert_mode='L'  # Luminance(휘도), Grayscale

그런데 저렇게만 하고 이미지를 `.show()` 를 통해서 보면 흑백으로 보이지 않는다. 사실 저 `convert_mode='L'` 는 빛의 밝기로 변환하겠다는 뜻이므로 색 자체 표현을 `binary` 로 변환해줘야 한다. fastai에서는 defaults 네임스페이스를 정의하고, cmap을 파라미터로 받는다. 

    defaults.cmap = 'binary'

이제 이 이미지 리스트를 가지고 폴더 이름에 맞게 training/testing으로 분리한다.

    splitted_dataset = image_list.split_by_folder(train='training', valid='testing')

training/testing 폴더 안을 살펴보면 다음과 같이 나온다.

    (path/'training').ls()
    
    [PosixPath('/home/deploy/.fastai/data/mnist_png/training/2'),
     PosixPath('/home/deploy/.fastai/data/mnist_png/training/7'),
     PosixPath('/home/deploy/.fastai/data/mnist_png/training/9'),
     PosixPath('/home/deploy/.fastai/data/mnist_png/training/8'),
     PosixPath('/home/deploy/.fastai/data/mnist_png/training/5'),
     PosixPath('/home/deploy/.fastai/data/mnist_png/training/3'),
     PosixPath('/home/deploy/.fastai/data/mnist_png/training/4'),
     PosixPath('/home/deploy/.fastai/data/mnist_png/training/6'),
     PosixPath('/home/deploy/.fastai/data/mnist_png/training/0'),
     PosixPath('/home/deploy/.fastai/data/mnist_png/training/1')]

이제 폴더 이름인 레이블을 그 폴더 안에 있는 데이터와 매핑시키는 과정을 진행한다.

    labeled_list = splitted_dataset.label_from_folder()
    
    labeled_list.train[0]  # 확인
    
    (Image (1, 28, 28), Category 2)

이제 약간의 가공을 위해 transform을 선언해 봅시다. [rand_pad](https://docs.fast.ai/vision.transform.html#rand_pad) 을 사용해서 random crop과 패딩을 적용할 수 있습니다. 

    tfms = rand_pad(padding=3, size=28, mode='zeros')
    ll = ll.transform(tfms)  # transform을 적용한 labeled_list

이렇게 구성된 데이터셋을 학습에 사용할 수 있도록 배치크기로 나누고 normalize하는 과정 등을 진행해야하는데, fastai에서는 `databunch` 을 이용해서 손쉽게 진행할 수 있다.

    bs = 128  # batch size
    
    data = ll.databunch(bs=bs).normalize()  # 배치 크기로 나누고 normalize를 진행
    
    # 학습셋 접근
    data.train_ds
    
    # 테스트셋 접근
    data.test_ds

## 2. 모델 구성하기

이제 간단한 CNN 모델을 구성해서 분류를 진행해본다.

    def conv(ni, nf):
        return nn.Conv2d(ni, nf, kernel_size=3, stride=2, padding=1)

모델 구성 자체를 심플하게 만들기 위해서 `conv` 를 하나 선언해서 `nn.Conv2d`를 한번 감쌌다. 이제 이를 이용해서 [batchnorm](https://pytorch.org/docs/stable/nn.html)을 포함한 모델을 구성해보자

    model = nn.Sequential(
        conv(1,8),  # stride=2이므로 28->14로 축소
        nn.BatchNorm2d(8),
        nn.ReLU(),
        conv(8,16),  # 7
        nn.BatchNorm2d(16),
        nn.ReLU(),
        conv(16,32),  # 4
        nn.BatchNorm2d(32),
        nn.ReLU(),
        conv(32,16),  # 2
        nn.BatchNorm2d(16),
        nn.ReLU(),
        conv(16,10),  # 1
        nn.BatchNorm2d(10),
        Flatten()
    )

이제 이 모델을 가지고 fastai 모델학습의 기반인 `Learner` 를 활용해 학습을 진행해보려 한다.

    learn = Learner(data, model, loss_func=nn.CrossEntropyLoss(),
                    metrics=accuracy)

이제 학습을 진행하면 되는데, 모델 학습에 있어서 가장 간단히 조절할 수 있으면서도 중요한 하이퍼 파라미터는 바로 `learning rate` 이다. fastai에서는 어떤 lr이 가장 적합한지를 찾아주는 `lr_find` 메서드를 제공한다. 나중에 한번 다루게 될 것 같지만, 이 `lr_find` 메서드는 Leslie N. Smith의 논문인 "Cyclical Learning Rates for Training Neural Networks" 에 나온 방법론을 따랐다. 쉽게 말하면 single mini-batch SGD를 매우 작은 lr에서부터 점점 키워가며 진행하고 그 loss들을 기록해놓은 다음에 loss가 지속해서 감소하는 과정에 있는 lr들 중에서 가장 큰 lr을 찾는 것이다. 직관적으로 봤을 때, loss가 계속 감소한다는 것은 수렴을 향해 간다는 것이고, 그 중에서 가장 큰 lr을 찾는 것은 수렴 속도를 빠르게 하기 위함이다.

    learn.lr_find(end_lr=100)

위의 과정을 통해 찾은 lr은 다음과 같이 확인할 수 있다.

    learn.opt.lr
    
    4.0000000000000003e-07

이제 마지막으로 학습을 진행하면 다음과 같이 된다.

    learn.fit_one_cycle(3, max_lr=0.1)  # 3 epochs
    
    epoch	train_loss	valid_loss	accuracy
    1	0.223167	0.217859	0.930500
    2	0.136179	0.078651	0.976400
    3	0.072080	0.038664	0.988600

사실 이렇게만 하면 그래 되는건 알겠어 근데 그래서 뭐? 이런 질문이 들 수 있다. 세부적인 내용이 많이 생략되었기 때문인데, 그런 내용들은 이어지는 포스팅들을 통해서 채워나갈 예정이다.

끝.

