---
layout: default
title: 6월 2주차
parent: Weekly_Report
nav_order: 1
---

# 6월 2주차 Weekly Report
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{: toc}

---

## 주간 작업 내용

{: .note }
6월 2주차 작업내용이다. 팀원별 업무 내용은 다음과 같다.  
박종현 : 하드웨어 납땜 및 모듈 조립 / 수면자세 통계분석 및 정제 / 수면 측정

이제욱 : 하드웨어 납땜 및 모듈 조립 / 2 Pico - Flutter 비동기 어플리케이션 구축 / 디바이스 전력 소모량 감소 방안 고안

이태우 : MobileNet 및 딥러닝 수면단계 분류실험 / 전처리 및 특징 추출 연구



- 오프라인 작업시간

6월 3일(월) : 14:30 ~ 22:50 (8시간 20분)

6월 4일(화) : 14:30 ~ 21:30 (7시간)

6월 5일(수) : 15:00 ~ 19:30 (4시간 30분)

6월 7일(금) : 11:00 ~ 21:50 (10시간 50분)

---

## 하드웨어

### 흉골 디바이스 분해 및 재제작

기존 디바이스의 사이즈, 특히 두께가 매우 아쉬워 모두 분해하여 더욱 컴팩트하게 납땜 및 조립을 진행하였다. 두께가 약 0.7cm로 크게 줄였으며, 디바이스의 전체적인 폭 또한 줄어들었다.

![Untitled](/assets/images/W62/p1.png)

![기존 케이스(좌측)과 신규 케이스(우측)의 사이즈 비교.](/assets/images/W62/p2.png)

기존 케이스(좌측)과 신규 케이스(우측)의 사이즈 비교.

![손과 사이즈 비교 & 실착 시 사이즈](/assets/images/W62/p3.png)

손과 사이즈 비교 & 실착 시 사이즈

### 손목 디바이스

![Untitled](/assets/images/W62/p4.png)

 이전에 납땜이 완료된 손목 모듈과 완성된 케이스를 조립하여 착용한 결과이다. 실제 스포츠밴드의 스트랩을 이용하였기에 착용에 불편함은 없었다.

---

## 수면자세 분석

 지난 5월 30일부터 6월 5일까지 평범한 사용자의 수면자세를 측정하였다. 결과는 아래와 같으며, 6월 5일 이후부터는 진동자극을 통해 수면자세 개선을 진행하여 변화한 수면자세 비율을 측정하였다. 기존에 AWS EC2와 RDS로 서버를 열어 DB에 저장하였으나, 요금이 너무 많이 나와(52달러) 박종현 팀장의 PC에서 포트포워딩으로 중개서버를 열어 로컬호스트 DB에 저장하였다.

![Untitled](/assets/images/W62/p5.png)

### 수면자세 분석(진동 자극X)

- **24.05.30 → 05.31 수면 (수면시간 6시간 10분)**
    
    정자세 21%, 좌측 수면자세 36%, 우측 수면자세 43%
    
- **24.05.31 → 06.01 수면 (수면시간 6시간 40분)**
    
    정자세 49%, 좌측 수면자세 22%, 우측 수면자세 28%, 뒤집어진 자세 약 1%.
    
- **24.06.01 → 06.02 수면 (수면시간 3시간 40분)**
    
    정자세 42%, 좌측 수면자세 25%, 우측 수면자세 33%
    
- **24.06.02 → 06.03 수면 (수면시간 7시간 50분)**
    
    정자세 46%, 좌측수면자세 19%, 우측수면자세 34%, 뒤집어진 자세 약 1%
    
- **24.06.03 → 06.04 수면 (수면시간 6시간 40분)**
    
    정자세 51%, 좌측수면자세 20%, 우측수면자세 27%, 뒤집어진 자세 약 2%
    
- **24.06.04 → 06.05 수면 (수면시간 7시간 20분)**
    
    정자세 36%, 좌측수면자세 34%, 우측수면자세 24%, 뒤집어진 자세 약 5%
    

