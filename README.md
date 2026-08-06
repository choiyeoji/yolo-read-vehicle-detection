# 🚗 YOLO 기반 차량 인식·분류 및 도로 상황 감지 시스템

> YOLO 기반 객체 검출과 추적 기술을 활용하여 국도와 고속도로 환경의 차량을 실시간으로 인식하고, 정차 차량·불법 주정차·차로 주행 트럭 등의 도로 상황을 판단하는 시스템입니다.

---

## 📌 프로젝트 개요

도로 CCTV 또는 카메라 영상을 입력받아 차량을 실시간으로 검출·분류하고,  
국도와 고속도로 환경에 서로 다른 판단 조건을 적용하여 위험 상황을 감지하는 시스템을 구현했습니다.

국도 환경에서는 정차 차량 및 불법 주정차 차량을 판단하고,  
고속도로 환경에서는 정차 차량과 제한 차로를 주행하는 트럭을 감지하도록 구성했습니다.

---

## 📅 수행 기간

**2026.06.22 ~ 2026.06.30**

---

## 👥 팀 구성

**4인 팀 프로젝트**

---

## 🙋 담당 역할

- 국도용 차량 감지 알고리즘 구현
- 국도 환경의 정차 차량 판단 조건 설계
- No Parking Zone 기반 불법 주정차 판단 로직 구현
- 차량 Bounding Box와 Zone 간 IoU 기반 위치 판단
- Confidence Hold Filter 적용
- Cross-Class Duplicate Filter 적용
- 국도 시연 영상 수집 및 결과 분석

> 전체 시스템은 팀 프로젝트로 진행했으며, 본인은 국도 환경의 정차 차량 및 불법 주정차 판단 로직과 후처리 필터 구현을 담당했습니다.

---

## 🛠 사용 기술

### Programming

- Python
- OpenCV

### AI / Computer Vision

- YOLO11n
- ByteTrack
- IoU
- Bounding Box
- Object Detection
- Object Tracking

### Deployment

- TensorRT
- NVIDIA Jetson
- CUDA

---

## 🏗 시스템 구성

카메라 또는 도로 영상을 입력받아 YOLO 모델로 차량을 검출하고,  
ByteTrack을 이용해 차량별 ID를 유지하며 실시간으로 추적합니다.

검출 결과에는 Confidence Filter, IoU 판단, 중복 클래스 제거 등의 후처리를 적용하고,  
도로 유형별 조건을 이용하여 정차·불법 주정차·차로 위반 상황을 판단합니다.

```text
Camera / CCTV Input
        ↓
YOLO11n Vehicle Detection
        ↓
ByteTrack Object Tracking
        ↓
Confidence / IoU / Duplicate Filtering
        ↓
Road Condition Judgment
        ↓
Bounding Box / Alert / Capture Output
```

---

## ✨ 주요 기능

### 🛣️ 국도 환경

- 차량 실시간 검출 및 추적
- 일정 시간 이상 움직이지 않는 차량 감지
- 정차 차량의 사고 가능성 판단
- No Parking Zone 설정
- Zone 내부 장시간 정차 차량의 불법 주정차 판단
- 감지 상황 이미지 자동 저장

### 🛤️ 고속도로 환경

- 고속도로 주행 차량 실시간 검출
- 도로 위 정차 차량 감지
- 제한 차로 내 트럭 주행 여부 판단
- 위반 차량 감지 시 캡처 이미지 저장

---

## 🔍 핵심 구현 내용

### 1. IoU 기반 Zone 침범 판단

차량 Bounding Box와 사용자가 설정한 Zone의 겹침 정도를 IoU로 계산하여  
차량이 특정 도로 영역에 진입했는지 판단했습니다.

```text
IoU = 교집합 영역 / 합집합 영역
```

이를 통해 차량의 단순 검출 여부뿐 아니라,  
불법 주정차 구역 또는 제한 차로 내부에 차량이 위치하는지를 구분했습니다.

---

### 2. Confidence Hold Filter

조명 변화, 차량 가림, 프레임 변화로 인해 차량의 Confidence가 순간적으로 낮아지는 경우에도  
최근 검출 결과와 동일 차량으로 판단되면 검출 정보를 일정 시간 유지하도록 구현했습니다.

#### 적용 기준

- 현재 Confidence가 기준값 이상이면 검출 결과 갱신
- Confidence가 낮아지면 최근 2초 이내 결과 검색
- 동일 클래스 여부 확인
- IoU 기준을 만족하면 기존 검출 결과 유지
- 유지 시간이 지나면 객체 제거

이를 통해 순간적인 검출 누락으로 차량이 화면에서 사라지는 현상을 줄였습니다.

---

### 3. Cross-Class Duplicate Filter

하나의 차량이 `car`, `truck`, `bus` 등 여러 클래스로 중복 검출되는 문제를 해결하기 위해  
서로 다른 클래스의 Bounding Box를 비교하고, 중복된 경우 가장 높은 Confidence를 가진 클래스만 유지했습니다.

#### 처리 과정

1. Confidence가 높은 순서로 Bounding Box 정렬
2. 서로 다른 클래스인지 확인
3. 중심점 겹침 또는 IoU 기준 확인
4. 중복된 박스 제거
5. 가장 높은 Confidence의 클래스만 유지

---

## ⚠️ 문제 해결

### 일시적 차량 검출 누락

