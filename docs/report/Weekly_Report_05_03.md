---
layout: default
title: 5월 3주차
parent: Weekly_Report
nav_order: 1
---

# 5월 3주차 Weekly Report
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{: toc}

---

## 주간 작업 내용


{: .note }
5월 3주차 작업내용이다. 팀원별 업무 내용은 다음과 같다.  
박종현 : PPG 센서 테스트 및 가속도 기반 수면단계 분류 실험 진행
이제욱, 이태우 : 가속도 기반 수면자세 분류 및 진동 자극 실험 진행




 
- 05.13(월) 16:30~21:15 : tflite모델 이식 및 테스트 

- 05.14(화) 11:00 ~22:10 : PPG 센서 테스트, 가속도 기반 수면단계 분류 실험, 수면자세 분류, 진동 자극 실험 진행

- 05.17(금) 14:00 ~ 17:05 : 발표 ppt 제작 및 준비

이외 개인 작업으로 자택에서 진행

---


## 기존 PPG 센서 재측정

- **Crowtail PPG**

![](/assets/images/W8/p1.png)

 조교 피드백에 따라 PPG 신호의 불특정성을 극복하고 안정적인 심박수를 얻고자 지난 3주간 측정했던 모든 신호를 분석하며 가장 잘 그려졌던 조건을 기록. 그 중 위 사진처럼 100~200Hz의 신호에 MA Filtering을 적용한 결과가 이상적으로 판단되어 데이터를 재수집하고자 재측정 진행.
(과거에 PPG 센서로 측정을 정말 많이 해봤는데, 측정할때마다 신호가 아예 동일한 값으로 출력되어 파형이 그려지지 않는 경우도 존재했기 때문에 정확한 측정 조건을 다시 결정할 필요가 있었음.)

**하지만 센서 고장으로 이전과 같은 측정이 도저히 되지 않음(손가락 유무 상관없이 동일한 파형 무한 반복). 파형이 지속되는 시간도 3초 이상으로 비 정상적으로 길고, 결정적으로 피크간 거리(RR)은 정상적인 인간 심박수 상 1초보다 작아야하기 때문에 해당 파형을 정상적인 PPG 신호라고 보긴 어렵다.**

→ 센서 연결이 불안정한지 의심스러워 납땜 후에 재 측정 하였으나 마찬가지로 동일한 측정 결과.

![](/assets/images/W8/p2.png)

- **MAX30102 PPG**

![](/assets/images/W8/p3.png)

 여분 PPG인 MAX30102를 이용. 하지만 이전과 마찬가지로 호흡수만 측정됨. 공식 사이트에서 센서 시그널의 안정성을 위해 납땜을 진행하라는 조언을 듣고 납땜을 시도하였으나 **납땜 실수로 가운데 두 개의 핀에 합선이 일어나 연결 불가능 상태.** 미리 주문했던 PPG 센서가 아직 배송중이고 현재 이용가능한 센서가 모두 망가졌기 때문에, 차선책으로 준비했던 가속도 센서 기반 수면단계 판별을 진행하였음.

## 가속도 센서 처리

Raspberry Pi Pico의 온보드 Micropython에서 구동하는 것을 목적으로, 모델 지원 범위 내에서 가벼운 모델을 사용하는 연구를 참고하여 구현하였음.
<가속도 센서 데이터 기반 수면단계 예측 및 수면주기의 추정, 강경우 외 1명>과 <웨어러블 가속도 기기 측정에 의한 수면/비수면 동적 분류, 박재현 외 3명>를 참고하여 구현하였으며 가속도 센서와 랜덤포레스트, 서포트 벡터 머신만을 이용하여 80% 이상의 정확성을 보였음.

 우선 전처리 단계에서는 센서로부터 입력받은 3축 가속도 데이터의 차원 축소 및 잡음, 아웃라이어 (outlier) 제거 과정을 통해 연산량을 줄이면서 수면 및 비수면 신호간의 구분성을 증가시켰음.

- **운동강도 변환**

