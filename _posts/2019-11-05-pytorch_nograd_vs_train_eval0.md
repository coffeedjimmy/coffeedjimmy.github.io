---
layout: post
title: "Pytorch에서 no_grad()와 eval()의 정확한 차이는 무엇일까?"
tags: [pytorch]
comments: true
---

.

Pytorch를 사용해서 모델링을 하다보면 다음과 같은 궁금증에 도달할 수 있다. 

왜 `model.eval()`을 선언해놓고 또 `with torch.no_grad():`로 감싸주는거지?

처음 접했을 땐, 전자만 사용하면 되지않나라고 막연하게 생각할 수도 있다. 그렇지만, 이 둘 사이에는 차이가 있다.

### with torch.no_grad()

이와 같이 `no_grad()` with statement에 포함시키게 되면 Pytorch는 autograd engine을 꺼버린다. 이 말은 더 이상 자동으로 gradient를 트래킹하지 않는다는 말이 된다.
그러면 이런 의문이 들 수 있다. `loss.backward()`를 통해 backpropagation을 진행하지 않는다면 뭐 gradient를 게산하든지 말든지 큰 상관이 없는 것이 아닌가?

맞는 말이다. `torch.no_grad()`의 주된 목적은 autograd를 끔으로써 메모리 사용량을 줄이고 연산 속도를 높히기 위함이다. 사실상 어짜피 안쓸 gradient인데 inference시에 굳이 계산할 필요가 없지 않은가?

그래서 일반적으로 inference를 진행할 때는 `torch.no_grad()` with statement로 감싼다는 사실을 알면 된다.

### model.eval()

그럼 여기서 다시 처음 질문으로 돌아와서 위의 `torch.no_grad()`만 쓰면 되지 않나? gradient 계산 안하고 이제 됐잖아 라고 생각할 수 있다. 
맞는 말이지만, `model.eval()`의 역할은 약간 다르다. 현재(2019년) 시점에서는 모델링 시 training과 inference시에 다르게 동작하는 layer들이 존재한다. 예를 들면,
Dropout layer는 학습시에는 동작해야하지만, inference시에는 동작하지 않는 것과 같은 예시를 들 수 있다. BatchNorm같은 경우도 마찬가지다.

사실상 `model.eval()`는 이런 layer들의 동작을 inference(eval) mode로 바꿔준다는 목적으로 사용된다. 따라서, 우리가 보통 원하는 모델의 동작을 위해서는 위의 두 가지를 모두 사용해야하는 것이 맞다.

### 결론

사실 되게 간단하고 어떻게 보면 짜치는 내용이지만, 은근히 처음 접하시는 분들한테는 이게 뭔 차이야? 이럴 수 있어서 간단히 정리해 보았다.


끝.
