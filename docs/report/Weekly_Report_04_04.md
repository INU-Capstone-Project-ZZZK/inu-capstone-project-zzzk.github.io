---
layout: default
title: 4월 4주차
parent: Weekly_Report
nav_order: 1
---

# 4월 4주차 Weekly Report
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{: toc}

---

## 주간 작업 내용


{: .note }
4월 4주차 작업내용이다. 팀원별 업무 내용은 다음과 같다.  
박종현, 이제욱 : PPG 시그널 전처리 및 필터링 기술 서베이 및 심박수 시그널의 보간 코드 분석 진행
이태우 : 수면단계 모델 학습 및 성능평가



 

 - 04.22(화) 17:30 ~ 22:00 : PPG 센서 작동 테스트 및 시그널 전처리 

 - 04.23(수) 15:00 ~ 23:00 : PPG 센서 작동 테스트 및 시그널 전처리 
 
 - 04.24(목) 16:00 ~ 23:00 : PPG 센서 작동 테스트 및 시그널 전처리 / 납땜 및 실제측정 진행

 - 04.25(금) 15:00 ~ 20:00 : 블루투스 통신 및 모터 세기 제어


---
## Crowtail PPG 센서 수령 후 피코 결합 및 실제 시그널 측정

<aside>
💡 4월 16일날 센서를 수령하였고 해당 주차는 중간고사 주차였기에 다음주인 22일부터 본격적인 센서 테스트에 착수하였다. 테스트는 브레드보드에 라즈베리파이 피코 WH(헤더 납땜)이 되어있는 것을 연결해 사용했으며 PP 센서 또한 브레드보드에 연결하여 사용하였다.

</aside>

### PPG 센서 연결 및 데이터 수집

라즈베리파이 피코의 GPIO 맵과 실제 브레드보드에 연결한 상태의 모습은 다음과 같다.

![](/assets/images/W5/p1.png)

PPG 센서값은 아날로그 값으로 전달되기 때문에 피코 GPIO에서 지원하는 ADC 핀을 사용하였다. 피코의 ADC는 34, 32, 31번 포트가 존재하고 이 중 31번 포트의 경우 GPIO 26번으로, 이를 센서 시그널을 읽어오는데 사용하였다.

이때 라즈베리파이 피코의 ADC는 16비트 ADC를 지원하기 때문에 아날로그신호를 0~65535에 해당하는 값으로 변환시켜준다.

- **센서 시그널 측정 및 텍스트 파일 저장 코드**

```python
import machine
import time

# ADC 핀 설정
adc = machine.ADC(machine.Pin(26))

# 측정 주기 (밀리초)
sampling_interval_ms = 1000

# 파일 이름
file_name = "sensor_data.txt"

# ADC 값을 파일에 저장하는 함수
def save_to_file(data):
    with open(file_name, "a") as f:
        f.write(str(data) + "\n")

# 메인 루프
while True:
    # 센서에서 데이터 읽기
    value = adc.read_u16()

    # 파일에 데이터 저장
    save_to_file(value)

    # 테스트를 위해 콘솔에 출력
    print("value:", value)

    # 샘플링 주기마다 파일에 데이터 저장
    time.sleep_ms(sampling_interval_ms)
```

### 손목 센서에서의 심박수 측정을 위한 PPG 신호 측정 시도

센서로부터 데이터를 읽을 준비를 마치고, 전원을 공급하여 일정 샘플링 주파수마다 데이터를 측정하였음. 아래 그림을 손가락에서 측정한 PPG 신호를 파이썬 라이브러리로 플로팅하고, 사전에 준비해 놓았던 Peak, Onset Detection 알고리즘을 적용한 결과이다.

![](/assets/images/W5/p2.png)

플로팅에서 알 수 있듯, 약간의 파형이 존재하는 듯 보이나, 일정한 패턴없이 마구잡이로 흔들리는 노이즈와 같은 형태의 신호가 출력되었음.
→ 노이즈의 영향인지, 추가적인 전처리를 통하여 순수한 PPG 시그널을 추출해야 하는 것인지 판단이 힘들기에 다른 방법들을 시도해보기로 했음.

## 신체 측정 부위 변화

정확한 측정을 위해 외부 형광등의 가시광선요소가 센서를 방행하지 않도록 실제 운용 환경의 케이스에 부착시키고 어두운 환경을 조성한 뒤에 손목, 손가락, 흉골 부위 등 신체 여러 부위에서 PPG를 측정해가며 경과를 기록하였다.

