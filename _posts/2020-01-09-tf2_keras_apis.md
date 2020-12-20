---
title: TensorFlow 2.0 Keras API
date: 2020.01.09 12:00:00
categories:
- TensorFlow
tags:
- TensorFlow
- Keras
---

TensorFlow 2.0의 high-level API의 방향성에 대해 정리.

참고 글 [링크](https://medium.com/tensorflow/standardizing-on-keras-guidance-on-high-level-apis-in-tensorflow-2-0-bad2b04c819a)

- Keras는 딥러닝 모델을 구성하고 학습하는데에 매우 널리 알려진 high-level API이다.
- 빠른 프로토타이핑이 가능하고, SOTA 리서치, 그리고 실제 production에도 사용된다.
- 기존에 Tensorflow가 tf.keras 형태로 keras 지원을 진행해왔지만, 2.0부터는 보다 keras-tensorflow 연동이 강해진다.
    - 정확하게는 Keras가 TensorFlow의 공식적인 high-level API가 된다.
- Keras로 통일된 high-level API를 제공함으로서 개발자들이 좀 더 혼란없이 모델개발에만 집중할 수 있도록 하는 목적

### Keras 장점

- User-friendly
    - 간단하고 통일된 인터페이스를 가지고 있음.
- Modular and composable
- Easy to extend
- For beginners, and for experts

### TF 2.0 관련 Q&A

1. Keras는 독립적인 라이브러리 아닌가?

대부분의 경우에 Keras는 단순히 API스펙이다. 기존에 존재하던 오픈소스라이브러리인 Keras는 독립적이며, TensorFlow와도 독립적이다. TF 2.0은 이 Keras의 구현체를 가져다 TensorFlow에 적용했다고 이해하면 됨.

2. Keras가 TensorFlow의 wrapper격인가?

아니다. 앞에서도 말했지만, Keras는 API 스펙이며 특정 구현체에 한정되지 않는다. 다시 말해 백엔드로 TF, pytorch 등 다양한 구현체들을 사용할 수 있다.

3. TF에 포함된 Keras와 keras.io의 Keras 차이는?

TensorFlow는 tf.keras 모듈 형태로 Keras API구현체를 포함한다. 이는 eager execution과 TensorFlow SavedModel model exchange format,  distributed training 등을 포함한다.

Eager execution은 특히 tf.keras model subclassing API를 사용할 때 효율적이다. 이 API는 최근에 개발 종료된 Chainer에서 영감을 받아서 만들어 졌으며, 모델의 forward pass를 보다 명령형으로 작성할 수 있다.

- tf.data
- distribution strategies
- exporting models : SavedModel
- TF Lite, TF JS

4. TensorFlow가 초보자와 전문가에 따라 다른 API 스타일을 제공한다는데 무슨 뜻인가?

텐서플로 사용자들의 경험치 수준이 다양하므로 크게 3가지의 다른 형태의 API를 제공한다.

**Sequential API**

```python
import tensorflow as tf
from tensorflow import keras

mnist = keras.datasets.mnist

(x_train, y_train), (x_test, y_test) = mnist.load_data()
x_train, x_test = x_train/255., x_test/255.

model = keras.models.Sequential([
    keras.layers.Flatten(),
    keras.layers.Dense(512, activation=keras.activations.relu),
    keras.layers.Dropout(0.2),
    keras.layers.Dense(10, activation=keras.activations.relu)
])

model.compile(optimizer='adam',
              loss='sparse_categorical_crossentropy',
              metrics=['accuracy'])
model.fit(x_train, y_train, epochs=5)
model.evaluate(x_test, y_test)
```

- 초심자에게 적합한 스타일
- 생각하는대로 적으면 되므로 매우 간단함.

**Functional API**

- Sequential API는 단순히 layer를 쌓는 형태이므로 임이의 모델을 표현하지는 못한다.
- 이보다 좀 더 발전된 형태로 개발하기 위해서 Functional API를 사용하면 됨.
    - multi-input, multi-output모델이나 shared layer 모델 그리고 residual connection등을 구현할 수 있음.

```python
inputs = keras.Input(shape=(32,))

# A layer instance is callable on a tensor, and returns a tensor
x = keras.layers.Dense(64, activation='relu')(inputs)
x = keras.layers.Dense(64, activation='relu')(x)

predictions = keras.layers.Dense(10, activation='softmax')(x)

model = keras.Model(inputs=inputs, outputs=predictions)
```


**Model Subclassing API**

- Pytorch의 Subclassing처럼 완벽히 커스터마이징하여 모델링 할 수 있는 방식

```python
class CustomModel(keras.Model):
    def __init__(self):
        super().__init__()
        self.fn1 = keras.layers.Dense(32, activation='relu')
        self.fn2 = keras.layers.Dense(32, activation='relu')

    def call(self, x):
        x = self.fn1(x)
        x = self.fn2(x)
        return x
```

세 타입의 API모두 compile, fit 형태로 학습을 진행할 수도 있고, 스스로 작성한 training loop를 활용해서 학습을 진행할 수도 있다.

### Estimator의 미래는?

많은 곳에서 Estimator를 사용하고 있기 때문에 2.0에서도 계속 유지하기는 함. 다만 앞으로는 Keras를 권장하며  Estimator의 사용이 필요하더라도 Keras를 이용해서 작성한 다음 `model_to_estimator()` 로 변환해서 사용하는 것을 권장.
