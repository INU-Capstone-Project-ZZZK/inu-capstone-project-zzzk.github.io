---
layout: default
title: 3월 3주차
parent: Weekly_Report
nav_order: 1
---

# 3월 3주차 Weekly Report
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{: toc}

---

## 주간 작업 내용


{: .note }
3월 3주차 작업내용이다. 3주차 목요일 이후에 주제 선정 결과를 확인했기에 작업내용이 아직 많지 않아 추후 작업 배분을 좀더 신경써야 할것으로 보인다.

 - 03.21(목) 16:30 ~ 19:00 : 디바이스 3D프린트 설계도 완성 및 스트랩 변경

 - 03.21(목) 22:00 ~ 24:00, 03.22(금) 15:30 ~ 17:00 : 깃허브 모델 로드 및 Tensorflow Lite 변환 진행


---

### 3D 프린팅 설계 및 스트랩 선정

보고서에서 상정했던 크기대로 3D 프린팅 설계 두 외관 다 뚜껑이 있는 구조로 내부에 센서 및 라즈베리파이 피코를 넣고 뺄 수 있다.

![](/assets/images/W1/p0.png)


손목 디바이스의 경우 PPG 센서의 빛이 나가기 위한 구경 1CM의 구멍이 존재하며 스트랩을 끼우기 위한 양 옆의 고리가 존재한다. 

![](/assets/images/W1/p1.png)


