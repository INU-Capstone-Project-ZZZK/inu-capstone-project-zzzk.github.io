---
layout: default
title: 5월 4주차
parent: Weekly_Report
nav_order: 1
---

# 5월 4주차 Weekly Report
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{: toc}

---

## 주간 작업 내용


{: .note }
5월 4주차 작업내용이다. 팀원별 업무 내용은 다음과 같다.  
공동작업: 흉골 디바이스 납땜 및 작동 테스트   
박종현 : 가속도 센서 특징 추출 연구    
이제욱 : 흉골 디바이스 3D 모델링, 하드웨어 배치 설계    
이태우 : 가속도 기반 수면단계 모델 학습 및 샘플링 변화    




 
- 05.21(화) 16:30~21:15 : 납땜 계획, 하드웨어 배치 설계, 주간 회의 

- 05.22(수) 11:00 ~22:10 : 흉골 디바이스 납땜

- 05.23(목) 15:00 ~ 22:50 : 발표 ppt 제작 및 준비, 흉골 디바이스 납땜
  
- 05.24(금) 11:00 ~ 20:05 : 흉골 디바이스 최종 납땜 , 발표 

이외 개인 작업으로 자택에서 진행

---

## 흉골 디바이스 납땜 및 작동 테스트

 진동 테스트 및 진동 강화 알고리즘까지 테스트를 마친 뒤에 직접 납땜을 하여 디바이스를 제작하였다. 

소형 모듈들을 매우 밀착시켜 납땜하였는데, 납땜 숙력도가 높지 않아 실수가 좀 많이 있었고, 합선이 일어나기도 하여 납을 제거하는 과정, 부피 축소를 위해 최소한의 선 길이 배치에 집중하는데 3일이 꼬박 걸렸다.

### 디바이스 작동 테스트 - 유선 상태 자세 탐지 여부(진동모터X)

- 시작 - 정자세

![](/assets/images/W9/1.png)

- 정자세 → 우측자세

![](/assets/images/W9/2.png)

- 우측자세 → 좌측자세

![](/assets/images/W9/3.png)

- 다시 정자세로 돌아온 경우

![](/assets/images/W9/4.png)

### 디바이스 작동 테스트 - 무선 상태 자세 탐지 여부(진동모터)O

- 정자세
    
   ![](/assets/images/W9/5.png)
    
- 좌측자세
    
    ![](/assets/images/W9/6.png)
- 우측자세
    
    ![](/assets/images/W9/7.png)

- **최종 동작 영상 및 진동자극 발생 여부(카메라를 가까이 대니 진동모터 소리가 조금 들려서 링크로 영상을 첨부하였습니다.)**
    
    https://drive.google.com/file/d/1FP3XRanrr28wMWh1ZEKpfjoPw7WeMkg5/view?usp=sharing
    

<aside>
💡 최종적으로 자세탐지 및 진동자극까지 기능 확인을 완료하였다. 다만 흉골 디바이스 케이스 사이즈에 오차가 조금 있어 재주문을 넣은 상태. 케이스에 넣어서 완전히 테스트는 못했지만, 돌아오는 주 월~화에 케이스 수령후 넣기만 하면 운용이 바로 가능함.   

</aside>




<aside>
💡 사용자가 깨지 않을 정도의 적절한 진동을 찾는 실험이 필요

</aside>

## 손목 디바이스 가속도 기반 수면단계 판별

가속도 데이터 값을 이용하여 수면 단계 분류 딥러닝 학습을 진행하였다. 사용된 데이터셋은 physionet의 데이터셋의 가속도 데이터와 수면단계 라벨링 값을 이용하였다.  

데이터 전처리 값에는 bandpass filter를 이용하여 noise값을 보정해주었다. 

```python
def bandpassFilter(x, fs=64, lowcut=2.0, highcut=3.5, order=4):
    nyq = 0.5 * fs
    low = lowcut / nyq
    high = highcut / nyq

    b, a = butter(order, [low, high], btype='band')
    filtered_x = lfilter(b, a, x)

    return filtered_x
```

![](/assets/images/W9/8.png)

위의 그림은 하나의 데이터에 대하여 bandpass filter를 통과한 그래프이다. 노이즈가 제거된 모습을 확인 할 수 있다. 

그 후 sampling을 저번 주 박종현 학생이 조사한 sampling을 가지고 실험을 진행하였다. 

이 때 sampling을 하는 이유는 다음과 같다. 데이터셋은 64hz로 가속도 센서가 64hz로 flutter와 pico간의 bluetooth 통신을 진행 할 때 overhead가 매우 커 버려지는 값이 생길 수 있기 때문이다. 

이와 같은 이유로 sampling을 진행하였을 때 accuracy가 얼마나 줄어드는지 확인 해 볼 필요가 있었다.


### 실험결과

