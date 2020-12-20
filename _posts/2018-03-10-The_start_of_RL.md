---
title: 강화 학습의 시작
date: 2018.03.10 12:00:00
categories:
- Reinforcement Learning
tags:
- RL
---


머신러닝을 공부하면서 또는 여러 논문들을 읽어나가면서 항상 뜻이 헷갈리는 단어를 고르자면 `Induction` 과, `Deduction` 일 것입니다.

> Induction: 귀납법

> Deduction: 추론, 연역법

쉽게 말해 `Induction` , 또는 `Inductive Reasoning` 은 구체적인 관찰으로부터 일반화로 진행해가는 과정을 말합니다. 예를 들어 `1->1`, `2->4`, `3->9` 의 관계를 가지는 인풋과 아웃풋이 있다면, 다음 인풋이 4인 경우 16이라고 근사(approximation) 하는 것입니다.



반대로  `Deduction`, 또는 `Deductive Reasoning` 은 일반적인 관계들 속에서 특별한 또는 구체적인 관계를 추론해 내는 것을 일컫습니다. 예를 들어 다양한 픽셀들이 있을 때, 다리가 네 개 그리고 등받이가 있는 픽셀들이 있는 경우 우리는 이 픽셀들이 나타내는 그림은 의자 그림이라고 추론할 수 있습니다.



이것들을 좀더 명확하게 이해하기 위해 `supervised learning` 과 `unsupervised learning` 의 개념과 연결해서 이해를 한다면 다음과 같습니다.



![Unsupervised/Supervised Learning](/images/unsupervised_supervised.png)



이와 다르게 앞으로 다룰 `Reinforcement Learning` 은 몇 가지 다른 특징을 가집니다.

- Supervisor 없이 오직 reward 시그널에 의해 학습을 진행함.
- 피드백이 즉각적인 것이 아닌 지연되서 발생함.
- 순차적인 것이 중요함.
