# Yolov5_Car_Distance

YOLOv5 기반 블랙박스 영상 차량 · 보행자 탐지 및 차간거리 추정

<br/>

<img src="img/car_vd_result.png">

<br/>

<img src="img/people_vd_result.png">

<br/><br/>

## Object

<br/>

이 프로젝트는 블랙박스(전방 카메라) 영상에서 **차량과 보행자를 탐지하고, 내 차량으로부터의 거리를 추정**하는 것을 목표로 합니다.

별도 거리 센서 없이 카메라 영상 한 대만으로 주변 객체와의 거리를 표시해, 차간 거리 경고 같은 사고 예방 기능의 기초 모델로 활용할 수 있도록 구현했습니다.

<br/><br/>

## Content

<br/>

1. Module 소개

2. 프로세스 구성

3. 거리 추정 방식

4. 실행 방법

5. Version

6. Feedback

<br/><br/>

## 1. Module 소개

<br/>

- `main.py` : 영상 기반 거리 추정을 수행하는 메인 모듈입니다.

  ① `DistanceEstimationDetector` class : 영상 로드 → 프레임별 탐지 → 바운딩 박스 · 거리 · FPS 표시까지 한 번에 처리

  ② 탐지 대상은 COCO 기준 `person`(0), `car`(2) 두 클래스로 제한 (신뢰도 0.40, NMS IoU 0.45)

  ③ GPU(CUDA)가 있으면 자동으로 사용

- `test.py` : 단일 이미지(`input/input.png`)로 차량 탐지와 거리 계산을 먼저 검증한 테스트 모듈입니다.

- `input/` : 테스트 영상 (차량 주행 2종, 보행자 거리 1종) · 이미지

- `img/`, `pics/demo.gif` : 결과 화면

- `yolov5/` : YOLOv5 패키지 소스 (Ultralytics)

<br/><br/>

## 2. 프로세스 구성

<br/>

```
영상 입력 (OpenCV) → YOLOv5s 탐지 (person · car) → 바운딩 박스 표시 → 객체별 거리 계산 → 거리 · 신뢰도 · FPS 표시
```

<br/>

- 프레임마다 탐지 결과(정규화 좌표 · 신뢰도 · 클래스)를 받아 원본 해상도로 변환합니다.
- 화면에 객체 이름, 거리(m), 신뢰도(%), 처리 속도(FPS)를 함께 표시하고 콘솔에도 출력합니다.
- `q` 키를 누르면 종료합니다.

<br/><br/>

## 3. 거리 추정 방식

<br/>

- 기준점 : 화면 하단 중앙 (내 차량 위치로 가정)
- 대상점 : 객체가 화면 왼쪽에 있으면 박스 오른쪽 아래, 오른쪽에 있으면 왼쪽 아래, 가운데면 아래 중앙
- 거리 : 기준점 ~ 대상점 사이 픽셀 거리를 박스 높이로 나누고, 화면 폭 기준 픽셀-길이 환산값을 곱해 추정

<br/>

> 카메라 보정(calibration) 없이 화면 비율로 환산한 **근사값**입니다. 가까운 객체일수록 박스가 커져 거리가 작게, 먼 객체일수록 크게 나오는 상대적 경향을 보는 용도입니다.

<br/><br/>

## 4. 실행 방법

<br/>

```bash
pip install yolov5 opencv-python torch

# main.py 하단의 video_path 를 input/ 폴더의 영상으로 바꾼 뒤 실행
python main.py
```

<br/><br/>

## 5. Version

<br/>

- python 3.11
- torch
- yolov5 (pip 패키지)
- opencv-python
- numpy

<br/><br/>

## 6. Feedback

<br/>

- 현재 거리는 화면 비율 기반 근사값이므로, 카메라 초점거리 · 설치 높이를 이용한 보정을 적용하면 실제 거리에 가까워집니다.
- 일정 거리 이내로 가까워질 때 경고 신호를 보내는 기준과 신호 처리 방식을 추가할 예정입니다.
- 영상 경로가 코드 안에 절대경로로 지정되어 있어, 실행 인자로 받도록 개선이 필요합니다.
