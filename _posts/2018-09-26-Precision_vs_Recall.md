---
title: Precision vs Recall vs F1 Score
date: 2018.09.26 12:00:00
categories:
- Machine Learning
tags:
- Metrics
- Precision
- Recall
- F1 Score
---


.

# Precision vs Recall vs f1 score

머신러닝 또는 딥러닝 모델을 학습시킨 다음 이 모델이 잘 학습되었는지 확인하기 위한 지표(metrics)는 여러가지가 있다. 이럴 때 어떻게 하면 정확한(accurate) 모델을 찾을 수 있을까?라는 질문에 accuracy라는 답을 내놓는다면 다소 문제가 생길 수 있다. Accuracy는 말 그대로 one-and only model metrics가 아니기 때문이다.

예를 들어 다음과 같은 Confusion Matrix 가 있다고 생각해보자.

|                  | Nagative(Predicted) | Positive (Predicted) |
| :--------------: | :-----------------: | :------------------: |
| Negative(Actual) |         998         |          0           |
| Positive(Actual) |          1          |          1           |

이 경우에 accuracy는 무려 99.9%가 된다. 물론 좋은 결과라고 생각할 수 있지만, 데이터를 조금만 더 살펴보면 전부 Negative라고 분류해도 99.8%의 정확도를 얻을 수 있다는 걸 알 수 있다. 따라서 이런 imbalance한 데이터에는 accuracy라는 지표가 그리 좋다고 말할 수 없다.

여기서 Precision과 Recall이라는 지표가 등장한다. 각각은 다음과 같이 표현된다.

$$Precision = \frac{True Positive}{True Positive + False Positive} = \frac{True Positive}{Total Predicted Positive}$$

$$Recall = \frac{True Positive}{False Nagative + True Positive} = \frac{True Positive}{Total Acutal Positive}$$

Precision은 `False Positive`가 발생했을 때의 비용이 비쌀 경우에 효과적인 metric이다. 예를 들어, 이메일 spam filter의 경우에 False Positive가 발생하게 되면 사용자가 중요한 메일을 놓칠 수도 있기 때문에 이 경우에는 Precision을 높이는 방향을 설정하는 것이 좋다. 반면, Recall은 `False Negative`가 발생했을 때의 비용이 비쌀 경우에 효과적이다. 예를 들어, 암환자 진단의 경우에 실제 암환자인데 암이 아니라고 판명하면 큰 문제가 생기므로 Recall을 높이는 방향으로 설정하는 것이 바람직하다.

### F1 Score

위에서 설명한 Precision과 Recall을 적절히 혼합해서 만든 지표가 `F1 score`이다. 이 지표를 통해 Precision과 Recall 사이의 균형을 맞출 수 있다.

$$ F1 Score = 2 *\frac{Precision*Recall}{Precision + Recall} $$

앞서 봤듯이 Accuracy는 `True Negative`가 큰 imbalance한 데이터에 대해서 제대로 작동하지 못하지만, `F1 Score`는 이 `True Negative`가 큰 imbalance한 데이터에 대해서 상대적으로 robust한 결과를 가질 수 있게 도와준다.

.
