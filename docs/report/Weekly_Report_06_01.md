---
layout: default
title: 6월 1주차
parent: Weekly_Report
nav_order: 1
---

# 6월 1주차 Weekly Report
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{: toc}

---

## 주간 작업 내용


{: .note }
6월 1주차 작업내용이다. 팀원별 업무 내용은 다음과 같다.    
박종현 : 가속도 특징 추출 & 흉골 디바이스 케이스 실험 및 환경 구축 & 손목 디바이스 설계 및 납땜      
이제욱 : 흉골 디바이스 케이스 설계, 안정성 보완 작업 & 손목 디바이스 설계 및 납땜      
이태우 : 가속도 기반 수면단계 판별 개선 작업     


 - 05.21(화) 14:00 ~ 22:10 : 신규 PPG 측정 및 수면단계 분류 실험   

 - 05.29(수) 12:30 ~ 19:30 : 안정성 보완 & 수면단계 분류 실험   

 - 05.30(목) 14:00 ~ 22:30 : 손목 디바이스 설계 및 납땜   

이외 개인 작업은 자택에서 진행


---


<aside>
💡 진행상황 요약

</aside>


- **흉골 디바이스(완료)**
1. 3D 프린팅 디바이스 케이스 제작 완료(완료)   
2. BLE 통신 및 수면 자세 판별 구현 완료(완료)   
3. 흉골 디바이스 납땜 완료(완료)   

- **손목 디바이스**   
1. 데이터 전처리 및 특징 추출(완료)   
2. 수면 단계 모델 구현 및 개선   
3. 손목 디바이스 납땜(완료)   
4. 3D 프린팅 디바이스 케이스 제작(완료)   

<aside>
💡 금주 업무 내용 요약

</aside>


 - 흉골 디바이스 케이스 수정 및 시연 & 안정성 보완 작업   

 - 손목 디바이스 납땜 및 케이스 제작   

 - 가속도 기반 수면단계 판별 개선 작업   



## 신규 PPG 센서 측정