![](/assets/images/W8/p4.png)

수면 및 비수면 상태 구분에는 사용자의 운동 세기와 운동 빈도를 고려할 뿐 방향성은 필요하지 않은 것은 합리적으로 보인다. 따라서 3축의 가속도 신호를 종합하여 1차원의 Intensity 값으로 변환하면 분류에 필요한 정보를 보존하면서 연산량을 줄일 수 있다. 이 방식은 위 수식으로 나타낼 수 있다.

- **BandPass Filtering(2~3.5Hz)**

![](/assets/images/W8/p5.png)

가속도 센서 자체에도 노이즈가 존재하기 때문에 1차원으로 변환된 움직임 값 I는 2~3.5Hz의 대역통과 필터를 통해 잡음을 제거하게 됨.

### **Sampling 방법**

 학습에 이용한 데이터는 PhysioNet에서 제공하는 가속도 - 수면단계 데이터셋을 사용하였다. 데이터 구조는 아래와 같으며, 가속도 시그널은 64Hz (0.015625초)의 샘플링 주파수로 기록된 반면에, 수면단계는 30초마다 기록이 되며 이후 새로 기록이 될 때까진 이전 수면단계가 동일하게 기록된 구조를 지닌다. 이 모든 데이터를 사용하기엔 양이 너무나 많고, 모델 학습을 위해 적절한 크기와 합리적인 데이터 샘플링을 위해서 아래와 같은 방법들을 시도하였다.

![](/assets/images/W8/p6.png)

1. **초당 데이터 샘플링**

 가장 먼저 일정 시간(1초)마다 가속도 데이터를 샘플링하였다. 수면단계 기록 주기가 30초로 매우 느리기 때문에 수면 단계에 맞춰서 가속도 데이터를 샘플링하였는데, 0.015624초마다 들어와 30초 동안 1920개가 기록되는 가속도 데이터에서 1초마다 1개씩 뽑아 총 30개를 추출하고 이때의 수면단계를 라벨로 엮어주었다.

![](/assets/images/W8/p7.png)

- **(추가) 가속도와 수면단계를 매핑한 방법**
    
    데이터 샘플링 및 학습 데이터 생성 시 가속도 데이터와 수면단계를 매핑하는 과정을 간단히 두가지로 나누어서 진행했음. 아래는 예시 사진으로, 특정 수면단계 발생 후에 지속된 가속도 데이터를 매핑해주거나, 특정 움직임(가속도) 후에 발생한 수면단계를 매핑한 것으로, 라벨링에 약간의 차이가 존재한다. 보다 실시간성이 반영된 수면단계 판별을 위해선 과거의 가속도를 보고 현재의 수면단계를 판별하는 2번 방식이 합리적이라 판단하였으나, 실 운용에서는 두 가지 경우를 모두 구현하여 실험을 진행하였다. (실제로는 모델 성능에 큰 차이가 없었음.)
    
    1. 특정 수면단계 발생 후에 지속된 가속도 데이터를 매핑
    
    ![](/assets/images/W8/p8.png)
    
    1. 특정 움직임(가속도) 후에 발생한 수면단계를 매핑한 것으로
    
    ![](/assets/images/W8/p9.png)
    

2. **운동강도 변화량(Intensity Difference) 기반 샘플링**

$$
ID_n = (I_n - I_{n-1})^2
$$

 위 논문에서 입력으로 사용한 가속도 벡터(Intensity)는 가속도 센서가 X,Y,Z축으로 받는 중력 가속도의 크기에 비례하지만 그 자체의 값 뿐만 아니라 가속도 벡터의 변화량도 중요한 특징일 것이라 생각했다. 이에 64Hz로 측정되는 Raw Data에서 위 1번에 해당하는 초당 데이터 샘플링을 수행한뒤, 샘플링된 가속도 데이터에서 가속도 벡터를 계산한 후에 인접한 Time point들 간의 크기 차이를 구하여 이를 모델의 입력으로 사용하였다.