> 전체 수면시간은 총 2300분이며, 각 수면자세의 비율은 다음과 같음.
> 
- **정자세 (spine)**: 41.07%
- **좌측 수면자세 (left)**: 25.87%
- **우측 수면자세 (right)**: 31.18%
- **뒤집어진 자세 (prone)**: 1.68%

### 수면자세 분석(진동 자극O)

- **24.06.05 → 06.06 수면 (수면시간 7시간 20분)**
    
    정자세 59%, 좌측 수면자세 36%, 우측 수면자세 3%, 뒤집어진 자세 약 1%
    
- **24.06.06 → 06.07 수면 (수면시간 6시간 50분)**
    
    정자세 62%, 좌측수면자세 34%, 우측수면자세 4%
    
- **24.06.07 → 06.08 수면 (수면시간 4시간 40분)**
    
    정자세 55%, 좌측수면자세 40%, 우측수면자세 5%
    
- **24.06.08 → 06.09 수면 (수면시간 3시간 50분)**
    
    정자세 63%, 좌측수면자세 35%, 우측수면자세 2%
    

> 전체 수면시간은 총 1320분이며, 각 수면자세의 비율은 다음과 같음.
> 
- **정자세 (spine)**: 61.36%
- **좌측 수면자세 (left)**: 37.09%
- **우측 수면자세 (right)**: 3.64%
- **뒤집어진 자세 (prone)**: 0.18%

<aside>
💡 현재 진동자극은 수면단계 반영없이 순전히 수면자세에 따라 자극을 가한 것이며, 6월 9일 밤부터 시행하는 수면측정에는 수면단계가 반영된 수면자세 개선 결과를 기록할 것이다.

</aside>

---

## 흉골, 손목 디바이스 <-> Flutter 앱

### 비동기 처리 및 코드

라즈베리파이 피코와 플러터 앱을 연결하는 과정 그리고 플러터 내에서 데이터를 전처리하고 추론하는 과정 모두 비동기 처리가 굉장히 중요하다. 

피코로부터 값을 가져오는 과정은 subscription 기반으로 처리되고 64hz라는 빠른 속도로 가속도 센서의 값이 전달된다. 만약 추론이나 전처리 과정이 채 끝나기 전에 값이 도달한다면 앱은 정상적인 작동을 하지 못할 수 있다. 따라서 대부분의 과정은 비동기로 처리된다.  

실사용된 코드는 다음과 같다. 

```python
void flutterBlueInit() async {
    await flutterBlueInit1();
    await flutterBlueInit2(); // 위 함수와 흡사하기에 보고서에는 담지 않겠다.
  }
  
  Future<void> flutterBlueInit1() async {
    // 스캔 결과 listen
    var subscription = FlutterBluePlus.onScanResults.listen(
      (results) {
        if (results.isNotEmpty) {
          ScanResult r = results.last; // the most recently found device
          device1 = r.device;
          connect_device(device1, 1);
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
    await FlutterBluePlus.startScan(
        timeout: const Duration(seconds: 7),
        withServices: [Guid("6e400001-b5a3-f393-e0a9-e50e24dcca9e")]);
    await FlutterBluePlus.isScanning.where((val) => val == false).first;
  }
```

```python
Future<void> run() async {
    await Provider.of<SensorDataProvider>(context, listen: false)
        .processQueue();
    var output = List<double>.filled(5, 0.0).reshape([1, 5]);
    print("chrun start");
    _interpreter.run(
        [Provider.of<SensorDataProvider>(context, listen: false).queue],
        output);
    print(output[0]);
    // 초기화
    Provider.of<SensorDataProvider>(context, listen: false).queue = [];

    setState(() {
      running_cnt = running_cnt + 1;
      sleep_class = _output[argmax(output[0])];
    });
  }
```

![Untitled](/assets/images/W62/p6.png)

### 앱 화면 분석

디버깅 혹은 작동 테스트를 하면서 현재 앱이 정상 작동중인지 로직에 문제가 없는지 판단을 위해 다음과 같은 숫자 혹은 문자열을 앱에 출력하였다.

