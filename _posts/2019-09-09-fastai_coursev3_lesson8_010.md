---
layout: post
title: "[Fast.ai] Lesson 8 (1부)"
tags: [fastai, 머신러닝, course-v3, lesson8]
comments: true
---

이 포스트는 Fastai course-v3의 lesson 8 내용을 정리한 내용을 담고 있습니다.

Fastai는 최근에 흥미롭게 바라보며 익혀가고 있는 라이브러리 중 하나이다. 평소에 케라스가 와닿지않고 이게 정말 쉬운게 맞냐는 이야기를 하고 다니는데, 왠 새로운 high-level 격 라이브러리를 들고와서 괜찮다고 하니 이상하게 느껴질 수도 있다. 

아직 둘러보고 있는 단계라 뭐가 확실히 좋다고 명확히 말할 수는 없지만, 지금까지 느낀 점으로는 무엇보다 데이터 구성하는 부분만 좀 편리하게 fastai를 사용하고, 원한다면 얼마든지 pytorch랑 섞어 쓸 수 있는 자유도가 마음에 든다 (물론 tf.keras를 쓰면 keras에 tf를 섞어 쓸 수 있다고는 하는데.. 모르겠다 개인 취향인듯 → 그래서 이제부터 좋은 점만 골라보는 걸로...)

무엇보다 강의를 제공해 준다는 점은 꽤나 큰 메리트인 것 같다. 더군다나 라이브러리의 개발자가!! (Jeremy Howard). 이 아저씨는 무려 캐글 1등 출신이라는 것 같다..  잡설은 이까지 하고 앞으로 가벼운 마음으로 공식 홈페이지에서 제공해주는 강의를 들으며 한 강의 씩 정리를 해볼까 한다. 그럼 시작해보자.

근데 왜 시작인데 Lesson1부터 안하냐는 생각을 할 수도 있는데, Lesson1~7는 약간 실제로 사용하는 법?에 초점이 맞춰져 있다고 한다면  Lesson8부터는 이 라이브러리에 대한 `Implementation from scratch`와 같은 느낌이다. 그래서 이것 또한 개인의 취향에 따라 + 이 쪽이 이 라이브러리를 이해할 때 먼저 보는 게 좋을 것 같아서. (Lesson 1~7도 정리할 예정)

그럼 이제 진짜 시작.

제레미가 강조하는 문장 중 하나는 다음과 같다.

> There are no stupid questions

참으로 매우매우 맞는 표현인데, 특히 우리나라에서는 참 이게 안되는 것 같다. 특히나 Fastai와 같은 경우에는 포럼이 매우 잘 활성화되어 있으므로 궁금한 부분이 생기면 거기다가 질문을 남기면 된다. (외국 애들은 생각보다 쉬운 질문에도 타박을 주지않고 잘 대답해주는 편이니 걱정하지 말자)

## Introduction

제레미는 모델의 학습 과정을 크게 다음의 3가지 과정으로 구성한다.

1. Overfit
2. Reduce overfitting
3. Visualising & Experiments

일단은 오버피팅시키고, 그걸 완화시킬 방법들을 찾고 모델학습이 완료된 다음 시각화 및 실험을 하는 것? 어떻게 보면 당연한 말인 것 같다. 처음부터 학습을 돌렸는데 underfitting이 난다면 뭔가 잘못되가고 있다는 신호일테니..

그리고는 세부적으로 어떻게하면 Overfitting을 피할 수 있는 지를 5가지 단계로 나눠서 말해준다. 

**Overfitting을 피하기 위한 5가지 단계**

1. More data
2. Data Augmentation
3. Generalisable architectures
4. Regularisation (Dropout or weight decay)
5. Reduce architecture complexity

앞에서는 Resnet을 간단히 쓰는 법을 배웠다면 이번 강의의 목적은 그 Resnet까지 가는 과정이 어떻게 구성되어 있는지를 하나하나 뜯어보는 것이라고 한다.

여기서부터는 굵직굵직한 내용들만 소개하고 전체 코드는 모두 쉽게 돌려볼 수 있도록 캐글 커널로 정리해서 제공하려고 한다. 여기서부터 링크를 걸어놓으면 분명 지금 들어갈 것 같으니 맨 아래에 두겠다. 개인적으로 제일 빨리 라이브러리에 익숙해지는 방법은 일단 눈에 익히는게 먼저라고 생각해서 이 글을 그냥 가벼운 마음으로 읽은 다음 나중에 코드 보는 것을 추천한다.

## Exports

이 부분은 캐글 커널을 통해서 설명하기가 조금 불편한 부분이 있어서 포스팅을 통해서 사용법? 정도만 전달하려고 한다.

팀에서 Jupyter Notebook을 기반으로 한 코드리뷰를 진행하면 어떨지 고민할 때도 제레미랑 비슷한 고민을 한 것 같은데, 여기서 다룰 내용은 `"어떻게 하면 노트북을 기반으로 유닛테스트를 좀 더 쉽게 할 수 있을까"` 이다.

