---
layout: single # 페이지에 single 레이아웃 적용 -> 이 값은 default로 적용해서 안적어도 되긴함
title: "Robust_Speech_Recognition_via_Large-Scale_Weak_Supervision(Whisper)"
categories: ["paper-summary"]
tags: ["minimal-mistakes", "post"]

sidebar:
    nav: "docs"
---

{% assign posts = site.docs.paper-summary %}

Robust_Speech_Recognition_via_Large-Scale_Weak_Supervision(Whisper) 논문은 OpenAI에서 작성한 ICML'23에 출판된 논문으로 STT(Speech-To-Text)관련 모델 Whisper를 제시한다.

## Introduction

이전의 음성 인식에서는 Word2Vec 2.0과 같은 unsupervised 사전 학습 기술들에 의해 발전되어왔다. unsupervised 학습 방식은 레이블이 없는 학습 데이터로 학습을 진행하기 때문에 대규모 음성 데이터셋을 생산해 학습할 수 있고 supervised 데이터셋의 1,000시간보다 훨씬 많은 100만 시간의 학습 데이터를 통해 확장하였다. 그 결과 표준 벤치마크에서 fine-tuning될 때 unsupervised 접근 방식이 low-data 세팅에서 state-of-the-art를 개선했다.

하지만 이러한 학습 방식은 음성의 표현을 학습하는 고성능의 인코더를 얻을 수 있지만 레이블이 없기 때문에 그러한 표현을 출력으로 내보낼 수 있는 고성능의 디코더를 얻기 힘들어 fine-tuning 단계가 필요하다. fine-tuning은 아직까지도 실무자의 역량에 따라 유용성과 견고성 영향을 준다. 저자들은 이런 약점을 극복하고 광범위한 환경에서 안정적으로 작동하는 것을 목표로한다.

그래서 적용한 방식이 weakly-supervised 학습 방식이다. GigaSpeech와 The People's Speech는 정교한 자동화 파이프라인을 사용하여 weakly-supervised 음성 인식을 1만 시간과 3만 시간의 noisy한 학습 데이터로 확장한다. 그리고 음성 인식 분야에서 weakly-supervised 학습 방식이 유용성과 견고성을 보장해준다는 것에 충분한 연구가 되진 않았지만, 컴퓨터 비전의 최근 연구들에서는 입증이 되었기 때문에 이런 학습 방식을 적용해보았다.

weakly-superivsed 방식은 supervised 방식보다 데이터셋이 훨씬 많지만 unsupervised 방식보다는 훨씬 적다. 하지만 저자들은 확보한 데이터셋의 전처리를 통해 68만 시간의 레이블이 지정된 오디오 데이터를 확보하여 unsupervised 방식의 데이터셋과 격차를 줄였고 이런 접근 방식을 Whisper라고 부른다고 했다.

이외에도 데이터셋에는 음성 인식을 넘어 다국어 및 멀티태스킹으로 확장할 수 있도록 되어있다. 68만 시간의 오디오 중 11.7만 시간은 96개의 다른 언어를 포함하고, 12.5만 시간은 X->en 번역 데이터도 포함되어 있다. 그러면서 저자들은 충분히 큰 모델의 경우에는 다국어 및 멀티태스킹 학습에 단점이 없고 오히려 이점도 있음을 발견하였다고 한다.

## Approach

저자들이 데이터셋을 확보할때 우선 인터넷에서 오디오-텍스트 쌍을 이루는 데이터를 모았다. 그 이후 텍스트의 품질을 개선하기 위해 몇 가지 자동화된 필터링 방법을 개발하여 개선하였다. 이유는 인터넷에 있는 많은 오디오-텍스트 데이터 셋은 인간이 직접 생성한 것이 아니라 기존의 ASR 시스템의 결과물이고 최근 연구에 따르면 사람과 기계가 생성한 데이터로 혼합한 데이터 셋을 만들어 학습하면 성능을 크게 저하시킬 수 있다고 하였기 때문이다.

