KOREAN | [ENGLISH](https://github.com/DVA-LAB/DVA_LAB-data/blob/main/README.md)

# Jeju Dolphin & Ship Detection Dataset

## 개요
이 데이터셋은 제주 해역에서 촬영된 항공 및 해상 영상 데이터를 바탕으로 제주남방큰돌고래와 선박을 탐지하기 위한 목적으로 제작되었습니다.  
주요 특징으로는 다양한 환경 조건(기상, 시간대 등)에서 수집된 데이터를 포함하고 있으며, 객체의 크기 및 형태 변화를 고려한 어노테이션이 적용되었습니다.

## 펠로우 조직 정보
이 데이터셋은 MARC에서 원천 데이터를 수집하였고, DVA LAB 2기 멤버들이 직접 라벨링하였습니다.

- **조직명**: MARC(Marine Animal Research and Conservation)
- **담당자**: 김미연 부대표  
- **이메일**: marckorea718@gmail.com  

## 라이센스
- **라이센스**: CC BY-NC-SA (Attribution-NonCommercial-ShareAlike)
- **활용 범위**: 비상업적 연구 및 교육 목적으로 사용 가능하며, 반드시 출처를 명시해야 합니다.
- **2차 저작물 가이드라인**: 2차 저작물 생성 시 동일한 라이센스로 배포해야 하며, 원본 데이터에 대한 변경 사항을 명확히 기술해야 합니다.

## 데이터 명세

### 데이터 구조
이 데이터셋은 이미지 데이터 및 해당 이미지에 대한 어노테이션 파일로 구성되어 있으며, 데이터의 저장 경로는 다음과 같습니다.

- **ROOT 디렉토리**: `private_files_for_service/AI/object_detection`
- **데이터셋 경로**: `ROOT/dataset_2024/unpatched/`
- **이미지 디렉토리**:
  - `train_1 ~ train_7`
  - `val_1 ~ val_2`
  - `test_1`
- **어노테이션 디렉토리**:
  - 이미지 데이터와 동일한 스플릿을 가지는 10개의 bbox 어노테이션 파일 포함
  - 어노테이션 형식: **CVAT 1.1 (XML 형식)**

### 데이터 통계
- **전체 데이터 크기**: 약 50GB
- **이미지 개수**: 1,085장
- **해상도**: 대부분 `3840×2160`, 일부 `1920×1080`
- **레이블 종류**: Dolphin, Ship, Jetski
- **수집 기간**: 2023년 1월 ~ 2024년 12월

### 객체별 bbox 기초 통계

| Class   | bbox Count | bbox Per Image | Mean Size | Median Size | Min Size | Max Size |
|---------|------------|----------------|------------|------------|-----------|-----------|
| Dolphin | 834        | 0.8            | 1,662      | 1,018      | 66        | 22,302    |
| Jetski  | 85         | 0.1            | 2,339      | 1,736      | 66        | 8,875     |
| Ship    | 952        | 0.9            | 40,276     | 26,992     | 90        | 501,071   |

### 객체별 bbox 크기 분포
- **Jetski**: 다양한 크기의 객체가 균등하게 분포함.
- **Dolphin, Ship**: 상대적으로 작은 크기에서 많은 분포를 보이며, 일부 큰 객체가 존재하는 극단적인 분포를 보임.

## 전처리 정보

### 데이터 정제 과정
- **중복 데이터 제거**: 유사한 프레임이 반복적으로 존재하는 데이터 필터링
- **화질 보정**: 밝기 및 대비 보정, 흐림 현상 보정
- **노이즈 제거**: 영상 흔들림 및 불필요한 정보 제거

### 레이블링 방식
- **CVAT 1.1을 이용하여 라벨링 진행**
- **BBox Annotation**:
  - 초기에는 Axis-Aligned Bounding Box (AABB) 사용
  - OBB(Oriented Bounding Box) 기반 모델을 실험하기 위해 회전 가능 직사각형 형태의 라벨 추가 생성

### 품질 검증 방법
- **다단계 검수 진행**:
  - **1차 검수**: 작업자가 기본 가이드라인을 적용하여 레이블링 수행
  - **2차 검수**: 검수자가 라벨링의 일관성을 확인하고 재검토
  - **3차 검수**: 모델 테스트 후 오탐지 및 미탐지 사례 보완

## 접근 정보

### 데이터 다운로드
- **데이터 다운로드 링크**: (추후 제공)
- **접근 권한 요청 방법**: taylor.7@kakaoimpact.org
- **데이터 제공 형식**:
  - 이미지 데이터 (`.png`)
  - 어노테이션 데이터 (`.xml` - CVAT 1.1)

## 활용 예시

### 데이터 로드 예시 코드
```python
import os
import xml.etree.ElementTree as ET
import cv2

# 데이터셋 경로
ROOT = "private_files_for_service/AI/object_detection"
DATASET_PATH = os.path.join(ROOT, "dataset_2024/unpatched/images/train_1")

# 어노테이션 파일 불러오기
ANNOTATION_PATH = os.path.join(ROOT, "dataset_2024/unpatched/annotations/train_1.xml")

# XML 파싱
tree = ET.parse(ANNOTATION_PATH)
root = tree.getroot()

# 이미지 및 bbox 정보 출력
for image in root.findall('image'):
    image_name = image.get('name')
    img_path = os.path.join(DATASET_PATH, image_name)
    img = cv2.imread(img_path)
    
    for box in image.findall('box'):
        label = box.get('label')
        xtl = int(float(box.get('xtl')))
        ytl = int(float(box.get('ytl')))
        xbr = int(float(box.get('xbr')))
        ybr = int(float(box.get('ybr')))
        
        # bbox 그리기
        cv2.rectangle(img, (xtl, ytl), (xbr, ybr), (0, 255, 0), 2)
        cv2.putText(img, label, (xtl, ytl-10), cv2.FONT_HERSHEY_SIMPLEX, 0.5, (0, 255, 0), 2)

    cv2.imshow("Image", img)
    cv2.waitKey(0)

cv2.destroyAllWindows()
```

## 주의사항

### 제한사항
- 연구 및 교육 목적 외 상업적 활용 금지
- 데이터셋에 포함된 이미지의 무단 재배포 금지

### 알려진 이슈
- **어두운 이미지**: 돌고래가 수면 아래에 있을 경우 탐지가 어려움
- **물보라**: 선박의 궤적에 형성되는 물보라가 객체 탐지를 방해함
- **빛반사**: 수면의 빛반사가 객체의 경계를 흐리게 만듦

### 권장사항
- **사전 데이터 보정**: 밝기 및 대비 조절을 통해 객체 탐지 성능 개선 가능
- **데이터 증강**: 다양한 조명 조건을 고려한 데이터 증강 기법 적용 추천

## 업데이트 이력

| 버전  | 날짜        | 내용                       | 담당자   |
|-------|------------|--------------------------|---------|
| 0.1   | 2024-02-18 | 초기 데이터셋 문서 README.md 설정 | 한서우 랩장 |

## 문의처
- **슬랙 채널**: `#99-lab-qna`
- **담당자**: 김민석 매니저(카카오임팩트)
- **이메일**: taylor.7@kakaoimpact.org
