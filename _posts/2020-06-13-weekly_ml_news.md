---
title: Weekly ML/DL News (6월 3주차)
date: 2020.06.13 12:00:00
categories:
- news
tags:
- ML
- DL
---


새로운 기술에 대해 관심이 많은 입장에서 어느샌가 페이스북은 친구들과의 소통창구가 아닌 기술뉴스 스크랩(나만보기)하는 장소가 되었다.

그렇지만 여기서의 맹점은 나만보기로 저장만 해놓을 분 "나도 보지 않는다"는 것이다 (아마 정말 많은 분들이 이러고 있지 않을까..)

그러다 우연한 기회에 회사 동료들에게 이런 뉴스들을 소개했더니 생각보다 반응이 좋았고, "나만보기"에 쌓여있는 뉴스들을 이제는 글로 정리해보고자 한다.

목표는 매주이지만 지켜질지는.. 한번 도전해보는걸로.

## 목차

- [Meta / Multi-task Learning](#meta-learning--multi-task-learning)
- [고려대연구실 자료](#고려대-dsba)
- [huggingface nlp viewer](#huggingface-nlp-viewer)
- [Jukebox](#jukebox)
- [RL blog](#rl-blog)
- [Fake data generator](#fake-data-generator)
- [NLP 100제](#nlp-100제)
- [d2l pytorch](#d2l-pytorch)
- [Super Duper NLP](#super-duper-nlp)
- [DS handbook](#ds-handbook)
- [Keras cheatsheet](#keras-cheatsheet)


### Meta Learning / Multi task Learning

[자료링크](https://cs330.stanford.edu/)

아무리 생각해도 스탠포드는 참 자비?로운듯하다. 항상 이렇게 좋은 강의를 자료 뿐만 아니라 영상까지 올려주다니..

이번에는 Meta Learning과 Multi Learning에 관련된 강의 CS330의 영상/자료를 공개했다. 조만간 한 번 정주행해봐야할듯.

### 고려대 DSBA

[자료링크](https://www.youtube.com/playlist?list=PLetSlH8YjIfVdobI2IkAQnNTb1Bt5Ji9U)

스탠포드 자랑에 이어서 우리나라에는 고려대 강필성교수님 연구실에서 이렇게 내부 세미나 자료를 공개해주시고 있다.

강필성 교수님 강의를 들은 적이 있는데 물론 정말 뛰어난 교수님들이 계시지만 이 분 강의는 등록금내고 들을만한 가치가 있다고 생각했었다. 정말 이해가 잘되게 설명해주셨던..

최근에는 이렇게 자료공개에까지 앞장서 주시니 정말 좋은 것 같다. 이제 한국어 관련 머신러닝/딥러닝 자료들이 점점 늘어나는듯해서 긍정적인듯.

### huggingface nlp viewer

[자료링크](https://huggingface.co/nlp/viewer/)

최근 NLP에서 빼놓고 이야기하기 힘든 연구집단? 회사?인 허깅페이스가 이번에는 NLP 데이터셋 뷰어를 공개했다.

그러면서 본인들이 TensorFlow Dataset 기반으로 만들었다는 [nlp 라이브러리](https://github.com/huggingface/nlp)를 슬쩍 홍보하고 있다.

시간이 가면 갈수록 이렇게 데이터를 불러오고 가공하는 부분은 점점 편해져가는 느낌. 그렇지만 한 편으로는 모델링에 대한 집중도를 높이는 것이 아닌가 하는 생각도 든다.

실제 회사에 있는 데이터는 저렇게 깨끗하고 말끔하지 않기에.. 취업을 준비하시는 분들은 데이터 클렌징에 대한 관심도를 더 높이는게 좋지 않을까하는 생각.

### Jukebox

[자료링크](https://openai.com/blog/jukebox/)

OpenAI에서 music generator에 대한 논문과 POC를 공개했다. 음원을 들어보니 꽤나 그럴싸한데.. 어쩌면 몇 년 뒤 쯤엔 정말 음악을 공장에서 찍어낼 수 있지 않을까하는 생각이 든다.

물론 창작의 영역을 무시할 수 없지만, 최근 아이돌 노래나 유행하는 노래는 어느정도의 패턴이 있다는 이야기들을 자주 들어서 어쩌면 그런 타입의 작곡은 인간보다
기계가 더 잘할 수 있는 영역일 수도 있지 않을까..

### RL blog

[자료링크](https://teamdable.github.io/techblog/Reinforcement-Learning)

컨셉 자체가 너무나도 매력적인 RL. 항상 공부를 시작하다가 DQN 쯤 볼 때가 되면 다른 일이 생기거나 열정이 떨어져서 공부를 미뤄뒀었는데 이렇게 DQN까지 잘 정리해주신 블로그가 있었다.

관심있는 분들은 쭉 읽어보시면서 개념을 잡아보는 것도 좋을듯.

### Fake data generator

[자료링크](https://github.com/lk-geimfari/mimesis)

개발자들이 제일 힘들어하는 것이 변수명 짓기라면 데이터 쪽에서 가장 별거아닌데 귀찮고 하기 힘든 것 중 하나 (사실 그런게 너무 많아서..)는 가짜 데이터 생성하는게 아닌가 싶다.

그러다보니 이렇게 라이브러리 형태로 다양한 타입의 데이터를 생성해주는 라이브러리가 등장했다.

사실 예전에도 이런 라이브러리를 본 듯하여, 어떤 큰 차이가 있는지는 모르겠지만 나름 2800스타를 받은 프로젝트이니 한 번쯤 활용해봐도 좋을듯.

### NLP 100제

[자료링크](https://nlp100.github.io/ko/)

머신러닝/딥러닝에 대한 관심이 커져가다보니.. 이제 문제집이 나왔다. 말 그대로 NLP 100제. 사실 이런 식의 접근이 정말 효과적일까하는 생각을 하기는 하는데
내용을 대충보니 어디부터 공부를 시작해야할 지 감을 못잡는 친구들에게는 효과적일 수도 있겠다는 생각을 했다.

영어 원문자료가 따로있고 한글번역을 진행한 것으로 보이니 번역이 이상하다면 원문자료를 참고하면 좋을듯.

### d2l pytorch

[자료링크](https://d2l.ai/)

이 자료가 얼마나 유명한지는 사실 잘 모르겠지만, 개인적으로는 잘 정리된 교재라고 생각한다. 원래 예제코드가 mxnet 기반이였는데 최근에 pytorch 코드도 지원하기 시작했다고 한다.

정말 공부할 수 있는 자료는 넘쳐나는듯.

### Super Duper NLP

[자료링크](https://notebooks.quantumstat.com/)

NLP분야의 유명한 또는 유용한 구현체들을 코랩에 구현해놓은 모음집이다. 새로운 모델을 구현할 때 참고하면 정말 편할듯.

이런 점에서 구글 코랩의 등장은 이런 코드공유를 촉진시키는 효과가 나는 듯하다. 거기다 최근에 유료화를 진행했음에도 무료로 GPU 할당을 어느 정도는 해주는 혜자스러움..

### DS handbook

[자료링크](https://colab.research.google.com/github/jakevdp/PythonDataScienceHandbook/blob/master/notebooks/Index.ipynb)

이 자료 또한 코랩에 예제코드를 정리해놓았음. 책 자체를 이렇게 코랩에 옮겨놓아서 예제를 바로바로 실행할 수 있는 장점이!

데이터 시각화 쪽 정리도 잘 되어 있는듯 해서 한번 쯤 훑어보면 좋을듯.

### Keras cheatsheet

[자료링크](https://drive.google.com/file/d/1OvLEa_WYA_4_lHptVfevZhHtrcx_-Jn1/view)

개인적으로는 케라스를 잘 활용하지는 않는 편이지만, 최근 TF2에 포함된 Keras API를 사용해보니 처음 머신러닝/딥러닝을 접하는 사람들이 사용하기에 좋은 프레임워크라는 생각이 들었다.

케라스를 주로 활용하는 분들은 한 번 훑어보면 좋을듯.


끝.
