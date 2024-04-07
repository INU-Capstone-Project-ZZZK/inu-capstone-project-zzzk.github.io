---
layout: default
title: 4월 1주차
parent: Weekly_Report
nav_order: 1
---

# 4월 1주차 Weekly Report
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{: toc}

---

## 주간 작업 내용


{: .note }
4월 1주차 작업내용이다. 팀원별 업무 내용은 다음과 같다.  
박종현 : Flutter - Pico 양방향 통신 개발  
이제욱 : 데이터셋 전처리 / 수면단계 모델 학습  
이태우 : Flutter 환경에서 TFLite 파일 이식  

 - 04.02(화) 17:30 ~ 22:00 : TFLite 환경 이식 가능한 모델 탐색 / 데이터셋 탐색 / TFLite 연산자 문제 분석 / 블루투스 스터디
 
 - 04.04(목) 21:00 ~ 02:00 : Flutter to  Pico 코드 작성 및 분석 / 데이터셋 전처리 / Flutter 환경 TFLite 파일 이식 문제 분석

 - 04.06(토) 14:00 ~ 02:00 : Flutter to  Pico 코드 작성 및 분석 / 데이터셋 전처리 / Flutter 환경 TFLite 파일 이식 문제 분석 완료 및 다른 모델 탑재 가능 확인

 - 04.07(일) 14:00 ~ 20:30 : Flutter to  Pico 블루투스 양방향 통신 테스트 / 수면단계 모델 학습 및 분석


---

## Flutter - Pico 양방향 통신 개발

지난 주에 테스트했던 피코 모듈과 Flutter 기반 BLE 통신을 기반으로 이를 응용하여 최종적으로 Flutter-Pico 간 BLE 통신 개발을 진행하였다.

### 구조 설명

기본적으로 라즈베리파이 피코에서는 MicroPython으로, 앱에서는 Dart기반 Flutter로 스크립트를 작성하였으며, 두 플랫폼이 BLE로 통신하는 과정을 도식화 해보면 다음과 같다.

1. 피코측에서 BLE Advertisie 진행.
2. Flutter 앱에서 주변 BLE 기기들을 Scan하고, 발견된 피코 디바이스와 Connect 진행.
(이때 디바이스의 구분자는 MAC 주소 혹은 서비스 UUID로 진행한다.)
3. 피코는 Flutter에서 요청한 Connect 요청을 수락하고, 미리 설정해두었던 두가지 캐릭터리스틱(송신, 수신)을 등록하여 통신을 시작한다.
4. Flutter는 연결이 완료되면 피코로부터 서비스를 제공받는데, 서비스 내의 캐릭터리스틱은 미리 설정해두었던 두가지 캐릭터리스틱의 UUID(송신의 UUID, 수신의 UUID)혹은 미리 설정해두었던 Flag로 구분하여 피코로부터 들어오는 값(Rx Value)는 별도로 저장하거나 화면에 띄운다. 반대로 보내는 값(Tx Value)는 write 기능을 이용하여 키보드로 입력한 값을 피코로 데이터를 전송한다.

<aside>
💡 피코에서 주기적으로 전송하는 값은 전처리된 PPG 시그널이어야 하지만, 현재 심박수 센서가 준비되지 않았기 때문에 임의로 1씩 늘어나는 카운트 값을 전송해주도록 하였다.
PPG 센서를 수령하면 전송값을 해당 시그널 값으로 바꾸어 줄 것이다.

</aside>

### MicroPython(Pico)

지난주에 작성하였던 advertise_payload 함수를 이용하여 advertise를 진행하여 자신의 MAC 정보와 서비스, 캐릭터리스틱 UUID 정보를 넘긴 뒤에, 이를 인식한 Flutter 앱과 Connect하여 값을 주고받는 코드를 작성했다.

- read_write_ble.py

