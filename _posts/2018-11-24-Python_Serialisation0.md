---
layout: post
title: "Python 객체를 파일로 저장하는 방법"
tags: [python, serialisation]
comments: true
---

.

파이썬 객체를 파일로 저장하는 것 => `Serialisation`

- 직렬화(Serialization) : 객체를 파일로 내보냄
- 역직렬화(Deserialization) : 파일로 내보낸 객체를 다시 읽어들임

```python
# 직렬화
import pickle

with open("myobject.lq", "wb") as file:
    pickle.dump(object(), file)

# 역직렬화

with open("myobject.lq", "rb") as file:
    my_obj = pickle.load(file)
```