(결과적으로 성능이 오히려 떨어졌음..)

3. **주기별 운동강도 최대값 샘플링**

 초당 샘플링과 유사하게 30초 이내에서 일정 주기로 데이터를 샘플링하는 것은 동일한 구조를 지닌다. 다만 2초 간격으로 가장 큰 Intensity를 지니는 가속도 벡터만을 30초 동안 추출하여 총 15개의 데이터를 샘플링한다. 해당 방법은 3축 가속도를 사용하여 일정시간마다 사용자의 움직임을 감지하기 위해 가속도 벡터의 크기가 일정 임계값을 넘는 구간을 구하거나, 일정 시간마다 가장 큰 가속도 벡터를 구하여 신체 움직임을 판단하는 논리와 동일하다. (실제로 1번의 간단한 초당 샘플링 방법보다 성능이 약간 상승하였음.)

- **실험 결과**

|  | Random Forest | SVM | KNN | Logistic Regression | Decision Tree |
| --- | --- | --- | --- | --- | --- |
| Max Accuracy | 0.89 | 0.87 | 0.83 | 0.84 | 0.83 |
| Average Accuracy | 0.61 | 0.58 | 0.55 | 0.58 | 0.46 |

→ 되돌아보면 모델 학습이 원활히 이루어지지 않은 것으로 보인다. 학습이 진행되는 와중에 모델 정확도 그래프가 오락가락하는 모습이 많이 보여졌기 때문에, 데이터 전처리 코드에 실수가 있거나, 모델 학습 부분에 문제가 있을 수도(사실 이 부분은 아니라고 확신한다. 머신러닝 모델 학습 코드가 매우 간단하기 때문에 학습 파트의 코드에는 오류가 없을 거라 생각함), 혹은 입력 데이터의 특징이 부족하기 때문으로 보인다. 정확한 학습을 보장하기 위해 아래 방법들을 시도 중이다.

### 문제 분석

1. **데이터셋의 라벨링 문제**

![](/assets/images/W8/p10.png)

 **학습에 사용한 수면단계 데이터의 분포가 비정상적임.** 일반적인 수면 비율을 고려할 때 N1, N2 수면단계는 30~40% 가량, REM은 30%, N3은 약 10% 정도를 차지하지만, 학습에 이용한 데이터에는 N1이 아예 없거나, REM이 없는 경우도 존재하는 매우 비정상적인 데이터가 다수 발견됨. 
(위 사진의 경우에는 N2와 Wake가 지나치게 많고, N3가 존재하지 않는다.)

 따라서 우리 디바이스에서는 작동/비작동 구간만 판별하면 되기에 라벨링 재설정이 가능하고, 성능 향상을 기대할 수 있음.
 -> 실제로 다음과 같이 라벨링을 변경해서 실험했을 때 모델 정확도가 상승하였음.   
    - 0: Wake, REM / 1: N1, N2, N3  => **68%(8%p 상승)**



2. **새로운 특징 추출**

 입력 특징 추출을 위한 연구로 <웨어러블 가속도 기기 측정에 의한 수면/비수면 동적 분류, 박재현 외 3명>를 서베이하여 이를 참고하였음. 아래에서 소개하는 새로운 3가지의 특징은 본 논문에서 소개하는 가속도 데이터에서의 추출가능한 특징들이며, 현재 이를 구현 중이나 아직 코드가 완성되지 않아 차주 보고서에 즉시 싣도록 하겠다.

{: .note }
**PIM(Proportional Integration Mode)**   
 : Intensity 값이 입력되었을 때 epoch 구간에 대한 넓이를 계산하여 얻는 값이며, 움직임의 크기 정도를 의미한다.
**ZCM(Zero Crossing Mode)**   
 : Epoch 구간에서 가속도 파형이 0의 값을 가로 지른 횟수를 나타내며 움직임의 빈도를 표현한다.