```python
import bluetooth
import random
import struct
import time
from machine import Pin
from ble_advertising import advertising_payload

from micropython import const

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
		
		# 값을 받을 시 출력하도록 함수 설정
    def on_rx(value_handle, v):
        print(value_handle)
        print("RX : ", v)
		
		# Rx 수신 시 이벤트 함수 지정
    p.on_write(on_rx)

    i = 0
    while True:
        if p.is_connected():
		        # 연결되었을 경우 LED를 키며 0부터 카운트를 하나씩 ㄷ
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

### Flutter(Application)

최종적으로 Flutter에서 BLE 통신을 위해 사용한 패키지는 Flutter_blue_plus 이다. MicroPython으로 작성한 BLE 통신코드와 굉장히 유사하며, BLE 통신의 기본인 Scanning, Connection, Subscribe까지 직관적인 API가 장점이었기에 선택하였다.

[공식 문서]

[flutter_blue_plus | Flutter package](https://pub.dev/packages/flutter_blue_plus)

 아래에서 위젯 디자인부터 프로바이더 및 라이브러리 이용을 위한 블루투스 세팅 등의 자세한 코드가 많지만 핵심 코드만 추려서 설명하도록 하겠다.
(현재 코드에서 사용중인 유일한 상태관리 기법은 Provider이며, ChangeNotifier 기반의 SensorDataProvider를 생성하여 사용중이다. 자세한 코드는 깃허브에 업로드 하였음.)

- **Scan**

주변에서 advertise를 수행하는 BLE 기기들을 검색하는 함수이다. startScan시 스캔이 시작하여 실시간으로 검색되는 디바이스들에 대하여 피코 디바이스인지 검사하고, 일치한다면 해당 디바이스 정보를 저장하였다가 connect_device 함수를 이용하여 디바이스와 페어링을 진행한다.

```kotlin
void flutterBlueInit() async {
    // 스캔 결과 listen
    print("스캔 시작");
    var subscription = FlutterBluePlus.onScanResults.listen(
      (results) {
        if (results.isNotEmpty) {
          ScanResult r = results.last; // the most recently found device
          print(r.advertisementData.advName);
          // print(
          //     '${r.device.remoteId}: "${r.advertisementData.advName}" found!');
          if (r.advertisementData.advName == "mpy-uart") {
            //if (r.advertisementData.serviceUuids.contains(Guid.fromString("6e400001-b5a3-f393-e0a9-e50e24dcca9e"))) {
            print(r.advertisementData.serviceUuids[0]);
            print(
                '${r.device.remoteId}: "${r.advertisementData.advName}" found!');
            print("디바이스 : ");
            print(r.device);
            print("광고데이터 : ");
            print(r.advertisementData);
            device = r.device;
            connect_device(device);
          }
        }
      },
      onError: (e) => print(e),
    );

    // 스캔 종료 시 위 listen 종료
    FlutterBluePlus.cancelWhenScanComplete(subscription);

    // 블루투스 활성화 및 권한 부여 테스트
    // In your real app you should use `FlutterBluePlus.adapterState.listen` to handle all states
    await FlutterBluePlus.adapterState
        .where((val) => val == BluetoothAdapterState.on)
        .first;

    // 스캔 시작 및 스캔 끝날때까지 기다리기
    await FlutterBluePlus.startScan(timeout: const Duration(seconds: 7));
    await FlutterBluePlus.isScanning.where((val) => val == false).first;
  }