- 부착 결과

![](/assets/images/W5/p3.png)

- 손가락

![](/assets/images/W5/p4.png)

- 흉골

![](/assets/images/W5/p5.png)

- 손목

![](/assets/images/W5/p6.png)

하지만 측정 부위 변경에도 불구하고 기존과 마찬가지로 신호의 일정한 패턴 발견 불가능. 
따라서 신호 자체에 노이즈가 존재할 것이라고 판단하고 PPG 신호에서 노이즈 필터링 및 심박수를  추출했던 다양한 논문들을 참고하며 다양한 신호 전처리 방법을 도입하였음.

## Crowtail PPG 시그널 전처리 진행

### Scailing

신호 대 잡음비를 향상시키고 피크를 더 잘 감지하기 위해 센서 데이터의 y축 스케일을 조정하는 방법이다. 여러 필터링 방법들을 시도해 본 결과, 스케일링이 가장 먼저 선행되지 않으면 밴드패스 필터링 등 주파수 대역의 필터링 방법이 적용되지 않는 문제를 발견해 가장 먼저 수행한 필터링이다. 여기서 이용한 스케일링은 Min-max Scailing과 Normalization이며, 두 스케일링 모두 유사한 그래프 형태를 보였고, 이후 필터링의 적용 과정에서도 큰 차이를 보이지 못했다. 

- **Min-max Scailing**

![](/assets/images/W5/p7.png)

- **Normalization**

![](/assets/images/W5/p8.png)

### MA Filtering

이동평균 필터(Moving Average Filter)는 시계열 데이터 또는 신호에서 잡음을 제거하고 데이터를 부드럽게 처리하는 데 사용되는 방법이다. 연속적인 데이터 포인트의 평균을 계산하여, 각 데이터 포인트의 값으로 대체하거나 별도의 처리 과정을 거치는데, 여기서 이용한 것은 단순이동평균(**Simple Moving Average, SMA**)과 지수가중이동평균(**Exponential Weighted Moving Average, EWMA**)이다.

참고논문(**Use Moving Average Filter to Reduce Noises in Wearable PPG During Continuous Monitoring)**

- 적용 코드

```kotlin
def MAfiltering(data):
    window_size = 5
    averages = []

    window_sum = sum(data[:window_size])
    averages.append(window_sum/window_size)

    for i in range(len(data)-window_size):
        window_sum = window_sum - data[i] + data[i + window_size]
        average = window_sum / window_size
        averages.append(average)
    
    return averages

def EWMA(data):
    alpha = 0.2
    ewma = [data[0]]  # 첫 번째 EWMA는 첫 번째 데이터 값으로 초기화
    for i in range(1, len(data)):
        next_value = alpha * data[i] + (1 - alpha) * ewma[-1]
        ewma.append(next_value)
    return ewma
```

![](/assets/images/W5/p9.png)

### BandWidth Filtering

 원시 PPG 신호에는 심박수, 호흡수, 소음에 대한 정보가 포함되어 있다. 노이즈를 제거하고 관련 정보를 추출하기 위해 일반적으로 디지털 필터링 방법을 사용하는데, 원시 PPG 신호에서 심박수 정보가 포함된 주파수 범위는 일반적으로 0.5Hz~3.0Hz이며, 특정 연구에서 심박수 정보가 가장 적절히 포함된 범위는 0.6~3.3Hz라고 표현한 것을 참고하여 해당 범위 내의 주파수만 허용하기 위해 대역통과 버터워스 필터를 사용했다.

참고 논문(**Optimal filter characterization for photoplethysmography-based pulse
rate and pulse power spectrum estimation)**

- 적용 코드

```kotlin
def bandpass_filter(data, lowcut, highcut, fs, order=4):
    nyq = 0.5 * fs  # 나이키스트 주파수
    low = lowcut / nyq  # 하한 주파수 정규화
    high = highcut / nyq  # 상한 주파수 정규화
    b, a = butter(order, [low, high], btype='band')  # 밴드패스 필터 계수 생성
    y = lfilter(b, a, data)  # 필터 적용
    return y
```

![](/assets/images/W5/p10.png)

