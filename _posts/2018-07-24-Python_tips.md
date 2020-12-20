---
title: Python과 관련된 사소한 정보들
date: 2018.07.24 12:00:00
categories:
- Python
tags:
- Tips
---

파이썬을 공부해나가며 알면 좋은 내용들을 채워넣는 포스트입니다.


### Python dictionary와 DefaultDict의 차이

별 차이 없음. 다만 defaultdict(lambda: 0)과 같이 선언해놓으면 keyerror를 발생시키지 않고, key-value 쌍을 자동으로 생성해준다 정도?

[reference](http://khanrc.tistory.com/entry/PyCon-%EC%9C%84%EB%8C%80%ED%95%9C-dict-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B3%A0-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0)

### Python Counter와 직접 딕셔너리로 Counter 만드는 것 사이에 차이가 있을까?

따로 딕셔너리를 만들어 하는것과 동일

```python
from collections import Counter

a = [1,2,1,2,4]
print(Counter(a))  # Counter({1: 2, 2: 2, 4: 1})
```

### python max, min 시간복잡도

O(N)

[reference](https://wayhome25.github.io/python/2017/06/14/time-complexity/)

### Python max recursion depth

```python
import sys

print(sys.getrecursionlimit())  # 1000
```