샘플링에 대한 실험결과는 다음과 같았다.  FCN 모델보단 LSTM모델에서의 성능이 더 좋게 나왔다. 이는 LSTM모델이 시계열 데이터 값인 가속도 센서의 값에 더 적합했기 때문이다. 원본의 성능에서 75% 값이 나왔는데 이는 원본 데이터값이 제일 특징이 잘 드러났기 때문이다. 

sampling값에서는 interval max sampling 값이 제일 성능이 좋았지만 58.4%로 이를 그대로 이용하기에는 어려움이 있었다. 

앞서 박종현 학생이 분석한 논문에 따르면 가속도 센서를 이용한 수면 단계 분류의 정확도는 80% 대로 정확도를 더 끌어올릴 수 있어 보인다. 

![](/assets/images/W9/9.png)



## 가속도 특징 추출 연구

지난 주, 랜덤포레스트 모델과 3가지의 샘플링 방법을 적용하여 수면단계 분류 성능을 비교해보았다. 하지만 정확도가 지나치게 낮았었고, 이에 단순 가속도 데이터만을 입력으로 삼을 경우 정확도 보장이 되지 않는다고 판단하였다. 따라서 이번에는 가속도 데이터의 전처리 방법에 변화를 주고, 별도 특징을 추출하여 모델 분류 성능을 향상시키고자 공부하였던 부분을 기록하였다.



### 전처리

기본적으로 가속도 X,Y,Z 축 데이터를 모두 이용하지 않고 가속도 벡터로 변환하여 사용하는 것은 같다. 손목의 세세한 방향이 중요한게 아니라 얼마나 운동하고 있는지 운동강도가 중요하기 때문에 해당 값에 대한 처리가 이후 과정의 주요 타겟이 된다.
(뿐만 아니라 3축 데이터 모두 이용할 경우 3차원의 데이터가 필요하지만, 이처럼 변환하면 1차원 데이터로 차원 축소가 가능하다는 장점도 존재함.)

- **잡음 제거**

 사용자의 움직임에 의해 얻어진 가속도와 함께 잡음 역시 입력 신호에 포함된다. 이 잡음 값은 실제 가속에 의한 입력 값에 비해 매우 작기 때문에 수면/비수면 구분에 영향을 적게 미친다. 하지만 참고한 논문<웨어러블 가속도 기기 측정에 의한 수면/비수면 동적 분류>에서 사용한 특징 추출 방법 중 하나인 **ZCM (Zero Crossing Mode)의 경우 일정 구간 내에서 신호의 영교차 횟수를 특징 값으로 지정하고, 이 횟수의 많고 적음에 따라 상태를 분류하므로 이 경우에는 잡음이 수면/비수면 클래스간의 구분성을 떨어뜨리는 요인이 된다.** 따라서 ZCM 특징 추출 전에 잡음을 제거할 필요가 있다. 

 앞에서 언급하였듯 잡음 값의 범위는 실제 입력된 가속도 값에 비해 매우 협소하다. 따라서 전체 입력 신호에 대한 windowing을 통해 잡음을 제거한다. 이 방법은 구체적으로 특정 구간(앞에서 설정한 window)에 대한 표준편차가 특정 임계치 이하인 경우 사용자로부터의 가속도 신호가 입력되지 않은 상태로 가정하여 Intensity 값을 0으로 보정해주는 과정이며, 해당 상수값은 4로 지정하였다.

![](/assets/images/W9/10.png)

뿐만 아니라 window를 정하는 범위(초)는 수면단계가 기록된 간격을 기준으로 설정하였는데, 보유한 데이터에서 수면단계는 30초마다 기록되기 때문에 window의 간격 또한 30초의 배수로 설정하였으며, 구현 중에는 60초로 설정해주었다.

- **Low Pass Filter**

![](/assets/images/W9/11.png)

이번 전처리에는 2.5Hz의 차단주파수와 5의 차수를 지닌 저역통과필터를 사용했다. 지난 연구에는 밴드패스필터링을 이용했는데, 필터링 후의 그래프를 자세히 확대한 결과 이전 데이터의 파형과 지나치게 값이 달라졌던 문제를 확인하여 이외에도 가속도 센서에서 자주 사용되는 저역필터를 적용하였다. 



### 특징 추출

특징 추출에 앞서 수면단계 판별에 걸리는 간격(30초 배수)을 참조한 epoch 구간을 설정하였다. epoch 구간의 길이는 60초로 설정하였으며(전처리 단계에서 windowing을 60초 기준으로 하였기 때문에 맞춰주었다), 해당 구간동안 수집된 가속도 데이터에 대해서 아래 특징을 추출하는 과정을 거치게 된다.

현재 데이터의 조건 상, 64Hz 샘플링 주파수이기 때문에 60초동안 총 3840개의 데이터를 이용하게 되며 이 데이터로부터 각 특징을 하나씩 추출하게 된다.



- **PIM - 움직임의 크기**

 가속도 데이터의 1차원 변환을 통해 운동강도(가속도 벡터)를 얻고 나서, epoch 구간 동안 운동강도 값이 차지하는 넓이(적분)를 계산하여 추출하는 특징이다. epoch당 상수 하나로 리턴된다.



