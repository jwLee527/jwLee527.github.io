---
layout: single # 페이지에 single 레이아웃 적용 -> 이 값은 default로 적용해서 안적어도 되긴함
title: "Ubuntu에서 GPU 사용량 확인 및 메모리 공간 확보"
categories: ["others-summary"]
tags: ["minimal-mistakes", "post"]

sidebar:
    nav: "docs"
---

{% assign posts = site.docs.others-summary %}

최근 모델을 학습하는데 계속되는 오류 속에 무엇이 문제인지 확인하려고 디버깅을 하는 도중 GPU 메모리 문제에 직면했다...   
그래서 이런 경우 어떻게 해결해야 하는지 확인해보았고 이런 경우가 나중에도 자주 반복될까 싶어 미리 정리를 해두어야 겠다고 생각해 끄적거려본다.

이 포스트는 코딩오페라님의 <https://codingopera.tistory.com/34>를 보고 작성하게 되었다.

## GPU 사용량 확인

먼저 우분투가 아니더라도 nvidia-driver가 다운받아진 기기라면 `nvidia-smi`를 통해 본인의 GPU의 사용량 등을 확인할 수 있다.

```
nvidia-smi
```

![nvidia-smi](https://github.com/jwLee527/test/assets/144921672/7de2cf4c-39cc-4c59-b33d-8767bd38b22a){: .align-center}

명령어를 입력하면 위와 같은 찾이 나오고, Memory-Usage 부분의 아래 박스를 보면 현재 GPU의 메모리 용량과 사용량을 볼 수 있다.

## GPU 메모리 삭제

위에서 메모리 사용량이 용량에 거의 근접하게 되면 추가적인 학습을 하기 어렵기 때문에 이전의 활동 중에 불필요하게 메모리를 잡아먹고 있는 녀석을 삭제해줘야한다.   
이때 어떤 녀석을 지워야하는지 확인하는 코드가 `ps aux | grep python`인데, 마지막에 python를 작성한 이유는 본인은 python을 통해 학습하여 메모리를 가장 많이 먹고 있는게 python이기 때문이다.

```
ps aux | grep python
```

![ps](https://github.com/jwLee527/test/assets/144921672/fa556a2d-cef7-4a94-8acd-d3cf40325300){: .align-center}

명령어를 입력하면 위와 같이 나오고 이 중 불필요한 활동으로 파악되는 프로그램(여기서는 하늘색 강조 줄)을 찾고 해당하는 번호(가장 처음 보이는 숫자 4개)를 삭제해준다.

```
kill -9 7494
```

본인은 도커에서 수행하고 있어 sudo를 입력하지 않았지만 권한이 필요할 시에 명령어 가장 앞에 sudo를 입력하면 된다.

이후 다시 `nvidia-smi`로 GPU의 상황을 확인해보면 메모리에 여유 공간이 확보된 것을 볼 수 있다.

![nvidia-smi2](https://github.com/jwLee527/test/assets/144921672/1cd528f4-7f58-42ee-9f9e-c7feee3b9a67){: .align-center}