모델에 대한 설명에서 본 논문의 초점은 음성 인식을 위한 대규모 supervised 사전 학습 능력을 연구하는 것이므로 모델 개선과 저자들의 발견을 혼동하지 않도록 기존의 아키텍처를 사용했다. 저자들은 안정적으로 모델을 확장할 수 있는 인코더-디코더 트렌스포머를 사용했다. 인풋으로 모든 오디오는 16kHz로 다시 샘플링되고 80채널 log Mel Spectrogram 표현은 strde가 10ms인 25ms window에서 계산된다. Feature 정규화를 위해 사전 학습 데이터셋에서 평균이 대햑 0인 -1과 1 사이가 되도록 인풋을 글로벌하게 조정한다. 인코더는 필터 크기가 3인 2개의 convolution layer와 GELU로 구성된 작은 stem으로 이 인풋 표현을 처리한다. 여기서 두 번째 convolution layer의 stride는 2이다. 그런 다음 sinusoidal position embedding이 stem의 출력에 추가된 후 인코더 트렌스포머 block이 적용된다. 트렌스포머는 pre-activation residual block을 사용하고 최종 layer normalization이 인코더 출력에 적용된다. 디코더는 학습된 position embedding과 입출력 토큰 표현을 사용한다. 인코더와 디코더는 트렌스포머 block의 크기와 개수가 동일하다. 아래 그림은 모델의 아키텍처를 표현한 것이다.