- **수면 단계 queue length**(1277)
    
    1277이란 수는 어떤 특별한 의미를 가지고 있는 것이 아닌 현재 수면 단계 판별을 위한 queue에 어느 정도 수가 찾느냐를 나타낸다. 
    
    해당 수가 **1920**에 달하면 라즈베리파이 피코로부터 송신되는 **enmo**(intensity) 값과 **timestamp**를 통해 lids 값을 구해낸다. 
    
- **흉골 가속도 센서 각도 값**(2.47, 1.47, 87.13)
    
    마찬가지로 해당 수는 특별한 의미는 없고 현재 흉골 디바이스로부터 들어오는 가속도 센서의 각도 값을 나타낸다.  0.4초마다 라즈베리파이 피코로부터 송신된다.
    
- **수면 단계**(sleep class is wake)
    
    ['wake', 'n1', 'n2', 'n3', 'rem'] 다음과 같은 5개의 라벨링 값으로 큐에 1920개의 데이터가 모두 도착했을 때 추론을 시작하고 결과값을 보여준다. 
    
- **수면 자세**(sleep position is supine)
    
    흉골 가속도 센서 각도 값으로 수면 자세 모델로 추론하고 결과 자세를 보여준다. 라벨은 다음과 같다.  ['supine', 'left', 'right', 'prone']
    
- **블루투스 연결**(connected1, connected2)
    
    플러터와 라즈베리파이 피코가 블루투스 연결이 됐는지를 나타낸다. 연결되면 true 끊기면 false로 변한다. 
    
- **수면 단계 추론 횟수**(10)
    
    수면 자세 모델과 달리 수면 단계 모델은 추론하는데 오래 걸리기에 디버깅 시에는 현재 몇번이나 추론이 됐는지 확인이 필요했다. 최종 앱에는 삭제될 것으로 판단한다.
    

### 결과

두 디바이스로부터 값이 문제없이 도달하는 것을 확인하였고, 플러터에 실린 모델 또한 정상 작동하는 것을 확인하였다. 

수면 단계와 수면 자세를 연동하여 진동 모터가 정상 작동하는 것 또한 확인하였다. 

---

## 디바이스 전력 소모량 감소

평균 수면 시간인 6~7시간이고 라즈베리파이 피코는 평균적으로 50mah를 소비하기에 이를 맞추기 위해 350mAh의 배터리를 사용하였다. 

하지만 실제로 측정시 4~5시간만에 꺼지는 문제가 발생하였고 이를 해결하기 위해 다음과 같은 방안들을 적용하였다.

### 라즈베리파이 피코 cpu 언더클럭

라즈베리파이 피코의 default 클럭은 125mhz이다. 

최적의 언더클럭 지점을 찾기위해 한계 시험을 진행하였고, 65mhz가 최대로 줄일 수 있는 한계지점임을 알았다. 

만약 클럭이 더 낮아지게 되면 블루투스 연결이 되지않아 디바이스의 사용이 불가해진다. 

결론적으로 거의 2배 가까이 cpu의 클럭을 줄였고 정상 작동하였다. 

### 사용하지 않는 pin off

라즈베리파이 피코의 pin들은 총 30개 있고 우리가 이 중에서 실질적으로 사용하는 핀은 5~6개 뿐이다. 6~7시간동안 사용하지 않는 핀들을 계속 켜둔다면 전력 낭비가 될 수 있기에 꺼주는 작업을 실시하였다. 

| GPIO29 | 물리 시스템 전압(VSYS)을 측정하기 위한 내부 ADC (ADC3)  |
| --- | --- |
| GPIO24 | VBUS 존재 파악 |
| GPIO23 | 온보드 SMPS (스위칭 전력 모드 공급 장치) 제어 |
| GPIO17 | 가속도 센서 SCL |
| GPIO16 | 가속도 센서 SDA |
| GPIO 26 | 진동 모터 PWM |

### 결과

6~7시간동안 디바이스들이 꺼지지 않고 정상 작동하는 것을 확인하였다.

---