### Wavelet Transform

 Wavelet 변환은 푸리에 변환의 한계를 극복하는 데 주된 목적을 두고 만들어진 일종의 필터링 기법이다. 푸리에 변환은 신호가 시간적으로 변하지 않는다는 가정을 하는데, 이에 반해 웨이블릿 변환은 시간적으로 주파수 성분이 변하는 신호에 대해 시간과 주파수 성분을 표현하기 위해 사용하는 방법이다.

 푸리에 변환(Fourier Transform)은 데이터를 분석하는 강력한 도구이지만, 갑작스러운 변화를 효율적으로 나타내지는 못한다. 그 이유는 푸리에 변환이 시간 또는 공간에 국한되지 않는 사인(Sin)파의 합으로 나타내기 때문이다. 반면 PPG 신호는 과도현상으로 인해 천천히 변화하는 추세 또는 진동이 자주 나타나기 때문에 Wavelet 변환을 사용하는 것이 ppg 신호를 필터링 함에 있어 효과적임을 알 수 있다. 

참고논문(**Photoplethysmogram (PPG) signal analysis and wavelet de-noising)
→** Moving-Average filter와 푸리에 필터보다 Wavelet 필터가 PPG 시그널의 노이즈를 더 잘 제거하고 변동성에 민감하다는 것을 주장.

- 웨이블릿 함수 적용 코드

```python
import numpy as np
import pywt
import matplotlib.pyplot as plt

def wavelet_transform(data, wavelet='db4', level=3):
    # 웨이블릿 변환을 위해 리스트 데이터를 numpy 배열로 변환
    data_array = np.array(data)
    
    # 웨이블릿 변환 수행
    coeffs = pywt.wavedec(data_array, wavelet, level=level)
    
    return coeffs

# 테스트용 데이터
signal_data = sigs

# 웨이블릿 변환 수행
coefficients = wavelet_transform(signal_data)

# 원래 데이터 그래프 그리기
plt.figure(figsize=(10, 5))
plt.subplot(len(coefficients) + 1, 1, 1)
plt.plot(signal_data, color='blue')
plt.title('Original Signal')

# 웨이블릿 변환 결과 그래프 그리기
for i, coeff in enumerate(coefficients):
    plt.subplot(len(coefficients) + 1, 1, i + 2)
    plt.plot(coeff, color='red')
    plt.title(f'Level {i+1} Detail Coefficients')

plt.tight_layout()
plt.show()

# 역변환 수행
reconstructed_signal = pywt.waverec(coefficients, 'db4')

# 역변환된 신호 그래프 그리기
plt.figure(figsize=(10, 5))
plt.plot(reconstructed_signal, color='green')
plt.title('Reconstructed Signal')
plt.show()

```

- **결과**

![](/assets/images/W5/p11.png)

- **웨이블릿 함수 적용 최종 결과**

![](/assets/images/W5/p12.png)

## 필터링 결과 복기

- 스케일링 - 밴드패스 필터링(0.6Hz~3.3Hz) 결과

![](/assets/images/W5/p13.png)

시도했던 필터링 방법 중에서 min-max 스케일링을 적용한 데이터셋에 0.6~3.3Hz의 밴드패스 필터링을 적용하고, Peak Detection 알고리즘을 적용시켜 센서 데이터의 Peak를 뽑아낸 결과이다. 
알고리즘에서 적용된 피크까지 포함시켜 피크개수를 카운트 했을 때, 29개의 피크가 관찰되며 이는 800개의 샘플 내에서 800/29 = 27개의 평균 샘플 거리를 지니게 된다. 해당 데이터의 샘플링 주파수는 100Hz로 설정하였기 때문에 피크간의 시간간격인 RR거리는 27 * 0.01 = 0.27초가 된다.

최종적으로 평균 심박수 계산은 위한 60초 / RR거리(초)의 계산 결과는 60 / 0.27로 약 **222bpm**이라는 말도 안되는 수치의 높은 심박수가 계산되는데, 가만히 앉아서 센서를 측정했던 상황을 고려했을 때 평균적인 인간의 심박수인 60~100bpm에서 훨씬 벗어난 심박수가 기록되었다. 이는 현재 측정되는 시그널, 혹은 전처리 방법에 문제가 있음을 알 수 있게 되는데, 구체적으로 다른 PPG 데이터와 비교해보고자 한다.

- 논문에서 제공하는 PPG 형태(좌측), 오픈 데이터셋에서 제공되는 PPG 형태(우측)