[INU makerspace](http://makerspace.inu.ac.kr/)

해당 사이트에서 인천대 내에 있는 3D 프린터 사용의 예약이 가능하며 내부 디바이스들이 완성될 경우 사용하도록 한다. 

[스트랩 구매처](https://www.lenergys.com/shop/shopdetail.html?branduid=2616671&gad_source=4&gclid=CjwKCAjwte-vBhBFEiwAQSv_xVmF4DQCDUH8IXPDnAo_jXD9XWv3yJRuMUgZ_sLmXaptm4FHoAdX4hoC0D0QAvD_BwE&ref=www.google.com)


해당 제품 30cm 구매


![](/assets/images/W1/p2.png)


![](/assets/images/W1/p3.png)

---

### GRU 모델 추출 및 Flutter 앱 이식 준비
 가장 먼저 손목디바이스의 수면단계 판별을 위해 사전에 준비해둔 추론 모델을 사용할 것이다. 사용할 모델은 [해당 깃허브](https://github.com/cbrnr/sleepecg)에서 제공되는 사전학습이 완료된 심박수 기반 수면단계 분류 모델(GRU)이다.  
PPG 센서에서 전처리된 심박수 데이터를 기반으로 딥러닝 모델(GRU)를 이용하여 수면 단계를 판별하는 모델이다. 해당 모델은 Tensorflow로 구성되어 있으며, Flutter 앱에 올리기 위해서는 Tensorflow Lite 버전으로 바꿔주는 단계를 거쳐야 한다.
![저장된 모델 구성도](/assets/images/W1/p5.png)

제공된 모델은 saved_model 형태의 pb파일로 갖춰져 번수 및 메타 데이터와 함께 classifier 폴더 안에 구비되어 있어 이를 Tensorflow Lite로 전환하는 과정을 거쳤다.

```python
import tensorflow as tf

# SavedModel 디렉토리 경로
saved_model_dir = 'classifier'

# 변환기 생성 및 모델 로드
converter = tf.lite.TFLiteConverter.from_saved_model(saved_model_dir)

# 모델 변환
tflite_model = converter.convert()

# .tflite 파일로 저장
with open('converted_model.tflite', 'wb') as f:
    f.write(tflite_model)
```
 위 코드로 변환을 시도하였으나 모델을 불러오고 변환하는 과정과 내부 레이어의 Tensor 연산 변환에서 반복적으로 에러가 발생하여 아래 코드와 같이 모델 로드와 변환기 생성, 텐서 리스트 연산 옵션 부여를 분리하여 코드를 작성하였다.


```python
import tensorflow as tf
import numpy as np

print("TensorFlow 버전:", tf.__version__)

model_path = './classifier/'

# TensorFlow 모델을 로드
loaded_model = tf.keras.models.load_model(model_path)
# TensorFlow Lite 변환기 생성
converter = tf.lite.TFLiteConverter.from_keras_model(loaded_model)
# TensorFlow 연산으로 텐서 리스트 연산 변환
converter.target_spec.supported_ops = [tf.lite.OpsSet.TFLITE_BUILTINS, tf.lite.OpsSet.SELECT_TF_OPS]
# 변환 수행
tflite_model = converter.convert()

# 변환된 모델을 .tflite 파일로 저장
with open('converted_model.tflite', 'wb') as f:
    f.write(tflite_model)


# 변환된 TensorFlow Lite 모델의 경로
tflite_model_path = './converted_model.tflite'

# TensorFlow Lite Interpreter 생성
interpreter = tf.lite.Interpreter(model_path=tflite_model_path)
interpreter.allocate_tensors()

# 입력 및 출력 텐서 정보 가져오기
input_details = interpreter.get_input_details()
output_details = interpreter.get_output_details()

# 모델의 입력 크기 및 형태 확인
print("Input shape:", input_details[0]['shape'])
print("Input type:", input_details[0]['dtype'])

# 모델의 출력 크기 및 형태 확인
print("Output shape:", output_details[0]['shape'])
print("Output type:", output_details[0]['dtype'])

# 모델의 중간 레이어 및 파라미터 확인
for layer in interpreter.get_tensor_details():
    print(layer['name'], layer['shape'], layer['dtype'])
```

![변환이 완료된 모습](/assets/images/W1/p4.png)

 차주에 Tensorflow Lite 변환된 모델을 Flutter 앱에 올리는 작업을 진행할 것이며, 추후 동일한 데이터셋을 이용하여 변환전과 후의 분류 정확도를 테스트할 것이다.

---

### 추가 물품 변경 및 주문 물품 업데이트

보다 사이즈 변경에 용이한 손목디바이스의 착용 방식을 고민하여 헬스 스트랩 형태로 길이와 조임 정도를 조절이 가능한 스트랩으로 선정하였다.

추가적으로, 기존에 주문하려던 380mAh 리튬배터리가 디바이스마트에서 재고품절이 나버려서, 동일한 브랜드의 자사몰에서 별도로 구입해야 하기에 리스트업하였다.

[겔패치](https://ko.aliexpress.com/item/4000068417083.html?spm=a2g0o.detail.pcDetailTopMoreOtherSeller.1.1470mLh8mLh8q8&gps-id=pcDetailTopMoreOtherSeller&scm=1007.40050.354490.0&scm_id=1007.40050.354490.0&scm-url=1007.40050.354490.0&pvid=a3ce2662-f3c8-4496-a43f-cb9e4895a0b1&_t=gps-id:pcDetailTopMoreOtherSeller,scm-url:1007.40050.354490.0,pvid:a3ce2662-f3c8-4496-a43f-cb9e4895a0b1,tpp_buckets:668%232846%238114%231999&pdp_npi=4%40dis%21KRW%211964%211368%21%21%211.45%211.01%21%402140e84617104267588128301e3459%2110000000176094709%21rec%21KR%21%21AB&utparam-url=scene%3ApcDetailTopMoreOtherSeller%7Cquery_from%3A)

[레너지스 아웃도어](https://www.lenergys.com/shop/shopdetail.html?branduid=2616671&gad_source=4&gclid=CjwKCAjwte-vBhBFEiwAQSv_xVmF4DQCDUH8IXPDnAo_jXD9XWv3yJRuMUgZ_sLmXaptm4FHoAdX4hoC0D0QAvD_BwE&ref=www.google.com)

[더한 리튬폴리머 배터리 충전지 모음 3.7V KC인증필](https://m.thehanpart.co.kr/product/detail.html?product_no=196&cate_no=1&display_group=7)

---

## Todo
Flutter 앱의 UI를 완성하고, 라즈베리파이 피코 주문이 늦어졌기 때문에 임시로 라즈베리 파이 5로 Flutter 앱간 블루투스 통신 테스트를 진행한다.
다음 주차까지 손목 디바이스의 온디바이스 시그널 전처리 및 블루투스 통신을 통한 데이터 교환을 테스트할 것이다.  


| Name    | Work Content |
|:--------|:------------------------------|
| `박종현` | `Flutter 앱 및 블루투스 통신 코드 구축` |
| `이태우` | `PPG 시그널 전처리 코드 완성. 변환 모델의 수면단계 판별 정확도 테스트.` |
| `이제욱` | `라즈베리 파이 5 개발환경 세팅. → 라즈베리 파이 OS 업로드와 vscode 개발환경 세팅` |