# 가속도 기반 수면단계 분류

 가속도 센서를 이용해 수면단계를 판별하는 과정에서, 정확도를 산출하는 과정에서 많은 어려움이 있었다. 전처리와 특징 추출을 변화시키고 Random Forest, SVM, DT 등의 머신러닝과 Dense, LSTM, GRU, CNN 등의 딥러닝 모델들을 적용하며 정확도를 개선하는 실험을 진행하였으나, 성능을 끌어올리기가 여간 쉽지 않았다. 특히 Flutter에 학습 모델을 실어야 하기 때문에 최근 시계열 분야에서 압도적인 성능을 보이고 있는 트랜스포머 모델을 도입하지 못해 아쉬움이 있었는데, 이번에 Flutter에 tflite 모델을 올리는 과정에서 LSTN, GRU 등 RNN 모델들이 실리지 않는 상황이 발생하여 기존에 실험했던 RNN 모델들을 사용하지 못하게 되었다.

 따라서 다음과 같은 작업에 집중하였다.

- **가속도와 수면단계의 연관성 파악 및 통계 분석 → 경험적 상수값 도출**
- **가속도 전처리 및 특징 추출 실험**
- **MobileNetV3-small 등 CNN 기반의 새로운 모델 실험**

## 데이터 통계 분석

![Untitled](/assets/images/W62/p7.png)

 운동강도와 수면단계 사이의 구체적인 상관성 측정을 위해 데이터셋에서 실험자 별로 그래프 분석과 통계치 분석을 진행했다. 운동강도가 높은 경우 수면단계가 얕아지며 WAKE상태가 되는 경우가 많았다. 하지만 다음과 같은 문제가 존재했다.

1. **운동이 수면단계보다 후행함.**
2. **지저분한 수면단계의 변화.** 
    
    WAKE 상태에서도 가속도의 명확한 변화가 없음에도 다른 수면단계로 변화하는 경우가 굉장히 잦고, 가장 큰 문제가 REM 수면의 명확한 구분이 힘듬.
    
3. **수면단계의 통계치가 매우 상이함.**
    
    비록 후행성이 존재하지만, 얕은 수면(WAKE, REM)에서 운동강도가 강한 경우가 많아 이를 참고하기 가장 좋은 지표가 수면단계 별 운동강도의 표준편차로 보임. 앞서 말했듯 WAKE상태에는 운동강도의 표준편차가 매우 크지만, 깊은 수면(N1~3)과 REM 수면의 표준편차가 실험자마다 매우 상이한 것을 알수 있다. 특정 환자는 REM수면의 표준편차가 깊은수면보다 크거나, 다른 환자는 작은 경우도 존재한다.
    

실험자마다 수면장애 여부가 다르고, 심지어 사람마다 수면시 움직의 크기가 다르기 때문에 보편적인 통계특성을 도출하기 힘들 것으로 판단함.

### 데이터 정제

 위처럼 수면장애의 여부에 따라 수면단계와 가속도의 상관관계가 다양해지만, 데이터 자체에 라벨링 문제가 있는 경우가 존재한다. 현재 우리가 사용하고 있는 데이터는 지난 4월 PhysioNet에 업로드된 120명 가량의 실험자에 대한 PSG(수면다원검사) 데이터이다. 수면다원검사는 수면단계의 기본적인 정답을 제공하지만, 아래와 같은 데이터를 보면 데이터의 신뢰성에 의구심이 든다.

- 예시
1. **REM이 존재하지 않는 경우**

![Untitled](/assets/images/W62/158b0243-f007-4e24-ad99-80f2cd4a978c.png)

사람의 수면에서 REM수면은 일반적으로 20~30%를 차지한다. 하지만 위 데이터처럼 아예 REM수면이 잡히지 않는 경우는 수면다원검사가 제대로 시행되지 않았음을 알 수 있다.

1. **WAKE가 비정상적으로 많고, REM이 비정상적으로 적은 경우**

![Untitled](/assets/images/W62/7b68ede2-7e8c-46ee-aa0e-1d097c4581fc.png)

