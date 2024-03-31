---
layout: default
title: 3월 4주차
parent: Weekly_Report
nav_order: 1
---

# 3월 4주차 Weekly Report
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{: toc}

---

## 주간 작업 내용


{: .note }
3월 4주차 작업내용이다. 팀원별 업무 내용은 다음과 같다.
박종현 : Rasberry Pi Pico 개발환경 구축 / Pico - Flutter 블루투스 통신 구축
이제욱 : PPG Signal 전처리 코드 분석 및 MicroPython&C 코드 변환
이태우 : TFLite 모델의 Flutter 업로드 테스트

 - 03.26(화) 17:30 ~ 22:00 : PPG Signal 전처리 코드 분석 및 변환 가능성 판단
 
 - 03.28(목) 22:30 ~ 03:00 : PPG Signal 전처리 코드 Matlab -> C & MicroPython 변환 / Rasberry Pi Pico 수령 및 작동테스트 / 블루투스 스터디 및 Flutter BLE 라이브러리 테스트

 - 03.30(토) 19:00 ~ 04:00 : Rasberry Pi Pico C/C++ & MicroPython SDK 개발환경 구축 / Flutter BLE 코드 분석 및 Pico MicroPython Bluetooth 통신 코드 분석 및 테스트

 - 03.31(일) 14:00 ~ 21:30 : Flutter BLE 코드 분석 및 Pico MicroPython Bluetooth 통신 코드 분석 및 테스트


---


## PICO 개발환경 세팅
 라즈베리파이 피코는 단일칩 PC형태인 여타 라즈베리파이 시리즈와는 다르게 MCU에 가까운 소형 개발보드이다. 따라서 일명 라즈비안 OS가 피코에서는 이용되지 않기 때문에, 직접 SDK를 설치하며 개발환경을 구성해야만 한다. 특히 C/C++ SDK 개발환경 구축이 상당히 어렵지만, Pico를 단독 PC처럼 운용하여 별도 커맨드 창과 VS code 개발환경 구축을 위해서 구축을 어느정도 해놓는 것이 추후 개발에 용이할 것으로 판단했다. (피코는 일반 PC 커맨드창으로 접근이 불가능함.)  
 아래는 두가지 개발환경인 C/C++ SDK와 MicroPython SDK 개발환경을 구성한 결과를 간략하게 제시하였다.


### C/C++ 개발환경 구성

 라즈베리파이 피코에서 C/C++ SDK 윈도우 개발환경 구축을 위해선 다음과 같은 요소들이 필요하다.  

- ARM GCC Compiler
- CMake
- Build Tools for VS 2019
- Python 3.9
- Git
- VS code
- Doxygen

신경쓸게 상당히 많기 때문에 아래 공식 깃허브를 참고해가면서 개발환경을 구축하였다.

