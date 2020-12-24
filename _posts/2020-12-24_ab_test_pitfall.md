---
title: AB Test Pitfall
date: 2020.12.24 12:00:00
categories:
- AB Test
tags:
- AB Test
---

AB 테스트를 진행함에 있어 빠질 수 있는 문제에 대해 다룹니다.

`원문자료`: [Interference: a tricky pitfall of A/B Testing](https://towardsdatascience.com/interference-a-tricky-pitfall-of-a-b-testing-f940464cb5a0)

데이터 사이언티스트들은 반드시 A/B 테스트를 할 때 interference, endogeneity issues 등으로 생길 수 있는 위험에 대해 인지하고 있어야 한다. 

이 글은 그 중에서도 interference의 위험성과 예시를 보여주며 어떻게 해야 interference를 피하는 A/B 테스트를 구성할 수 있는지 다룬다.

## Interference due to shared resource

우버와 같은 공유차량 업체가 있다고 가정했을 때 이 업체는 차량을 부르는데까지 걸리는 스탭을 줄이려는 시도를 하였고, 그에 대해 A/B테스트를 진행한다고 가정해보자.

이 경우 평가지표로 특정 시간 내에 한 기사에게 얼마나 많은 승차가 일어났는지를 정했다.

데이터 사이언스 팀은 뉴욕에 있는 기사들 중 랜덤하게 실험군 / 대조군을 분리하였고 2주동안 A/B 테스트를 진행하였다. 그리고 팀은 2주 동안 평가지표 관련 로그들을 수집하고 2-sample t-test를 진행했다.

결과에 대한 통계치가 나온 후에 팀은 p-value가 유의미하다고 판단했고, 새로운 UI가 기사들이 더 많은 승객을 태우고 더 많은 돈을 벌게 해준다고 주장했다. 당신이 매니저라고 가정했을 때, 이런 결과를 DS팀을 통해 들었고 이제 실제 개발팀에 이 새로운 UI를 적용해야한다 혹은 아니다라는 결정을 내려줘야 한다.

사실 이런 얘시는 일반적인 A/B 테스트 인터뷰 문제와 다르지 않다.

우리는 이 실험결과로 어떠한 결론도 내릴 수 없는데, 그 이유는 바로 두 그룹 (실험군/대조군) 운전자들이 뉴욕이라는 같은 장소에 있고 그들은 시장의 공급과 수요를 공유하고 있기 때문이다.

만약 새로운 UI가 적용된 기사들이 이득을 봤다면 그들은 시장의 수요에 영향을 주었을 것이다. 결론적으로 대조군에 속한 기사들은 상대적으로 적은 승객들을 수요로 가지고 있었을 것이고 평소보다 적은 승객을 태웠을 것이다. 

요약컨데 이 경우에 관찰된 실험군 효과는 실제 실험효과 대비 과장된 측면이 있다. 그래서 우리는 실제 실험효과가 통계적으로 유의미했을지 알 방법이 없다.

## The idea of Interference

위의 예시를 통해 아마 interference에 대한 대략적인 개념을 잡았을 것이다. 이 개념의 본질은 정말 명확하다. 다수의 실험군 그룹이 주어졌을 때 한 그룹의 행동이 다른 그룹의 행동에 영향을 준다. 이것이 바로 위의 예시에서 드러난 문제이다. 더 많은 승객이 한 그룹의 기사에게로 간다면 다른 그룹의 기사에게는 상대적으로 적은 승객들이 갈 수 밖에 없는 것과 같다.

## Types of interference

일반적으로 interference는 두 가지 방식으로 발생한다: indirect/direct connection

### Indirectly Connection

Indirectly connection은 두 개의 유저군이 잠재변수나 공유된 리소스 때문에 연결되어 있는 경우에 발생한다. 실험군과 대조군이 같은 고객 풀을 공유했기에 위에서 다뤘던 예시가 정확하게 이 indirect connection의 예시다.

Sub-user experimentation unints 또한 이 예시 중 하나라고 볼 수 있다

: 새로운 UI를 적용하는 것에 대한 실험이 수행되었고 실험 단위가 유저레벨이 아닌 페이지뷰 레벨에 있다고 가정해보자. 이 경우에 유저는 잠재변수이며 실험군/대조군에 연계되어 있다. 그 결과 두 버전의 UI는 같은 유저에게 노출될 수가 있고 실험 결과를 신뢰하지 못하게 만들 수 있다.

### Direct Connection

반대로 Direct connection은 두 단위가 같은 소셜 네트워크에 속하거나 물리적인 접점이 있는 경우 발생할 수 있다. 그렇기에 LinkedIn 엔지니어링 팀에서는 interference를 네트워크 효과라고 간주한다. Saint-Jacques(2019)의 예시를 보자.

상상해보자. 내 친구가 실험 타겟이 되었고 그 실험을 통해 내 친구는 더 나은 메시징 경험을 가지게 되었다. 그렇지만 나는 타겟이 아니였고, 내 메시징 경험은 변화하지 않았다. 그러나 내 친구의 보다 나은 경험은 더 많은 시간을 메시지를 보내는데 사용하게 만들었고 난 친구이기에 더 많은 시간을 메시지에 사용하게 된다. 그 결과 나는 실험에 속하지 않았지만 내 친구로 인해 사용량이 늘게 되었고 이는 실험에 영향을 미치게 되는 Interference를 발생시킨다.

## Identify interference when designing an experiment

Interference 는 가정 기반 테스트에서 bias error를 발생시키고 전체 실험결과를 신뢰하지 못하게 만든다. 이런 이슈를 피하기 위해서 극단적인 케이스를 고려해보자. 만약 실험군에 적용된 변화가  중요  평가지표를 10배 상승시킨다고 하면 대조군에도 그 영향이 미치는가? 만약 영향을 미친다고 하면, Interference가 발생할 수 있고 아래의 솔루션을 참고하길 바란다.

## Possible solutions to interference (after you are aware of it)

가능한 한 가지 해결책은 randomisation unit을 변경하는 것이다.

만약 interference이 지리적인 단위로부터 기인한다면 우리는 지역단위 랜덤화를 진행해야한다. Vaver and Koehler (2011)은 관심 지리지역 (예: 국가)를 일련의 지역들로 분할하는 실험을 설계했다. 그리고 각 지역은 랜덤하게 실험군과 대조군에 할당되었다. 논문저자들은 또한 실험군과 대조군 지역의 밸런스를 맞추기 위해 블럭 랜덤화와 같은 것을 진행할 수도 있다.

비슷하게 소셜네트워크의 관점에서 간섭 가능도에 따라 네트워크 노드들의 클러스터를 구성할 수 있다. 그리고 이 클러스터를 실험의 단위로 삼을 수 있다.

위의 방식과는 다르게 랜덤화 과정에서 시간 간격을 사용할 수도 있다. 이 과정에서 시간적인 요인 (주중/주말 등) 을 고려하는 것을 잊어서는 안되며 paired t-test는 분산을 줄일 수 있는 좋은 방법이다.

그렇지만 실험군이 변경될 때마다 bias-varience tradeoff는 여전히 존재한다는 것을 인지해야 한다. 보다 구체적인 레벨인 유저와 같은 레벨은 interference 효과로 분산은 줄이지만 편향이 커지게 된다. 반대로 시간 간격이나 지역 등과 같은 보다 일반적인 레벨에서는 편향은 적겠지만 보다 큰 분산을 가지게 된다. (그룹 내 분산 때문)

## Last but not least

이 글의 목적은 사람들에게 기본적인 interference에 대한 개념을 전달하기 위함이다. 그리고 이 글을 읽는 사람들에게 아래의 레퍼런스 리스트를 같이 읽기를 권장하며 이를 통해 보다 넓은 내용들에 대한 지식을 얻을 수 있을 것이다.

## References

- Chamandy, N. (2016, September 2). Experimentation in a Ridesharing Marketplace. Retrieved from [https://eng.lyft.com/experimentation-in-a-ridesharing-marketplace-b39db027a66e](https://eng.lyft.com/experimentation-in-a-ridesharing-marketplace-b39db027a66e)

- Kohavi, R., Tang, D., & Xu, Y. (2020). *Trustworthy Online Controlled Experiments: A Practical Guide to A/B Testing*. Cambridge University Press.

- Saint-Jacques, G. (2019, June 5). Detecting interference: An A/B test of A/B tests. Retrieved May 22, 2020, from [https://engineering.linkedin.com/blog/2019/06/detecting-interference--an-a-b-test-of-a-b-tests](https://engineering.linkedin.com/blog/2019/06/detecting-interference--an-a-b-test-of-a-b-tests)

- Vaver, J., & Koehler, J. (2011). Measuring ad effectiveness using geo experiments.