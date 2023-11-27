---
layout: single # 페이지에 single 레이아웃 적용 -> 이 값은 default로 적용해서 안적어도 되긴함
title: "Context Encoder paper review"
categories: ["paper-summary"]
tags: ["minimal-mistakes", "post"]

sidebar:
    nav: "docs"
---

{% assign posts = site.docs.paper-summary %}

우선 Context Encoder는 2016년도에 발간된 image inpainting task에 관련한 논문이다.   

## Image Inpainting Task
논문의 모델을 살펴보기 전에 우선 image inpainting task에 간단히 이야기를 해보면,

![Fig. 1](https://github.com/jwLee527/test/assets/144921672/1f56f736-ee67-4ec2-ba51-3e1ea5e8b16e){: .align-center}
*Figure 1*

위의 Figure 1와 같이 빈칸이 있는 이미지에 대해 빈칸 주변의 영역과 문맥적으로 의미 있는 inpainting을 해내는 작업이라고 할 수 있다.   
이 논문이 나오기 이전에도 diffusion-based approach, patch-based approach 그리고 CNN을 이용한 방식들이 있었지만 각 방식마다 문제점들이 있었다.   

그리고 이 논문에서는 위 방식들의 문제점 중 위의 그림에서처럼 이미지에 넓은 영역의 빈칸이 있는 경우 의미 있는 inpainting을 하지 못하는 것을 해결할 수 있는 Context Encoder를 제시하였다.

## Model Architecture
Context Encoder의 구조는 auto-encoder의 encoder-decoder 구조와 GAN의 generator-discriminator 구조를 사용하였다.

![context encoder](https://github.com/jwLee527/test/assets/144921672/fb9fe630-fb38-4943-883a-c99dd697f040){: .align-center}
*Figure 2*

Figure 2와 같이 Context Encoder는 encoder-decoder로 생성된 이미지에 대해 기존의 ground truth와 비교하는 과정을 통해 모델을 학습하고 모델 파라미터를 업데이트하는 과정을 반복하는 모델이다.

### Generator

![context encoder](https://github.com/jwLee527/test/assets/144921672/c9e7d4f6-0f61-4397-9fa0-15f196b0a620){: .align-center}
*Figure 3*

Figure 3에서는 모델의 generator 부분에 대한 구조인데 그림에서는 인풋 이미지가 이미 구멍이 뚫린것으로 보이지만 실제로는 원본 이미지와 빈칸 부분은 0 아닌 부분은 1로 된 binary mask를 인풋으로 받아 요소 단위의 곱셈을 해주어 그림에서와 같은 인풋 이미지를 얻게 되고 generator의 아웃풋은 빈칸에 관한 생성 이미지이다.

#### Encoder
encoder는 기존의 AlexNet 구조에서 각 convolution layer 사이에 pooling layer를 추가한 모델을 사용하고 있다.

#### Channel-wise Fully Connected
encoder의 feature map을 decoder로 전달하기 위해 저자들은 Fully Connected layer보다 계산량이 적은 Channel=wise Fully Connected layer를 활용하였다.   
채널 단위의 계산 방식은 계산량도 적을뿐만 아니라 채널 단위로 분리된 업데이트를 통해 특정 특성에 대해 더 민감할지 둔감할지 결정할 수 있는 이점이 있다.

#### Decoder
decoder는 5개의 up-convolution layer를 활용하여 이미지를 복원하는 방식을 선택하였다.   
이 논문이 나온 당시(2016년도)에는 up-convolution이라는 용어에 대한 정의가 없어 어떤 계산 방식을 활용하였는지 자세하게 나와있지 않았는데 구글링을 좀 더 해보니 Deconvolution(Transepose Convolution)를 활용한 것으로 확인되었다.

### Discriminator

![discriminator](https://github.com/jwLee527/test/assets/144921672/11c50fb5-6c10-47fc-9007-77ab206cdafd){: .align-center}
*Figure 4*

discriminator는 generator에서 나온 결과와 원래 이미지를 인풋으로 받아 5번의 convolution layer에서 마지막 convolution layer에서 sigmoid를 통해 생성된 이미지가 real인지 fake인지 맞춰보는 방식으로 학습한다.

## Loss Function
Context Encoder의 loss function은 두개의 함수를 함께 사용한 것이 특징인데, generator에서의 loss function인 Reconstruction loss(L2 loss)와 Adversarial Loss(GAN loss)가 함께 사용되었다.

![loss function](https://github.com/jwLee527/test/assets/144921672/edf211c8-f0e4-416b-a3bc-4b39c3e0f5a2){: .align-center}
*Figure 5*

Figure 5에서 F는 generator, D는 discriminator이고, 수식을 확인해보면

### Reconstruction Loss(L2 loss)
reconstruction loss는 모델에서의 아웃풋(빈칸 생성) 이미지와 원래 이미지의 요소 단위 곱으로 나온 생성 이미지와 원래의 이미지를 비교(빼기)하여 나온 값이 작아지도록 학습하는데 L2 loss를 사용하였다.   
이러한 재구성을 위한 loss에는 L1과 L2를 주로 사용되는데 L2가 일반적으로 제곱을 하여 오차가 큰 값에 더 큰 페널티를 부여하고 오차가 적은 값에는 작은 페널티를 부여해 이상치에 민감하게 반응하기 때문에 이미지의 문맥을 이해하는데 L2 loss가 더 높은 성능을 나타내는 경우가 많다고 한다.

### Adversarial Loss
adversarial loss는 GAN loss로 generator에서 생성한 아웃풋과 원래 이미지에 대해 비교하고 D가 최대가 되는 건 같지만 F가 최소가 되는 제약조건이 없어 GAN loss의 min-max operator에서 min를 뺀 max operator이다.   
이유는 학습 초반에 generator의 성능이 낮아 discriminator가 너무 쉽게 판별하여 빠르게 수렴하는 것을 방지하기 위한 조치이다.

### Joint Loss
위의 두 loss function에 각각 하이퍼 파라미터를 곱하여 더한 방식으로 전체 모델에 대한 loss function을 정의하였고 학습시 저자들은 $\lambda_{rec}=0.999, \lambda_{adv}=0.001$로 설정하였다.

## Result

![result_1](https://github.com/jwLee527/test/assets/144921672/c658f7f9-b735-4463-8365-ce97cbefb8f7){: .align-center}
*Figure 6*

모델의 결과는 Figure 6에서 확인할 수 있는데 확실히 reconstruction loss나 adversarial loss를 단독으로 사용하였을 때보다 함께 사용하였을 때가 더 높은 성능을 갖는 걸 볼 수 있다.

![result2](https://github.com/jwLee527/test/assets/144921672/2843adc1-a69f-4910-9b20-ac2ecbb5fdfc){: .align-center}
*Figure 7*

그리고 Figure 7의 경우에는 Context Encoder가 image inpainting 이외에 다른 task에 대해서 괜찮은 성능을 보일 수 있다는 것을 나타내는 지표이다.

## 정리
논문에 대한 세밀한 정리보다는 전반적으로 어떤 모델인지 대충 정리를 해보았는데,   
결과적으로 Context Encoder는 이때까지 넓은 영역에 대한 빈칸을 그리지 못한 문제에 대해 유의미한 성능을 보였다는 것에서 의미가 있는 논문이라고 생각하고 내가 찾아보기론 image inapainting task에서 처음으로 auto-encoder와 gan를 같이 활용한 모델이라 판단되는데 지금 보면 당연해보이는 생각을 이 당시에는 어떻게 생각해냈는지 정말 대단한 것 같다.   

하지만 여전히 복잡한 텍스쳐(사람의 얼굴 또는 동물 등)에 대해서는 좋은 성능을 보이지 못하는 것과 이 모델은 이미지 전체에 대해서 학습하고 빈칸을 생성하기 때문에 local consistency가 부족하다는 단점 등을 갖고 있어 이를 해결해야 하는 챌린지가 있다는 것도 생각해봐야 할 점인것 같다.