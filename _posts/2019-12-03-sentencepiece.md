---
title: SentencePiece 튜토리얼
date: 2019.12.03 12:00:00
categories:
- SentencePiece
tags:
- BERT
- BPE
- SentencePiece
---

.

SentencePiece는 구글에서 2018년 공개한 Unsupervised 형태소 분석 패키지이다. 기본적으로 BPE, unigram, char, word 알고리즘을 제공하며, 이 튜토리얼에서는 BERT에서 쓰이는 BPE를 사용해보려고 한다. BPE가 가장 널리 쓰이는 분야는 Machine Translation이다.

BPE는 Byte Pair Encoding(BPE)의 약자로 1994년 제안되었다. 기본적인 개념은 말뭉치(corpus)에서 가장 많이 등장한 문자열을 병합해 문자열을 압축하는 것이다.

예를 들어, `aaabdaaabac`라는 문자열이 있다고 해보자.

`aa`가 가장 빈출했으므로 `Z`로 치환해보자. `ZabdZabac` 로 압축됨을 알 수 있다.

이제는 ab가 가장 빈출하므로 `Y`로 치환해보자. `ZYdZYac` 로 압축되게 된다.

위의 예시에서 알 수 있듯이 BPE를 적용한다는 뜻은 (사용자가 설정한) vocab 사이즈가 될 때까지 반복적으로 고빈도 문자열들을 병합해서 vocab에 추가한다는 것이다.

학습 후 Inference 시에는 문장 내 각 어절 (띄어쓰기로 문장을 나눈 것)에 vocab에 있는 subword가 포함되는 경우 해당 subword를 최장 일치 기준으로 어절에서 분리한다.

추가적으로 `_` 는 해당 어절의 첫 번째 subword임을 나타내는 구분자이며, BPE는 문자열의 발현 빈도를 통해 vocab을 구성하는 방법론이므로 언어에 관계 없이 적용 가능하다는 장점을 가지고 있다.

## 1. SentencePiece 설치하기

```bash   
pip install sentencepiece == 0.1.83
```

## 2. SentencePiece 실습

```python
import pandas as pd
import sentencepiece

input_file = './data/spm_input.txt'  # 학습하고자 하는 텍스트 파일

df = pd.read_csv('./data/test.csv')

with open(input_file, 'a+', encoding='utf-8') as f:
    for idx, row in df.iterrows():
        f.write('{}\n'.format(row['text']))
```

이 튜토리얼에는 csv파일을 pandas로 읽어서 다시 텍스트 파일을 구성했다. 그 이유는 sentencepiece가 input으로 반드시 텍스트 파일만을 받기 때문이다.

```python
templates = "--input={} --model_prefix={} --vocab_size={} --model_type={} --user_defined_symbols={} --hard_vocab_limit=false"

vocab_size = 2000
prefix = 'test'
user_defined_symbols = '[PAD],[UNK],[CLS],[SEP],[MASK]'
model_type = 'bpe'

cmd = templates.format(input_file, prefix, vocab_size, model_type, user_defined_symbols)

spm.SentencePieceTrainer.Train(cmd)
```

이렇게 학습을 완료하고 나면 `.vocab`, `.model` 파일 두 개가 생긴다.

```python
sp = spm.SentencePieceProcessor()
sp.Load('{}.model'.format(prefix))
```

다음과 같이 모델을 로드할 수 있다.

이제 테스트를 진행해보자.

```python
sp.EncodeAsPieces('안녕하세요, 새로운 기술을 배우고 있습니다')
# 학습에 사용한 데이터량이 적다는 점을 고려.
"""
['▁안',
 '녕하세요',
 ',',
 '▁',
 '새',
 '로',
 '운',
 '▁기',
 '술',
 '을',
 '▁',
 '배',
 '우',
 '고',
 '▁있습니다']
"""

sp.EncodeAsIds('안녕하세요, 새로운 기술을 배우고 있습니다')

"""
[35, 112, 0, 123, 386, 154, 272, 31, 753, 165, 123, 409, 208, 129, 79]
"""
```

```python
with open('{}.vocab'.format(prefix), encoding='utf-8') as f:
    vocabs = [doc.strip() for doc in f]

print('num of vocabs = {}'.format(len(vocabs)))
# num of vocabs = 1000

print(vocabs)
"""
['<unk>\t0',
 '<s>\t0',
 '</s>\t0',
 '[PAD]\t0',
 '[UNK]\t0',
 '[CLS]\t0',
 '[SEP]\t0',
 '[MASK]\t0',
 'XX\t-0',
 'XXX\t-1',
...
"""
```

다음과 같이 정상적으로 user_defined_symbols도 추가되어 있는 것을 볼 수 있다.

```python
sp.bos_id()
# 1

sp.eos_id()
# 2

sp.unk_id()
# 0

sp.IdToPiece(8)
# XX
```

.