제레미는 두 스크립트를 이용해서 이를 해결하고자 하는데 바로 [run_notebook.py](https://github.com/fastai/course-v3/blob/master/nbs/dl2/run_notebook.py) 과 [notebook2script.py](https://github.com/fastai/course-v3/blob/master/nbs/dl2/notebook2script.py) 이다. 먼저 notebook2script.py부터 살펴보자.

```python
# ==== Jupyter Cell ====
#export
TEST = 'test'
# ==== Jupyter Cell ====

# ==== Jupyter Cell ====
!python notebook2script.py 00_exports.ipynb
# ==== Jupyter Cell ====
```

위와 같이 노트북 셀에 `#export` 라고 쓰게되면 해당 셀에 있는 내용을 `.py` 파일로 export 시켜주는 스크립트이다. 개인적인 유즈케이스라면.. 노트북으로는 깃헙 기반 코드리뷰가 힘드니 nbconvert같은걸로 .py로 떨어뜨린 다음 커밋하게 해야하나?하는 고민들을 했었는데, 제레미가 한 것 처럼 진행을 해도 좋을 것 같다는 생각을 했다.

위의 예시와 같이 가능한 이유는 노트북의 기본적인 구조가 json으로 되어 있기 때문인데, 그래서 다음과 같이 노트북을 여는 것 또한 가능하다.

```python
# ==== Jupyter Cell ====
import json
d = json.load(open('00_exports.ipynb','r'))['cells']
# ==== Jupyter Cell ====

{'cell_type': 'code',
 'execution_count': 1,
 'metadata': {},
 'outputs': [],
 'source': ['#export\n', "TEST = 'test'"]
}
```

그 다음은 `run_notebook.py`다.

```python
# ==== Jupyter Cell ====
#export
from exp.nb_00 import *
import operator

def test(a,b,cmp,cname=None):
    if cname is None: cname=cmp.__name__
    assert cmp(a,b),f"{cname}:\n{a}\n{b}"

def test_eq(a,b): test(a,b,operator.eq,'==')
# ==== Jupyter Cell ====

# ==== Jupyter Cell ====
test_eq(TEST,'test')
# ==== Jupyter Cell ====
```

유닛테스트를 위해서 다음과 같이 구성할 수 있을텐데, 노트북 내에서만 실행해야 하나? 다른 방법은 없나? 하는 궁금증이 들 수도 있다. 제레미도 이런 부분을 간단히 하기 위해 `run_notebook.py`라는 간단한 스크립트를 만들어서 노트북 밖 즉 쉘에서도 유닛테스트를 쉽게 진행할 수 있도록 해놓았다.

이미 개발을 잘하는 개발자의 경우에는 이게 뭐라고..별 거 아니네 라고 생각할 수도 있지만, 사실 이런 건 있으면 좋겠는데 내가 짜기는 귀찮은 부분이지 않는가. 그런 점에서는 대단히 개발자 친화적인 스크립트들이 많다는 느낌을 받았다.

## Matrix Multiplication

이제 본격적으로 강의 내용에 대해 설명하려고 한다. 먼저 신경망 학습의 기초 중의 기초인 Matrix Multiplication에 대해서 다뤄본다.

```python
# Basic Matrix Multiplication

def matmul(a,b):
    ar,ac = a.shape
    br,bc = b.shape
    assert ac==br
    c = torch.zeros(ar, bc)  # result matrix of matmul
    for i in range(ar):
        for j in range(bc):
            for k in range(ac):
                c[i,j] += a[i,k]*b[k,j]
    return c
```

가장 나이브한 Matmul 구현이다. 우리가 직접 행렬 곱을 하듯이 진행을 하는 꽤나 직관적인 방법인데, 사실 이를 활용해서 Matmul을 사용하기에는 적합하지 않다.

```python
# ==== Jupyter Cell ====
%time t1 = matmul(m1, m2)
# ==== Jupyter Cell ====

CPU times: user 760 ms, sys: 44 ms, total: 804 ms
Wall time: 766 ms
```

위와 같이 0.7초 가량이 걸린다. 하나의 간단한 행렬 곱에도 이정도 시간이 걸린다면 더욱 더 큰 신경망을 학습시키는데는 엄청난 시간이 소요될 것이다. 이렇게 느리게 나온 이유는 아무래도 Python이라는 점이 한 몫을 했다. Python이 느리다는 내용은 다들 잘 알테니, 왜 그런지는 다른 블로그들을 참고하면 좋을 듯 하다. 아무튼 위의 구현은 오직 Python level에서의 구현이기 때문에 당연히 느릴 수 밖에 없다. 

그럼 어떡해야 할까? 다행히도 이보다 50,000배는 빨리 matmul을 진행할 수 있는 방법이 있다. Python이라 느렸다고 했으니 조금 더 low-level로 내려보면 어떨까? 예를 들면 Pytorch에게 연산들을 넘겨줘서 계산하게 하는 것이다.

실제로 Pytorch는 `aten` 이라는 C++ API를 가지고 있는데, 여기에는 기본적인 텐서와 수학적 연산들이 정의되어 있다. ([참고자료](https://pytorch.org/cppdocs/)). 그래서 우리는 지금부터 `aten` 이 지원해주는 elementwise operation을 가지고 matmul을 구현해본다.

```python
# 간단한 Elementwise operation 설명
a = tensor([10., 6, -4])
b = tensor([2., 8, 7])

# Elementwise addition
a+b
>> tensor([12., 14.,  3.])

# Elementwise comparison
(a<b).float().mean()
>> tensor(0.6667)
```
 

간단히 예시를 봤으니 이제 어떻게 matmul이 변할 수 있는지 보자.

```python
def matmul(a,b):
    ar, ac = a.shape
    br, bc = b.shape
    assert ac == br
    c = torch.randn(ar, bc)
    for i in range(ar):
        for j in range(bc):
                        # :을 생략해도 됨.
                        # c[i,j] = (a[i,] * b[,j]).sum()
            c[i,j] = (a[i,:] * b[:,j]).sum()
    return c

>> 989/1.53
```

이로써 하나의 loop를 줄였고 (정확히는 c level로 내려보냈고), 실행환경에 따라 다르겠지만 캐글 커널을 통한 실행에서는 `646.405` 배 정도 빨라졌다.

## Broadcasting

`Broadcasting`이라는 단어는 수학적 연산을 할 때, 서로 다른 모양을 가진 배열들이 어떻게 처리되는지를 뜻하는 말이다. Numpy에서 처음 이 개념을 차용해서 사용했다고 한다. 세부적인 실습은 캐글 커널을 통해 진행하면 될 것 같고, 간단히 matmul에서 broadcasting이 가지는 의미를 찾자면 다음과 같다.

앞에서 봤던 matmul에서 대부분의 loop를 제거할 수 있고, C level 속도를 가지며 GPU에서 돌릴 경우 CUDA 속도를 가진다는 점이다.

Broadcasting하게 되면 그만큼 메모리를 더 잡는 것 아닌가 하는 걱정을 할 수도 있지만, 실상은 그렇지 않다.

```python
c = tensor([10., 20, 30])
m = tensor([[1., 2, 3], [4,5,6], [7,8,9]])

t = c.expand_as(m)

print(t.storage())

#result
 10.0
 20.0
 30.0
[torch.FloatStorage of size 3]

print(t.stride())

# result
(0, 1)
```

이를 활용해서 실제 matmul을 구현하면 다음과 같다.

```python
def matmul(a,b):
    ar, ac = a.shape
    br, bc = b.shape
    assert ac == br
    c = torch.randn(ar, bc)
    for i in range(ar):
        # c[i] = (a[i,None]*b).sum(dim=0)
        c[i] = (a[i].unsqueeze(-1) * b).sum(dim=0)
    return c
```

`torch.sum(dim=0)` 에 대해서 되게 헷갈리는 경우가 많은데, 나도 조금만 안보다 보면 헷갈려서 다시 찾아보곤 한다. 쉽게 말하자면 저기에서의 dim은 해당 dimension을 줄여버리겠다는 뜻이다. 그래서 2차원 행렬에 대한, `torch.sum(dim=0)`의 결과는 (1, ncols) 그리고 `torch.sum(dim=1)` 의 결과는 (nrows, 1)이 된다. 이렇게 적어놓고 보니 좀 와닿지 않는가?

## Einstein summation

이제 많은 사람들한테 새로울 수 있는 Einstein summation으로 구현하는 법을 소개한다.

```python
def matmul(a,b):
    return torch.einsum('ik,kj->ij', a,b)

%timeit -n 10 _=matmul(m1, m2)
# result
>> 989/0.0733
>> 13492.4965893588
```

속도가 훨씬 빨라지긴 했지만, 제레미는 `language in language` 라며 horrendous라는 표현까지 쓰며 별로라고 이야기 한다. 

그럼 이것보다 더 빠른 속도를 내면서 좀 더 general한 형태로 될 수 있는 방법이 있다는 뜻인데 그건 바로 Pytorch자체에 있는 operation이다.

## Pytorch operation
   
```python 
%timeit -n 10 t2 = m1.matmul(m2)

#result
>> 989/0.019
>> 52052.63157894737
```

처음에 비해서 약속한대로 50,000배 이상 빨라졌다. 이를 가능하게 했던 건 Pytorch가 `BLAS` 라는 Basic Linear Algebra Subprograms을 이용해서 matmul을 구현해놨기 때문이다.

쉽게 말하면 Pytorch에 구현된 matmul을 사용하는 것이 가장 빠르다고 생각하면 편하다. 그 외에 [tensor comprehensions](https://pytorch.org/blog/tensor-comprehensions/) 라는 방향으로 나아가려고 한다고 하니 한번 참고해보면 좋을 듯 하다.

마저 마무리를 할까 싶었는데, 강의 내용 안에서 약간의 내용 변화가 있어서 남은 30분 가량의 내용은 2부에서 이어나가고자 한다.

[캐글 커널 실습자료 링크](https://www.kaggle.com/coffeedjimmy/fastai-course-v3-01-matmul)

1부 끝.

