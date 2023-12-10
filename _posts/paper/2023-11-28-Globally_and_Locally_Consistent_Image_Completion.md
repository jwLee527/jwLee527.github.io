---
layout: single # 페이지에 single 레이아웃 적용 -> 이 값은 default로 적용해서 안적어도 되긴함
title: "Globally and Locally Consistent Image Completion paper review"
categories: ["paper-summary"]
tags: ["minimal-mistakes", "post"]

sidebar:
    nav: "docs"
---

{% assign posts = site.docs.paper-summary %}

Globally and Locally Consistent Image Completion 논문은 2017년도에 나온 image inpainting task관련 논문이다.

이 논문은 이전에 리뷰했던 [Context Encoder](https://jwlee527.github.io/paper-summary/Context_Encoder/) 이후에 나온 논문으로 같은 작업에 대한 논문이라 논문 모델의 baseline도 Context Encoder이다.   
그래서 이번 논문의 리뷰에서는 Image Inpainting task에 관한 내용은 넘어가고 바로 모델에 대한 리뷰를 진행하겠다.

## Model architecture
![Model architecture](https://github.com/jwLee527/test/assets/144921672/f2b25c1e-7c42-4497-a3b1-584952ca65bc){: .align-center}
*Figure 1*

Context Encoder 모델을 기반으로 구성한 모델이라 대체적으로 모델의 형태는 같아보이지만 구성되어 있는 요소들은 모두 다르다.

![Context Encoder](https://github.com/jwLee527/test/assets/144921672/0c2d8e86-3b71-44df-be5c-141f77c561c6){: .align-center}
*Figure 2(context encoder)*

이후 모델에 대해 좀 더 살펴보겠지만 Figure 1(논문의 모델)과 Figure 2(context encoder)를 비교해보면   
auto-encoder에서 code라 불리는 encoder에서 생성한 아웃풋을 decoder로 전달해주는 부분이 context encoder에서는 channel-wise fully connected layer로 되어있지만 논문의 모델은 dilated convolution layer로 구성되어 있고   
discriminator 부분에서는 Figure 2에서는 보이지 않지만 context encoder는 Figure 1 기준으로 Global Discriminator 하나가 있다고 생각하면 쉬울 것 같지만 논문의 모델은 두 개의 discriminator를 갖고 있다.
이러한 모델의 차이점이 어떤 결과를 불러일으키는지는 뒤에서 알아보자.

## Completion Network

우선 generator인 Completion Network의 구조를 살표보면 다음과 같다.

![Completion Network](https://github.com/jwLee527/test/assets/144921672/afca50c3-ab49-4d09-8156-03cca23df707){: .align-center}
*Figure 3(Completion Network)*

### Encoder and Decoder
~~논문 모델을 GLCIC라고 하겠다.~~
GLCIC에서 encoder와 decoder는 이전 모델인 context encoder보다 적은 convolution layer을 활용하여 모델의 구조를 구성했는데,   
encoder는 이후에 dilated convolution을 이용하여 넓은 영역에 대한 convolution 연산을 할 수 있기 때문이고   
decoder는 적은 deconvolution layer가 모델의 성능에 좋은 영향을 준다고 하였다.

여기서 적은 deconvolution layer를 사용하는 이유는 아래와 같다.
1. deconvolution layer가 많을 수록 이미지의 디테일을 잃음   
2. deconvolution layer가 많을 수록 이미지가 세밀한 텍스쳐를 생성할 수 없음   

### Dilated Convolution
[dilated convolution](https://jwlee527.github.io/concept-summary/Dilated_Convolution/)은 이전에 정리한 내용이고

이 모델의 경우에는 