![](/assets/images/W5/p14.png)

 제공되는 데이터셋을 관찰해볼 때, 피크의 형태가 굉장히 깔끔하며 피크의 높이(y축)또한 일정하게 유지되며 약간의 상승, 혹은 하락만이 관찰되고 있다. 하지만 우리가 측정한 PPG 시그널은 노이즈의 영향을 고려하더라도 그  피크의 높낮이가 지나치게 제각각이며 제공되는 그래프 형태와 굉장히 다른 형태를 보이고 있음을 알 수 있다. 센서의 결합에 문제가 있나 여러번 체크해봤지만 제공된 데이터시트와 예제대로 올바르게 결합하였고, 전원 공급에도 문제가 없었기 때문에 어디서 문제가 발생한 것인지 판별이 힘들었음.

 뿐만 아니라 결정적으로 허공을 바라보는 파형이나, 책상을 바라보는 파형이나 손목 등 신체부위를 바라보는 경우나 파형이 비슷하고, 특히 알수없는 이유로 허공을 바라볼 때 규칙적인 파형이 발생하는 이상한 상황이 발생
→ 하지만 이는 심박 파동이 아님.

 이러한 이유로 구매한 Crowtail PPG 센서에 이상이 있다 판단하여 다른 PPG 센서 구입을 진행하였다. 빠른 배송이 가능한 네이버 스토어에서 사용 예제가 많고, 라즈베리파이 피코에서 운용가능하도록 별도의 라이브러리까지 준비된 새로운 PPG 센서 MAX30102를 선정하여 주문하였음.

## 납땜 및 실제측정 진행

 5월 1일까지 스마트워치와 실제 성능 비교를 위해 케이스와의 실제 결합을 진행했어야 함.
하지만 PPG 센서에 문제가 있어 새로운 센서를 주문하였고, 아직 수령하지 못하여 리튬배터리와 충전모듈, 피코를 먼저 납땜 진행. 완료

 주말에 수령한 MAX30102(Red, IR 광선)과 납땜 부품 임시 결합. 손목에서의 PPG 신호를 출력하여 파형을 확인할 결과, 이전과는 다르게 매끄럽고 일정한 신호값이 출력되며, 손목에 부착시 호흡(들이쉬고 내쉴때마다) PPG 파형이 진동하는 것을 확인.

 그러나 측정 도중 케이스 내부에 들어있던 충전모듈과 피코 간의 납땜 부위가 끊어지면서 더이상의 측정이 불가능했음. 급하게 이어 붙이려 순간접착제를 사용하다가 접착제가 두 핀끼리 겹쳐지면서 작동하지 않는 상황이 발생. 더이상의 측정이 불가능했음.

- **납땜 부위가 끊어진 상황**

![](/assets/images/W5/p15.jpg)

- **접착제로 부착 시도..**

![](/assets/images/W5/p16.jpg)


## 진동모터 테스트

진동모터가 pwm모드를 통하여 정상작동이 되는지 확인을 해보았다. 그래서 예제로 진동의 세기가 순차적으로 증가하는 코드로 실험을 해보았다.

```python
from machine import Pin, PWM
from time import sleep

# GPIO 15에 PWM 객체 생성
pwm = PWM(Pin(15))

# PWM 주파수 설정 (1000Hz)
pwm.freq(1000)

try:
    while True:
        # 0에서 100%까지 듀티 사이클을 증가시키면서 모터 강도 제어
        for duty in range(65535):
            pwm.duty_u16(duty)
            sleep(0.0001)
        # 100%에서 0%까지 듀티 사이클을 감소시키면서 모터 강도 제어
        for duty in range(65535, 0, -1):
            pwm.duty_u16(duty)
            sleep(0.0001)
except KeyboardInterrupt:
    pwm.deinit()  # PWM 정지

```

라즈베리파이 피코에서 pwm모드를 통한 모터 제어는 가능하다는 결론이 나왔다.

### 블루투스 통신으로 모터 세기 제어하기

pwm모드로 진동모터 세기 제어가 가능했으며 이제는 블루투스 통신으로 모터세기를 제어해볼 차례이다.

먼저 기존에 블루투스 피코간 통신 코드에서 rx부분의 코드를 변경 하여야한다. on_rx 함수은 바이트 데이터를 그대로 수신하기 때문에 이를 정수 형식으로 바꿔줘야 한다. 여기서 라즈베리파이피코는 little endian이기 때문에   옵션으로 int.from_bytes(v, 'little')를 사용해준다.