**FFT(Fast Fourier Transform)**   
 : Intensity 값을 주파수 영역으로 변환한 특징으로 단일 epoch 에서 나타나는 움직임 패턴의 구성을 표현하는데 적합한 특징이다. 일반적인 사람의 움직임은 2초 이내에서 발생하기 때문에 FFT window는 2초 (200 샘플) 로 설정한다. 
→ 본 논문에서는 100Hz 샘플링 주파수를 지니기 때문에 200샘플이지만, 우리 데이터의 경우에는 64Hz의 샘플링 주파수를 지니기 때문에 128 샘플로 변경하였다.

3. **모델 변경**

 온보드 환경을 고려하여 MicroPython이 제공하는 머신러닝 모델만 사용하였지만, Flutter에 모델을 탑재가 가능하기 때문에 보다 무거운 딥러닝 모델도 탑재가 가능함. 추가적인 실험을 진행할 경우 성능 향상을 기대할 수 있음.

---

## **Flutter에 TFLite 모델 이식하기**

이 과정은 [flutter-tflite GitHub](https://github.com/tensorflow/flutter-tflite) 레포지토리를 참조하였다.

### **종속성 설정**

먼저, **`pubspec.yaml`** 파일에 **`tflite_flutter`** 종속성을 추가하고, pre-trainning 된 모델 파일을 저장할 **`assets`** 폴더를 생성한다.

```yaml
yaml코드 복사
dependencies:
  flutter:
    sdk: flutter
  tflite_flutter: ^0.9.0  # 최신 버전 확인 필요

assets:
   - assets/

```

### **모델 이식**

모델은 x, y, z 위치 데이터를 받아서 이를 수면 자세로 변환하는 모델이다. 현재는 가속도 센서 값을 받지 않으므로 텍스트 입력을 통해 데이터를 읽어와서 변환한다.

### **Flutter 코드**

아래 코드는 텍스트 입력을 받아 TFLite 모델을 통해 수면 자세를 분류하는 Flutter app의 demo이다. 모델을 불러오기전에 임의의 텍스트 입력을 받아 TFLite 모델로 추론을 하여 이식된 모델이 제대로 테스트가 되는지 확인하기 위하여 demo앱을 만들어서 테스트를 해보았다.

```dart
dart코드 복사
import 'package:flutter/material.dart';
import 'package:tflite_flutter/tflite_flutter.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'TFLite Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: MyHomePage(),
    );
  }
}

class MyHomePage extends StatefulWidget {
  @override
  _MyAppState createState() => _MyAppState();
}

class _MyAppState extends State<MyHomePage> {
  late TextEditingController _controller1;
  late TextEditingController _controller2;
  late TextEditingController _controller3;
  late Interpreter _interpreter;
  List<double> _output = [0, 0, 0, 0];

  @override
  void initState() {
    super.initState();
    _controller1 = TextEditingController();
    _controller2 = TextEditingController();
    _controller3 = TextEditingController();
    _loadModel();
  }

  void _loadModel() async {
    _interpreter = await Interpreter.fromAsset('assets/model.tflite');
  }

  void classify() {
    List<double> input = [
      double.tryParse(_controller1.text) ?? 0.0,
      double.tryParse(_controller2.text) ?? 0.0,
      double.tryParse(_controller3.text) ?? 0.0
    ];

    var output = List<double>.filled(4, 0).reshape([1, 4]);
    _interpreter.run([input], output);
    setState(() {
      _output = output[0];
    });
  }

  @override
  void dispose() {
    _interpreter.close();
    _controller1.dispose();
    _controller2.dispose();
    _controller3.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('TFLite Demo'),
      ),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          children: [
            TextField(
              controller: _controller1,
              decoration: InputDecoration(labelText: 'X Position'),
              keyboardType: TextInputType.number,
            ),
            TextField(
              controller: _controller2,
              decoration: InputDecoration(labelText: 'Y Position'),
              keyboardType: TextInputType.number,
            ),
            TextField(
              controller: _controller3,
              decoration: InputDecoration(labelText: 'Z Position'),
              keyboardType: TextInputType.number,
            ),
            SizedBox(height: 20),
            ElevatedButton(
    dw          onPressed: classify,
              child: Text('Classify'),
            ),
            SizedBox(height: 20),
            Text('Output: $_output'),
          ],
        ),
      ),
    );
  }
}

```

1. **의존성 설정**:
    - **`pubspec.yaml`** 파일에 **`tflite_flutter`** 패키지를 추가하여 TFLite를 사용할 수 있도록 한다.
2. **모델 로드**:
    - **`assets`** 폴더에 TFLite 모델 파일(**`model.tflite`**)을 추가한다.
    - **`_loadModel`**  method를 통해 모델을 불러온다.
3. **input 및 output**:
    - **`TextEditingController`**를 사용하여 사용자의 입력(x, y, z 위치)을 임의값으로 입력받는다.
    - **`classify`** method를 통해 입력 값을 모델에 전달하고, 모델의 출력을 받아온다.
    - **`setState`**를 호출하여 모델의 출력 결과를 반환한다.
4. UI화면

![](/assets/images/W8/p11.jpeg)

![](/assets/images/W8/p12.jpeg)

text로 입력받은 결과 정자세일때 잘 classfier 하는 것을 확인 해 볼 수 있다. 이후 가속도 센서의 값을 input으로 받아 온 후 센서 데이터 값으로 모델을 추론할 계획이다.

---

## 수면자세 판별

### 피코에서 가속도 보내고 플러터에서 double 값으로 변환

가속도 value를 각으로 변환한 뒤 str의 형태로 ‘,’를 구분자로 하여 플러터 앱으로 보내준다.

- 피코 코드

```python
# 가속도 읽기
acceleration = imu.accel
gyrometre = imu.gyro

acx = acceleration.x
acy = acceleration.y
acz = acceleration.z

angle_acx = atan2(acy,sqrt(pow(acx,2) + pow(acz,2)))
angle_acx = degrees(angle_acx)
angle_acy = atan2(-acx,sqrt(pow(acx,2) + pow(acz,2)))
angle_acy = degrees(angle_acy)
angle_acz = atan2(acz,sqrt(pow(acx,2) + pow(acy,2)))
angle_acz = degrees(angle_acz)

print("각 x: ", round(angle_acx,2), " y:", round(angle_acy,2) , " z:", round(angle_acz,2) )
data = str(round(angle_acx,2)) + ',' + str(round(angle_acy,2)) + ',' + str(round(angle_acz,2))
p.send(data)
x += 1 
utime.sleep(1)
```

- 플러터 코드

String을 double형 list에 분할해서 넣어준다. 

[X축 데이터, Y축 데이터, Z축 데이터] 가 되게끔 한다.

```python
void convertAscii(asciiSignals) {
    display_data = "";

    for (int asciiValue in asciiSignals) {
      display_data += String.fromCharCode(asciiValue);
    }
    input = display_data
        .split(',')
        .map((signal) => double.parse(signal.trim()))
        .toList();
    print('변환된 시그널: $display_data');
    print(input);
    notifyListeners(); // 상태 변경 알림
  }
```

### 모델 플러터에 싣기

- 모델 불러오는 코드

```python
// 모델 이름
  final _modelFile = 'assets/sleep_position_model.tflite';

  late Interpreter _interpreter;
  
  @override
  void initState() {
    super.initState();

    // 권한 및 초기 설정 세팅
    // 모델 불러오기
    flutterBlueSettings();
    _loadModel();
    // // 앱 시작 스캔
    // flutterBlueInit();
  }
  
  void _loadModel() async {
    // 모델 생성
    _interpreter = await Interpreter.fromAsset(_modelFile);
    print('Interpreter loaded successfully');
  }
  
  
```

- 모델 실행

가속도 값이 도착하면 double형의 list로 변환한 뒤 2차원의 형태로 (1,3)의 크기로 넣어준다. 

output도 마찬가지로 2차원의 형태로 (1,4)의 형태로 만들어주며 이는 TFLite 모델로 변환될 때 설정된 것이다. 앱 환경상 단일 입력만 판단할 수 있도록 파라미터 수를 최대한 줄인 것이다. 

나온 추론 값을 argmax 함수를 통해 하나의 class로 확인할 수 있게끔 변환해주고 수면 자세를 기반으로 진동모터를 제어하는 함수를 호출한다. 

```python
final valueSubscription = c.onValueReceived.listen((value) {
          print("값 도착 $value");
          Provider.of<SensorDataProvider>(context, listen: false)
              .convertAscii(value);
          var output = List<double>.filled(4, 0.0).reshape([1, 4]);
          _interpreter.run(
              [Provider.of<SensorDataProvider>(context, listen: false).input],
              output);

          setState(() {
            sleep_position = _output[argmax(output[0])];
            vib();
            former_sleep_pos = sleep_position;
          });
        });
        
int argmax(List<double> data) {
  double maxValue = double.negativeInfinity;
  int maxIndex = -1;

  // 이중 List를 순회하며 최대값과 그 인덱스를 찾습니다.
  for (int i = 0; i < data.length; i++) {
    if (data[i] > maxValue) {
      maxValue = data[i];
      maxIndex = i;
    }
  }

  return maxIndex;
}
```

### 결과 출력

자세별 결과이다. 들어오는 가속도 데이터에 대해서 잘 분류하는 것을 확인할 수 있다.

![](/assets/images/W8/supine.jpg)


![](/assets/images/W8/left.jpg)


![](/assets/images/W8/right.jpg)


![](/assets/images/W8/prone.jpg)

### 수면자세별 진동 전달

계획한 대로 왼쪽 자세나 정자세로만 자게끔 오른쪽 자세이거나 뒤집힌 자세일 때 진동 모터를 컨트롤하는 라즈베리파이 피코에 진동을 하라고 신호를 보낸다. 

보내는 신호의 형태는 0~8의 인덱스 값으로 String의 형태로 치환된 것이다. 피코에서 받았을 때 이 값이 16진수의 형태로 들어오기에 적절한 처리 함수를 통해 integer 값으로 변환한 뒤 pwm 방식으로 진동 모터를 제어한다. 

**진동 전달 알고리즘**

1. 진동을 전달할 떄 0~8의 사이로 9단계로 나눠 자세의 변화가 없을 시 진동의 세기를 강화한다.
    
    진동의 세기는 다음과 같다. [0,15000,19662,26216,32770,39324,45878,52432,58986]
    
2. 진동을 받는 중 수면 단계가 악화된다면 예를 들어 깊은 수면에서 얕은 수면으로 변화한다면 진동의 세기를 절반으로 낮추고 자세가 변화할 때까지 계속 진동을 전달한다. 1번으로 돌아간다.
3. 자세가 적절한 수면 자세로 바뀌었다면 모든 flag 및 컨트롤 용 cnt 변수들을 초기화하고 진동 전달을 중지한다.

적절하지 못한 자세일 때마다 바뀔 떄까지 계속한다. 

2번 알고리즘은 수면 단계 파악이 완료되는 순간 적용한다. 현재는 1번 3번만 구현하였다.

- 플러터 진동 알고리즘 코드

```python
void vib() async {
    // 오른쪽이거나 뒤집힌 자세일 때
    // 진동을 가한다.
    if (sleep_position == _output[2] || sleep_position == _output[3]) {
      // 이전 수면자세와 같을 시 즉 진동에도 변화가 없을 시
      // cnt 25를 기준으로 진동 레벨을 강화 시킨다.
      // 1초마다 가속도 데이터가 들어오기에 5초가 지난 것 5초동안 진동에도 변화가 없었던 것
      if (former_sleep_pos == _output[2] || former_sleep_pos == _output[3]) {
        cnt++;
        if (cnt == 5 && now_vib_level < vib_level.length) {
          now_vib_level += 1;
          cnt = 0;
        }
      }
      // 이전 수면자세가 올바른 자세였다면
      // 초기 세팅으로 진동 전달 시작
      else {
        now_vib_level = 1;
        cnt = 0;
      }
      await write_characteristic.write([vib_level[now_vib_level]]);
    }

    // 올바른 자세라면 진동을 주지 않는다.
    else {
      now_vib_level = 0;
      cnt = 0;
      // 진동을 주지 않음에도 보내는 이유는 계속 진동 모터가 작동중이기 때문이다.
      if (former_sleep_pos == _output[2] || former_sleep_pos == _output[3]) {
        await write_characteristic.write([0]);
      }
    }
  }
```

- 라즈베리파이 피코 코드

직접 테스트하였을 때 15000의 레벨부터 진동이 느껴졌기에 vib_level을 다음과 같이 설정하였다. 

60000대의 레벨은 잠이 깰 가능성이 다분하기에 제외하였고 4~50000대도 가능성이 존재하지만 현재는 이렇게 설정하였다. 

후에 납땜을 진행하고 실제 착용하여 진동이 어느정도로 다가오는지 판단한 후에 적정 진동 레벨을 찾아나갈 것이다.

```python
# 모터를 연결한 GPIO 핀 번호 설정 및 PWM 인스턴스 생성
vibration_motor = PWM(Pin(15))
vibration_motor.freq(1000)  # PWM 주파수 설정 (예: 1000Hz)

# 직접 테스트했을 때 15000부터 직접적인 떨림이 느껴졌음
# 65535은 너무 과도하다 사실 50000대부터 깨는 위험이 충분히 있지만 현재는 이렇게 설정
vib_level = [0,15000,19662,26216,32770,39324,45878,52432,58986]
# 값을 받을 시 출력하도록 함수 설정
  def on_rx(value_handle, v):
	    print("Received Data: ", v)
	    # 바이트 데이터를 정수로 직접 변환
	    intensity = int.from_bytes(v, 'little')  # 'little' 또는 'big' 엔디안에 따라 선택
	    vibration_motor.duty_u16(vib_level[intensity])  # 진동 세기 설정
	    print("Vibration intensity set to:", vib_level[intensity])
```

- 피코 들어오는 결과 출력

초기에 정자세 각도가 들어올 때는 진동이 없고 오른쪽으로 자세가 변경된 것을 판별하자 진동 레벨을 15000으로 하여 진동을 주기 시작한 것을 확인할 수 있다. 플러터에서는 1의 index로 오는 것을 확인할 수 있다.

또 5초가 지났을 때 진동레벨을 올려 19662의 진동을 주고 또 플러터에서 2의 index로 오는 것을 확인할 수 있다.

이렇게 전달되는 index들은 위에 선언한 진동 레벨에 매핑된다. 

![](/assets/images/W8/p13.png)

---

## To do

- **가속도 데이터의 특징 추출 코드 완성 및 모델 성능 개선(박종현)**
→ 모델 학습을 위한 코랩 추가 결제 여부 결정.
(현재 코랩에서 5만원어치의 크레딧을 다 써버렸음. 과방에 있는 PC를 사용하려면 학교에 가야하는데, 전부 집이 멀어서 PC 쓰려고 학교 오는데 들어가는 교통비+식비 감안하면 코랩 크레딧 결제가 더 싸게 먹힐 수 있음. 월요일(20일)에 회의 진행 예정.
- **흉골 케이스 제작 및 납땜, 흉골 디바이스 최종 하드웨어 제작 완료(이제욱)**
- **신규 수령 예정 PPG 센서 테스트 및 정상작동 여부 확인(이태우)**
→ 정상 작동시 가속도 센서 기반 수면단계 판별과 PPG 기반 수면단계 판별 결정 회의 진행.

<aside>
💡 흉골 디바이스는 납땜만 하면 실운용이 가능하고, 문제가 되는건 손목 디바이스지만 아무리 늦어도 이번달 이내에는 최종 디바이스를 완성해서 6월 14일 전까지 실제 수면간에 자세가 개선되었는지 실험을 진행할 것.

</aside>
