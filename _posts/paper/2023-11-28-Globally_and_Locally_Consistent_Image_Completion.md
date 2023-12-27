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

이 모델의 경우에는 이미지의 큰 빈 공간에 대해 dilated convolution을 활용하여 정보에 대한 손실을 최소화 하였다.

## Discriminator   
![discriminator](https://github.com/jwLee527/test/assets/144921672/4b4ae995-d72d-490b-8454-fe8816585f55){: .align-center}
*Figure 4(Discriminator)*

Context Encoder에서 Global Discriminator만을 사용하여 Local consistency가 부족한 단점을 보완하기 위해 GLCIC에서는 Local Discriminator를 추가하여 Local consistency를 채웠다.   

Global Discriminator는 Completion Network의 output을 input(256*256)으로 하고 구조는 5개의 convolution layer로 이루어져 있으며,   
Local Discriminator는 Completion Network의 output에서 mask 부분이였던 생성 부분을 input(128*128)으로 하여 4개의 convolution layer로 이루어져 있다.

이후 두 Discriminator의 output을 합친 후 Fully Connected layer를 통해 결과값을 얻는다.

## Loss Function
![GAN loss](https://github.com/jwLee527/test/assets/144921672/fbc06b75-ab34-41d7-b35c-fa13c05dfe69){: .align-center}
*Figure 5(GAN Loss)*

GLCIC에서도 Context Encoder와 같이 MSE loss와 GAN loss를 합친 방식 loss를 활용 하였는데,   
다른 점은 GAN loss에서 discriminator의 잘못된 수렴을 방지하기 위해 Context Encoder에서는 generator에 제약조건을 걸지 않았었는데 본 논문에서는 generator에 대한 편미분으로 해결하려는 모습을 보였다.

## Model Algorithm
![Model Algorithm](https://github.com/jwLee527/test/assets/144921672/fdb2beea-1bc1-46ac-b6ee-6c2d70af6532){: .align-center}
*Figure 6(model algorithm)*

모델의 알고리즘을 확인해보면 아래와 같다.   
- completion network를 MSE loss에 대해 $$T_C$$번만큼 학습
- completion network는 고정하고 discriminator에 대해 $T_D$번만큼 학습
- 전체 모델에 대해 $$T_{train}-(T_C+T_D)$$번만큼 학습

여기서 저자들이 설정한 값은 $$T_C=90000, T_D=10000, T_{train}=500000$$이다.

## Result
아래의 사진들에서 `ours`가 GLCIC에 해당한다.

![result](https://github.com/jwLee527/test/assets/144921672/24e967dd-ed66-4d44-a713-f56f66163efb){: .align-center}
*Figure 7(result)*

Figure 7을 보면 다른 논문의 모델들과 비교해서 GLCIC의 결과물이 더 좋은 결과를 보인다고 판단할 수 있다.   
이렇게 모델들과 비교에서는 GLCIC의 방식이 효과적임을 알 수 있지만 이전 논문들과 같이 주변의 정보들을 모아 인페이팅을 하기 때문에 한 물체 전체를 제거한 이미지에 대해서는 의도된 결과물을 얻지 못한다는 점과, 사람의 얼굴이나 동물과 같이 복잡한 이미지에 대해서는 좋지 못한 결과물을 낸다는 문제점을 여전히 갖고 있었다.(Figure8, 9에서 확인할 수 있음)   
~~GLCIC에 대해서 얼굴을 위한 fine-tuning을 하여 학습한 모델에서는 유의미한 결과를 보인다고 설명하고 있다.~~

![result2](https://github.com/jwLee527/test/assets/144921672/0f36ccd8-3ab5-4112-95eb-e0e12584ffec){: .align-center}
*Figure 8(result)*

![result3](https://github.com/jwLee527/test/assets/144921672/456933a2-1f8e-4b66-9db6-16b2f34593b4){: .align-center}
*Figure 9(result)*