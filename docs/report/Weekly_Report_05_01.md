---
layout: default
title: 5월 1주차
parent: Weekly_Report
nav_order: 1
---

# 5월 1주차 Weekly Report
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{: toc}

---

## 주간 작업 내용


{: .note }
5월 1주차 작업내용이다. 팀원별 업무 내용은 다음과 같다.  
박종현 : PPG 신호처리 및 미밴드 심박수 추출, 서버 및 DB 실험 환경 구축   
이제욱 : PPG 신호처리 및 가속계 센서 테스트   
이태우 : 수면단계 모델 성능평가 및 개선   

 - 04.29(월) 15:00 ~ 22:00 : PPG 시그널 전처리 및 필터링 진행

 - 04.30(화) 17:30 ~ 21:00 : PPG 대안 모색 회의 진행 / 스마트워치 - Flutter 연결 방법 구상
 
 - 이외 시간 자택 작업 진행


---


## PPG 센서 작업 내용

<aside>
💡 금주 작업 내용은 다음과 같다.
</aside>

지난주까지 약 2주간 PPG 센서 개선을 위한 시도(다른 센서 구매, 전처리 및 신호처리 방법 적용)를 진행했다. 하지만 적합한 PPG 신호를 측정할 수 없었고, 시중에서 구할 수 있는 다른 PPG 센서도 존재하지 않았기 때문에 PPG 센서 측정을 포기하기로 결정하였다.
남은 선택지는 다음과 같았는데,   


1. **가속계 센서를 통한 수면단계 판별**   

2. **스마트워치의 심박수 정보 추출**   


1번의 가속계 센서를 이용하는 방법은 비교적 정확도가 떨어지는 방법이기 때문에 비교적 정확도가 높은 스마트워치의 심박수 정보 및 수면 정보를 추출하는 것으로 우회하기로 결정했다.




### 새로운 PPG 센서 이용

 지난 주에 주문했던 새로운 PPG 센서가 도착하여 실험을 진행하였다. 해당 제품의 이름은 **MAX30102**으로, 마이크로파이썬에서 구동 가능하도록 깃허브 라이브러리도 함께 제공해주었으나 실제로 측정한 결과 해당 센서는 **PPG 신호값이 아닌 호흡수(Respiratory rate)를 측정하고 있었다.**
해당 MAX30102 센서로 측정한 그래프를 관찰하면, 실험자가 숨을 쉴때 파형이 진동하며, 호흡을 멈추면 진동하지 않고 있음을 알 수 있다.


![](/assets/images/W6/p1.png)


 호흡수도 주요한 신체 정보이긴 하나, **기존에 PPG 센서의 도입 목적은 궁극적으로 수면단계 판별을 위함인데 호흡수로는 수면단계를 판별할 수가 없다**(선행 연구도 적고, 모델 사전 학습을 위한 데이터도 존재하지 않음.). 결론적으로 수면단계 판별을 위한 심박수 추출에서 해당 PPG 센서는 적합하지 않다고 판단하였고 따라서 PPG 센서는 잠시 접어두고 다른 방법을 모색하기로 결정하였다.



---


## 갤럭시 워치, 구글 헬스 피트니스 이용

 심박수 추출을 위한 또다른 방법으로, 시중에서 이용되고 있는 스마트 워치(갤럭시워치, 애플워치)의 측정값을 이용하는 방법이 존재한다. 아래는 사용자 앱(Flutter)에서 사용자가 착용한 스마트워치 및 스마트폰에 설치된 피트니스 앱(헬스 모니터링 앱. 스마트워치와 연동되어 있음)이 통신하는 사용자 신체 데이터를 추출하는 방법을 시도한 내용이다.

(현재 이용가능한 애플워치가 없기 때문에 IOS외에도 Android에서 구동 가능한 방법 위주로 조사하였다.)

### flutter_health_connect 라이브러리

