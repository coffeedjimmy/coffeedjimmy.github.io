---
title: Tensorflow 2.0 GPU 사용하기
date: 2019.09.14 12:00:00
categories:
- TensorFlow
tags:
- TensorFlow
- TensorFlow 2
---

.

현재 2019년에는 GPU 없이 딥러닝 모델을 빨리 학습시키는 것을 상상할 수 없을 정도로 GPU를 통한 병렬처리 및 최적화는 일반화 되었습니다. 각 딥러닝 프레임워크들이 당연하게도 GPU 연산을 지원하고 있고, Tensorflow 2.0 또한 GPU 연산을 지원합니다.

## 1. 설정하기

Tensorflow 2.0에서는 GPU 연산을 위해 다시 코드를 작성할 필요없이, 즉 **코드의 변경 없이** 기존의 코드를 GPU 연산을 통해 수행할 수 있도록 해줍니다.

```python
from __future__ import absolute_import, division, print_function, unicode_literals

try:
    # Colab에서만 적용되는 부분입니다 (%tensorflow_version)
  %tensorflow_version 2.x
except Exception:
  pass

import tensorflow as tf

print("GPU Available: ", tf.test.is_gpu_available()")
```

결과

```
GPU Avaiable:  True
```

> Colab에서 GPU 환경을 사용하려면 Runtime 변경이 필요합니다. 1.2장의 Google Colab 사용법을 참고 부탁드립니다.

이제부터 GPU를 사용해 보려 합니다. 그런데 그 전에 우리가 몇 개의 GPU를 가지고 있는지 먼저 알면 좋지 않을까요? 현재 사용하고 계신 로컬 머신이나 Colab에 어떤 CPU와 GPU가 장착되어 있는지 알 수 있는 방법은 다음과 같습니다.

```python
    from tensorflow.python.client import device_lib

    device_lib.list_local_devices()
```

결과

```
# Colab에서 실행했을 경우

[name: "/device:CPU:0"
 device_type: "CPU"
 memory_limit: 268435456
 locality {
 }
 incarnation: 14651414794317527222, name: "/device:XLA_CPU:0"
 device_type: "XLA_CPU"
 memory_limit: 17179869184
 locality {
 }
 incarnation: 2158607737880706437
 physical_device_desc: "device: XLA_CPU device", name: "/device:XLA_GPU:0"
 device_type: "XLA_GPU"
 memory_limit: 17179869184
 locality {
 }
 incarnation: 3842897686706556916
 physical_device_desc: "device: XLA_GPU device", name: "/device:GPU:0"
 device_type: "GPU"
 memory_limit: 11330115994
 locality {
   bus_id: 1
   links {
   }
 }
 incarnation: 15101550400344153513
 physical_device_desc: "device: 0, name: Tesla K80, pci bus id: 0000:00:04.0, compute capability: 3.7"]
```

또는 다음과 같은 방식으로 확인할 수 있습니다.

```python
gpus = tf.config.experimental.list_physical_devices('GPU')
print(gpus)
```

결과
```
[PhysicalDevice(name='/physical_device:GPU:0', device_type='GPU')]
```

이제 우리가 몇 개의 GPU를 사용할 수 있는지 알았으니 실제 사용을 해보겠습니다.

Tensorflow를 1.x 버전부터 사용해 보신 분들은 이미 익숙하실 수 있지만, Tensorflow에서는 다음과 같은 방식으로 코드를 어떤 장치 (CPU 또는 GPU)에서 동작시킬지 결정합니다.

- `/device:CPU:0`: 앞서 확인했던 장치들 중 **CPU:0** 을 사용한다는 뜻입니다.
- `/GPU:0` : Tensorflow가 인식한 GPU들 중에서 첫 번째 GPU를 사용하겠다는 뜻입니다.
- `/job:localhost/replica:0/task:0/device:GPU:1` : Tensorflow가 인식한 두 번째 GPU가 있을 경우, 그 GPU의 전체 이름입니다.

장치의 측면에서 주의할 점이 있는데, Tensorflow에서는 CPU와 GPU 둘 다에서 돌릴 수 있는 연산의 경우에는 기본값이 GPU라는 점입니다. 따라서 해당 연산에 대해서는 따로 우리가 연산을 실행할 장치를 명시하지 않으면 기본적으로 GPU에서 실행이 됩니다. 좀 더 쉽게 예를 들면 다음과 같이 이해하시면 됩니다.

예를 들어, `tf.matmul` 이라는 기본적인 연산이 있습니다. 이 연산은 매트릭스 곱을 뜻하며 딥러닝의 기반이 되는 연산입니다. 그래서 당연하게도 해당 연산은 CPU와 GPU 두 장비 모두에서 실행할 수 있도록 구현되어 있는데, 이 경우 우리가 **"CPU에서 이 연산을 실행해줘"** 라고 명시적으로 말해주지 않는 이상은 기본적으로 GPU에서 실행된다는 뜻입니다. 물론 우리가 코드를 실행시키는 장비에 GPU가 존재할 때만 적용됩니다.