```python
def on_rx(value_handle, v):
    print("Received Data: ", v)
    try:
        # 바이트 데이터를 정수로 직접 변환
        intensity = int.from_bytes(v, 'little')  # 'little' 또는 'big' 엔디안에 따라 선택
        # 진동 세기 값이 PWM 범위 내에 있도록 조정
        pwm_intensity = max(0, min(intensity, 65535))
        set_vibration_intensity(pwm_intensity)  # 진동 세기 설정
        print("Vibration intensity set to:", pwm_intensity)
    except ValueError:
        print("Invalid intensity value")
```

여기서 바이트 데이터를 정수로 변환을 하여 전달한다. 위에서 진동수의 최대 값이 65535이기 때문에 min max의 범위를 정해준다. 이렇게 한다면 통신이 완료되어 rx로 데이터가 넘어왔을때 이를 정수로 변환하여 진동 세기를 제어 할 수 있게 해준다. 하지만 이는 비트단위이기 때문에 값이 1~255사이의 값만을 보낼 수 있다. 

이제 진동 제어 코드를 함수로 나타내보자. 

```python
# 모터를 연결한 GPIO 핀 번호 설정 및 PWM 인스턴스 생성
vibration_motor = PWM(Pin(15))
vibration_motor.freq(1000)  # PWM 주파수 설정 (예: 1000Hz)

def set_vibration_intensity(intensity):
    # 듀티 사이클을 조정하여 진동 세기 변경
    vibration_motor.duty_u16(intensity)
```

진동제어 함수는 간단하다. 핀번호의 pwm객체를 생성해주고 주파수를 설정한다. 이때 호출 함수는 duty_u16을 함수로 이용한다.

이를 전체 코드로 나타내면 다음과 같다.

