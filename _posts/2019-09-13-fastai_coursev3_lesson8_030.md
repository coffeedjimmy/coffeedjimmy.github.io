---
layout: post
title: "[Fast.ai] Lesson 8 번외편"
tags: [fastai, 머신러닝, course-v3, lesson8]
comments: true
---

이 포스트는 Fastai course-v3의 lesson 8 내용을 정리한 내용을 담고 있습니다.

사실 이 내용은 실제 강의에서는 9강에서 다뤄지는 내용이다. 그렇지만 전체적인 흐름 상 제레미가 8강에서 궁금증을 가진 부분에 대해서 9강 초반에 설명해주는 방식으로 진행되기 때문에, 그리고 9강의 내용과는 직접적인 연관이 없는 것 같아서 아예 8강의 번외편으로 빼서 작성한다.

weight 초기화에 대해서 설명하던 제레미는 conv layer의 default 초기화에 대해 말하며 `math.sqrt(5)`가 어디서 왔는지 모르겠다며 리서치를 한 내용을 쭉 설명해준다. 그 내용을 쭉 따라가겠지만, 내가 개인적으로 이해한 방식으로 재구성하여 한번 설명해보려 한다.

먼저, conv 레이어의 기본 초기화가 어떻게 이루어지는지 보려면 아래와 같이 쥬피터 노트북에서 입력하면 된다.

```python
torch.nn.modules.conv._ConvNd.reset_parameters??

# init.kaiming_uniform_(self.weight, a=math.sqrt(5))
```

그러면 아래 주석과 같은 부분을 확인할 수 있다. 앞서 언급한 `math.sqrt(5)`가 `kaiming_uniform_`에 전달되는데 일단 그러면 `kaiming_uniform_`에 대해서 알아보자.

```python
init.kaiming_uniform_??

# std = gain / math.sqrt(fan)
# bound = math.sqrt(3.0) * std
```

여기서 gain은 kaiming he가 제안한 `2/math.sqrt(1+a^2)`이다. 그래서 결론적으로 Conv 레이어는 `U[-1/math.sqrt(fan_in), 1/math.sqrt(fan_in)]`이 된다.

그럼 그냥 어련히 알아서 pytorch가 잘 초기화 해줬겠지 생각하면서 쓰면 안되냐는 생각을 할 수도 있는데, 사실상 그렇지 않다. 

```python
x = x_valid[:100]

x.shape, stats(x)
# (torch.Size([100, 1, 28, 28]), (tensor(-0.0362), tensor(0.9602)))

stats(l1.weight), stats(l1.bias)
# ((tensor(0.0038, grad_fn=<MeanBackward0>),
#  tensor(0.1155, grad_fn=<StdBackward0>)),
# (tensor(-0.0041, grad_fn=<MeanBackward0>),
#  tensor(0.1107, grad_fn=<StdBackward0>)))

t = l1(x)

stats(t)
# (tensor(0.0054, grad_fn=<MeanBackward0>),
# tensor(0.6679, grad_fn=<StdBackward0>))
```

위와 같이 기본 초기화를 사용하게 되면 weight의 std가 매우 작은 값으로 설정되고, 실제 normal distribution을 가지는 데이터를 conv layer에 통과시키게 되면 std가 1에서 멀어지게 된다. 이렇게 되는 이유가 뭘까? 

```python
torch.zeros(10000).uniform_(-1,1).std()
# tensor(0.5768)

1/math.sqrt(3.)
# 0.5773502691896258

(1/math.sqrt(fan_in))/math.sqrt(3)
# 0.11547005383792516
```

코드를 보면 알 수 있다시피 `Uniform distribution [-a, a]` 의 std는 a가 아니라 `a/math.sqrt(3)`에 가까운 값이다. 그래서 위에서 weight의 std가 0.11 정도의 값이 나왔던 것이다! 

사실 pytorch 코드에는 이 Uniform distribution의 std를 보정하기 위해 math.sqrt(3)을 std에 곱해서 bound를 설정한다. (`bound = math.sqrt(3.0) * std`) 그런데 a 값으로 math.sqrt(5)가 주어지게 되면서 이 부분이 상쇄되어 버리고, 결국 아래와 같은 결과가 되게 된다.

```python
init.uniform_(l1.weight, -1/math.sqrt(fan_in), 1/math.sqrt(fan_in))
stats(l1.weight)
# (tensor(-0.0054, grad_fn=<MeanBackward0>),
# tensor(0.1160, grad_fn=<StdBackward0>))

t = l1(x)
stats(t)
# (tensor(-0.0167, grad_fn=<MeanBackward0>),
# tensor(0.7040, grad_fn=<StdBackward0>))
```

이와 관련된 [링크](https://github.com/pytorch/pytorch/issues/15314)를 좀 찾아들어가다 보면 2018년 7월까지만 해도 바로 위처럼 단순 `init.uniform_` 을 사용해서 초기화를 진행하다가 2018년 7월에, 링크에서 언급되는 사람이 `init.kaiming_uniform_`으로 변경했다. 그런데 이 과정에서 kaiming init 논문대로 구현하지 않고 기존과 같은 초기화 방식을 택했다. 다시 말해 표현 방식만 변했지, functional하게는 달라진 점이 없도록 만들었다는 뜻이다. 그 과정에서 math.sqrt(5)가 a값으로 들어가면 기존과 같은 초기화를 진행할 수 있게 되기에 `math.sqrt(5)`가 들어갔다는 내용이다. 그리고 이 초기화 방식은 Pytorch의 기반인 lua로 쓰여진 torch가 Lecun 초기화를 차용하면서 구현되었던 걸 그대로 가져와서 썼다고 한다. 이러한 문제로 기본 초기화가 아닌 kaiming_normal을 사용하면 다음과 같은 결과를 얻을 수 있다.

```python
init.kaiming_normal_(l1.weight, a=0.)
stats(f1(x))
# (tensor(0.5663, grad_fn=<MeanBackward0>),
# tensor(1.1403, grad_fn=<StdBackward0>))
```

흥미롭지 않은가? 사실 일을 하면서도 초기화가 중요하다 정도만 이해하고 있었지, 이렇게 Pytorch에 어떻게 구현되어 있는지와 같은 부분에 주의를 기울이지 않아왔다. 그런 점에서 매우 흥미로웠던 내용이였다. 우리가 자주 쓰는 torchvision에 구현된 모델들에서도 이런 문제를 정말 자각하고 진행을 했는지는 모르겠지만, 이 부분을 피해 나간다.

```python
# VGG16
nn.init.kaiming_normal_(m.weight, mode='fan_out', nonlinearity='relu')
```

VGG16에서는 기본 초기화를 쓰지 않고, 다음과 같이 nonlinearity를 relu로 주는 방식으로 다시 초기화를 하는데, 이렇게 하면 `gain=math.sqrt(2.0)` 이 된다. 그래서 앞서 언급 되었던 문제를 피하게 된다. Leaky ReLU를 쓸 때만 해당 문제를 발생한다고 이해하면 편한데, 아무래도 ReLU가 de facto처럼 되면서 이 문제가 자연스럽게 묻혀버린 것 아닌가 하는 생각도 들었다.

실제 코드들은 이 [링크](https://www.kaggle.com/coffeedjimmy/fastai-course-v3-03-why-sqrt5)에 있으니 실제로 한 번 돌려보는 것도 추천한다.

이렇게 이 포스팅은 마무리하고 추가적으로 초기화가 왜 중요한지를 설명하는 부분도 제레미가 다뤄줬기에 다음 포스팅도 번외편으로 그 내용을 다뤄보려고 한다.

번외편 1탄 끝.