## 2. 연산에 실제 적용해보기

이제 실제 연산에 GPU를 사용하는 법을 연습해보려고 합니다.

```python
tf.debugging.set_log_device_placement(True)

a = tf.constant([[1.0, 2.0, 3.0], [4.0, 5.0, 6.0]])
b = tf.constant([[1.0, 2.0], [3.0, 4.0], [5.0, 6.0]])
c = tf.matmul(a, b)

print(c)
```

결과

```
Executing op MatMul in device /job:localhost/replica:0/task:0/device:GPU:0
tf.Tensor(
[[22. 28.]
 [49. 64.]], shape=(2, 2), dtype=float32)
```

무언가 익숙한 문구가 실행 결과에서 보이지 않나요? 바로 `/job:localhost/replica:0/task:0/device:GPU:0` 이 부분입니다. 하나의 GPU를 가지고 있는 경우, 이 연산이 GPU에서 원하는대로 제대로 실행되고 있는지 확인하기 위해서 그리고 하나 이상의 GPU를 가지고 있는 경우 어떤 GPU에서 해당 연산이 실행되고 있는지 궁금한 경우가 생깁니다. 이런 경우 위의 코드 블럭처럼 `tf.debugging.set_log_device_placement(True)` 이 부분을 코드 실행 전에 추가해주면 아래와 같이 해당 연산이 어떤 장치에 할당 되었는지 확인할 수 있습니다.

앞서 설명한 내용과 같이 우리가 어떤 장비에서 해당 연산을 실행하기 원하는지 명시하지 않았고, GPU에서 기본적으로 실행된 것을 확인할 수 있습니다.

그렇다면 이제 우리가 원하는 대로 이 연산을 GPU 대신 CPU에서 실행 시키려면 어떻게 해야 할까요?

```python
tf.debugging.set_log_device_placement(True)

with tf.device('/CPU:0'):
  a = tf.constant([[1.0, 2.0, 3.0], [4.0, 5.0, 6.0]])
  b = tf.constant([[1.0, 2.0], [3.0, 4.0], [5.0, 6.0]])
  c = tf.matmul(a, b)
  print(c)
```

결과

```
Executing op MatMul in device /job:localhost/replica:0/task:0/device:CPU:0
tf.Tensor(
[[22. 28.]
 [49. 64.]], shape=(2, 2), dtype=float32)
 ```

이제 연산이 GPU가 아닌 CPU에서 실행되는 것을 확인할 수 있습니다.

### 2.1. GPU 메모리 제한하기

기본적으로 Tensorflow가 GPU를 할당받게 되면 해당 GPU의 메모리 대부분을 다 점유해 버립니다. 해당 GPU에 할당된 연산이 그만큼의 메모리가 필요하지 않더라도 메모리 대부분을 점유해 버리기 때문에 GPU를 효율적으로 활용하지 못하게 됩니다. 따라서, Tensorflow에서는 바라보는 GPU의 수를 조절하거나 해당 GPU의 메모리 사용에 있어 제한할 수 있는 방법을 제공합니다.

**GPU 개수 조절하기**

```python
gpus = tf.config.experimental.list_physical_devices('GPU')
if gpus:
  try:
        # 첫 번째 GPU만 사용하도록 제한
    tf.config.experimental.set_visible_devices(gpus[0], 'GPU')
  except RuntimeError as e:
    print(e)
```

**GPU 메모리 제한하기**

먼저 첫 번째 방법은 필요한 만큼만 메모리를 할당하는 방식입니다. 초기에는 매우 작은 메모리만 할당 해놓고, 필요할 때마다 조금씩 메모리 할당량을 늘려가는 방식으로 이해하시면 됩니다.

```python
gpus = tf.config.experimental.list_physical_devices('GPU')
if gpus:
  try:
    tf.config.experimental.set_memory_growth(gpus[0], True)
  except RuntimeError as e:
    print(e)
```

이와 같은 효과를 내는 또 다른 방법은 `TF_FORCE_GPU_ALLOW_GROWTH` 을 true로 설정하는 것이지만, 이 경우에는 플랫폼에 따라 다르게 동작할 수 있습니다.

이와 관련해 주의할 부분은 메모리가 일정 부분 더 할당 되었더라도 이 메모리를 다시 해제 하지는 않습니다. 메모리를 해제하고 다시 할당하는 과정 속에서 메모리 단편화가 일어나기 때문입니다.

두 번째 방법은 가상으로 GPU 장치를 설정하고, 그에 대한 설정을 적용하는 방식으로 GPU에 할당되는 메모리 자체를 제한시키는 방법입니다.

```python
gpus = tf.config.experimental.list_physical_devices('GPU')
if gpus:
  # 첫 번째 GPU에 1GB 메모리만 할당하도록 제한
  try:
    tf.config.experimental.set_virtual_device_configuration(
        gpus[0],
        [tf.config.experimental.VirtualDeviceConfiguration(memory_limit=1024)])
  except RuntimeError as e:
    print(e)
```

