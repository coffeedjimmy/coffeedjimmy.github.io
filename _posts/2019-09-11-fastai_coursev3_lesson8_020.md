---
layout: post
title: "[Fast.ai] Lesson 8 (2부)"
tags: [fastai, 머신러닝, course-v3, lesson8]
comments: true
---

이 포스트는 Fastai course-v3의 lesson 8 내용을 정리한 내용을 담고 있습니다.

사실 이 8강의 후반부는 weights initialisation에 대한 내용이 주를 이룬다. 이 부분을 Fastai 포스트에 자세히 처음부터 다루기에는 무리가 있어보여, 일단 여기서는 제레미가 언급하는 내용들만 다루기로 하고 전반적인 흐름 설명에 대해서는 별도의 포스트를 통해 정리하고자 한다. 자 그럼 시작해보자.

우리가 신경망에 대해서 어느정도 공부하다 보면 무조건 마주하게 되는 내용이 초기화가 중요하다는 부분일 것이다. 그래 뭐든 처음 어떻게 해놓는지가 중요하지라고 그냥 넘기기에는 신경망에서의 초기화의 위치가 생각보다 중요하다. 2019년 1월에 나온 [논문](https://arxiv.org/abs/1901.09321)을 보게 되면, weight 초기화만 잘해주면 normalization 없이도 Resnet SOTA 성능을 얻을 수 있다는 내용까지 나온다. 이 정도면 초기화가 얼마나 중요한지 느껴지지 않는가?

왜 시작부터 초기화 얘기를 하냐면, 8강의 후반부 내용이 이 초기화에 대해 다루기 때문이다. 다들 이미 알겠지만, weight가 너무 크거나 작아서 activation 값이 극단으로 가게 되면 `Vanishing Gradient` 가 일어나고, weight가 크고 activation 값의 gradient가 클 경우에 `Exploding Gradient`가 일어난다. 둘 중에서 그래도 우리가 자주 마주치는 `Vanishing Gradient`의 경우에는 신경망이 깊어질 수록 초기 레이어의 학습이 더뎌지고, 이는 입력값에 대한 정보를 많이 잃어버리게 됨을 뜻한다. 그래서 아무리 후반 레이어의 학습이 잘 일어나더라도 입력값에 대한 정보량 자체가 작게 반영되므로 모델의 성능이 좋지 않을 수 밖에 없는 것이다. 

그래서 우리는 이제 간단한 레이어의 상황을 보며 초기화를 어떻게 하는게 좋은지 테스트 해보자.

```python
# x_valid.shape: torch.Size([10000, 784])

hidden = 50
w1 = torch.randn(x_valid.shape[1], hidden)
b1 = torch.zeros(hidden)

print(w1.mean(), w1.std())
# (tensor(-0.0033), tensor(1.0015))

print(x_valid.mean(), x_valid.std())
# (tensor(-0.0057), tensor(0.9924))
```

정규분포로 초기화를 했기 때문에 평균은 거의 0, 표준편차는 1에 가까운 것을 볼 수 있다.

```python
def lin(x, w, b):
        return x@w + b

t = lin(x_valid, w1, b1)
print(t.mean(), t.std())
# (tensor(4.8306), tensor(28.4001))

activ = torch.nn.Tanh()
t2 = activ(t)
print(t2.mean(), t2.std())
# (tensor(0.1465), tensor(0.9750))
```

단순 수치만 보기에 평균이 0에 가깝고, std가 1에 가까우니 잘 초기화가 된 것 같다고 생각할 수도 있지만, 실상은 다르다. 

`Tanh` 통과 전의 std를 보게 되면 매우 큰 것을 확인할 수 있다. 이는 대다수의 linear activation 값들이 activation function의 양 극단에 치우침을 뜻하고, 이는 결국 `gradient vanishing`으로 이어진다. 그래서 우리가 알다시피 `Sigmoid`나 `Tanh`가 아닌 `ReLU`를 사용하는데, 그럼 ReLU를 한번 사용해보자.

```python
activ = torch.nn.ReLU()
t3 = activ(t)
print(t3.mean(), t3.std())
# (tensor(10.7017), tensor(16.2206))
```

실제로 plot을 그려보면 0보다 작은 많은 뉴런들이 꺼뜨려지지만, 그래도 0보다 큰 값을 가지는 뉴런들은 결국 1이라는 gradient를 가지게 되므로 앞선 문제는 해결이 되었다. 그렇지만 이제는 activation value가 더 이상 normal distribution을 따르지 않게 되었다.

그럼 다른 방법이 없을까?

조금 옛날이긴 하지만 유명한 LeCun 교수가 주장 했었던 초기화 방법을 한번 사용해보자.

```python
# Lecun Initialisation

w1 = torch.randn(m, hidden)/math.sqrt(m)  # fan_in
b1 = torch.zeros(hidden)
w2 = torch.randn(hidden,1)/math.sqrt(hidden)  # fan_in
b2 = torch.zeros(1)

t = lin(x_valid, w1, b1)
t.mean(), t.std()
# (tensor(0.0426), tensor(1.0264))

activ = torch.nn.Tanh()
t2 = activ(t)
t2.mean(), t.std()
# (tensor(0.0254), tensor(1.0264))
```

앞이랑 같지 않냐고 생각할 수 있지만, 실제로 plotting 해보면 -1과 1로 치우치는 현상이 매우 개선된 걸 볼 수 있다. 그렇지만 본질적인 `Tanh`의 문제점은 남아 있으므로 `ReLU`로 바꿔보자.

```python
activ = torch.nn.ReLU()
t3 = activ(t)

t3.mean(), t3.std()
# (tensor(0.4301), tensor(0.6140))
```

`Tanh` 사용 시에는 std가 잘 유지되었지만, ReLU를 쓰니 std가 반쪽이 되었다. 이에 대해 Kaiming He라는 사람이 간단하면서도 똑똑한 아이디어를 냈다. 두 배를 해보자!

```python
# Kaiming He initialization

w1 = torch.randn(m, hidden)*math.sqrt(2/m)

w1.mean(), w1.std()
# (tensor(-0.0002), tensor(0.0503))

t1 = relu(lin(x_valid, w1, b1))
t1.mean(), t1.std()
# (tensor(0.5567), tensor(0.8172))
```

이전보다 훨씬 더 분산 및 std를 유지하게 되었다. 이렇게 매뉴얼하게 쓸 것이 아니라 pytorch에는 이미 이런 초기화 방법들이 구현되어 있다.

```python
from torch.nn import init

w1 = torch.zeros(m, hidden)
init.kaiming_normal_(w1, mode='fan_out')
t = relu(lin(x_valid, w1, b1))
```

그런데 왜 `fan_in`이 아니고 `fan_out`일까? 이건 Pytorch 구현을 자세히 살펴보면 이해가 가능하다.

```python
import torch.nn

torch.nn.Linear(m, hidden).weight.shape
# torch.Size([50, 784])

torch.nn.functional.linear??
# ret = torch.addmm(bias, input, weight.t())
```

분명 (784,50)을 기대하고 만들었는데 (50,784)가 나온 것을 알 수 있다. 이는 pytorch 내부 구현 상에서 weight를 transpose 한 다음 계산을 하기 때문임을 알 수 있다.

그럼 다시 돌아가서 std가 `He initialization`을 통해서 좀 개선 되었는데, 좀 더 개선시킬 수 있는 방법이 없을까?라는 의문이 들 수 있다. 제레미 또한 이런 의문에 따라, ReLU를 조금 개선하는 방향으로 테스트를 진행해봤다고 한다.

```python
def relu(x):
    return x.clamp_min(0.) - 0.5

w1 = torch.randn(m, hidden)*math.sqrt(2./m)
t1 = relu(lin(x_valid, w1, b1))
t1.mean(), t1.std()
# (tensor(0.0905), tensor(0.8547))
```

매번 조금씩 달라지기는 하지만, 기존의 `He initializatin +  ReLU` 조합 보다 조금 더 나아졌다는 걸 볼 수 있다. 그렇지만 제레미가 말하듯이 무조건적인 답은 없고 이것저것 더 나은 것이 없는지 시도해 보는 것이라고 보면 될 것 같다.

이 뒤에 이어지는 내용들은 Fully Connected Layer를 구성하기 위한 Loss function 구현, Gradient & backprop 등에 대한 내용들이므로 아래의 커널 링크를 통해서 직접 실습 과정으로 배우는 것이 나을 것이라 생각한다.

고작 2시간 짜리의 한 강의인데 생각보다 깨달은 점과 놓치고 있는 포인트들을 많이 찾을 수 있었다. 정말 8강은 꼭 Fastai에 관심이 없더라도 한번 쯤 공부해보면 좋을만한 강의라고 다시 한 번 강조하며 이번 포스팅을 마친다.

[캐글 커널 링크](https://www.kaggle.com/coffeedjimmy/fastai-course-v3-02-fully-connected)


lesson8 끝.