| 구분 | 내용 |
| --- | --- |
| 문제 | 조명, 가림, 프레임 변화로 차량 Confidence가 순간적으로 낮아지거나 검출이 끊김 |
| 원인 | 프레임 단위 검출 결과만 사용할 경우 동일 차량도 새로운 객체 또는 미검출로 판단됨 |
| 해결 | 최근 검출 결과와 IoU 및 클래스 조건을 비교하는 Confidence Hold Filter 적용 |
| 결과 | 차량 검출 연속성 확보 및 정차 판단 안정성 향상 |

### 동일 차량의 중복 클래스 검출

| 구분 | 내용 |
| --- | --- |
| 문제 | 하나의 차량이 car, truck, bus 등 여러 클래스로 동시에 검출됨 |
| 원인 | 형태가 유사한 차량 클래스 간 경계가 모호하여 여러 Bounding Box가 생성됨 |
| 해결 | 중심점 및 IoU 기준으로 중복 여부를 판단하고 가장 높은 Confidence의 클래스만 유지 |
| 결과 | 중복 Bounding Box 제거 및 도로 상황 판단 정확도 향상 |

---

## 📊 실행 결과

### 국도 환경

| 정차 차량 감지 | 불법 주정차 감지 |
| --- | --- |
| <img src="images/national_stopped_vehicle.png" width="100%"> | <img src="images/national_illegal_parking.png" width="100%"> |

> 본인은 국도 환경의 정차 차량 및 불법 주정차 판단 로직을 담당했습니다.

### 고속도로 환경

<table>
  <tr>
    <th>정차 차량 감지</th>
    <th>제한 차로 트럭 감지</th>
  </tr>
  <tr>
    <td align="center">
      <img src="images/highway_stopped_vehicle.png" width="360">
    </td>
    <td align="center">
      <img src="images/highway_truck_detection.png" width="360">
    </td>
  </tr>
</table>

---

## 🎥 시연 영상

### 🛣️ 국도 환경

#### 정차 차량 감지

https://github.com/user-attachments/assets/https://github.com/user-attachments/assets/a1df30c2-be1d-4da5-b8ce-9798ebbe565e

#### 불법 주정차 감지

https://github.com/user-attachments/assets/https://github.com/user-attachments/assets/9cb2302e-b36d-47e5-945d-b8f014c61369



### 🛤️ 고속도로 환경

#### 정차 차량 및 제한 차로 트럭 감지

https://github.com/user-attachments/assets/https://github.com/user-attachments/assets/81df3ba5-5faa-498a-91e8-7b72ed72e4f7


---

## 🤖 모델 학습 및 최적화

- YOLO11n 사전 학습 모델 활용
- 차량 5개 클래스 구성
  - truck
  - trailer
  - bus
  - car
  - motorcycle
- 250 Epoch 학습 수행
- 데이터 증강 적용
  - Rotation
  - Horizontal Flip
  - Mosaic
- 학습 모델을 ONNX 형식으로 변환
- TensorRT Engine으로 최적화
- NVIDIA Jetson 환경에서 실시간 추론 수행

---

## 📂 프로젝트 구조

```text
yolo-road-vehicle-detection/
├── README.md
├── src/
│   ├── national_road_detection.py
│   ├── highway_detection.py
│   ├── confidence_hold_filter.py
│   └── duplicate_filter.py
├── models/
│   ├── best.pt
│   ├── best.onnx
│   └── vehicle_yolo11n.engine
├── images/
│   ├── system_overview.png
│   ├── national_stopped_vehicle.png
│   ├── national_illegal_parking.png
│   ├── highway_stopped_vehicle.png
│   └── highway_truck_detection.png
├── videos/
│   ├── national_stopped_vehicle.mp4
│   ├── national_illegal_parking.mp4
│   └── highway_demo.mp4
├── docs/
│   └── project_presentation.pdf
└── requirements.txt
```

> 실제 업로드하는 파일 구성에 따라 폴더명과 파일명은 수정할 예정입니다.

---

## ✅ 검증 내용

- 국도 영상에서 차량 검출 및 클래스 분류 결과 확인
- 일정 시간 이상 정차 차량의 사고 판단 결과 확인
- No Parking Zone 내 차량의 불법 주정차 판단 확인
- Confidence Hold Filter 적용 전후 검출 연속성 비교
- Cross-Class Duplicate Filter 적용 후 중복 박스 제거 확인
- 고속도로 제한 차로 내 트럭 감지 결과 확인
- Jetson 환경에서 실시간 추론 동작 확인

---

## 💡 프로젝트를 통해 배운 점

단순히 차량을 검출하는 것만으로는 실제 도로 상황을 정확히 판단하기 어렵다는 점을 알게 되었습니다.

차량의 위치, 정차 시간, Zone 침범 여부, Confidence 변화, 클래스 중복 등  
여러 조건을 함께 고려해야 안정적인 도로 상황 판단이 가능했습니다.

또한 객체 검출 모델의 성능뿐 아니라 후처리 알고리즘이 전체 시스템의 신뢰성에 큰 영향을 준다는 점을 경험했습니다.

향후에는 CCTV 기반 교통 모니터링, 사고 차량 자동 감지, 불법 주정차 단속 시스템 등으로 확장할 수 있습니다.

---

## 📄 발표 자료

전체 시스템 설계 및 검증 과정은 아래 발표 자료에서 확인할 수 있습니다.

<p align="center">
  <a href="docs/260630_yolo_vehicle_detection.pdf">
    <b>📑 프로젝트 발표 자료 보기</b>
  </a>
</p>