[Flutter package - flutter_health_connect](https://pub.dev/packages/flutter_health_connect)

플러터에서 Android/IOS 스마트폰에 설치된 구글 피트니스 앱의 데이터에 접근하도록 해주는 라이브러리 이다. 해당 라이브러리를 이용하여 예제 앱을 설정하였고, 권한 부여 및 Manifest 세팅도 제공된 메뉴얼대로 해주었으나 연동을 위한 **“헬스 커넥트” 권한을 얻는데 반복적으로 에러가 발생하였다**. 라이브러리를 배포한 개발자의 저장소에서 이와 동일한 에러를 겪은 사람들이 피드백을 남긴 것을 다수 확인하였고 현재는 해결이 되어있지 않아 포기하였다

![](/assets/images/W6/p2.png)




### Samsung Health 이용

구글 피트니스 외에도 갤럭시 스마트폰의 기본 어플인 Samsung Health를 이용해서 갤럭시 워치에서 실시간으로 측정되는 심박수 정보를 알 수 있다. Flutter에서도 정보에 접근하기 위한 라이브러리가 존재하며, 이때 이용되는 것이 Android 개발용 SDK인 Samsung Health SDK이나 **해당 SDK를 사용하기 위해선 파트너 협약을 맺은 곳만 사용가능하다는 제약이 존재했다.**

[samsung_health_handler Flutter package](https://pub.dev/packages/samsung_health_handler)

[Samsung Health SDK for Android Samsung Developer](https://developer.samsung.com/health/android/overview.html)




> 앱 배포는 하지 않을 것이며 개인적인 개발을 위해 사용이 가능하냐는 질문이 삼성 포럼에 올라와있었지만, 개발진 측에서 2019년 EU GDPR 규약 이후 아예 불가능하다는 답변이 달려있는 모습이다. 아쉽지만 이용은 불가능해보인다.
> 

![](/assets/images/W6/p3.png)

[Questions about accessing Samsung Health data](https://forum.developer.samsung.com/t/questions-about-accessing-samsung-health-data/21714)



---



## 샤오미 미밴드 이용

차선책으로 헬스 밴드용으로 개발된 샤오미의 `미밴드4` 를 이용해보았다. 미밴드는 샤오미에서 제공하는 웨어러블 디바이스로, 운동하는 환경에 보다 적합한 활동 추적기이다. 미밴드의 경우 스마트폰에 설치된 자사 앱 `Zepp Life` 혹은 `Mi Fitness` 와 블루투스로 연결되어 **사용자의 걸음 수, 심박수 및 건강정보를 송신한다**. 이때 이용하는 블루투스 프로토콜은 BLE로, 이전에 피코와 Flutter앱간 통신에서 적용했던 프로토콜과 동일한 방식이다. **따라서 커스텀 Flutter 앱에서 미밴드와 BLE 통신을 수행한다면. 혹은 패킷을 빼낼 수 있다면** 실시간으로 전송되는 심박수 데이터를 추출할 수 있을 것이라고 판단했다.

![](/assets/images/W6/p4.png)


### **WireShark 패킷 분석**

패킷 분석에서 주로 이용되는 **WireShark**를 이용하여 미밴드 - 스마트폰 간의 블루투스 패킷 교환을 검사하였다. 가장 먼저 스마트폰의 개발자 설정에서 **블루투스 HCI 스누프 로그**를 활성화 및 버그 리포트 제출을 통하여 30분간 내 스마트폰과 블루투스 통신을 수행한 로그를 추출하였다.

이후 HCI 스누프 로그로 얻은 btsnoop_hci.log를 WireShark를 통해 패킷 분석을 진행하였다. 착용한 미밴드와 통신한 기록을 확인할 수 있었고, 특히 심박수 데이터를 요청 및 응답하는 패킷을 확인이 가능했다.

![](/assets/images/W6/p5.png)

- 심박수 응답 패킷

![](/assets/images/W6/p6.png)

- 심박수 요청 패킷

![](/assets/images/W6/p7.png)

- 심박수 응답 패킷 내용

![](/assets/images/W6/p8.png)

심박수 데이터가 응답으로 제공되는 위의 패킷을 분석하여 다음과 같은 정보를 얻을 수 있었다.


 - 소유한 미밴드의 MAC 주소    


 - 심박수 제공 서비스의 UUID    


 - 심박수 제공 서비스의 특성 UUID   




### **패킷 탈취 시도**

WireShark로 패킷 분석을 통해 얻은 정보를 이용하여 Flutter에서 미밴드와 BLE 통신을 수행하는 앱을 작성하였다. 미밴드측에서 심박수 서비스를 advertisie할 것이기 때문에 Scan 후에 위에서 얻은 미밴드의 MAC 주소와 Paring 및 연결을 수행해주었고, 마지막으로 획득한 심박수 서비스의 UUID와 특성 UUID를 이용하여 해당 특성을 찾은 후 심박수 값에 대한 Notify를 설정해주었다.

![](/assets/images/W6/p9.png)

**성공적으로 심박수값이 전송되는 것을 확인할 수 있었으며**, 심박수 값 업데이트 주기는 **약2~3초로**, 굉장히 빠른 주기로 값이 업데이트 됨을 확인할 수 있었다.



### **데이터 수집 및 실험 계획**

 실시간으로 데이터가 들어오는 것을 성공적으로 확인하였고, 수집한 심박수 데이터를 이용하여 수면단계를 분류하고 이에 대한 정확도를 측정하기 위한 환경을 조성하기로 계획했다.

자세한 데이터 수집 계획 및 실험 방안은 다음과 같다.

1. 백그라운드 서버 및 DB 가동으로 Flutter 앱에서 탈취한 심박수 정보를 DB에 저장시킨다.
2. 기상 후 데이터 측정을 정지하고, 하루밤동안 측정된 심박수 데이터를 미리 학습해두었던 모델로 추론하며 수면단계를 분류해본다.
3. 비교 대상은 갤럭시 워치에서 제공하는 Samsung Health의 수면분석, 미밴드의 Zepp Health에서 제공하는 수면분석이며 해당 수면 분석 결과들은 단순히 이미지 형태로만 제공되기 때문에 위에서 모델이 추론한 수면단계 분류 결과를 동일하게 이미지 형태로 변환하여 얼마나 일치하는지 비교하여 정확도를 판별할 것이다(기존과 동일).

![](/assets/images/W6/p10.png)

1. 이후 정확도 개선을 위해 모델 구조 변환 및 심박수 전처리, 특징 추출을 연구할 수 있으며 새롭게 측정된 심박수 데이터로 사전 학습 모델을 재학습 시키는 등 모델 정확성 향상 방안을 모색할 것임.

<aside>
💡 위 과정으로 실험을 진행할 것이며, 차주 예정된 **중간보고평가에서** 이에 대해 자세히 발표할 것이다. **그동안 약 5~6일 정도의 수면 데이터를 수집할 수 있을 것으로 보이며 해당 데이터에 대한 분석 및 수면단계 분류 결과, 정확성 비교 결과를 발표할 것이다.**

</aside>



### **서버 세팅**

**AWS의 EC2 및 RDS를 사용하여 ubuntu 백그라운드 서버와 MySQL DB를 설정하였다.** 퍼블릭 액세스, 고정 IP 할당 및 페어 키 생성, 인바운드 규칙 등 서버와 DB의 연결 과정은 본문에서 언급하기엔 지나치게 길기 때문에 생략하겠다.

Flutter 앱에서 탈취한 심박수 정보를 서버에 POST 요청으로 전송하는 코드를 추가햇고, nodejs로 간단한 서버를 작성하여 해당 POST 요청을 받아 연결된 MySQL에 저장하도록 하였다. nodejs 서버는 pm2를 이용하여 ubuntu EC2 서버에서 백그라운드로 구동하였고, DB에 성공적으로 데이터가 저장되는 것을 확인하였다.

아래는 지난 5월 3일 밤 ~ 5월 4일 아침 간 수면에서 측정된 박종현 팀장의 심박수 정보가 저장된 결과이다.

![](/assets/images/W6/p11.png)




---



## 가속도 센서 작동 테스트

### 가속도 센서 설명

mpu 6050(가속도 센서)는 3.3v ~ 5v 전압에서 작동 가능하며 라즈베리파이 피코는 3.3v의 전압을 output으로 내보내기에 정상 작동 가능하다.

해당 센서는 3축 각속도 센서 + 3축 가속도 센서 (+ 온도 센서)로 구성되어 있다.

![](/assets/images/W6/p12.png)

가속도 센서의 모습은 위와 같다. 

만약 가속도 센서가 평평한 바닥에 있다면 즉 z축이 위를 향한다면 가속도 센서 값이 x축 0g, y축 0g, z축 +1g로 측정된다. 반대의 경우엔 z축이 -1g로 측정되며 이는 x축, y축도 동일하다. 

이렇게 가속도 센서의 값이 어떻게 나오느냐에 따라 사용자의 수면 자세 판별이 가능하다.


**센서 로직**

1. 측정된 각속도, 가속도 데이터들은 1차적으로 16비트 adc를 통해 센서 데이터 레지스터에 저장된다. 
2. i2c 통신을 사용하여 센서 데이터 레지스터에 저장된 가장 최근의 각속도, 가속도 측정 데이터를 접근한다. 
3. 인터럽트 스테이터스 레지스터로 일련의 과정(새로운 데이터가 사용 가능한지?)들을 컨트롤한다.


**센서 측정 범위**

각속도와 가속도의 측정 범위 그리고 샘플링 헤르츠는 프로그래밍적으로 설정이 가능하며 그 범위는 다음과 같다.

각속도 측정 범위 : ±250, ±500. ±1000, ±2000 deg/sec

가속도 측정 범위 : ±2g, ±4g, ±8g, ±16g (g : 중력 가속도)

각속도 센서 샘플링 레이트 : 4 ~ 8kHz

가속도 센서 샘플링 레이트 : 4 ~ 1kHz


**i2c 통신**

i2c 통신 속도 또한 조절 가능하다.

Fast-mode : 400kHz

Standard-mode : 100kHz

i2c 통신은 선 두개로 통신하며  각각 SDA (signals serial data) + SCL (serial clock)가 그 두 선이 되겠다. 선들은 open-drain이고 양방향이다. 

내가 사용하려는 보드가 마스터 MPU6050이 슬레이브가 되어 통신이 진행된다. 


![](/assets/images/W6/p13.png)

SDA, SCL선은 풀업저항에 의해 기본적으로 high 상태를 유지한다.

i2c 신호의 시작과 종료는 다음과 같다.

시작 신호 : SDA신호가 high->low 이면 SCL에서 클럭 신호 발생

종료 신호 : 모든 비트 전송이 끝나면 SCL신호가 high가 되고 SDA신호가 low->high로 바뀐다.



### 가속도 센서 연결

![](/assets/images/W6/p14.png)



### 가속도 센서 테스트

테스트를 위한 코드가 너무 길어 생략하였음.


- **테스트 결과**

1. x축이 아래를 향할 떄

![](/assets/images/W6/p15.png)

2. x축이 위를 향할 떄 

![](/assets/images/W6/p16.png)

3. y축이 아래를 향할 때

![](/assets/images/W6/p17.png)

4. y축이 위를 향할 때

![](/assets/images/W6/p18.png)

5. z축이 아래를 향할때

![](/assets/images/W6/p19.png)

6. z축이 위를 향할 때

![](/assets/images/W6/p20.png)

위와 같이 센서를 중심으로 여러 각도를 취했을 때 모두 결과가 잘 나오는 것을 확인할 수 있다. 


### 가속도 센서 움직임 감지 코드

- 움직임 감지 코드

```python
from imu import MPU6050  
import time
from machine import Pin, I2C

i2c = I2C(0, sda=Pin(8), scl=Pin(9), freq=400000)
imu = MPU6050(i2c)

# LED 설정
led = Pin(25, Pin.OUT)

while True:
    # 가속도 읽기
    acceleration = imu.accel.magnitude
    print(acceleration)
    
    # 정지 상태의 값은 1
    if abs(acceleration - 1) > 0.1:
        print("움직임 감지!")   
        led.value(1) # LED 켜기
    else: 
        led.value(0) # LED 끄기
        
    time.sleep(0.2) 

```


- **테스트 결과**

![](/assets/images/W6/p21.png)

보드를 잡고 움직일 때마다 움직임을 감지해 출력하는 것을 확인할 수 있었다. 



### 가속도 센서 데이터 실제 각도로 바꾸기

위에서 뽑아냈던 가속도 데이터와 자이로 데이터를 이용하여 3축 각도를 얻어낸다. 바꿔진 각도는 데이터셋으로 모아질 것이며 수면 자세 학습에 사용될 것이다.



- **각도 출력 결과**

바닥에 브레드보드를 평평하게 두었을 때의 각도 결과이다.  

![](/assets/images/W6/p22.png)


- **상보 필터**

![](/assets/images/W6/p23.png)

가속도 센서는 센서 특성상 고주파 영역에서 노이즈가 많이 발생하게 되어 정확한 값을 얻기 어렵다. 그래서 노이즈 영역을 제거하고자 Low Pass Filter를 적용한다.

자이로 센서는 센서 특성상 저주파 영역에서 값이 변하는 Drift현상이 발생하여 마찬가지로 정확한 값을 얻기가 힘들다. 그래서 High Pass Filter를 적용한다.

상보필터에서는 가속도 센서의 저주파 영역에서의 장점과 , 자이로센서의 고주파 영역에서의 장점만을 융합한 필터로서 , 방법과 코드가 간단하여 적용하기 쉬운 장점이 있다.   

상보필터에서는 ALPHA라는 가중치 값을 통해 자이로센서와 가속도센서 각각으로부터 얻은 각도를 어떤 비중으로 적용시킬지 정하여 계산한다.  

아래가 바로 보정값(가중치)  ALPHA를 적용한 보정된 각도 값의 식(angleFiX) 이다.   angle_fix = ALPHA * angle_tmpx + (1.0 - ALPHA) * angle_acx 여기서 angle_tmpx 는 자이로센서 값을 기준으로 만든 상보필터 처리를 위한 임시각도이다. ****

X축에 대해 정리하면  angle_tmpx = angle_fix + angle_gyx * dt 가 된다. 

가중치 ALPHA 값은 α = T/(T+Δt) 식으로 구하며,  α (ALPHA) = 1 / (1+0.04)  가 돼,  α 값은 0.96이다.

이 값은 고정값은 아니며 시스템에 맞게 변경할 수 있다. 

식을 보면  자이로 값에  0.96의 가중치를 주고, 가속도 값에  0.04의 가중치를 줘서 합하기에 자이로센서에 더 의존하는 형태라고 말 할 수 있다.  

이런 형태가 비교적 정밀한(안정적인) 출력값을 내는 이유는 각도 변화가 상당히 느린 구간(저주파 영역)에서는 자이로센서의 데이터 값이 작아지기 때문에 가중치 적용으로 작았던 가속도 데이터 값이 상대적으로 커지게 된다. 

따라서 자이로스코프의 저주파 영역에서 발생하는 Drift(드리프트)에 대한 단점을 보완 할 수 있게 된다.


- **필터 적용 유무 결과**

의도적으로 센서 근처에 진동을 주었을 떄 흔들림을 주었을 때 결과이다. 

필터가 적용된 경우 큰 변동이 없는 것을 확인할 수 있지만 본래의 값은 그렇지 않다. 값이 요동치는 것을 확인할 수 있다.

디바이스에는 진동모터가 함께 위치하기 때문에 필연적으로 센서의 근처에 진동이 전해질 수 밖에 없기에 이런 상보 필터의 적용이 필수적이다.

![](/assets/images/W6/p24.png)


여기서 한가지 생각해볼 수 있는 사항은 필터링전 그래프와 필터링후의 그래프간 출력의 시간차가 조금씩 나는 것이다.  이는 더 매끈하게 노이즈를 제거 하려고 ALPHA값 수치를 조정하면 더 커지며 적절한 수치를 찾는 것이 중요해 보인다. 현재는 0.96을 사용할 것이다.



---


## 딥러닝 모델 추출

4월 2주차에 만든 수면단계 판별 모델의 검증 정확도는 78%로 비교적 정확하였다. 하지만 이 모델을 tflite로 변환하는 과정에서 검증 정확도가 69%로 약 10% 하락하였다. 예상되는 원인은 총 3가지로 예상된다.

- 모델이 변환되면서 생기는 연산의 왜곡
- 모델이 변환되면서 float형식의 변환 float-64에서 float-32
- 모델이 변환되면서 시퀀스 길이가 1로 고정

여기서 어떤것이 원인인가 알아보기 위하여 모델의 구조를 한번 살펴보았다.

![](/assets/images/W6/p25.png)

h5파일을 tflite로 변환한 코드는 다음과 같다.

```python
import tensorflow as tf

# Load the existing TensorFlow model
model = tf.keras.models.load_model('/content/my_model.h5')

# Convert the model to TensorFlow Lite
converter = tf.lite.TFLiteConverter.from_keras_model(model)
converter.target_spec.supported_ops = [
    tf.lite.OpsSet.TFLITE_BUILTINS,  # Enable TensorFlow Lite ops.
    tf.lite.OpsSet.SELECT_TF_OPS    # Enable Select TF ops.
]
converter._experimental_lower_tensor_list_ops = False #tensor의 연산수 감소 x

# Convert the model to TensorFlow Lite
tflite_model = converter.convert()

# Save the TFLite model to a file
with open('/content/my_model.tflite', 'wb') as f:
    f.write(tflite_model)

print("Model has been converted to TFLite and saved.")

```

여기서 연산에 왜곡이 없었음을 알수 있었다. 그러면 원인은 크게 두가지로 나눠볼 수 있는데, 첫번째로, h5 모델이 float-64 형식이었는데, 이가 tflite로 변환되면서 float-32로 변환된 것이고. 두번째로, tflite 모델에서는 시퀀스 길이를 1로 고정시킨 것이 성능 하락이다. 이 두가지 원인은 필연적인 것 이므로 이는 accurancy의 감소는 필연적으로 일어나는 것임을 알 수 있다. 그렇다면 h5 모델의 accurancy를 향상 시켜 accurancy가 감소하더라도 변환된 모델이 일정한 accurancy를 유지하면 되는 것이다. 


### 모델 정확도 향상

모델의 정확도를 향상시키기 위한 방법으로 class의 갯수를 줄여서 예측하게 하였다. 먼저 우리의 모델은 수면 단계를 0~5단계로 총 6개의 class가 존재하는데 이를 0~2 단계로 줄여서 예측을 진행하게 하였다. 우리의 모델은 깨어있음, 얕은 수면, 깊은 수면 단계가 필요하기에 나머지 단계들은 모두 2단계로 바꾸어 진행하였다.

![](/assets/images/W6/p26.png)

수면단계

```python
import numpy as np
import os
import matplotlib.pyplot as plt
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import Dense, LSTM
from tensorflow.keras.utils import to_categorical
from sklearn.model_selection import train_test_split

def load_feature_data(base_path, feature_pattern):
    all_data = []
    for filename in os.listdir(base_path):
        if feature_pattern in filename:
            file_path = os.path.join(base_path, filename)
            try:
                # Load all data from file
                data = np.loadtxt(file_path)
                if data.ndim == 1:  # Ensure data is two-dimensional
                    data = data.reshape(-1, 1)
                all_data.append(data)
            except Exception as e:
                print(f"Error reading {file_path}: {e}")
    if all_data:
        return np.concatenate(all_data, axis=0)
    return np.array([])  # Return an empty array if no data

# Set file path
base_path = '/content/drive/MyDrive/features'

# Load all feature data
cosine_features = load_feature_data(base_path, 'cosine_feature')
hr_features = load_feature_data(base_path, 'hr_feature')
time_features = load_feature_data(base_path, 'time_feature')
psg_labels = load_feature_data(base_path, 'psg_labels')  # Assuming this is the label data

# Modify labels as required (limit classes to 0, 1, 2)
psg_labels = np.where(psg_labels >= 3, 2, psg_labels)
y_one_hot = to_categorical(psg_labels, num_classes=3)

# Combine features into one array (adjust as necessary)
features = np.stack([cosine_features, hr_features, time_features], axis=1)

print(features.shape)
print(psg_labels.shape)

# Reshape data to include time step dimension for LSTM
X = features.reshape(features.shape[0], features.shape[1], 1)  # Adding time dimension

# Split data
X_train, X_test, y_train, y_test = train_test_split(X, y_one_hot, test_size=0.2, random_state=42)

# Construct the model
model = Sequential([
    LSTM(50, activation='tanh', recurrent_activation='sigmoid', input_shape=(X_train.shape[1], X_train.shape[2]), dropout=0, recurrent_dropout=0),
    Dense(100, activation='relu'),
    Dense(50, activation='relu'),
    Dense(25, activation='relu'),
    Dense(3, activation='softmax')  # Change the output layer to match the number of classes
])

model.compile(optimizer='adam', loss='categorical_crossentropy', metrics=['accuracy'])

history = model.fit(X_train, y_train, epochs=3000, validation_split=0.2)

model.save('my_model.h5')

loss, accuracy = model.evaluate(X_test, y_test)
print(f"Test Accuracy: {accuracy*100:.2f}%")
```


![](/assets/images/W6/p27.png)

![](/assets/images/W6/p28.png)


h5모델을 불러와서 모델 정확도를 측정한 결과 91.8%의 accurancy가 나왔다. 이는 class가 3개로 줄어들었기 때문에 accurancy가 향상 된 것으로 보인다. tflite파일의 accurancy도 78.2%의 정확도가 나왔다. 비록 12% 정도 accurancy가 감소하였지만 78%의 비교적 높은 정확도를 보였다.












--- 


## To do
- **단기 목표(중간발표 전)**
    - 수면 심박 데이터 수집 및 수면단계 판별 진행
    - 스마트워치와의 간단한 성능 비교 진행
- **장기 목표(중간발표 후)**
    - 수면단계 판별 정확성 향상을 위한 모델 개선 시도
    - 이후 흉부 디바이스 측 개발(가속도 센서 기반 수면자세 판별) 합류 예정