```

[스캔 성공 사진]

- **Connect**

연결된 서비스에서 캐릭터리스틱을 조사하여 read와 write 플래그의 구분으로 Rx와 Tx 캐릭터리스틱을 구분하는 코드이다. read가 enabled 되어있는 캐릭터리스틱인 경우 피코에서 전송하는 Tx 특성이기 때문에 값을 읽어 저장할 필요가 있는데, 센서값이 주기적으로 들어올것으로 판단하여 해당 값을 구독(subscribe)하여 Notify 기능을 활성화하였다. 해당 기능을 활성화하면 값이 들어올때마다 listen 콜백 함수로 값을 알아서 읽어올 수 있어 해당 값으로 화면의 Text를 업데이트 해주었다. 
해당 텍스트 위젯의 상태 관리 기법은 미리 생성해놓은 SensorDataProvider를 이용하였다.

```kotlin
void connect_device(device) async {
    // listen for disconnection
    var connectSubscription =
        device.connectionState.listen((BluetoothConnectionState state) async {
      if (state == BluetoothConnectionState.disconnected) {
        print("연결 끊김");
      }
    });

    // cleanup: cancel subscription when disconnected
    device.cancelWhenDisconnected(connectSubscription,
        delayed: true, next: false);

    // 해당 디바이스와 연결
    await device.connect().then((result) => print("연결 성공"));

    // 서비스 찾기
    List<BluetoothService> services = await device.discoverServices();
    print("찾은 서비스");
    print(services);
    print("=======================");
    for (var service in services) {
      // 캐릭터리스틱 읽기
      var characteristics = service.characteristics;
      for (BluetoothCharacteristic c in characteristics) {
        print("캐릭터리스틱 : ");
        print(c);
        print("=======================");
        if (c.properties.read) {
          List<int> value = await c.read();
        } else {
          // 쓰기
          write_characteristic = c;
        }

        final valueSubscription = c.onValueReceived.listen((value) {
          print("값 도착 $value");
          Provider.of<SensorDataProvider>(context, listen: true)
              .convertAscii(value);
        });

        // 연결 끊겼을때 subscribe 해제
        device.cancelWhenDisconnected(valueSubscription);

        // subscribe 설정 - Notify
        await c.setNotifyValue(true);
      }
    }
  }
```

```kotlin
Text(
	Provider.of<SensorDataProvider>(context, listen: true)
	    .display_data,
	style: Theme.of(context).textTheme.headlineMedium,
	),
```

[연결 성공 및 서비스, 캐릭터리스틱 출력 사진]

- **write**

반대로 flutter에서 값을 실어 피코쪽으로 보내기 위해선 피코의 Rx 캐릭터리스틱에 write 함수를 사용하여 값을 전송해주어야 한다. 따라서 TextField 위젯으로 입력받은 값을 컨트롤러를 이용하여 전달해주었으며, 해당 과정에서 write로 쓰이는 값은 Int형식을 지켜야하기 때문에 입력받은 숫자 문자열을 파싱하여 숫자로 바꿔주는 전처리 작업을 진행하였다.

```kotlin
TextField(
  controller: inputController,
  decoration: const InputDecoration(
    hintText: 'Enter your data..',
    labelStyle: TextStyle(color: Colors.redAccent),
    focusedBorder: OutlineInputBorder(
      borderRadius: BorderRadius.all(Radius.circular(10.0)),
      borderSide: BorderSide(width: 1, color: Colors.redAccent),
    ),
    enabledBorder: OutlineInputBorder(
      borderRadius: BorderRadius.all(Radius.circular(10.0)),
      borderSide: BorderSide(width: 1, color: Colors.redAccent),
    ),
    border: OutlineInputBorder(
      borderRadius: BorderRadius.all(Radius.circular(10.0)),
    ),
  ),
  keyboardType: TextInputType.emailAddress,
),
const SizedBox(
  height: 10.0,
  width: 30.0,
),
ElevatedButton(
	onPressed: () {
	  setState(() async {
	    print("작성한 값 : ${inputController.text}");
	    int parsedInt = int.parse(inputController.text);
	    // 숫자 int값만 전송가능.
	    await write_characteristic.write([parsedInt]);
	  });
	},
	child: const Text("send"))
```

### 최종 작동

- Flutter 어플리케이션 작동

[피코로부터 데이터 수신하는 화면]

[텍스트필드에 값 입력해서 전달]

- Pico Micropython

[Flutter로 카운트 값 송신하는 화면(출력창)]

[Flutter로부터 수신 받은 값 출력(출력창)]
