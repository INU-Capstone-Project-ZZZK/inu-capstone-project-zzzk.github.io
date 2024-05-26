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
박종현, 이제욱, 이태우: 흉골 디바이스 납땜 및 작동 테스트  
박종현, 이태우 : 가속도 기반 수면단계 모델 학습 및 샘플링 시도  
이제욱 : 흉골 디바이스 3D 모델링, 하드웨어 배치 설계




 
- 05.21(화) 16:30~21:15 : 납땜 계획, 하드웨어 배치 설계, 주간 회의 

- 05.22(수) 11:00 ~22:10 : 흉골 디바이스 납땜

- 05.23(목) 15:00 ~ 22:50 : 발표 ppt 제작 및 준비, 흉골 디바이스 납땜
  
- 05.24(금) 12:00 ~ 20:05 : 흉골 디바이스 최종 납땜 , 발표 

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
💡 현재는 진동이 너무 커 사용자가 깰 우려가 있음 적절한 진동을 찾는 실험이 필요

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

### RawSampliing

64개의 데이터 즉 1초마다 값을 30개의 값을 뽑아주는 sampling이다. 이렇게 한다면 값이 64 → 30으로 줄어 overhead가 줄어들어 원할한 진행이 될 것이다.

```python
def RawSampliing(data, time_window=30, sample_freq=64):
    sampled_intensity = []
    sampled_stage = []

    for i in range(int(data.iloc[::sample_freq].shape[0] / time_window)):
        sampled_intensity.append(data["intensity"].iloc[::sample_freq].iloc[time_window*i : time_window*(i + 1)].values)
        sampled_stage.append(data["Sleep_Stage"].iloc[::sample_freq].iloc[time_window*i])

    sampled_intensity = np.array(sampled_intensity[:-1])
    sampled_stage = np.array(sampled_stage[1:])

    return sampled_intensity, sampled_stage
```

### DiffSampling

미분값을 이용한 샘플링은 가속도 값의 변화를 강조하여 특징을 추출한다. 이는 가속도의 변화율에 주목함으로써 데이터에서 변화를 명확하게 관찰할 수 있다. 

또한 값이 1개만 줄어들지만 디지털 그래프 같이 파형이 변화 되므로 데이터의 overhead가 줄어들 것이다.

```python
def DiffSampling(sampled_intensity):
    # 가속도 값의 변화 기반 샘플링

    sampled_intensity_diff = np.diff(sampled_intensity, axis=1)

    return sampled_intensity_diff
```

### IntervalMaxSampling

데이터를 30초 단위로 끊어서 각 30초 구간에서 2초 최대값 샘플링하는 방식이다. 

제일 처리속도가 적을 것으로 예상이 된다. 

```python
def intervalMaxSampling(data, interval=2):
    data['TIMESTAMP'] = pd.to_datetime(data['TIMESTAMP'], unit='s')
    data.set_index('TIMESTAMP', inplace=True)

    sampled_intensity_per_sec = data['intensity'].resample(f'{interval}S').max()
    sampled_intensity_per_label = sampled_intensity_per_sec.groupby(pd.Grouper(freq='30S')).apply(list)[:-1]

    max_length = max(len(lst) for lst in sampled_intensity_per_label)
    sampled_intensity = np.array([lst + [np.nan] * (max_length - len(lst)) for lst in sampled_intensity_per_label])

    sampled_stage = data.resample('30S')["Sleep_Stage"].first().values[1:]
    return sampled_intensity, sampled_stage

```

### 실험결과

샘플링에 대한 실험결과는 다음과 같았다.  FCN 모델보단 LSTM모델에서의 성능이 더 좋게 나왔다. 이는 LSTM모델이 시계열 데이터 값인 가속도 센서의 값에 더 적합했기 때문이다. 원본의 성능에서 75% 값이 나왔는데 이는 원본 데이터값이 제일 특징이 잘 드러났기 때문이다. 

sampling값에서는 interval max sampling 값이 제일 성능이 좋았지만 58.4%로 이를 그대로 이용하기에는 어려움이 있었다. 

앞서 박종현 학생이 분석한 논문에 따르면 가속도 센서를 이용한 수면 단계 분류의 정확도는 80% 대로 정확도를 더 끌어올릴 수 있어 보인다. 

![](/assets/images/W9/9.png)

## To do

- 수면단계 판별 모델 개선
- 적절한 sampling 기법 탐색
- 손목 디바이스 납땜 완성
- 적절한 진동 세기 결정