```python
import bluetooth
import random
import struct
import time

from machine import PWM,Pin
from ble_advertising import advertising_payload

from micropython import const

# 모터를 연결한 GPIO 핀 번호 설정 및 PWM 인스턴스 생성
vibration_motor = PWM(Pin(15))
vibration_motor.freq(1000)  # PWM 주파수 설정 (예: 1000Hz)

def set_vibration_intensity(intensity):
    # 듀티 사이클을 조정하여 진동 세기 변경
    vibration_motor.duty_u16(intensity)

    
def on_rx(value_handle, v):
    print("Received Data: ", v)
    try:
        # 바이트 데이터를 정수로 직접 변환
        intensity = int.from_bytes(v, 'little')  # 'little' 또는 'big' 엔디안에 따라 선택
        # 진동 세기 값이 PWM 범위 내에 있도록 조정
        pwm_intensity = max(0, min(intensity * 100, 65535))
        set_vibration_intensity(pwm_intensity)  # 진동 세기 설정
        print("Vibration intensity set to:", pwm_intensity)
    except ValueError:
        print("Invalid intensity value")

## 플래그 선언
# 장치 연결 이벤트
_IRQ_CENTRAL_CONNECT = const(1)
# 장치 연결 해지 이벤트
_IRQ_CENTRAL_DISCONNECT = const(2)
# GATT를 이용한 데이터 write 이벤트
_IRQ_GATTS_WRITE = const(3)

_FLAG_READ = const(0x0002)
_FLAG_WRITE_NO_RESPONSE = const(0x0004)
_FLAG_WRITE = const(0x0008)
_FLAG_NOTIFY = const(0x0010)
_FLAG_INDICATE = const(0x0020)

## UUID 설정
# 캐릭터리스틱 선언
# 상대방이 읽을 수 있도록 Read, Notify 활성화를 위해 플래그도 선언
_UART_TX = (
    bluetooth.UUID("6E400003-B5A3-F393-E0A9-E50E24DCCA9E"),
    _FLAG_READ | _FLAG_NOTIFY,
)

# 상대방이 쓸수 있도록 Write, 무응답쓰기도 지원하도록 플래그 선언
_UART_RX = (
    bluetooth.UUID("6E400002-B5A3-F393-E0A9-E50E24DCCA9E"),
    _FLAG_WRITE | _FLAG_WRITE_NO_RESPONSE,
)

# 서비스 UUID 선언
_UART_UUID = bluetooth.UUID("6E400001-B5A3-F393-E0A9-E50E24DCCA9E")
_UART_SERVICE = (
    _UART_UUID,
    (_UART_TX, _UART_RX),
)

## =====================

class BLESimplePeripheral:
    def __init__(self, ble, name="mpy-uart"):
        
        ## BLE 모듈 생성
        self._ble = ble
        self._ble.active(True)
        self._ble.irq(self._irq)
        
        # GATT 서비스 등록 - 수신, 송신 핸들러
        ((self._handle_tx, self._handle_rx),) = self._ble.gatts_register_services((_UART_SERVICE,))
        self._connections = set()
        
        # 상대방이 쓰기 작업시 콜백 이벤트 함수
        self._write_callback = None
        self._payload = advertising_payload(name=name, services=[_UART_UUID])
        self._advertise()
    
    # 이벤트 콜백 함수
    def _irq(self, event, data):
        # 연결 성공 시,
        if event == _IRQ_CENTRAL_CONNECT:
            conn_handle, _, _ = data
            print("New connection", conn_handle)
            self._connections.add(conn_handle)
        
        # 연결 실패 시, advertise 진행
        elif event == _IRQ_CENTRAL_DISCONNECT:
            conn_handle, _, _ = data
            print("Disconnected", conn_handle)
            self._connections.remove(conn_handle)
            self._advertise()
        
        # 쓰기 이벤트 발생 시(상대 디바이스가 데이터 송신 시)
        elif event == _IRQ_GATTS_WRITE:
            # 데이터 수신시 두 개의 핸들러가 고정되는데, 이는 캐릭터리스틱 별로 값이 고정되어 있다.
            # 예시로, 내 스마트폰과 연결시 value_handle은 2개의 RX 캐릭터리스틱에 대하여 12와 14의 고정값을 지닌다.
            conn_handle, value_handle = data
            value = self._ble.gatts_read(value_handle)
            
            # 쓰기 이벤트 콜백 함수 호출 파트
            if value_handle == self._handle_rx and self._write_callback:
                self._write_callback(self._handle_rx, value)    
    # 패킷 전송 함수
    def send(self, data):
        for conn_handle in self._connections:
            self._ble.gatts_notify(conn_handle, self._handle_tx, data)
    
    # 패킷 수신 콜백 함수 지정
    def on_write(self, callback):
        self._write_callback = callback
    
    # 연결 여부 확인 함수
    def is_connected(self):
        return len(self._connections) > 0
    
    # 지정된 인터벌마다 advertise 진행
    def _advertise(self, interval_us=500000):
        print("Starting advertising")
        self._ble.gap_advertise(interval_us, adv_data=self._payload)

def demo():
		# 피코 LED 핀 지정
    led_onboard = Pin("LED", Pin.OUT)
    ble = bluetooth.BLE()
    p = BLESimplePeripheral(ble)
		

		
		# Rx 수신 시 이벤트 함수 지정
    p.on_write(on_rx)

    i = 0
    while True:
        if p.is_connected():
		        # 연결되었을 경우 LED를 키며 0부터 카운트를 하나씩 늘려나감.
            for _ in range(1):
                data = str(i) + "_"
                print("TX", data)
                p.send(data)
                i += 1
        time.sleep_ms(1000)

if __name__ == "__main__":
    demo()

```

flutter app으로 라즈베리파이 피코와 통신이 완료되고 app에서 숫자를 전송하면 진동 모터 세기가 제어가 된다. 기존코드에서 연결이 되었을 경우 카운트를 하나씩 늘려가며 출력하는 부분은 연결이 확인 되었는지 확인 하기위하여 그대로 남겨놓았다.

<aside>
💡 TODO:
측정을 위한 최선의 과정을 거쳤으나 위와같이 쉽지않은 과정이 있었음. 먼저 PPG 센서의 신뢰도에도 문제가 존재했지만 작은 케이스 내부에 모듈들을 우겨넣다보니 설계했던것과는 다르게 납땜한 부위의 구리선이 꺾여버리며 끊어지는 상황이 발생. 과사에서 납땜기를 다시 빌려 납땜 재진행 필요.
뿐만 아니라 조교님과 상의 후 스마트워치와의 성능 비교 일정 및 현 상황 관련하여 면담이 필요할 것으로 보임. 
또한 진동의 단계를 나누어 숫자를 state로 나누는 함수를 작성해야한다. 이는 진동의 세기를 어떻게 느끼는 지에 따라 회의가 필요한 부분이다.

</aside>
