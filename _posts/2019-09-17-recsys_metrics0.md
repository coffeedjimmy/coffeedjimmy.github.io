---
layout: post
title: "추천 시스템 평가 Metrics"
tags: [recsys, metrics]
comments: true
---

## Precision at K

- top k의 결과로 Precision 계산
- 관련이 있는지 없는지를 binary 로 표현해서 계산
- 예를 들어, 어떤 키워드에 대한 검색 결과에 대한 실제 유사도가 다음과 같다면,
    - O, X, X,O,X
    - Precision 3는 1/3
    - Precision 4는 1/2
    - Precision 5는 2/5

## Mean Average Precision (MAP)

- 각 검색 결과에 대한 Average Precision을 평균하는 개념
- 첫 번째 검색 결과의 Average Precision이 0.62, 두 번째 검색 결과의 Average Precision이 0.44라면, MAP는 (0.62+0.44)/2 = 0.53

## Normalising Discounted Cumulative Gain (NDCG)

- 추천의 순서에 좀 더 가중치를 두기를 원해서 고안된 방식
- top1과 top2의 차이는 큰 차이지만 top100과 top101의 차이는 상대적으로 적음 (크게 의미 있지 않음) → log function
- NDCG = DCG / IDCG
- 예를 들어, relevance를 측정하는 지표가 있다고 할 때, 실제 검색의 결과의 rel 지표가 4,2,0,1 와 같이 나왔고, Ideal한 rel 지표가 4,2,1,0 이라고 하면 다음과 같은 값들을 가진다.
    - DCG = 4 + (2/log2) + 0/log3 + 1/log4 = 6.5 (밑이 2인 로그)
    - IDCG = 6.64
    - NDCG = 0.979

## Entropy Diversity

- 추천의 다양성을 측정하기 위한 지표
    - [http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.459.8174&rep=rep1&type=pdf](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.459.8174&rep=rep1&type=pdf)