[raspberrypi/pico-sdk](https://github.com/raspberrypi/pico-sdk) → raspberrypi pico SDK Github


- 구축완료된 상태

    ![](/assets/images/W2/p1.png)


- Pico 전용 cmd 창과 vs code 환경 구축완료
    
    ![](/assets/images/W2/p2.png)



### MicroPython 개발환경 구성

- **Pico W uf2 업로드**

 아래 공식사이트에서 제공하는 uf2파일을 피코에 업로드하여 MicroPython SDK를 구동해주었다. BOOTSEL을 누른채 연결하여 USB모드에서 업로드를 완료한 뒤에 SDK 준비가 끝났다면 Thonny IDE를 이용하여 스크립트를 작성할 수 있다.

[MicroPython - Python for microcontrollers](https://micropython.org/download/RPI_PICO_W/)


구축이 완료되면 연결된 USB 포트와 Rasberry 전용 MicroPython 인터프리터를 적용하여 개발이 가능해진다.

![](/assets/images/W2/p3.png)

[Thonny, Python IDE for beginners](https://thonny.org/)




 ---



## 블루투스 연결


### 블루투스 통신 및 UUID 개요

- **BLE란**

 라즈베리파이 피코W에는 infineon(인피니어)의 CYW43439 무선 칩셋이 내장되어 있어서  BLE를 통해 무선 블루투스 통신이 기본적으로 가능하다. BLE로 약칭되는 Bluetooth Low Energy(BLE)는 절전 기능을 주목할만한 기능으로 개발된 Bluetooth 무선 기술의 변형으로, 항상 활성화되어 있는 Bluetooth Classic과 달리 BLE는 데이터를 송수신하지 않을 때 절전 상태를 유지할 수 있다는 장점이 있음. 적은 전력으로 고수준의 무선통신을 할 수 있다는 장점 덕분에 우리 디바이스처럼 배터리로 작동되는 장치에 사용할 때 배터리 절약 효과를 크게 누릴 수 있다. 

 특히 Bluetooth Classic은 비교적 데이터 전송률과 연결 범위가 우수하지만, 우리 디바이스의 작동 flow를 생각해보았을 때 주기적으로 센서 하나의 값을 보내는 정도의 적은 데이터는 BLE로도 충분히 전송이 가능할 것으로 생각되며, 마찬가지로 수면 시 스마트폰은 일반적으로 수면자와 인접한 곳에 위치하기 때문에 수면자 신체에 부착된 우리 디바이스와 통신하는데에는 무리가 없을 것으로 보임.

![](/assets/images/W2/p4.png)

- **서비스 용어 정리**
    
    **클라이언트(나)** - 어플리케이션
    
    **서버(상대)** - 블루투스 장치
    
    송신과 수신의 역할이 분리된 단방향 통신의 경우 이 역할이 고정되어 있지만, 두 디바이스 모두가 데이터 송수신을 담당할 때(Ex. 앱 - Pico)는 양방향 통신으로 매번 클라이언트와 서버가 달라진다고 볼 수 있다. 
    예를 들어 Pcio에서 심박수 데이터를 Flutter 앱에 송신해줄 경우 클라이언트는 Flutter 어플리케이션, 서버는 Pico라고 볼 수 있다.
    
    - **서비스**
    
    블루투스 장치(서버)에서 제공하는 기능을 나타낸다. BLE에서는 캐릭터리스틱을 사용하여 서비스의 속성을 설명한다. 서비스는 일반적으로 하나 이상의 캐릭터리스틱을 포함하며, 각각은 UUID와 함께 정의된다.
    
    **서비스 UUID :** 16비트 또는 128비트이며, 공식적으로 정의된 서비스 UUID와 사용자 정의 UUID를 사용할 수 있다.
    
    - **캐릭터리스틱**
    
    블루투스 LE 장치의 서비스 내부에 있는 속성으로 측정 데이터, 상태정보, 알림 등과 같은 값을 포함하며, UUID,  값, 속성, 퍼미션, 디스크립터를 통해 정의한다. 하나 이상의 값과 가질 수 있으며, 속성과 허가는 응용 프로그램에서 변경될 수 있다. 전송되는 실제 데이터 값은 Read와 Notify 속성을 통해 읽을수 있다. 바이트배열 형태로 표현된다.
    
    **캐릭터리스틱 속성(Properties)**
    
    - **Read :** 블루투스 장치에서 캐릭터리스틱 값을 읽는데 사용. 해당 요청에 대한 응답으로 블루투스 장치는 해당 캐릭터리스틱 값의 현재 상태를 응답으로 반환.
    - **Write :** 블루투스 장치에 값을 쓰는데 사용된다. 해당 요청에 대한 응답으로 블루투스 장치는 해당 캐릭터리스틱 값이 변경되었음을 알리거나, 쓰기가 실패했음을 알림.
    - **Notification :** 블루투스 장치간에 캐릭터리스틱 값 변경 사항을 실시간으로 알리기 위해 사용한다. 클라이언트는 서버에 알림을 요청하고, 서버는 캐릭터리스틱 값이 변경되면 클라이언트에게 알림 메시지를 보냄.
    - **Indication :** Notification과 유사하지만 서버는 클라이언트로부터 인디케이션 확인 응답을 받을 때까지 캐릭터리스틱 값을 업데이트 하지 않는다. 이를 통해 클라이언트는 캐릭터리스틱 값 변경을 보장받는다.
    
    **디스크립터(Descriptor)** : 캐릭터리스틱에 대한 추가 정보를 포함한다. 캐릭터리스틱이 사용자에게 설명하는 값을 포함할 수  있다.
    
    ** **중요한점**은 캐릭터리스틱을 통해 송수신을 한다는점. 그리고 각 캐릭터리스틱은 반드시 하나 이상의 속성을 가진다는 점이다.  예를들어 Generic Access Profile 서비스에 캐릭터 리스틱 A 가 Read와 Write를 가지고 있다면 클라이언트는 캐릭터리스틱 값을 읽을수도, 쓸수도 있는것이다.
    
    ![](/assets/images/W2/p5.png)
    
    위 그림처럼 생겼으며 각 캐릭터리스틱은 반드시 UUID를 가지고 있고, Value가 있다. 속성은 여러개일수 있으며, 디스크립터는 여러개가 존재할 수 있으며 이역시 UUID로 식별된다.
    
    즉 특정 캐릭터리스틱의 특정 디스크립터를 찾기 위해선 서비스의 UUID와 캐릭터리스틱의 UUID 그리고 디스트립터의 UUID를 모두 알아야 하며, 
    특정 캐릭터리스틱의 값을 가져오려면 서비스 UUID와 캐릭터리스틱의 UUID를 알고 있어야한다.
    



### Pico Bluetooth LE

BLE를 이용하여 Flutter 앱 간에 통신하기 이전에, Pico에서 마이크로 파이썬을 이용한 BLE 작동 구조를 알아보고, 모듈이 올바르게 작동하는지 테스트할 필요가 있다. 따라서 공식 문서에서 제공되는 아래 두가지 코드를 이용하여 온도 송수신 테스트를 진행하였고, 이외에 추후 Flutter 앱간 양방향 BLE 통신을 위한 코드를 알아볼 것이다.

- **ble_advertising.py**

    아래 코드는 라즈베리파이 피코에서 마이크로파이썬을 이용하여 주변장치들에게 자신이 제공하는 서비스와 캐릭터리스틱의 UUID를 광고(advertise)하는 파트이다. 흔히 블루투스 페어링을 위해 주변장치를 검색하는 단계에서 이 광고(advertise)를 수행하여 주변장치들에게 자신이 수행하는 서비스에 대한 정보와 연결을 원한다는 요청을 흩뿌린다. 여기서는 서비스 UUID와 캐릭터리스틱 UUID를 다음과 같이 설정하였는데, 특히 서비스 UUID는 Bluetooth SIG에 의하여 정해진 표준이다.
    
    **서비스 UUID** : 0x181A  
    
    **캐릭터리스틱 UUID** : 6E400001-B5A3-F393-E0A9-E50E24DCCA9E  


```python
# ble_advertising.py
# BLE advertising payloads.

from micropython import const
import struct
import bluetooth
import machine

# Advertising payloads are repeated packets of the following form:
#   1 byte data length (N + 1)
#   1 byte type (see constants below)
#   N bytes type-specific data

_ADV_TYPE_FLAGS = const(0x01)
_ADV_TYPE_NAME = const(0x09)
_ADV_TYPE_UUID16_COMPLETE = const(0x3)
_ADV_TYPE_UUID32_COMPLETE = const(0x5)
_ADV_TYPE_UUID128_COMPLETE = const(0x7)
_ADV_TYPE_UUID16_MORE = const(0x2)
_ADV_TYPE_UUID32_MORE = const(0x4)
_ADV_TYPE_UUID128_MORE = const(0x6)
_ADV_TYPE_APPEARANCE = const(0x19)

# Generate a payload to be passed to gap_advertise(adv_data=...).
def advertising_payload(limited_disc=False, br_edr=False, name=None, services=None, appearance=0):
    payload = bytearray()

    def _append(adv_type, value):
        nonlocal payload
        payload += struct.pack("BB", len(value) + 1, adv_type) + value

    _append(
        _ADV_TYPE_FLAGS,
        struct.pack("B", (0x01 if limited_disc else 0x02) + (0x18 if br_edr else 0x04)),
    )

    if name:
        _append(_ADV_TYPE_NAME, name)

    if services:
        for uuid in services:
            b = bytes(uuid)
            
            
            print(f'uuid : {uuid}')
            if len(b) == 2:
                _append(_ADV_TYPE_UUID16_COMPLETE, b)
            elif len(b) == 4:
                _append(_ADV_TYPE_UUID32_COMPLETE, b)
            elif len(b) == 16:
                _append(_ADV_TYPE_UUID128_COMPLETE, b)

    # See org.bluetooth.characteristic.gap.appearance.xml
    if appearance:
        _append(_ADV_TYPE_APPEARANCE, struct.pack("<h", appearance))

    return payload

def decode_field(payload, adv_type):
    i = 0
    result = []
    while i + 1 < len(payload):
        if payload[i + 1] == adv_type:
            result.append(payload[i + 2 : i + payload[i] + 1])
        i += 1 + payload[i]
    return result

def decode_name(payload):
    n = decode_field(payload, _ADV_TYPE_NAME)
    return str(n[0], "utf-8") if n else ""

def decode_services(payload):
    services = []
    for u in decode_field(payload, _ADV_TYPE_UUID16_COMPLETE):
        services.append(bluetooth.UUID(struct.unpack("<h", u)[0]))
    for u in decode_field(payload, _ADV_TYPE_UUID32_COMPLETE):
        services.append(bluetooth.UUID(struct.unpack("<d", u)[0]))
    for u in decode_field(payload, _ADV_TYPE_UUID128_COMPLETE):
        services.append(bluetooth.UUID(u))
    return services

def demo():
    payload = advertising_payload(
        name="micropython",
        services=[bluetooth.UUID(0x181A), bluetooth.UUID("6E400001-B5A3-F393-E0A9-E50E24DCCA9E")],
    )
    print(payload)
    print(decode_name(payload))
    print(decode_services(payload))

if __name__ == "__main__":
    demo()
```


- **picow_ble_temp_reader.py**

    pico에 내장된 온도센서와 위에서 작성한 advertising 코드를 이용해 주변 디바이스와 BLE연결을 수립하여 10초 간격으로 온도값을 전송하는 코드이다. 핵심은 다음과 같다.

    **_ENV*_*SENSE_SERVICE** : “0x181A” 의 UUID를 가지는 서비스이다. 아래에서 설명하는 온도 캐릭터리스틱을 보유한다.

    **_TEMP_CHAR** : “0x2A6E” 의 UUID를 가지는 캐릭터리스틱이다. READ, NOTIFY 및 INDICATE의 플래그를 모두 활성화 하였으며, BLE 모듈 생성시 선언된 서비스 안에 GATT로 등록되어 위에서 선언한 서비스의 캐릭터리스틱으로 등록된다.

    **메인함수 demo()** : notify를 이용하기 때문에 캐릭터리스틱인 온도값에 변화가 생길 경우, 연결된 클라이언트에 알려주어 새로운 값을 전달한다.

```python
# picow_ble_temp_reader.py
# The sensor's local value is updated, and it will notify
# any connected central every 10 seconds.

import bluetooth
import random
import struct
import time
import machine
import ubinascii
from ble_advertising import advertising_payload
from micropython import const
from machine import Pin

_IRQ_CENTRAL_CONNECT = const(1)
_IRQ_CENTRAL_DISCONNECT = const(2)
_IRQ_GATTS_INDICATE_DONE = const(20)

_FLAG_READ = const(0x0002)
_FLAG_NOTIFY = const(0x0010)
_FLAG_INDICATE = const(0x0020)

# org.bluetooth.service.environmental_sensing
_ENV_SENSE_UUID = bluetooth.UUID(0x181A)
# org.bluetooth.characteristic.temperature
_TEMP_CHAR = (
    bluetooth.UUID(0x2A6E),
    _FLAG_READ | _FLAG_NOTIFY | _FLAG_INDICATE,
)
_ENV_SENSE_SERVICE = (
    _ENV_SENSE_UUID,
    (_TEMP_CHAR,),
)

.... 생략 .....

def demo():
    ble = bluetooth.BLE()
    temp = BLETemperature(ble)
    counter = 0
    led = Pin('LED', Pin.OUT)
    while True:
        if counter % 10 == 0:
            temp.update_temperature(notify=True, indicate=False)
        led.toggle()
        time.sleep_ms(1000)
        counter += 1

if __name__ == "__main__":
    demo()

```


- **실행 결과**

위에서 작성한 코드를 피코에 업로드한후 실행하여 BLE를 시작하고, NRF Android 앱을 이용하여 이를 페어링하여 값을 확인할 수 있다. 
가장 먼저 확인되는 Generic Access와 Generic Attribute(GATT)는 BLE 장치에 연결 및 기본관리를 담당하는 서비스로, 주목할 서비스는 0x181A UUID의 Environmental Sensing이다. 캐릭터리스틱 UUID는 위에서 설정한대로 0x2A6E이며, 활성화한 플래그와 동일하게 READ, NOTIFY, INDICATE가 모두 활성화되어있다. 신뢰성 낮은 온도센서와 보정 알고리즘의 영향으로 온도가 비교적 높고, 등락도 심하게 관찰되지만 온도 송신과 수신은 정상적으로 이뤄지고 있는걸 확인할 수 있으며, Pico의 LED도 깜빡거리며 정상동작하는 것을 확인할 수 있다.

![](/assets/images/W2/p6.png)

![](/assets/images/W2/p7.png)

- **read_write_ble.py**

이번에 작성해 볼 것은 추후 양방향 데이터 송수신을 위한 Pico Server 코드이다. 마찬가지로 서비스와 캐릭터리스틱에 UUID를 부여해주어야 하는데, 아래 코드에서 확인 가능하듯이 전송 패킷인 **_UART_TX**와 수신 패킷인 **_UART_RX**에 서비스 UUID와 별개의 UUID를 부여해 주었다. 뿐만 아니라 각 패킷의 역할에 맞게 읽고 구독하는 Flag와, 응답대기송신과 무응답대기송신을 담당하는 Flag를 설정해주었다.

메인함수읜 demo()가 실행되면, BLE 모듈이 생성되며 advertising을 수행하고, 주변기기와 연결되기 전까진 LED가 꺼져있다가 연결이 수립되면 LED가 켜지며 0부터 1초씩 카운팅한 값을 TX에 담아 전송한다. **이때 핵심은 반대편인 연결 기기에서도 Pico 서버에 데이터 송신이 가능하고, Pico 서버는 이를 수신이 가능하다는 점인데, 이는 BLE 모듈의 on_write()를 이용함으로써 데이터 수신시 작동하는 이벤트 핸들러를 이용하기 때문이다.** 따라서 연결된 스마트폰의 NRF Android에서 어떠한 값을 송신하면 피코 서버에서는 미리 할당해둔 이벤트 핸들러에 의해서 후속 처리가 가능해진다. 여기서는 받은 값을 출력창에 표시하기만 하였다.


```python
import bluetooth
import random
import struct
import time
from machine import Pin
from ble_advertising import advertising_payload

from micropython import const

_IRQ_CENTRAL_CONNECT = const(1)
_IRQ_CENTRAL_DISCONNECT = const(2)
_IRQ_GATTS_WRITE = const(3)

_FLAG_READ = const(0x0002)
_FLAG_WRITE_NO_RESPONSE = const(0x0004)
_FLAG_WRITE = const(0x0008)
_FLAG_NOTIFY = const(0x0010)

_UART_UUID = bluetooth.UUID("6E400001-B5A3-F393-E0A9-E50E24DCCA9E")
_UART_TX = (
    bluetooth.UUID("6E400003-B5A3-F393-E0A9-E50E24DCCA9E"),
    _FLAG_READ | _FLAG_NOTIFY,
)
_UART_RX = (
    bluetooth.UUID("6E400002-B5A3-F393-E0A9-E50E24DCCA9E"),
    _FLAG_WRITE | _FLAG_WRITE_NO_RESPONSE,
)
_UART_SERVICE = (
    _UART_UUID,
    (_UART_TX, _UART_RX),
)

... 생략 ...

def demo():
    led_onboard = Pin("LED", Pin.OUT)
    ble = bluetooth.BLE()
    p = BLESimplePeripheral(ble)

    def on_rx(v):
        print("RX", v)

    p.on_write(on_rx)

    i = 0
    while True:
        if p.is_connected():
            led_onboard.on()
            for _ in range(1):
                data = str(i) + "_"
                print("TX", data)
                p.send(data)
                i += 1
        time.sleep_ms(1000)

if __name__ == "__main__":
    demo()
```


- **실행결과 -** 연결 전

![](/assets/images/W2/p8.png)

- **실행결과** - 연결 후

선언했던 서비스와 캐릭터리스틱의 UUID가 올바르게 할당된 것을 알수 있으며, 이외에 Flag 설정과 수신받는 데이터도 정상적으로 작동함을 확인가능하다.

![](/assets/images/W2/p9.png)

![](/assets/images/W2/p10.png)

→ 좌측은 피코 서버에서 송신하는 데이터를 출력한 것이고, 우측은 NRF에서 문자열을 송신하였을 때 피코 서버가 이를 정상적으로 수신하여 출력한 결과이다.





 ---




## Flutter


### flutter_reactive_blue

Flutter에서 Bluetooth LE를 지원하는 라이브러리 중 하나인 flutter_reactive_blue이다. Reactive Programming 패턴을 사용하여 데이터 스트림을 처리하는 라이브러리로 프로젝트에서 사용을 위해 아래와 같이 AndroidManifest.xml에 퍼미션과 pubspec.yaml에 종속성을 추가하였다.

![](/assets/images/W2/p11.png)

이 외에, 안드로이드 앱 빌드 시 minSdkVersion이 19로 돌아가고 있었는데, 라이브러리 권장사항으로 21로 수정해주었다.


하지만 라이브러리 로드 중 다음과 같은 에러가 발생했는데, 안드로이드 앱으로 빌드 시 Kotlin 버전이 중복된다는 에러가 반복적으로 뜨기에 build time kotlin 버전을 수정해보았으나 해결이 되지 않아 진척이 막힌 상태임. 다른 라이브러리를 추가로 알아보겠음.

![](/assets/images/W2/p12.png)



### flutter_blue_plus 변경

Flutter에서 Bluetooth LE 분야의 가장 유명한 라이브러리이다. 마찬가지로 퍼미션과 종속성을 추가한 뒤, 안드로이드 build.gradle의 minSdkVersion을 21로 설정해주었다.

![](/assets/images/W2/p13.png)


라이브러리 로드가 정상적으로 진행되는것을 확인하고, 오피셜 라이브러리 사이트에서 제공하는 BLE 연결 어플리케이션을 스마트폰에 빌드하여 아래와 같이 Pico와의 연결을 테스트해보았다.

![](/assets/images/W2/p14.png)


이전에 테스트했었던 picow_ble_temp_reader.py를 피코에서 구동시키고, 0x1801의 UUID를 가지는 Environmental Sensing 서비스에서 0x2A6E의 UUID를 가지는 캐릭터리스틱으로 올바르게 데이터를 수신받고 있음을 확인할 수 있다.

![](/assets/images/W2/p15.png)



### 추후 계획

{: .note }
블루투스 통신 구축에 생각보다 많은 공부와 시간이 필요한것으로 판단됨. 차주까지 Flutter 앱과 Pico에서 각각 BLE 기반 양방향 통신이 완벽하게 이뤄지도록 코드 작성과 테스트를 마치겠음.




 ---




## PPG Signal 전처리 코드 변환


**PPG란?**

PPG는 조직의 미세혈관층에서 혈액량 변화를 감지하는 데 사용할 수 있는 광학적으로 획득된 혈류량이다. PPG 센서는 피부에 빛을 조명하고 빛 흡수의 변화를 측정하는 펄스 산소 측정기를 사용하여 현재 혈류에 산소가 얼마나 포함되어 있는지, 부피가 얼마나 달라지는 지를 구한다. 

그렇기에 PPG센서를 통해 실제로 얻는 data는 실제 heart rate(bpm)이 아닌 혈류량의 변화(pleth) data이다. 

따라서 수면 단계 판별에 활용하기 위해선 추가적인 작업이 필요하다. pleth data 파형의 주기 즉peak와 peak사이 시간 간격(초)를 구해 실제 heart rate를 구해야 한다.

![](/assets/images/W2/p16.png)

다음 논문으로부터 peak를 추출하는 알고리즘을 알았고 구현하였다. 

마이크로파이썬 환경과 c++ sdk에서 사용이 가능하게끔 두 언어로 구현하였다.


```cpp
#include <vector>
#include <algorithm>
#include <iostream>

using namespace std;

vector<double> upslopes(const vector<double>& ppg) {
    int th = 6;                 // 임계값 초기화
    vector<double> pks;       // 초기화
    vector<double> pos_peak;  // 초기화
    int pos_peak_b = 0;         // 초기화
    int n_pos_peak = 0;         // 초기화
    int n_up = 0;               // 초기화
    int n_up_pre = 0;           // 초기화
    for (size_t i = 1; i < ppg.size(); ++i) {
        if (ppg[i] > ppg[i - 1]) {
            n_up++;
        } 
        else {
            if (n_up >= th) {
                pos_peak.push_back(i);
                pos_peak_b = 1;
                n_pos_peak++;
                n_up_pre = n_up; 
            } 
            else {
                if (pos_peak_b == 1) {
                    if (ppg[i - 1] > ppg[pos_peak[n_pos_peak-1]]) {
                        pos_peak[n_pos_peak-1] = i - 1;
                    } 
                    else {
                        pks.push_back(pos_peak[n_pos_peak-1]);
                    }
                    th = static_cast<int>(0.6 * n_up_pre);
                    pos_peak_b = 0;
                }
            }
            n_up = 0;
        }
    }
    return pks;
}

// peak beat의 고유한 값만 유지
// vector<double> tidy_beats(vector<double> beats){

//     vector<double> result ;
//     result.reserve(beats.size());
//     result.insert(result.end(),beats.begin(),beats.end());

//     sort(result.begin(), result.end());
//     result.erase(unique(result.begin(),result.end(),result.end()));

//     return result;
// }

vector<double> pulse_onsets_from_peaks(const vector<double>& sig, const vector<double>& peaks) {
    // 펄스 온셋 식별
    vector<double> onsets(peaks.size()-1);

    for (size_t wave_no = 0; wave_no < peaks.size() - 1; ++wave_no) {
        int min_index = 0;
        double min_value = numeric_limits<double>::max();
        
        // 신호에서 최소값 찾기
        for (int i = peaks[wave_no]; i < peaks[wave_no + 1]; ++i) {
            if (sig[i] < min_value) {
                min_value = sig[i];
                min_index = i;
            }
        }
        
        // 펄스 온셋 설정
        onsets[wave_no] = min_index+1;
    }

    return onsets;
}

int main() {
    // 메인 함수에서는 데이터를 읽고 처리하는 부분을 구현합니다.

    // 입력 데이터 불러오기 
    // sig로 

    vector<double> sig = {0.002108
,0.000777
,-0.000236
,-0.00081
,-0.000986
,-0.00093
,-0.00085
,-0.000906
,-0.001152
,-0.001538
,0.013324
,0.013387
,0.013144
,0.012553
,0.011678
,0.010663
,0.009678
,0.008852
,0.008237
};

    // peak 디텍션
    vector<double> peaks = upslopes(sig);

    // 출력 
    // 남아있는 peak들만 솎아내기
   // peaks = tidy_beats(peaks);
    vector<double> onsets = pulse_onsets_from_peaks(sig, peaks);

    for(int i = 0; i<peaks.size(); i++) cout <<peaks[i] <<' ';
    cout <<"\n";
    for(int i = 0; i<onsets.size(); i++) cout <<onsets[i] <<' ';

    return 0;
}

```


마이크로파이썬 환경에서는 라이브러리 사용의 제약이 있기에 그래프 출력을 제외하곤 라이브러리 없이 구현하였다.


 - 파이썬 구현 및 결과 그래프 출력

```python
import csv
import matplotlib.pyplot as plt

def upslopes(ppg, v=0):

    if v == 1:
        color = ['tab:blue', 'tab:orange']
        fig, (ax1, ax2) = plt.subplots(2, 1, figsize=(10, 8))
        ax1.plot(ppg, color='k', linewidth=1.2)
        ax1.set_title('Peak Detection')
        ax1.set_xlabel('Samples')
        ax1.set_ylabel('PPG')
        ax1.grid(True)

        ax2.plot(ppg, color='k', linewidth=1.2)
        ax2.set_title('Upslopes')
        ax2.set_xlabel('Samples')
        ax2.set_ylabel('PPG')
        ax2.grid(True)

    th = 6
    pks = []
    pos_peak = []
    pos_peak_b = 0
    n_pos_peak = 0
    n_up = 0

    for i in range(1, len(ppg)):
        if ppg[i] > ppg[i - 1]:
            n_up += 1
            if v == 1:
                ax2.plot(i, ppg[i], '.', color=color[0])
        else:
            if n_up >= th:
                pos_peak.append(i)
                pos_peak_b = 1
                n_pos_peak += 1
                n_up_pre = n_up
                if v == 1:
                    ax2.plot(pos_peak[n_pos_peak-1], ppg[pos_peak[n_pos_peak-1]], 'ok', markerfacecolor=color[1])
            else:
                if pos_peak_b == 1:
                    if ppg[i - 1] > ppg[pos_peak[n_pos_peak - 1]]:
                        pos_peak[n_pos_peak - 1] = i - 1
                    else:
                        pks.append(pos_peak[n_pos_peak - 1])
                        if v == 1:
                            ax2.plot(pks[-1], ppg[pks[-1]], 'ok', markerfacecolor='r')
                    th = int(0.6 * n_up_pre)
                    pos_peak_b = 0
            n_up = 0

    if v == 1:
        ax1.plot([int(i) for i in pks], [ppg[int(i)] for i in pks], 'ok', markerfacecolor=color[0])
        plt.tight_layout()
        plt.show()
    return pks 
    
```

```python
def tidy_beats(beats):
    result = sorted(list(set(beats)))
    return result

def pulse_onsets_from_peaks(sig, peaks):
    onsets = []

    for wave_no in range(len(peaks) - 1):
        min_index = 0
        min_value = float('inf')

        for i in range(int(peaks[wave_no]), int(peaks[wave_no + 1])):
            if sig[i] < min_value:
                min_value = sig[i]
                min_index = i

        onsets.append(min_index+1)

    plt.plot(sig, color='k', linewidth=1.2)
    plt.scatter(onsets, [sig[i] for i in onsets], color='red', label='Pulse Onsets')
    plt.xlabel('Samples')
    plt.ylabel('PPG')
    plt.title('Pulse Onsets from Peaks')
    plt.legend()
    plt.grid(True)
    plt.show()
    
    return onsets
```

```python
data = open('./subject1.csv')
reader = csv.reader(data)
sig = []

for row in reader:
    if(row[0] == 'pleth') : continue
    sig.append(float(row[0]))

```


 - **upslopes**

주어진  PPG 데이터에서 상승 기울기를 계산하고, 일정 임계값(threshold)을 초과하는 상승 기울기가 발생한 지점을 찾아서 반환

1. 주어진 PPG 데이터의 각 지점마다 이전 지점과 비교하여 상승하는 경향을 확인.
2. 상승하는 경향이 감지되면 상승 횟수(n_up)를 증가.
3. 상승하는 경향이 감지되지 않으면, 이전에 감지된 상승 횟수가 임계값(threshold)을 초과하는지 확인.
4. 임계값을 초과하는 상승 횟수가 있을 경우, 해당 지점을 피크(pos_peak) 리스트에 추가하고, 이전에 발생한 상승 횟수를 기억.
5. 이전에 발생한 상승 횟수를 기준으로 새로운 임계값을 설정합니다(0.6배로 줄임).
6. 다음 상승을 찾기 위해 상승 횟수를 다시 0으로 초기화.
7. 이 과정을 모든 데이터 지점에 대해 반복하고, 최종적으로 찾은 양성 피크(pos_peak)의 위치를 반환.


 - **tidy_beats**

겹치는 peak 지점이 있는지 확인하고 중복 제거


 - **pulse_onsets_from_peaks**

원 신호(sig)와 신호 내의 피크(peaks) 지점들을 기반으로 신호의 파형에서 각 펄스의 시작 지점(onset)을 찾는다.


1. peaks 리스트에 저장된 피크 지점을 기준으로 각각의 펄스 파형을 처리. 이때, peaks의 길이에서 1을 빼는 이유는 마지막 peak는 다음 펄스의 시작점을 가리키지 않기 때문.
2. wave_no 변수는 현재 펄스의 번호를 나타내며, 이를 이용하여 현재 펄스와 다음 펄스 사이의 영역을 결정.
3. min_index와 min_value 변수는 현재 펄스의 최소값을 찾기 위한 초기화를 수행. min_value는 무한대로 초기화.
4. 현재 펄스의 영역(int(peaks[wave_no])부터 int(peaks[wave_no + 1])까지)을 반복 해당 구간에서 최솟값을 찾는다.
5. 최솟값을 찾으면 그 위치(min_index)를 onsets 리스트에 추가. 이때, 인덱스가 0부터 시작하기 때문에 1을 더해야 실제 위치를 나타낸다.
6. 이 과정을 모든 peak 지점들에 대해 반복하고, 각 펄스의 시작 지점을 모은 onsets 리스트를 반환.

절대 경로로 실제 ppg raw data를 불러온 뒤 2000개 가량 정도로 그래프 출력하였다.

데이터가 매우 많아 그래프가 출력이 안되기 때문이다. 


```python

sigs = sig[1000000:1002000]
peaks = upslopes(sigs,1)

tidy_peaks = tidy_beats(peaks)

onsets = pulse_onsets_from_peaks(sigs, peaks)
```


![](/assets/images/W2/p17.png)

![](/assets/images/W2/p18.png)



이렇게 구해 낸 각 peak간의 시간 간격을 통해 실제 심장의 bpm을 다음과 같은 공식을 통해 구할 수 있다.

$$
bpm = 60 / Peak 간의 \ 간격(sec)
$$


**추후 계획**
{: .note }
 현재 가지고 있는 데이터에선 시간 간격 정보가 없었기에 heart rate까지 구할 수는 없었다. 
실제로 제품(센서 등)을 받았을 때 시간 정보 또한 받아들여 bpm까지 구하고자 한다.





 ---




## TFLite 모델 업로드 테스트

 - **플러터 내에서 .tflite 파일을 실행하기**

![](/assets/images/W2/p19.png)

pubspec.yaml 파일에 종속성 추가 및 assets 추가하여 dart파일에 tflite파일을 로드하고 tflite를 이용가능.


 - **모델의 입출력 사이즈**

![](/assets/images/W2/p20.png)


 - **main.dart**
 임의의 input값을 넣어 모델을 거쳐 output값이 제대로 나오는 지 확인 해보는 작업임.


```dart
import 'package:flutter/material.dart';
import 'package:tflite_flutter/tflite_flutter.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(
          title: Text('TFLite Flutter Example'),
        ),
        body: Center(
          child: ElevatedButton(
            onPressed: runTFLiteModel,
            child: Text('Run TFLite Model'),
          ),
        ),
      ),
    );
  }

  void runTFLiteModel() async {
    try {
      // Load the model
      final interpreter = await Interpreter.fromAsset('assets/sleepst.tflite');
      // Assuming your model expects a 1x1x36 input tensor and produces a 1x1x4 output tensor
      var input = List.generate(
          1, (i) => List.generate(1, (j) => List.filled(36, 0.0)));
      var output = List.filled(1 * 1 * 4, 0.0).reshape([1, 1, 4]);

      // Run the model on the input data
      interpreter.run(input, output);

      // Process the output as needed
      print(output);

      // Close the interpreter
      interpreter.close();
    } catch (e) {
      print('Failed to load model: $e');
    }
  }
}
```



![](/assets/images/W2/p21.png)

아래 와 같은 오류가 발생. 모델의 연산자를 사용할 수 없다는 뜻임.


```python
# 입력 텐서에 데이터 설정
tensor = np.zeros((1, 1, 36), dtype=np.float32)
interpreter.set_tensor(input_details[0]['index'], tensor)

# 모델 실행
interpreter.invoke()

# 추론 결과 얻기
output_data = interpreter.get_tensor(output_details[0]['index'])
print(output_data)
```


![](/assets/images/W2/p22.png)

{: .note }
파이썬 코드에서는 위와 같이 정상적으로 모델이 출력되는것을 확인함. 하지만 Flutter 내에서 업로드한 모델의 경우 위와 같은 연산자 오류로 작동에 오류가 발생한 것으로 보임.  
따라서 추후에 연산자를 보정하는 내용을 코드에 추가시킬것.




 ---




## Todo
 - Tensorflow Lite 환경에서 구동가능한 모델 찾아보기 → 직접 학습시켜 정확도 비교할 것.  
여러 모델 별 특징(복잡도, 일반적인 성능 등)을 정리해놓고, 전부 학습시켜서 최종적으로 정확도를 비교하여 높은 성능의 모델을 선택할 것.

 - Flutter 내의 TFLite 모델 정상작동을 위한 연산자 보정 방법 찾기

 - Flutter BLE 라이브러리와 코드 분석 진행 및 Pico 서버와의 송수신 코드 작성  
    / MicroPython 기반 Pico BLE 서버 코드 완성 및 최종 테스트 진행



| Name    | Work Content |
|:--------|:------------------------------|
| `박종현` | `Flutter BLE 라이브러리와 코드 분석 진행 및 Pico 서버와의 송수신 코드 작성 및 MicroPython 기반 Pico BLE 서버 코드 완성 및 최종 테스트 진행` |
| `이제욱` | `Tensorflow Lite 환경에서 구동가능한 모델 찾아보기 → 직접 학습시켜 정확도 비교할 것. 여러 모델 별 특징(복잡도, 일반적인 성능 등)을 정리해놓고, 전부 학습시켜서 최종적으로 정확도를 비교하여 높은 성능의 모델을 선택할 것.` |
| `이태우` | `Flutter 내의 TFLite 모델 정상작동을 위한 연산자 보정 방법 찾기 & TFLite 모델 분석 및 학습 보조` |