![model architecture](https://github.com/jwLee527/test/assets/144921672/7135b0ac-3d6a-4d10-8d55-73e93b3eb437){: .align-center}
*Figure 1*

GPT2에서 사용된 것과 동일한 바이트 레벨의 BPE text tokenizer를 영어 전용 모델에 사용한다. GPT2 BPE vocabulary가 영어 전용이므로 다른 언어에 대한 단편화를 피하기 위해 다국어 모델에 대한 vocabulary를 다시 맞춘다.

이 모델이 멀티 테스크를 수행하기 위해서 모든 작업과 컨디셔닝 정보를 디코더에 대한 일련의 입력 토큰으로 지정했다. <|startoftranscript|> 토큰으로 예측의 시작을 나타내고, 학습 셋의 각 언어(총 99개)에 대한 고유한 토큰으로 표현되는 말하는 언어를 예측한다. 오디어 세그먼트에 음성이 없는 경우는 <|nospeech|> 토큰을 예측하도록 모델을 학습한다. 다음 토큰은 <|transcribe|> 또는 <|translate|> 토큰을 사용하여 테스크(전사 또는 번역)를 지정한다. 이후 각 케이스에 대한 <|notimestamps|> 토큰을 포함하여 타임스탬프를 예측할지 여부를 지정한다. 이 시점에서 테스크와 원하는 형식이 완전히 지정되고 출력이 시작된다.

타임스탬프 예측을 위해 현재 오디오 세그먼트와 관련된 시간을 예측하고, Whisper 모델의 기본 시간 해상도와 일치하는 가장 가까운 20ms로 모든 시간을 양자화하고, 이들 각각에 대한 vocabulary에 추가 토큰을 추가한다. 저자들은 예측을 캡션 토큰과 인터리브하였다. 시작 시간 토큰은 각 캡션의 텍스트 전에 에측되고 종료 시간 토큰은 그 후에 예측된다. 최종 전사 세그먼트가 현재 30초 오디오 청크에 부분적으로만 포함된 경우 타임스탬프 모드에 있을 때 세그먼트의 시작 시간 토큰만 예측하여 해당 시간과 일치하는 오디오 window에서 후속 디코딩을 수행해야 함을 나타낸다. 그렇지 않으면 오디오를 자른다. 마지막으로 <|endoftranscript|> 토큰을 추가한다. 이전 컨텍스트 텍스트에 대한 학습 로스만 가리고 다른 모든 토큰을 예측하도록 모델을 학습시킨다. 형식 및 학습 설정에 대한 개요는 위 그림에 요약되어있다.

추가로 저자들은 Whisper의 스케일링 속성을 연구하기 위해 다양한 크기의 모델을 학습시켰다.

![model scaling](https://github.com/jwLee527/test/assets/144921672/ff107070-0772-4bd5-9507-93d4fbae298b){: .align-center}
*Figure 2*

## Experiments

1. Zero-shot Evaluation   
Whisper의 목표는 데이터 셋별 fine-tuning 없이 안정적으로 작동하는 강력한 단일 음성 처리 시스템을 개발하는 것이다. 이 능력을 연구하기 위해 다양한 기존 음성 처리 데이터 셋을 재사용하여 Whisper가 도메인, 테스크, 언어 전반에 걸쳐 잘 일반화할 수 있는지 확인한다. 각 데이터 셋에 대한 학습 데이터를 사용하지 않고 zero-shot 세팅에서 Whisper를 평가하여 광범위한 일반화를 측정한다.

2. Evaluation Metrics   
음성 인식 연구는 일반적으로 WER(단어 오차율)을 기반으로 시스템을 평가하고 비교한다. 그러나 문자열 편집 거리를 기반으로 하는 WER은 전사 스타일의 무해한 차이를 포함하여 모델의 출력과 레퍼런스 사이의 모든 차이에 페널티를 준다. 결과적으로 시스템이 출력한 전사를 모두 인간이 올바르다고 판단하더라도 사소한 형식 차이로 인해 여전히 큰 WER을 가질 수 있다. 이것은 특정 데이터 셋의 전사 형식의 예를 관찰하지 않은 Whisper와 같은 zero-shot 모델의 경우 특히 심각하다.

이러한 관점은 새로운 것이 아니고 인간의 판단과 더 잘 연관되는 평가 지표의 개발은 활발한 연구 분야이지만 아직 음성 인식에 널리 채택된 것이 없다. 그래서 저자들은 non-semantic 차이에 대한 페널티를 최소화하기 위해 WER 계산 전에 텍스트를 광법위하게 표준화하여 이 문제를 해결하기로 했다. 본 논문에 사용된 text normalizer는 naive한 WER이 무해한 차이에 대해 Whisper 모델에 페널티를 부여한 일반적인 패턴을 식별하기 위해 반복적인 수동 검사를 통해 개발되었다. 여러 데이터셋의 경우 WEr이 최대 50%까지 감소하는 것을 관할했다. 일반적으로 데이터 셋의 레퍼런스 전사가 공백이 있는 단어와 약어를 구분하는 등의 특이점으로 인해 발생한다.이 개발 절차가 Whisper 모델의 전사 스타일에 overfitting될 위험이 있음을 주의해야 한다.

3. English Speech Recognition
2015년 Deep Speech 2는 LibriSpeech test-clean split을 전사할 때 인간 수준의 성능과 일치하는 음성 인식 시스템을 보고했다. 하지만 7년 후 LibriSpeech test-clean의 SOTA WER은 기존 5.3%에서 1.4%로 73% 성능이 더 향상 되었다. 하지만 기존의 세팅이 아닌 다른 세팅에서 사용될 때는 사람의 오차율보다 높다. 즉, in-distribution에서는 좋은 성능을 보이지만, out-of-distribution에서는 인간보다 좋지 못한 성능을 보인다.

저자들은 이러한 격차의 상당 부분이 테스트 셋에서 인간과 기계 성능으로 측정되는 서로 다른 능력을 통합하기 때문이라고 한다. 즉, 인간과 기계가 같은 시험을 치르고 있지만 서로 다른 능력으로 시험된다는 것이다. 인간은 연구 중인 특정 데이터 분포에 대한 supervision이 거의 또는 전혀 없는 테스크를 수행하도록 요청받아서 인간의 성능은 out-of-distribution 일반화의 척도이다. 하지만 기계 학습 모델은 일반적으로 평가 분포에서 많은 양의 supervision에 대한 학습 후에 평가되므로, 기계 성능은 in-distribution 일반화의 척도가 된다.

그렇지만 광범위하고 다양한 오디오 분포에 대해 학습되고 zero-shot 세팅에서 평가되는 Whisper 모델은 잠재적으로 기존 시스템보다 훨씬 더 인간의 행동과 일치할 수 있다. 이를 증명하기 위해 Whisper 모델을 인간의 성능과 표준 fine-tuning 모델과 비교하여 어느 것이 더 밀접하게 일치하는지 확인할 수 있다.

차이를 정량화하기 위해 많은 분포/데이터 셋의 평균 성능인 overall robustness과 effective robustness을 모두 조사한다. 일반적으로 in-distribution인 레퍼런스 데이터 셋과 하나 이상의 out-of-distribution 데이터 셋 간의 예상 성능 차이를 측정한다. Effective robustness가 높은 모델은 레퍼런스 데이터 셋에 대한 성능의 함수로 out-of-distribution 데이터 셋에서 예상보다 더 잘 수행되며 모든 데이터 셋에서 동일한 성능에 접근한다. LibriSpeech는 최신 음성 인식 연구의 중심 역활과 많은 릴리스 모델의 가용성으로 인해 레퍼런스 데이터 셋으로 사용한다. 저자들은 12개의 다른 학술 음성 인식 데이터 셋을 사용하여 out-of-distribution 행동을 연구하였다.

다음은 LibriSpeech에서의 WER과 3가지 out-of-distribution 데이터 셋(Common Voice, CHiME-6, TED-LIUM)의 평균 WER을 plot한 그래프이다.

![plot](https://github.com/jwLee527/test/assets/144921672/d534cb95-3ce6-4616-bf2f-2e1a62b661e5){: .align-center}
*Figure 3*

다음은 다양한 데이터 셋에 대한 effective robustness를 비교한 표이다.

![graph](https://github.com/jwLee527/test/assets/144921672/9dbd6b2f-3298-4189-b115-0b6fd1fbddd5){: .align-center}
*Figure 4*

위 그래프와 표에서 볼 수 있듯이 zero-shot Whisper 모델은 supervised LibriSpeech model과 매우 다른 robustness 속성을 가지며 다른 데이터 셋에서 모든 벤치마크된 LibriSpeech model와 큰 차이가 난다.

이 발견은 모델의 zero-shot과 out-of-distribution 평가를 강조할 것을 제안하고 특히 인간 성능과 비교하려고 시도할 때 오해의 소지가 있는 비교로 인해 기계 학습 시스템의 능력을 과장하는 것을 방지할 수 있다.

4. Multi-lingual Speech Recognition   

![graph](https://github.com/jwLee527/test/assets/144921672/ee2e0131-d972-4684-8099-3444c7f8811f){: .align-center}
*Figure 5*

위의 표는 Multilingual LibriSpeech(MLS)와 VoxPopuli의 다국어 음성 인식 성능을 비교한 표이다. 두 벤치마크는 15개의 고유한 언어만 포함하기 때문에 언어의 폭이 다소 좁다고 판단된다. 그에 비해 Whisper는 75개국의 언어에 대해 음성 인식을 위한 데이어를 학습하였다.

저자들은 Whisper의 성능을 보다 광범위하게 연구하기 위해 Fleurs 데이터 셋에 대한 성능도 측정하였다. 특히 저자들은 주어진 언어에 대해 가지고 있는 학습 데이터의 양과 해당 언어에 대한 downstream zero-shot 성능 사이의 관계를 연구하는 데 관심이 있었다. 아래 그래프는 이 관계를 시각화한다.

![plot](https://github.com/jwLee527/test/assets/144921672/1023af70-c850-4cc2-a519-8063d0006b7f){: .align-center}
*Figure 6*

WER과 언어당 학습 데이터 양 사이에 0.83의 강한 제곱 상관 계수가 있음을 볼 수 있고 이런 선형 맞춤에 대한 회귀 계수를 확인하면 학습 데이터가 16배 증가할 때마다 WER이 반으로 줄어드는 추정을 할 수 있다. 하지만 이 추세에 따르지 못하고 성능이 낮은 언어 중 많은 수가 고유한 스크립트를 가지고 있고 인도-유럽 언어와 관련이 먼 언어이다.(히브리어(HE), 텔루구어(TE), 중국어(ZH), 한국어(KO)). 이러한 성능 차이는 언어적 거리로 인한 전송 부족, 바이트 레벨 BPE tokenizer가 이러한 언어와 일치하지 않거나 데이터 품질의 차이로 인해 발생할 수 있다.

5. Translation   

![graph](https://github.com/jwLee527/test/assets/144921672/feda4fd1-a48e-4917-9b81-0d185036075f){: .align-center}
*Figure 7*

다음은 타 언어에서 영어로 음성 번역 성능을 나타낸 표이고,

![plot](https://github.com/jwLee527/test/assets/144921672/d39f090b-373b-48d6-91a3-ce4abc380e2d){: .align-center}
*Figure 8*

다음은 언어당 번역 학습 데이터 양과 Fleurs에서의 zero-shot 번역 성능의 상관 관계를 시각화한 그래프이다.

학습 데이터가 증가함에 따라 명확한 개선 추세가 있지만 제곱 상관 계수는 음성 인식에서 관찰된 0.83보다 훨씬 낮고 상관 계수를 잘 따르지 않는다.

6. Language Identification

![graph](https://github.com/jwLee527/test/assets/144921672/702d9ba1-5938-4915-924d-7a0c635b9cb1){: .align-center}
*Figure 9*

다음은 Fleurs에서의 언어 식별 성능을 나타낸 표이다. Whisper의 zero-shot 성능은 다른 supervised SOTA보다 떨어진다. 이는 Fleurs가 다루는 102개의 언어 중 20개가 Whisper 데이터 셋에 없기 때문이다. 겹치는 82개의 언어에 대해서한 평가하면 성능이 80.3%까지 나온다고 한다.

7. Robustness to Additive Noise

![plot](https://github.com/jwLee527/test/assets/144921672/5b314e3c-8ffe-4ed3-8ac5-aa8a23317aa4){: .align-center}
*Figure 10*

위 그래프는 additive noise가 더욱 집중됨에 따라 ASR 성능이 저하되는 것을 보여준다. 낮은 noise(40dB SNR)에서 zero-shot 성능을 능가하는 모델이 많이 있다. 이러한 모델은 주로 LibriSpeech에서 학습된다. 그러나 모든 모델은 noise가 더 강해짐에 따라 빠르게 성능이 저하되어 10dB미만의 SNR의 additive pub noise에서 Whisper모델보다 성능이 떨어진다. 이는 특히 pub noise와 같은 보다 자연스러운 분포 변화에서 noise에 대한 Whisper의 견고성을 보여준다.

8. Long-form Transcription

![plot](https://github.com/jwLee527/test/assets/144921672/0b340df8-8ed2-49e3-9f9f-755d3d21fff3){: .align-center}
*Figure 11*

다음 그래표는 긴 형태의 전사를 가지는 7개의 데이터 셋에 대해 Whisper가 다른 SOTA ASR 시스템들과 경쟁력이 있음을 보인다.

9. Comparison with Human Performance

![plot](https://github.com/jwLee527/test/assets/144921672/ab4f91b2-127d-4180-989b-1c6f04b9eb30){: .align-center}
*Figure 12*

다음은 Kincaid46 데이터 셋에서의 전사 능력을 전문가와 함께 비교한 그래프이다.

## Analysis and Ablations

1. Model Scaling

![plots](https://github.com/jwLee527/test/assets/144921672/29108f2c-90e6-4e97-adf0-51986f155c8f){: .align-center}
*Figure 13*

다음은 다양한 task에서 모델 크기에 따른 WER을 측정한 그래프이다.

2. Dataset Scaling

![graph](https://github.com/jwLee527/test/assets/144921672/c3cc86da-0246-4a7d-9849-de23827fcf56){: .align-center}
*Figure 14*

다음은 데이터 셋 크기에 따른 성능을 나타낸 표이다.

3. Multitask and Multilingual Transfer

![plot](https://github.com/jwLee527/test/assets/144921672/af0b8924-a1fd-404e-9c85-a978c5544ad3){: .align-center}
*Figure 15*

다음은 영어 전용 모델과 다국어 멀티테스킹 모델을 학습량에 따라 평균 WER로 비교한 그래프이다. 그래프에서 적당한 양의 계산으로 학습된 작은 모델의 경우 실제로 테스크와 언어 간의 부정적인 번역이 있음을 보여주지만 다국어 멀티테스킹 모델은 잘 확장되며 가장 큰 실험의 경우 다른 테스크에서 긍정적인 번역을 보여주는 영어 전용 모델보다 성능이 뛰어남을 보인다.

4. Text Normalization

무해한 단어 오차를 줄이기 위해 Whisper와 공동으로 텍스트 정규화를 개발했기 때문에 정규화가 전사의 일반적인 변형을 해결하기보다 Whisper의 특성을 수정하는 데 오버피팅될 위험이 있다. 이를 체크하기 위해 저자들은 본 논문의 normalizer를 이용한 Whisper의 성능과 FairSpeech 프로젝트에서 독립적으로 개발한 normalizer를 비교했다.

![plot](https://github.com/jwLee527/test/assets/144921672/698a4dbe-0765-4ccd-9e26-c633c3ddcc70){: .align-center}
*Figure 16*

위 그래프는 차이점을 시각화한 것이다. 대부분의 데이터 셋에서 두 개의 normalizer는 Whisper 모델과 비교된 오픈 소스 모델 간의 WER 감소에 큰 차이 없이 유사하게 수행되지만, 일부 데이터 셋(WSJ, CallHome, Switchboard)에서는 본 논문의 normalizer가 Whisper 모델의 WER을 훨씬 많이 줄인다. 감소의 차이는 ground truth에서 사용하는 다양한 형식과 두 normalizer가 어떻게 패널티를 부과하는지 추적할 수 있다.

5. Strategies for Reliable Long-form Transcription

저자들은 긴 오디오에 대한 전사의 실패를 피하는 데 도움이 되는 다양한 휴리스틱을 개발하였다. 먼저 greedy decoding에서 더 자주 발생하는 반복 looping을 줄이기 위해 로그 확률을 score function으로 사용하여 5개의 beam으로 beam search를 사용하고 Temperature는 0부터 시작한다. 이는 항상 확률이 높은 토큰을 선택하고 생성된 토큰에 대한 평균 로그 확률이 -1보다 낮거나 생성된 텍스트의 gzip 압축률이 2.4보다 높을 때 temperature를 0.2씩 1.0까지 높인다. 적용된 temperature가 0.5미만일 때 이전 window에서 전사된 텍스트를 이전 텍스트 조건으로 제공하면 성능이 더욱 향상된다. 저자들은 <|nospeech|> 토큰의 확률만으로는 음성이 없는 세그먼트를 구별하기에 충분하지 않고 음성 없음 확률 임계값 0.6과 평균 로그 확률 임계값 -1을 결합하면 음성 활동 감지를 더 믿을 수 있게 만든다는 것을 발견했다. 마지막으로 모델이 입력의 처음 몇 단어를 무시하는 failure mode를 피하기 위해 초기 타임스탬프 토큰을 0.0~1.0초 사이로 제한했다.

![graph](https://github.com/jwLee527/test/assets/144921672/2f8522f0-b767-408f-9877-defd8b3bf9c7){: .align-center}
*Figure 17*

위 표는 위의 각 개입을 추가하면 전반적으로 WER이 점진적으로 감소하지만 데이터 셋 전체에 걸쳐 균등하지는 않음을 보여준다.