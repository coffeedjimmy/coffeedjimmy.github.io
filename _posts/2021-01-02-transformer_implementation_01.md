---
title: Transformer 직접 구현해보기 - 1편
date: 2021.01.02 12:00:00
categories:
- Transformer
tags:
- transformer
- ml
- dl
---

# Transformer 직접 구현해보기 - 1편

## 1. Introduction

`Transformer`는 2017년 NIPS에서 [Attention is all you need](https://arxiv.org/pdf/1706.03762.pdf) 논문을 통해 소개된 아키텍쳐입니다. 이번 아티클부터 우리는 직접 Transformer 를 구현해보면서 간단한 번역 테스크를 풀어보겠습니다.

단, 이 아티클은 기본적인 Transformer 구조에 대한 개념은 알고있다는 것을 기반으로 하며, 만약 처음 보시는 분이라면 [Jay의 블로그](http://jalammar.github.io/illustrated-transformer/)를 통해 한번 개념을 잡으신 뒤 Hands-on을 해나가시는 걸 추천드립니다.

## 2. 데이터셋 구성

본격적인 모델 구현에 앞서 `torchtext`와 우리가 활용할 데이터셋인 `Multi30k` 데이터셋을 가공하도록 하겠습니다. 

먼저 현재 활용하시는 환경에 세팅되어 있는 Pytorch 버전을 다음과 같이 확인하신 뒤, 해당 [링크](https://pypi.org/project/torchtext/)에 있는 compatibility 대로 그에 맞는 torchtext를 설치해줍니다.

```bash
$ pip list | grep torch
$ pip install -q torchtext==0.8.0
```

설치가 완료되고 나면 [spaCy](https://spacy.io/)와 torchtext를 활용해서 손쉽게 데이터 전처리를 진행하게 됩니다. *(다른 다양한 방법이 있습니다. 튜토리얼 느낌이라 보다 손이 덜 타는 방법을 택했으며, 다른 방식을 통해서 데이터를 가져오시고 가공하셔도 아무런 상관 없습니다.)*

```python
# 여기부터는 Jupyter 환경을 가정합니다.

%%capture
!python -m spacy download en
!python -m spacy download de
```

위와 같은 방식으로 우리가 원하는 영어와 독일어를 위한 모델을 다운받을 수 있습니다. 해당 작업이 끝나고 나면 해당 모델을 로드해준 뒤, 아래와 같이 모델에 포함된 tokenizer를 활용해서 어떻게 나눠지는지 테스트 해볼 수 있습니다.

```python
import spacy

spacy_en = spacy.load('en')
spacy_de = spacy.load('de')

tokenized = spacy_en.tokenizer("This is a test for tokenizer.")

for idx, token in enumerate(tokenized):
    print(f"Index {idx}: {token.text}")

"""
Index 0: This
Index 1: is
Index 2: a
Index 3: test
Index 4: for
Index 5: tokenizer
Index 6: .
"""
```

이제 텍스트를 입력받아서 토큰화 진행 후 다시 리턴해주는 함수를 각 언어별로 선언해준 뒤, 텐서로 표현 될 수 있는 텍스트 데이터 타입을 처리할 수 있는 `torchtext의 Field`를 활용해서 파이프라인을 구성해줍니다.

```python
def tokenize_de(text):
    return [token.text for token in spacy_de.tokenizer(text)]

def tokenize_en(text):
    return [token.text for token in spacy_en.tokenizer(text)]

from torchtext.data import Field, BucketIterator

SRC = Field(tokenize=tokenize_de, init_token="<sos>", eos_token="<eos>", lower=True, batch_first=True)
TGT = Field(tokenize=tokenize_en, init_token="<sos>", eos_token="<eos>", lower=True, batch_first=True)
```

이제 torchvision처럼 torchtext에 포함되어 있는 우리가 오늘 활용할 데이터셋인 Multi30k 데이터를 불러오고, 앞서 선언한 Field를 통해 가공해주게 됩니다. 

```python
from torchtext.datasets import Multi30k

train_dataset, valid_dataset, test_dataset = Multi30k.splits(exts=(".de", ".en"), fields=(SRC, TGT))

print(f"training dataset size: {len(train_dataset)}")
print(f"validation dataset size: {len(valid_dataset)}")
print(f"test dataset size: {len(test_dataset)}")

"""
training dataset size: 29000
validation dataset size: 1014
test dataset size: 1000
"""
```

데이터가 제대로 불러와졌는지는 다음과 같은 방식으로 확인할 수 있으며 Field를 통해 가공된 데이터는 `OOV(Out of Vocab) = 0, Padding=1, <sos> = 2. <eos> = 3`의 스페셜 토큰을 가집니다.

```python
# Sample data

sample_idx = 7

print(vars(train_dataset.examples[idx])['src'])
print(vars(train_dataset.examples[idx])['trg'])

"""
['ein', 'mann', 'lächelt', 'einen', 'ausgestopften', 'löwen', 'an', '.']
['a', 'man', 'is', 'smiling', 'at', 'a', 'stuffed', 'lion']
"""

# Build Vocab
SRC.build_vocab(train_dataset, min_freq=2)
TGT.build_vocab(train_dataset, min_freq=2)

print(f"len(SRC): {len(SRC.vocab)}")
print(f"len(TGT): {len(TGT.vocab)}")

"""
len(SRC): 7855
len(TGT): 5893
"""

print(TGT.vocab.stoi["abcabc"]) # 없는 단어: 0
print(TGT.vocab.stoi[TGT.pad_token]) # 패딩(padding): 1
print(TGT.vocab.stoi["<sos>"]) # <sos>: 2
print(TGT.vocab.stoi["<eos>"]) # <eos>: 3
print(TGT.vocab.stoi["hello"])
print(TGT.vocab.stoi["world"])

"""
0
1
2
3
4112
1752
"""
```

이제 데이터 구성의 마지막 작업으로 앞서 선언해둔 torchtext의 BucketIterator를 활용해서 train, valid, test 데이터셋을 구성합니다.

```python
import torch

device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')

batch_size = 128

train_iterator, valid_iterator, test_iterator = BucketIterator.splits(
    (train_dataset, valid_dataset, test_dataset),
    batch_size=batch_size,
    device=device
)
```

데이터가 잘 구성되었는지 샘플로 꺼내보려면 다음과 같이 진행하시면 됩니다.

```python
for idx, batch in enumerate(train_iterator):
    src = batch.src
    tgt = batch.trg

    print(f"batch shape: {src.shape}")

    for elem in range(src.shape[1]):
        print(f"index {elem}: {src[0][elem].item()}")

    break

"""
batch shape: torch.Size([128, 29])
index 0: 2
index 1: 17
index 2: 16
index 3: 895
index 4: 21
index 5: 14
index 6: 1517
index 7: 10
index 8: 339
index 9: 11
index 10: 172
index 11: 722
index 12: 4
index 13: 3
index 14: 1
index 15: 1
index 16: 1
index 17: 1
index 18: 1
index 19: 1
index 20: 1
index 21: 1
index 22: 1
index 23: 1
index 24: 1
index 25: 1
index 26: 1
index 27: 1
index 28: 1
"""
```

이번 1편에서는 기본적인 데이터 구성에 대해 다루었고, 다음 2편 부터는 Transformer 블록 및 Encoder-Decoder 구조를 구성해보도록 하겠습니다.