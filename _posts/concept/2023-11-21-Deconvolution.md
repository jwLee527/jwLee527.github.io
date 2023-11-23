---
layout: single # 페이지에 single 레이아웃 적용 -> 이 값은 default로 적용해서 안적어도 되긴함
title: "Deconvlution(Transpose Convolution)"
categories: ["concept-summary"]
tags: ["minimal-mistakes", "post"]

sidebar:
    nav: "docs"
---

{% assign posts = site.docs.concept-summary %}

<span style="font-size:60%">우선 이 post는 image inpainting task에 대해 공부하다 적게된 post로 image inpainting에 대한 관점으로 작성되었다는 것을 알립니다.</span>
--------

## background

CNN에서 convolution layer는 일반적으로 인풋에 대한 feature map의 크기를 줄이기 위해 사용된다.   
이와 반대로 auto-encoder와 같은 모델에 decoder부분을 보면 convolution layer를 통해 인풋에 대한 feature map의 크기를 키우기도 하는데, 이때 사용되는 계산 방식이 deconvolution이다.
-deconvolution 방식은 pooling layer에서 사용되는 방식도 있지만 이 post에서는 convolution layer에서 사용되는 transpose convolution에 대해서만 다룰 예정이다.-   

Deconvolution은 처음 Image Segmentation task를 위해 탄생한 계산 방식으로 알고있다.   
하지만 이후에 image inpainting task에서도 image에 대한 feature를 압축했다가 다시 재구성하는 auto-encoder 모델을 사용함에 따라 중요한 계산 방식으로 자리잡았다.

![auto-encoder](https://github.com/jwLee527/test/assets/144921672/ed77f4dc-4cfb-49ea-afcf-570e2be61e4a){: .align-center}
*auto-encoder*

## deconvolution

![deconvolution](https://github.com/jwLee527/test/assets/144921672/f083d6d9-9bed-45dd-927a-5b414724d9de){: .align-center}
*deconvolution*

deconvolution은 이름에서도 알 수 있듯이 convolution의 역을 구하는 방식이다.

그런데 다른 블로그의 deconvolution에 대한 포스트를 보아도 바로바로 이해가 잘 되지 않아 직접 그려보면서 해봤는데,

![deconvolution_1](https://github.com/jwLee527/test/assets/144921672/1986ce41-909f-49ae-b5b5-cf651d181b90){: .align-center}
*convolution 계산식*

위의 경우는 조건과 그림에서 보이듯이 간단한 convolution 계산을 표현한 식이다.   
이를 곱셈으로 표현하기 위해 식을 좀 변경했는데 곱셈으로 표현된 식은 아래와 같다.

![deconvolution_6](https://github.com/jwLee527/test/assets/144921672/038ef8ee-bdcb-46ce-bc27-45526ad3de07){: .align-center}
*convolution 변형식*

식을 보면 각 y에 만족하는 방식으로 kernel이 변환된 것을 볼 수 있다.   
그리고 deconvolution은 convolution의 역이라고 하였고 deconvolution 계산 중에 transpose convolution이라고 한 것에서 두가지 힌트를 얻을 수 있는데,   
첫 번째는 convolution의 역이기 때문에 convolution에서 인풋과 아웃풋이 deconvolution에서는 아웃풋과 인풋으로 뒤바뀐다는 사실이고   
두 번째는 transpose convolution인 것에서 식에서 어떤 한 행렬을 transpose할 것이라는 것이였는데   

그럼 인풋과 아웃풋을 뒤바꾼다는건 쉽게 이해가 되지만 식에서 어떤 한 행렬을 transpose하는 건지 모를 수 있지만 사실 식에서 transpose 할만한 것이 변형된 kernel밖에 없는 것 같고 실제로도 변형된 kernel을 transpose하는게 맞았다.

그래서 위의 convolution식을 deconvolution로 변경하면 다음과 같다.

![deconvolution_4](https://github.com/jwLee527/test/assets/144921672/810900df-ecb2-4a72-9867-1fce475bf726){: .align-center}
*transpose kernel*

그런데 여기서 잘 이해가 되지 않았던 부분이 나오는데, 위 사진에서의 변형 kernel에 대한 행렬을 보면 convolution에서 곱셈을 하기 위해 변형된 kernel에서 단지 transpose만 한 것이 아닌 것을 확인 할 수 있다.   
하지만 이를 다시 deconvolution에 대한 식으로 변형하면 

![deconvolution_5](https://github.com/jwLee527/test/assets/144921672/da5f68af-cf98-4eac-8adf-485f7fca48f5){: .align-center}
*deconvolution*

이렇게 kernel이 다시 원래의 모습을 잘 찾았다....   
사실 이런걸 직접적으로 이해하기에는 수식을 찾아보는게 직빵인데 개념에 대해서만 공부하느라 시간이 없어 수식을 못찾아봤는데 시간이 난다면 수식에 대해 공부해보고 포스팅을 채워봐야겠다.