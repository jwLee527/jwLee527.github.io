---
layout: single # 페이지에 single 레이아웃 적용 -> 이 값은 default로 적용해서 안적어도 되긴함
title: "Learning to Memorize Entailment and Discourse Relations for Persona-Consistent Dialogues"
categories: ["paper-summary"]
tags: ["minimal-mistakes", "post"]

sidebar:
    nav: "docs"
---

{% assign posts = site.docs.paper-summary %}

Learning to Memorize Entailment and Discourse Relations for Persona-Consistent Dialogues 논문은 AAAI2023에 submit된 dialogue generation관련 논문이다.   

이 논문은 이전 dialogue generation 모델들은 특이성과 성격 일관성이 부족하고, 그나마 큰 데이터 셋에서는 단편적이고 정보가 없는 응답을 생성한다는 문제점이 있다고 하였다.   
이에 대한 문제점에서 저자들이 생각하는 dialogue에서 일관성을 유지하는 한 가지 솔루션은 특징을 설명하고 Persona에 따라 응답을 생성하는 Persona 프로파일 셋을 제공하는 것이라고 하였다.   
여기서 Persona는 프로필, 개인적인 배경 사실 등 정체성 요소의 구성이다.   

위의 솔루션을 바탕으로 저자들은 다음과 같은 모델 architecture를 제시하였다.   

![model architecture](https://github.com/jwLee527/test/assets/144921672/34c81cb3-0fd9-4e71-a0cd-3a2b62e065db){: .align-center}   
이 모델의 백본은 BART 모델을 활용하였고, 제시된 모델의 키 포인트는 수반 관계와 담화 정보에 대한 latent spaces를 모두 활용했다는 것이다.  

![model algorithm](https://github.com/jwLee527/test/assets/144921672/a8e5cdc2-5bc7-4ac4-a9e7-d18acb6cd512){: .align-center}   
모델의 latent memory 학습 알고리즘을 보게 되면 2단계로 나누어져 있는데 이는 모델 파라미터를 크게 2번 학습시키는 것이고, 모델에 대한 설명을 할때는 3가지 파트(수반 관계, 담화 정보, 생성)로 나누어 설명하는 것이 편할것 같고 논문에서도 3가지 파트로 나누어져 있어 파트별로 알아보자.

### 수반 관계
ERM(Entailment Relation Memory)은 persona 일관성을 위해 수반 관계를 학습하고 저장하는 데 사용되는 외부 메모리 M이다. 논문에서는 '주어진 가설 H이 전제 P를 추론할 수 있다면, 그 관계는 수반이다.'라고 표현하였다. 그리고 persona 기반 대화의 경우, 저자들은 수반 관계를 도입하여 일관된 응답을 생성할 수 있다고 하였다.   
학습에는 persona를 인풋으로 두어 그에 대한 가설을 얻을 수 있도록 M를 학습시키는 방식으로 인코더에서 얻어지는 잠재 수반 관계 표현인 z를 디코더의 특별 토큰인 start-of-hypothesis 토큰에 더하는 방식을 선택하였고, 메모리 M와 모델 파라미터를 최적화 시키기 위해 loss function은 language modeling loss($$L_{ERM}$$)을 활용하였다.

### 담화 정보
DDM(Dialogue Discourse Memory)는 이후 답변 생성에 기반이 되는 내부 메모리 N이다. 논문에서는 대화 일관성은 담화 문장의 품질 측면에서 중요하다고 하였고, 이를 위해 저자들은 DDM을 학습하기 위해 persona와 대화 내용을 토큰으로 붙여 학습 데이터를 생성하였다.   
여기서 잠재 수반 관계와 대화의 맥락은 독립적이여야 하기 때문에 저자들은 서로 다른 잠재 공간 간의 상관 관계를 줄이기 위해 두 메모리 공간에 직교 제약 조건을 부과하였다. 직교 제약 조건은 잠재 메모리가 더 많은 기능을 학습하고 중복 기능을 줄이도록 할 수 있고, 여기서는 코사인 유사도와 L2 normalization를 활용하여 loss($$L_{DDM}$$)를 계산하였다.

### 생성
저자들의 설명으론 모델은 이전 단계였던 수반 관계와 담화 정보에서 얻은 메모리들을 활용하여 persona 기반 답변을 생성해낼 수 있다고 한다. 이 메모리들로 얻어지는 잠재 수반 관계 표현인 z와 잠재 대화 담화 표현인 $$z_d$$를 디코더 인풋에서 특별 토큰인 start-of-hypothesis 토큰에 더하는 방식으로 활용하였다.   
![added token](https://github.com/jwLee527/test/assets/144921672/6ad39e84-15bd-4f19-aec6-af4383db8568){: .align-center}   

그리고 학습에 대한 loss function으로서는 latent variable에 대해 bag-of-words loss($$L_{BOW}$$)를 활용하였고, 답변 생성에서 모델의 파라미터에 대해서는 language modeling loss function($$L_{LM}$$)을, 마지막으로 디코더가 출력한 마지막 토큰의 hidden state를 사용하여 각 후보 응답의 점수를 예측하고 실측 라벨을 사용하여 cross-entropy loss($$L_{CLS}$$)를 활용하였다.   

학습에서는 결국 위의 모델 알고리즘에서와 같이 두번에 나누어 학습하는데 첫번째는 수반 관계 단계에서의 $$L_{ERM}$$에 대해서만 진행하고 두번째는 나머지 단계의 로스들을 더하여 진행한다.
![stage 2 loss](https://github.com/jwLee527/test/assets/144921672/cd5f263f-64c4-4aa4-8286-5603ac55d15a){: .align-center}   

## 평가
최근 담화 생성 모델에 관하여 관심이 있어 직접 학습을 시켜보기 위해 논문에서 제시한 'DSTC7-AVSD' 데이터셋과 NLI에 대해서는 'MNLI'로 학습한 결과,   
답변이 이전의 모델들보다 주어진 정보에서의 답을 하긴 하지만 질문에 대한 답변이 아닌 경우도 아직은 많고 사람과 대화하는 것처럼 느껴지진 못한다고 느껴 논문에서 보여지는 것처럼 automatic evaluation은 확실히 점수가 괜찮을 순 있어도 아직 사람이 평가하는 점수에서는 좋은 점수를 받기 어렵다라는 생각을 하였다.   