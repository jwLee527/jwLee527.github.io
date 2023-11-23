---
layout: single # 페이지에 single 레이아웃 적용 -> 이 값은 default로 적용해서 안적어도 되긴함
title: "Dilated Convolution"
categories: ["concept-summary"]
tags: ["minimal-mistakes", "post"]

sidebar:
    nav: "docs"
---

{% assign posts = site.docs.concept-summary %}

Dilated Convolution은 확장 Convolution으로 불린다.

![dilated convolution](https://github.com/jwLee527/test/assets/144921672/be4559b6-a3c7-4c5d-9f33-f8c85bb27203){: .align-center}
*dilated convolution*

위와 같이 일반적인 convolution보다 더 넓게 퍼져서 계산되는 것을 볼 수 있을 텐데

## formula   
이를 식으로 표현하면 아래와 같고,

<center>$$y_\left(a,b\right) = \alpha\left(b+\sum_{i=-k'_h}^{k'_h} \sum_{j=-k'_w}^{k'_w} W_\left({k'_h}+i,{k'_w}+j\right)x_\left({u+\eta i},{v+\eta j}\right)\right)$$</center>   
<center>$$k'_h = \frac{k_h-1}{2}, k'_w = \frac{k_w-1}{2}$$</center>

이 식에서 $\alpha$는 activation function이고 b는 bias이며 $\eta$는 dilation factor이다.

위 식에서 알 수 있듯이 dilation factor가 1이면 일반적인 convolution이고 1보다 크면 dilated convolution이 되는 것이다.

## feature   
수식으로도 알 수 있듯이 모든 조건이 동일한 경우 dilated convolution은 convolution보다 넓은 지역에 대한 계산이 가능해진다는 특징이 있다.   

이러한 특징은 image inpainting task의 경우에 아래와 같이 그려야 하는 빈칸이 넓은 경우 일반적인 convolution 계산으로는 인풋에서 정보 압축해도 주변 정보를 얻지 못하기 때문에 dilated convolution을 이용하여 정보를 얻는 이득을 얻을 수 있다.

![deliated convolution 특징](https://github.com/jwLee527/test/assets/144921672/e6af2ec2-a0e7-450a-8579-f0a02f112fca)