해당 경우는 REM수면이 존재하긴 하지만, 비중이 전체 수면 대비 2%도 되지않고, 심지어 수면중 깸 상태(WAKE)가 수면의 절반 이상을 차지하는 매우 비정상적인 실험자의 데이터임을 알 수 있다.

위와 같은 이유로 신뢰성에 의심이 가는 데이터와, 수면장애 및 비정상적인 수면을 취하는 실험자의 데이터를 걸러내기 위해 직접 데이터의 통계분포를 확인해가며 데이터를 선별하였다. 총 102명의 환 자 중 REM이 존재하지 않는 경우 11건, WAKE가 비정상적으로 많은 경우 9건과 전처리 과정 중 수면단계 라벨링에 공백(Nan)값이 존재하는 7건을 제외하여 총 75명의 실험자 데이터만 사용하여 머신러닝 분류를 진행하였다. 하지만 걸러낸 데이터셋으로도 Random Forest(58%), SVM(57%)에 미치며 성능 향상을 이루지 못했다.

---

## 모델 수정

cnn - lstm 모델이 flutter 모델에서 작동하지 않는 오류가 발생했다. 원인을 찾아본 결과 lstm모델이 flutter앱에 올라가지 않는 것이 원인이었다. 모델을 수정할 필요가 있었다. 먼저 기존 1-d cnn 모델은 정확도가 낮았지만 flutter에 올라갔기 때문에 cnn계열의 모델로 학습을 진행해보기로 하였다.

### MobileNetV3-small

 

![Untitled](/assets/images/W62/p8.png)

MovilenetV3구조는 다음과 같다. 이 MovileNet은 이미지처리에 특화된 cnn이다. 하지만 우리의 데이터는 시계열 데이터 이므로 이를 1차원으로 구조를 변형 후 진행하였다.

- 모델의 accurcy가 기존 54%에서 58%로 올라갔다. 또한 class를 3개로 압축한 결과 63%의 accurcy가 나왔다.
- 기존 lstm모델의 71% accuracy가 나왔으나 이 모델에선 63%가 나왔으므로 모델을 수정할 필요가 있었다.

### MobileNetV3-Large

Large모델로 진행한 결과에도 성능이 나아지지 않고 오히려 성능이 하락하였다. 57.4%의 성능을 보였고 클래스 압축에도 61.8%로 성능이 하락하였다. 모델의 크기를 큰 모델로 변환하였음에도 성능이 오히려 하락 하였으며 이 원인은 기존 데이터에서 특징 추출이 원할하게 되지 않은 것을 원인으로 보았다. 그래서 데이터의 전처리를 수정할 필요가 있었다. 

## 데이터 전처리 수정

아무리 시도해도 accuracy가 이전 논문에 실린 85%의 정확도로 나오는 것을 재현하지 못하였고 6-70%대의 성능이 나왔기에 조교님과 면담을 진행했다. **이전 논문 같은 survey는 주어진 데이터 내에서의 성능을 평가하기 때문에 실제 테스트 결과 보다 좋은 결과가 나올 수도 있을 수도 있다는 이야기를 들었다.** 그래서 다른 전처리 방식을 모색해보았다.

 지난 21년, Nature저널에 실린 손목의 가속도 데이터를 이용한 수면단계 판별에서 random forest기법을 이용하여 73.93%의 정확도가 나왔다. 이전 국내 논문에서는 85%가 나왔었는데 이 저널은 그보다 한참뒤인 2021년도에 출판되었는데도 85%가 아닌 73.93%의 성능을 보여주었다. **신뢰성 높은 학회지와, 특정 데이터셋에 편향된 정확도를 감안하여 Nature에 기재된 논문에서 소개한 전처리 기법을 사용하여 새롭게 수면단계 분류**를 진행하였다. 추가적으로, **해당 논문에선 5중 Random Forest 모델을 이용하였으나, Flutter에 해당 모델을 올릴 수 없기에 CNN 계열의 경량화된 모델로 대체하여 실험하였다.**