## 3. Multi-GPU 환경

사용하시는 장비의 환경에 따라 하나 이상의 GPU가 장착되어 있는 환경에서 작업을 할 수도 있습니다. 이 경우에 다수의 GPU 장비들 중에서 번호가 낮은 GPU가 기본적으로 선택됩니다. 이를 원하지 않는다면 다음과 같이 명시적으로 원하는 GPU에 연산을 할당하면 됩니다.

```python
tf.debugging.set_log_device_placement(True)

try:
  # 유효하지 않은 GPU 장치를 명시
  with tf.device('/device:GPU:2'):
    a = tf.constant([[1.0, 2.0, 3.0], [4.0, 5.0, 6.0]])
    b = tf.constant([[1.0, 2.0], [3.0, 4.0], [5.0, 6.0]])
    c = tf.matmul(a, b)
except RuntimeError as e:
  print(e)
```

위에서 나온 예시들을 보다 보면 try~except로 적용되고 있다는 점을 알 수 있는데, 이는 해당 장치가 없을 경우를 대비한 예외 처리 부분입니다. 코드를 작성하다 GPU 이름을 잘못 적는다 거나, 해당 장치의 연결 상태가 좋지 않을 경우 장치를 찾을 수 없다는 에러가 발생하게 되는데 이 때 해당 장치가 아닌 Tensorflow가 인식하고 있는 다른 장치로 할당하여 실행 하기를 원한다면 아래와 같이 작성하면 됩니다.

```python
tf.config.set_soft_device_placement(True)  # 다른 장치로 할당할 수 있도록 함.
tf.debugging.set_log_device_placement(True)

# 텐서 생성
a = tf.constant([[1.0, 2.0, 3.0], [4.0, 5.0, 6.0]])
b = tf.constant([[1.0, 2.0], [3.0, 4.0], [5.0, 6.0]])
c = tf.matmul(a, b)

print(c)
```

이제 실제로 다수의 GPU를 이용해서 모델 학습을 진행하려면 어떻게 하면 될까요? Tensorflow 2.0에서 권장하는 방식은 `tf.distribute.Strategy`를 사용하는 방식입니다.

```python
strategy = tf.distribute.MirroredStrategy()

with strategy.scope():
  inputs = tf.keras.layers.Input(shape=(1,))
  predictions = tf.keras.layers.Dense(1)(inputs)
  model = tf.keras.models.Model(inputs=inputs, outputs=predictions)
  model.compile(loss='mse',
                optimizer=tf.keras.optimizers.SGD(learning_rate=0.2))
```

결과

```
Executing op RandomUniform in device /job:localhost/replica:0/task:0/device:GPU:0
Executing op Sub in device /job:localhost/replica:0/task:0/device:GPU:0
Executing op Mul in device /job:localhost/replica:0/task:0/device:GPU:0
Executing op Add in device /job:localhost/replica:0/task:0/device:GPU:0
Executing op VarHandleOp in device /job:localhost/replica:0/task:0/device:GPU:0
Executing op VarIsInitializedOp in device /job:localhost/replica:0/task:0/device:GPU:0
Executing op LogicalNot in device /job:localhost/replica:0/task:0/device:GPU:0
Executing op Assert in device /job:localhost/replica:0/task:0/device:GPU:0
Executing op AssignVariableOp in device /job:localhost/replica:0/task:0/device:GPU:0
Executing op ReadVariableOp in device /job:localhost/replica:0/task:0/device:GPU:0
Executing op Reshape in device /job:localhost/replica:0/task:0/device:GPU:0
Executing op VarHandleOp in device /job:localhost/replica:0/task:0/device:GPU:0
```

위에서 본 `tf.distribute.Strategy` 는 연산을 여러 GPU에 복제해서 진행하도록 해줍니다. 이를 `tf.distribute.Strategy` 를 사용하지 않고 진행하려면 다음과 같이 진행할 수 있습니다.

```python
tf.debugging.set_log_device_placement(True)

gpus = tf.config.experimental.list_logical_devices('GPU')
if gpus:
  # 여러 GPU에 계산을 복제
  c = []
  for gpu in gpus:
    with tf.device(gpu.name):
      a = tf.constant([[1.0, 2.0, 3.0], [4.0, 5.0, 6.0]])
      b = tf.constant([[1.0, 2.0], [3.0, 4.0], [5.0, 6.0]])
      c.append(tf.matmul(a, b))

  with tf.device('/CPU:0'):
    matmul_sum = tf.add_n(c)

  print(matmul_sum)
```

결과

```
Executing op MatMul in device /job:localhost/replica:0/task:0/device:GPU:0
tf.Tensor(
[[22. 28.]
 [49. 64.]], shape=(2, 2), dtype=float32)
```
