---
title: future 모듈과 Python2/3 호환성
date: 2019.12.28 12:00:00
categories:
- python
tags:
- python2
- python3
---

Python2의 공식 지원이 2020년 1월 1일 (글 쓰는 시점의 4일 뒤)라는 점에서 다소 뒷북인 내용이 될 수도 있지만
당분간은 python2가 그럼에도 쓰일 것으로 보이고 질문도 꽤나 받아서 해당 내용을 정리해 둔다.

다들 익숙한 파이썬 버전은 python3일텐데 기존에 python2라는 물건이 존재했다. python2는 2000년에 공개되었고, python3는 2006년에 공개되었다.
python2는 곧 공식적인 [보안 업데이트를 종료](https://www.python.org/doc/sunset-python-2/)하게 되니 왠만해선 3버전으로 업데이트를 권장한다.

다시 본론으로 돌아가서 TensorFlow 예제코드들을 보다 보면 python2와 python3의 호환성을 위한 부분을 구글이 포함시켜 놓은 것을 볼 수 있다.

`from future import absolute_import, division, print_function, unicode_literals`

Python을 3점대부터 접한 분들은 아마도 이건 대체 왜 넣는거지?라는 의문을 가질 수도 있다. 사실 Python2 -> Python3로의 transition에는
3가지의 큰 변화가 있었는데 각각을 해결하기 위해 future 모듈 속의 몇 가지를 임포트하게 되고, 이를 통해 호환성 있는 코드를 작성할 수 있다.

## 1. from future import print_function

python2 -> python3에서 가장 눈에 띄는 변화 중 하나는 print문이 statement에서 function으로 변경되었다는 점이다. 간단히 설명하자면
기존의 `print "Hello, jimmy"` 형태에서 `print("Hello, jimmy")` 형태로 변경되었다는 점이다. 따라서, python2/3 호환이 가능한 코드를
작성하기 위해서는 다음의 import 문을 추가해주어야 한다.

```python
## python2/3 호환이 가능한 코드
from future import print_function

print("Hello, Jimmy")
```

해당 내용에 대해 좀 더 깊이 알고 싶다면 이 글의 마지막 부분의 부록을 참고하자.

## 2. from future import unicode_literals

한국어를 모국어로 사용하는 사람으로써 슬픈 이야기지만, 프로그래밍 언어 자체를 영어 문화권에서 만들었기에 한국어에 대한 배려가 크게 높지 않다.
그래서 python3에서는 큰 문제가 없지만, python2에서 주석에다가 한글을 입력하면 다음과 같은 에러를 볼 수 있다.

```bash
SyntaxError: Non-ASCII character '\xed' in ...
```

그래서 다음과 같이 다음과 같이 사용해야 한다.

```python
#-*- coding: utf-8 -*-
from future import print_function

print(u'안녕')
```

한글을 unicode라고 선언해주고, unicode를 utf-8로 인코딩하라는 뜻인데, 이렇게 하면 정상적으로 동작한다. 그렇지만, 이렇게 사용하기엔
매번 문자열에 u를 붙여야 한다는 점에서 너무나도 귀찮지 않은가? 그래서 이렇게 진행할 수 있다.

```python
#-*- coding: utf-8 -*-
from future import print_function, unicode_literals

print('안녕')
```

python3에서 따로 선언을 하지 않아도 되는 이유는 python3의 문자열이 기본적으로 unicode로 설정되었기 때문이다.

## 3. from future import division

이 부분은 예시를 통해 이해하는 것이 보다 직관적일 것 같아 아래 예시를 보자.

```python
## python2
test = 3/2  # 1

## python3
test = 3//2 # 1
test = 3/2  # 1.5
```

해당 부분의 차이점을 해결하기 위해 division을 임포트 해주면 된다.

이렇게 python2와 python3의 차이에 대해 알아보았다. 앞선 1번에 대한 심화 내용이 궁금하신 분들은 아래의 부록을 참고하시면 된다.

### 부록

앞서 설명했지만, python2에서 print문은 statement이다. statement란 실행 됨으로 해서 특정 효과가 나타날 수는 있지만 값에 대한 평가는 하지 않는다는
뜻이다. 쉽게 말해 statement는 print하거나 변수에 할당할 수 없다는 뜻이다.

예를 들어 python2에서 다음과 같은 구문을 실행하면 에러가 발생하게 된다.

```python
## python2

test = print 'hello, jimmy'
# SyntaxError: invalid syntax
```

python에서 다른 statemenet들은 다음과 같다.

- assignment: =
- conditional: if
- loop: while, for
- assertion: assert

Statement는 주로 예약된 키워드들로 구성되는데 `if`, `for` 등의 예시를 보면 알 수 있다. 이 때문에 python2에서는 print가 statement이기
때문에 print를 재정의할 수가 없다. 또한, lambda 표현식은 statement를 받지 않기 때문에 print문을 익명함수 안에 넣을 수 없다.

```python
## python2
test = lambda: print "hello"
# SyntaxError: invalid syntax

## python3
test = lambda: print("hello")
```

또한, print 문이 statement인 python2에서는 다소 애매모호한 점들이 몇 가지 있다.

```python
## python2
print "hello"
# hello
print ("hello")
# hello

print "name:", "jimmy"
# name: jimmy
print ("name:", "jimmy")
# ('name:', 'jimmy')
```

String concatenation 과정에서도 문제가 발생할 수 있다.

```python
## python2
values = ['jimmy', 'is', 100, 'years old']
print ' '.join(values)
# TypeError: sequence item 2: expected string, int found

print ' '.join(map(str, values))
# jimmy is 100 years old

## python3
values = ['jimmy', 'is', 100, 'years old']
print(*values)
# jimmy is 100 years old
```

이런 문제가 발생하는 것을 막기 위해 본문에서 설명했던 `from future import print_function`을 사용하면 된다.

python의 print와 관련된 좀 더 자세한 내용을 보고 싶다면, 다음 [링크](https://realpython.com/python-print/#print-was-a-statement-in-python-2)를
참고하면 좋을 듯 하다.

.