[Sleep classification from wrist-worn accelerometer data using random forests](https://www.nature.com/articles/s41598-020-79217-x)

![Untitled](/assets/images/W62/p9.png)

### 1. ENMO (Euclidean Norm Minus One)

첫번째로 ENMO이다 기존의 우리가 이용한 intensity의 경우 유클리디언 거리만을 이용하였지만 이 intensity의 경우 -1을 해주었다. 이 논문에서 -1을 해주는 이유는 중력의 평균크기인 1을 제거하는 이유이기 때문이다. 중력을 제거하였으므로 당연히 intensity-1이 0보다 작은경우 0값으로 해준다. 이것을 수식으로 나타내면 다음과 같다.

![Untitled](/assets/images/W62/p10.png)

### 2. Tilt Angles

두번째 인자로 기울기 각도를 넣어주었다. 이값은 가속도 데이터에서 기기의 공간적 기울기를 유추할 수 있어 더욱 특징을 뽑아낼 수 있다. 즉  각 축의 기울기 각도를 계산하면, 기기의 공간적 방향과 회전 상태를 측정할 수 있다. 기울기 각도는 가속도 센서의 raw 데이터를 보정하고, 기기의 실제 기울기를 파악하는 데 사용된다. 이때 기울기 각도는 라디안 단위가아닌 도 단위를 사용한다.

![Untitled](/assets/images/W62/p11.png)

### 3. LIDS (Locomotor Inactivity During Sleep)

**ENMO_sub** :

![Untitled](/assets/images/W62/p12.png)

먼저 ENMO값에 0.02를 빼주어 값을 보정을 해준다. 이는 센서 자체의 노이즈나 미세한 환경 변화로 인해 작은 값의 진동이 발생할 수 있기 때문이다.또한 수면 중에는 미세한 움직임이 빈번히 발생할 수 있기 때문에 이러한 작은 움직임은 실제로는 수면 단계나 수면의 질에 큰 영향을 미치지 않는 경우가 많기 때문이다. 즉 실제로 의미 있는 움직임만을 반영하도록 하는 것이 의도이다.

**ENMO_sub_smooth**:

![Untitled](/assets/images/W62/p13.png)

ENMO_sub_smooth는 10분(600초) 간격으로 움직임의 총량을 계산한다. 이는 짧은 시간 동안의 움직임을 누적하여 더 안정적인 활동을 모니터링. 이렇게 하면 순간적인 움직임보다 더 지속적인 활동 패턴을 추출할 수 있다.

**LIDS_unfiltered 계산**:

![Untitled](/assets/images/W62/p14.png)

LIDS_unfiltered는 ENMO_sub_smooth 값의 역수를 취하여 활동량이 적을수록 값이 커지도록 한다. 이는 움직임이 적은 상태, 즉 비활동 상태를 강조하기 위함이다. +1을 하는 이유는 0으로 나누는 상황을 없애려고 하기 때문이다.

**LIDS 계산**:

![Untitled](/assets/images/W62/p15.png)

LIDS_unfiltered 값을 30분(1800초) 동안 이동 평균을 계산한다. LIDS 수면 중 신체의 움직임이 적은 상태를 평가하여, 수면 상태를 감지하는 데 도움이 된다.  

데이터의 전처리는 다음과 같았고 이에따라 총 입력 tensor는 [1920,5]의 크기가 되었다. 

구조가 [ENMO, ang_x, ang_y, ang_z, LIDS]인 입력tensor이다.

### 결과

데이터를 전처리 한 결과 mobileNetV3-small에서 accuracy 60%로 약 2%의 성능향상을 이루어냈다. class를 3개로 압축한 결과 68%의 정확도를 나타내었다. 하지만 아직 정확도가 좋지 못했는데 원인을 rem수면을 깊은 수면으로 예측하였기 때문이다. 이 rem수면을 깊은 수면 단계로 예측하였을때는 76%의 정확도를 보여주었다.

# Todo

- 지속적인 수면 측정
- 수면단계 모델 정확도 개선 연구
- 디바이스 외관 꾸미기