[아두이노용 심박 센서 Heart Rate Monitor Sensor For Arduino [SEN0203]](https://www.devicemart.co.kr/goods/view?no=1327536)

 - **엄지 & 검지**   
    
    ![Untitled](/assets/images/W10/p1.png)
    

 시중에서 마지막으로 구할 수 있는 PPG 센서를 배송받아 테스트를 진행해 보았다. 엄지와 검지에서는 드디어 정상적인 Raw Signal이 검출되는 것을 확인할 수 있었음(명확한 피크가 확인 가능). 특히 검지의 경우 큰 노이즈 없이 완벽한 파형이 검출되는 모습이다.

 - **손목**   
    
    ![Untitled](/assets/images/W10/p2.png)
    

 하지만 우리 디바이스의 측정 위치인 손목에서는 아래 그림에서처럼 피크를 전혀 검출하지 못하고 있다. 특히 측정거리에 따라 알수 없는 파형이 그려지기도 하며, 이전에 준비했던 필터링 방법을 적용하여도 엄지와 검지에서 검출된 형태와는 상당히 다른 비정상적인 파형(우측그림)이 나타나기 때문에 해당 센서로도 손목에서의 PPG 피크 검출은 불가능이라는 결론을 내렸다.




## 최종 디바이스 설계도

![Untitled](/assets/images/W10/6%E1%84%8B%E1%85%AF%E1%86%AF%202%E1%84%8B%E1%85%B5%E1%86%AF%20%E1%84%87%E1%85%A9%E1%84%80%E1%85%A9%E1%84%89%E1%85%A5%20cd39438472c14bff92d4d5490a76e44f/Untitled.png)


## 1. 흉골 디바이스

### 가속도 센서 값 흔들림 해결

 사용자의 수면 자세가 불안정 자세로 향해 진동을 시작할 때 가속도 센서 옆에 진동 모터가 위치해 가속도 센서의 측정 값이 흔들려 수면 자세가 잘못 판단되는 경우가 생겼고 해당 경우에 진동이 멈출 때가 있었다. 진동모터 옆에 솜을 위치해 센서로 향하는 영향을 줄이려고 시도했지만 별 다른 효과가 없었고 알고리즘의 수정을 하기로 결정했다. 

1. **알고리즘 수정**

불안정한 자세라 판단하고 진동을 가할 때 수면자세 판별을 멈췄으며 약 2~3초 가량 진동을 가한 뒤에 다시 수면자세 판별을 시작한다 그때도 불안정 자세라면 진동레벨을 강화시키고 반복한다.

1. **블루투스 연결 끊김 문제 해결**

간헐적으로 가속도 센서의 연결이 끊어져 pico 연결이 끊어지는 문제가 있었다. 이를 해결하기 위해 Flutter subscription에서 연결이 끊어짐을 감지하면 다시 연결을 시도하는 기능을 추가하였다.

```python
connectSubscription = device.connectionState
            .listen((BluetoothConnectionState state) async {
          if (state == BluetoothConnectionState.disconnected) {
            print("connection stopped");
            setState(() {
              connected = 'ggungim';
            });
            restart(connectSubscription, valueSubscription, device);
          }
        });

Future restart(dynamic sub1, dynamic sub2, dynamic device) async {
      // cleanup: cancel subscription when disconnected
      device.cancelWhenDisconnected(sub1, delayed: true, next: false);

      // 연결 끊겼을때 subscribe 해제
      device.cancelWhenDisconnected(sub2);
      //FlutterBluePlus.stopScan();
      //await device.disconnect();
      flutterBlueInit();
    }
```

### 최종 흉골 케이스 완성 및 실착

겔패치를 달아 가슴에 실제 착용 가능하며 하단에 존재하는 두 구멍을 통해 디버깅 및 충전이 가능하다. 추후 사포질을 통해 날카로운 부분을 뭉툭하게 만들 것이다.

![Untitled](/assets/images/W10/6%E1%84%8B%E1%85%AF%E1%86%AF%202%E1%84%8B%E1%85%B5%E1%86%AF%20%E1%84%87%E1%85%A9%E1%84%80%E1%85%A9%E1%84%89%E1%85%A5%20cd39438472c14bff92d4d5490a76e44f/Untitled%201.png)

### 시연 영상

흉골 디바이스를 착용하고 자세를 바꿔가며 자세 탐지와 진동 자극의 발생 여부를 확인하는 영상이다. 아래 링크로 첨부함.

[시연 영상 Google Drive](https://drive.google.com/file/d/1NuZMm_mkpdEn4rLLWm8E7PNAEarBL00D/view?usp=drive_link)

### 데이터 수집 및 입증 실험

Flutter 어플리케이션으로 EC2 nodejs 서버와 통신하여 MySQL DB에 수면자세 데이터를 수집하는 중이다(아래 사진). 자세는 10초마다 DB에 저장되며, UTC시간으로 기입되기 때문에 실제 수면시간은 해당 시간보다 9시간을 더해주어야 한다.

![Untitled](/assets/images/W10/6%E1%84%8B%E1%85%AF%E1%86%AF%202%E1%84%8B%E1%85%B5%E1%86%AF%20%E1%84%87%E1%85%A9%E1%84%80%E1%85%A9%E1%84%89%E1%85%A5%20cd39438472c14bff92d4d5490a76e44f/Untitled%202.png)

 06.05까지 일반적인 수면자세 비율을 측정할 것이며, 이후부터 최종발표날까지 진동 자극을 가해 수면자세의 비율이 얼마나 변화했는지, 우측 혹은 뒤집은 자세의 개선 성공 횟수를 측정할 것이다. 
최근 3일간 박종현 팀장의 수면자세 비율(일반적인 수면자세)은 다음과 같다.

![Untitled](/assets/images/W10/6%E1%84%8B%E1%85%AF%E1%86%AF%202%E1%84%8B%E1%85%B5%E1%86%AF%20%E1%84%87%E1%85%A9%E1%84%80%E1%85%A9%E1%84%89%E1%85%A5%20cd39438472c14bff92d4d5490a76e44f/Untitled%203.png)

- **24.05.30 → 05.31 수면**
    
    정자세 21%, 좌측 수면자세 36%, 우측 수면자세 43%
    
- **24.05.31 → 06.01 수면**
    
    정자세 49%, 좌측 수면자세 22%, 우측 수면자세 28%, 뒤집어진 자세 약 1%.
    
- **24.06.01 → 06.02 수면**
    
    정자세 42%, 좌측 수면자세 25%, 우측 수면자세 33%
    

---

## 2. 손목 디바이스

![Untitled](/assets/images/W10/6%E1%84%8B%E1%85%AF%E1%86%AF%202%E1%84%8B%E1%85%B5%E1%86%AF%20%E1%84%87%E1%85%A9%E1%84%80%E1%85%A9%E1%84%89%E1%85%A5%20cd39438472c14bff92d4d5490a76e44f/Untitled%204.png)

최소한의 사이즈로 컴팩트한 형태 완성을 위해 위와같이 납땜하였음. 가로 24mm, 세로 57mm, 두께 13mm로 매우 작은 사이즈로 구성되었다. 이 사이즈에 맞추어 작은 케이스를 출력 주문한 상황이며, 차주 평일에 수령후 모듈 조립 예정.

---

## 3. 수면단계 모델 성능개선

데이터셋은 PhysioNet의 가속도 센서 데이터셋에 대한 설명.

<aside>
💡 PhysioNet의 가속도 센서 데이터셋의 대한 설명

- **ACC** (32 Hz): 삼축 가속도계 데이터로, 각 축의 이름은 **ACC_X**, **ACC_Y**, **ACC_Z**입니다.
- **Sleep_Stage**: PSG에서 파생된 기술자가 주석을 단 수면 단계 라벨이 30초마다 기록됩니다.
</aside>

### 공통 데이터 전처리

1. LPF (cut frequency 2.5Hz)
2. 샘플링 rate 조정:.
    - 원본 32 Hz의 데이터를 여러 샘플링 rate로 표현: 16 Hz, 32 Hz, 64 Hz.
3. 윈도우로 합침:
    - 데이터를 30초 간격으로 분할하여 하나의 윈도우를 만든다. 하나의 윈도우 크기는 30초 * 샘플링 레이트
    - 각 윈도우에 대해 시간적 특성을 부여하기 위해 윈도우 번호를 붙임. 즉 사용자가 디바이스를 착용하는 시점의 30초가 윈도우번호 1임.

### 가속도 특징 추출

- **PIM - 움직임의 크기**

epoch 구간 동안 운동강도 값이 차지하는 넓이(적분)를 계산하여 추출하는 특징.

- **ZCM - 움직임의 빈도**

epoch 구간에서 가속도 파형이 0값을 가로지른 횟수를 나타내며, 움직임의 빈도를 카운터하여 추출하는 특징.

- **FFT - 주파수 특성**

epoch에서 나타나는 Intensity 값을 FFT 변환을 통해 주파수 도메인에서 추출하는 특징. 전처리 과정에서 저역통과 필터링을 이용하였기 때문에 DC 성분에 인접한 23개의 FFT coefficient의 최대값, 표준편차인 46차원 coefficient를 사용하였다.

### 3-1. 샘플링 rate별 실험결과

1D-CNN에 대하여 다음과 같은 실험을 진행하였다.

```python
model = Sequential([
    Conv1D(32, 3, activation='relu', input_shape=(1920, 2)),  # 입력 형태를 맞추기 위해 수정 필요 (ACC, window number)
    MaxPooling1D(2),
    Conv1D(64, 3, activation='relu'),
    MaxPooling1D(2),
    Flatten(),
    Dense(128, activation='relu'),
    Dropout(0.5),
    Dense(1, activation='sigmoid')  # 이진 분류를 위해 sigmoid 활성화 함수 사용
])
```

- 16hz로 다운샘플링.
- 32hz로 다운샘플링.
- 64hz로 진행

- 실험 결과

![20에포크에 대하여 실행한 결과](/assets/images/W10/6%E1%84%8B%E1%85%AF%E1%86%AF%202%E1%84%8B%E1%85%B5%E1%86%AF%20%E1%84%87%E1%85%A9%E1%84%80%E1%85%A9%E1%84%89%E1%85%A5%20cd39438472c14bff92d4d5490a76e44f/%25E1%2584%2583%25E1%2585%25A1%25E1%2584%258B%25E1%2585%25AE%25E1%2586%25AB%25E1%2584%2585%25E1%2585%25A9%25E1%2584%2583%25E1%2585%25B3_(3).png)

20에포크에 대하여 실행한 결과

현재 수면 단계 예측 모델의 정확도가 40%대로 매우 낮기 때문에 성능 개선이 필요하다. 이를 해결하기 위해 수면 단계 라벨을 전처리하는 방법을 적용하였다.

수면 단계 라벨을 전처리하는 이유는 PhysioNet의 데이터셋이 30초마다 수면 단계를 검사하여 라벨링을 수행하고, 이 라벨을 64Hz 신호 값에 매핑한 것이기 때문이다. 즉, 64Hz로 샘플링된 데이터에서 30초 윈도우의 값을 단일 라벨로 표현하는 것이 라벨 값을 정확하게 반영하지 못할 수 있다. 이러한 문제를 해결하고자 수면 단계 라벨의 전처리를 통해 데이터의 대표성을 향상시키고자 한다.

### 3-2. 수면 단계 라벨 전처리하여 실험

PhysioNet 데이터셋에서 제공하는 수면 단계 라벨은 30초 간격으로 작성되어 있다. 따라서 기존 수면 단계 라벨을 30초 윈도우 내의 최빈값을 사용하여 전처리하였다. 이 방법은 각 30초 구간 내에서 가장 빈번하게 나타나는 수면 단계를 대표 값으로 설정함으로써 라벨의 일관성을 높이고, 모델이 더 정확한 학습을 할 수 있도록 도와준다.

이와 같은 전처리 과정을 통해 수면 단계 라벨의 신뢰성을 개선하고, 모델의 정확도를 향상시킬 수 있을 것으로 기대된다.

- 실험 결과

![20에포크에 대하여 실험한 결과](/assets/images/W10/6%E1%84%8B%E1%85%AF%E1%86%AF%202%E1%84%8B%E1%85%B5%E1%86%AF%20%E1%84%87%E1%85%A9%E1%84%80%E1%85%A9%E1%84%89%E1%85%A5%20cd39438472c14bff92d4d5490a76e44f/1.png)

20에포크에 대하여 실험한 결과

수면 단계 라벨을 전처리한 후 모델을 다시 학습시킨 결과, 정확도가 약간 향상되었다. 정확도는 이전의 47%대에서 실험 결과 약 55%로 증가하였다. 이는 수면 단계 라벨의 일관성을 높인 것이 모델 성능에 좋은 영향을 미친 것으로 예상된다.

### 3-3. 모델 수정 및 실험

1D-CNN 모델이 성능을 발휘하지 못하여 모델 구조를 수정하고 실험을 진행하였다.

1. LSTM 모델 적용
    - 시계열 데이터를 더 잘 기억할 특정을 지닌 LSTM(Long Short-Term Memory) 모델을 적용하였다.
    - LSTM 모델도 성능이 좋진 못하였음.
2. 1D-CNN과 LSTM 혼합 모델
    - 1D-CNN과 LSTM을 결합한 모델을 적용하여 좀더  성능 항상을 해보기로 하였음.
        
        ![Untitled](/assets/images/W10/6%E1%84%8B%E1%85%AF%E1%86%AF%202%E1%84%8B%E1%85%B5%E1%86%AF%20%E1%84%87%E1%85%A9%E1%84%80%E1%85%A9%E1%84%89%E1%85%A5%20cd39438472c14bff92d4d5490a76e44f/Untitled%205.png)
        
- 혼합 모델은 1D-CNN 레이어로 특성을 추출한 후, LSTM 레이어로 시계열 정보를 처리하는 구조로 설계.
    
    ```python
    model = Sequential([
        Conv1D(64, 3, activation='relu', input_shape=(1920, 2)),
        MaxPooling1D(2),
        Conv1D(128, 3, activation='relu'),
        MaxPooling1D(2),
        LSTM(64, return_sequences=True),
        LSTM(64),
        Dropout(0.5),
        Dense(128, activation='relu'),
        Dropout(0.5),
        Dense(3, activation='softmax')  # 다중 클래스 분류를 위해 softmax 활성화 함수 사용
    ])
    ```
    

- **실험 결과**

![다운로드 (4).png](/assets/images/W10/6%E1%84%8B%E1%85%AF%E1%86%AF%202%E1%84%8B%E1%85%B5%E1%86%AF%20%E1%84%87%E1%85%A9%E1%84%80%E1%85%A9%E1%84%89%E1%85%A5%20cd39438472c14bff92d4d5490a76e44f/%25E1%2584%2583%25E1%2585%25A1%25E1%2584%258B%25E1%2585%25AE%25E1%2586%25AB%25E1%2584%2585%25E1%2585%25A9%25E1%2584%2583%25E1%2585%25B3_(4).png)

> 16hz,32hz의 데이터는 모델의 성능이 64hz와 차이가 많이 나는 모습이다. downsampling을 하였을때 데이터셋의 특징이 명확하게 보이지 않아 그 특징이 사라진 것이 원인이 된 것으로 보인다. 하지만 64hz의경우 71%의 accuracy를 보였다. 아직까진 accurcy가 많이 낮다. 데이터의 전처리 방법을 수정하여 데이터의 특징을 더욱 잘 드러나게 하는 방법을 적용하거나 모델을 수정하여 fine tuning의 방법이나 하이퍼파라미터 수정하는 방법들을 적용하여 성능을 더욱 개선해야한다. 개선의 여지는 있어보인다.
> 

---

## To do

- 계획대로 5월 안에 흉골과 손목 디바이스 또한 납땜을 완료하여 실험 진행이 가능한 상태.


- 현재 흉골 디바이스를 실제 수면 시 착용하여 수면 데이터를 수집 중. 수집한 데이터를 통하여 일반적인 수면 자세 비율 대비 디바이스 착용 시 진동 자극 유무에 따라 수면자세의 비율이 얼마나 변화하는지 기록할 예정.


- 수면단계 판별 모델은 아직 정확도 개선의 여지가 남아있어 지속적으로 모델 개선 시도 중. 다음주 중으로 완성하여 Flutter 탑재 예정.
