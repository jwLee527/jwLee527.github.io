---
layout: single # 페이지에 single 레이아웃 적용 -> 이 값은 default로 적용해서 안적어도 되긴함
title: "Contextual Knowledge Learning For Dialogue Generation"
categories: ["paper-summary"]
tags: ["minimal-mistakes", "post"]

sidebar:
    nav: "docs"
---

{% assign posts = site.docs.paper-summary %}

Contextual Knowledge Learning For Dialogue Generation 논문은 ACL 2023에 submit된 dialogue generation관련 논문이다.

본 논문에서는 dialogue generation task에서 최근 일반적으로 제시되는 방식인 Knowledge-grounded dialogue generation에 대해   
Context 정보가 주어지면 knowledge에서 필요한 정보를 찾아(knowledge selection task)과 최종 응답(response)를 생성하는 generator를 따르는 retriever를 고안(dialogue generation task)로 나눌 수 있다고 하였다.

우선 Knowledge-grounded dialogue generation 방식의 이전 연구들의 모델은 학습에서 end-to-end 형태가 아닌 모델들이 다단계 학습을 하게 되면서 오류가 누적이 되고 context와 knowledge에서 정확한 정보를 찾아 response를 생성하지 못하였다고 주장하였다.   
그래서 본인들은 end-to-end의 형식의 모델을 구현하고 모델 내부에서 input를 다른 모델들과 다른 방식으로 처리하여 위의 문제들을 해결하였다고 소개하였고,   
추가로 [Wang et al. 2019](https://arxiv.org/abs/1907.00607)에서 generation task에서는 학습하는 동안 discriminator에 가중치를 적게 두는 것이 성능에 좋다는 주장을 본인들의 dialogue generation에서도 적용하였다고 한다.   

## Model architecture
![model architecture](https://github.com/jwLee527/test/assets/144921672/85625cd7-1bec-48a0-98d7-c6286cd8f35e){: .align-center}
*Figure 1*

모델의 구조를 살펴보면   
encoder의 경우에는 Bart-base의 encoder를 그대로 활용하였고 context와 knowledge를 max input size에 맞게 맞춰 만들어 모델의 input으로 활용하였다.   
그렇게 encoder를 통과한 Context&Knowledge Representation를 context, knowledge 길이에 맞게 분리한 값을 각각 CLW Generator와 KLW Generator에 넣어주고 전체 값은 decoder에 넣어주게 된다.

CLW Generator와 KLW Generator는 각각 Context와 Knowledge에 대한 고정된 단어의 인덱스를 1로 두는 work embedding을 통해 Latent Vector를 생성한다고 하였고,   
CLW Generator는 두 개의 Latent Weight Head를 통해 CLWR, CLWK인 두 스칼라 값을 얻는데 이는 KLW Generator와 Decoder에서 LWE Cross Attention에 활용되고 두 값은 같은 구조이지만 파라미터는 공유하지 않는 Latent Weight Head를 통과한다. LWE Cross Attention은 좀 이따 간단하게 설명을 하기로 하겠다.   
KLW Generator는 CLW Generator와 마찬가지로 Latent Weight Head를 통해 얻은 KLW인 스칼라 값을 Decoder에 LWE Cross Attention에서 활용한다.

### LWE Cross Attention(Latent Weight Enhanced Cross Attention)
![LWE Attention](https://github.com/jwLee527/test/assets/144921672/e9b9562a-1ff9-43c3-ad37-7caf8b284648){: .align-center}
*Figure 2*

논문에서는 Attention은 두 sequence 사이에서의 word-level의 계산이라면 LWE Attention의 경우에는 sentence-level의 계산이라고 설명하였다.   
KLW Generator의 LWE Attention에서의 LW는 CLWK으로 치환해서 계산하면 되고, PE Attention은 Decoder에서의 Context&Knowledge LWE Cross Attention을 의미한다.

## Loss Function
모델의 loss function은 Figure 1에서의 상단부에서 볼 수 있는데,,   
$$Loss_{CLWR}, Loss_{CLWK}과 Loss_{KLW}$$은 각각 Ground Truth와 MSE Loss를 통해 계산되고 $$Loss_{NLL}$$은 기존의 Bart모델의 loss function인 Negative Log Likelihood(NLL)를 사용한다.   
최종적으로 서로 다른 네 loss function에 대해서 Automatic Weighted Loss(AWL)를 활용해 최적화를 시켜 위의 loss function과 같은 식이 완성되었다.

## Result
![experience1](https://github.com/jwLee527/test/assets/144921672/30f25bd0-eb5f-4cd4-b667-bbba9d89b4f8){: .align-center}
*Figure 3*

논문의 수치적인 결과들도 나와있지만 개인적인 생각으론 Dialogue Generation task는 서비스 제공을 위한 연구 성향이 강하다고 생각하여 수치적인 결과보다 실제 생성된 답변의 예시가 더 중요하다고 판단해 논문에서 보이는 Good, Bad case를 갖고 왔다.   
~~이 논문에서 제시하는 모델 코드가 github에 올라와 있지 않아 따로 실행은 시켜보지 못했다.~~   
Figure 3에서 보이는 예시에서 CKL모델이 생성한 response는 대화가 진행될 수록 문맥을 파악하지 못하다는 생각이 들게 되었다. 하지만 Dialogue Generation task에서 Latent Modeling이 대세인것 같은 현재에서 본 모델 구조 중 가장 관심이 가는 구조였고 좀 더 공부해보고 싶다는 생각이 들었다.