- **ZCM - 움직임의 빈도**

 운동강도 외에 가속도 데이터 X,Y,Z를 이용하며, epoch 구간에서 파형이 0의 값을 가로 지른 횟수를 나타내며, 움직임의 빈도를 카운터하여 추출하는 특징이다. epoch당 상수 하나로 리턴한다.



- **FFT - 주파수 특징**

 Intensity 값을 FFT 변환을 통해 주파수 영역으로 변환한 특징으로, epoch에서 나타나는 Intensity의 파형을 주파수 도메인에서 분석하여 추출하는 특징이다. 일반적인 사람의 움직임은 2초 이내에서 발생하기 때문에 FFT window는 2초 (200 샘플) 로 설정하였다. 그리고 총 512개의 bin을 사용하였으나 전처리 과정에서 저역통과 필터링을 이용하였기 때문에 DC 성분에 인접한 23개의 FFT coefficient만을 사용하였다. 23개의 coefficient가 1개의 epoch에서 30회 얻어지기 때문에 23개 각각의 bin에 대한 표준편차 및 최대값을 최종 특징으로 사용하였다. (총 46차원, 23+23)

```python
# PIM 계산
def calculate_pim(data, sample_rate):
    pim_value = np.trapz(data, dx=1/sample_rate)
    return pim_value

# 푸리에 변환 및 특징 추출
def extract_fft_features(epochs_data, n_bins=512, n_coeff=23):
    fft_features_list = []

    print(len(epochs_data) // (SAMPLE_FREQ*2))
    for i in range(len(epochs_data) // (SAMPLE_FREQ*2)):
        ## FFT 수행
        # DC 인접값만 추출.
        fft_result = np.fft.fft(epochs_data[i * (SAMPLE_FREQ):(i+2) * (SAMPLE_FREQ)], n=n_bins)
        fft_coeff = np.abs(fft_result[:n_coeff])

        fft_features_list.append(fft_coeff)

    fft_features_array = np.vstack(fft_features_list)
    
    # 추출된 인접값들을 epoch 사이에서 최대값과 표준편차 추출
    fft_max_feature = np.max(fft_features_array, axis=0)
    fft_std_feature = np.std(fft_features_array, axis=0)

    print("Shape of the fft_features_array:", fft_features_array.shape)

    return fft_max_feature, fft_std_feature

## 특징 추출하는 함수.
# 1. PIM
# 2. ZCM (실수 발견. 수정 필요)
# 3. FFT
def extract_features(data, lowpassed_data):
    features_list = []
    stages_list = []
    epochs = len(lowpassed_data) // (SAMPLE_FREQ * 60)
    for i in range(epochs):

        # 현재 에폭에서 다뤄야할 데이터
        segment_data = lowpassed_data[i * (SAMPLE_FREQ * 60) : (i + 1) * (SAMPLE_FREQ * 60)]
        stages = data["Sleep_Stage"].iloc[(i + 1) * (SAMPLE_FREQ * 60)]

        # 1. PIM 추출
        pim = calculate_pim(segment_data, SAMPLE_FREQ)
        # 2. ZCM 추출
        zcm = calculate_zcm(segment_data)
        # 3. FFT 추출
        fft_features = extract_fft_features(segment_data, SAMPLE_FREQ)

        features = np.concatenate(([pim], [zcm], fft_features[0], fft_features[1]))
        print(f"features : {features.shape}")

        features_list.append(features)
        stages_list.append(stages)

    features_array = np.array(features_list)
    stages_array = np.array(stages_list)

    return features_array, stages_array
```


💡 특징 추출을 마치고 SVM 모델을 이용하여 장시간 학습 중이나, ZCM 코드 작성 실수로 재학습할 필요가 있어 정확도를 산출하지 못했다. 학습 한번이 시간이 너무 오래걸려 서둘러 실험을 마치고 정확도를 산출해보겠다.


---

## To do

 이번 주가 하드웨어 디바이스를 마무리 지어야하는 단계이다. 흉골 디바이스를 제작하는데 상당한 시간이 소요되었고, 사이즈를 컴팩트하게 납땜하는데 난이도가 상당히 어려웠다. 하지만 손목 디바이스는 흉골 디바이스보다 더욱 크기가 작아야만 사용자의 착용성에 불편함이 없기 때문에 이번주에 진행할 손목 디바이스 납땜에 구체적인 설계와 높은 집중이 필요해 보인다.

 추가적으로, INU메이커스가 작업을 끝내주는대로 흉골 디바이스는 바로 측정에 들어갈 것이며, 사용자의 수면자세 비율 변화를 측정하기 위해 가속도 데이터를 Flutter에서 DB로 저장시키는 서버 세팅을 마친 상태다. 흉골 디바이스의 성능을 측정하면서, 손목 디바이스 또한 서둘러 완성시켜 수면단계를 연동하겠다.
