# 창원시 교통 데이터 수집 파이프라인 정리 (Step0\~Step4 + 데이터 사전)

> 목적: 공모전 제출/재현을 위해 **Colab 노트북** 흐름을 Step별로 정리하고, 각 단계의 **핵심 코드 스니펫**과 **산출물(파일) 사전**을 함께 제공합니다. 실제 실행은 노트북에서 셀 단위로 복사·실행하세요.

---

## Step 0. 환경 준비 & 비밀키 관리

### 개요

* Google Drive 마운트, 프로젝트 디렉토리/출력 경로 생성
* **.env** 에 API 키(공공데이터포털 일반 인증키; URL-Encoded) 저장 후 환경 변수로 로드
* 키는 **URL-Encoded(…%3D%3D)** 형태를 `.env`에 저장, 코드에서는 `unquote()` 로 Decoding

### 코드 스니펫

```python
from google.colab import drive
drive.mount('/content/drive')

import os, textwrap
DRIVE = "/content/drive/MyDrive"
SECRET_DIR = f"{DRIVE}/.secrets/changwon"
DATA_DIR   = f"{DRIVE}/data_j"
os.makedirs(SECRET_DIR, exist_ok=True)
os.makedirs(DATA_DIR, exist_ok=True)

# dotenv 로드 및 KEY 확인 코드 포함
```

---

## Step 1. TAGO BIS — 도시코드 조회 (자동)

### 개요

* TAGO 버스정류소정보 서비스의 도시코드 목록으로부터 **창원시 cityCode=38010** 자동 탐색
* HTTPS 오류가 발생하면 HTTP(80) + Decoded Key 조합으로 호출

### 코드 스니펫

```python
import requests, pandas as pd
from urllib.parse import unquote
import os

BASE = "http://apis.data.go.kr/1613000"
DEC_KEY = unquote(os.getenv("CHANGWON_KEY_URLENC"))

url = f"{BASE}/BusSttnInfoInqireService/getCtyCodeList"
params = {"serviceKey": DEC_KEY, "_type": "json"}
js = requests.get(url, params=params, timeout=30).json()
items = js["response"]["body"]["items"]["item"]
df_city = pd.DataFrame(items)
CITY_CODE = df_city.loc[df_city["cityname"].str.contains("창원"), "citycode"].iloc[0]
print("CITY_CODE:", CITY_CODE)  # '38010'
```

---

## Step 2. TAGO BIS — 정류소/노선/경유정류소 전수 수집

### 개요

* **정류소 전수**: `getSttnNoList`
* **노선 전수**: `getRouteNoList`
* **노선별 경유정류소 전수**: `getRouteAcctoThrghSttnList`
* 한글 인코딩 문제 방지 위해 `utf-8`(분석용), `utf-8-sig`(엑셀 공유용) 동시 저장
* 무결성 점검 결과: FK 모두 1.0 → 데이터 일관성 확보

---

## Step 3. 도착정보 샘플링 → 헤드웨이/정시성(CV)

### 개요

* 허브 정류소(경유 노선 수 상위 N=20)를 선정하여 파일럿 수집(30초 간격 × 약 8분)
* 이벤트 검출: 완화 규칙 + 버킷법(1분)
* 정류소 단위 평균 headway, CV 산출 후 저장

### 산출물 예시

* **arrivals\_raw\.csv**: 수집 원천 로그 (2389행)
* **headway\_station\_route\_bucketed.csv**: 노선×정류소 단위 headway
* **headway\_station.csv**: 정류소별 평균 headway, CV

---

## Step 4-A. 산업단지 경계도면(SHP) → 창원 추출

### 개요

* 공공데이터포털 **산업단지 경계도면(3069832)** 다운로드
* cp949 인코딩 SHP를 geopandas로 로드 → 좌표계 변환(ESRI:102082 → EPSG:4326)
* 창원 영역 필터 후 `park_id` 단위 dissolve
* 면적(m²), 센트로이드(lat/lon) 산출

### 결과

* 창원 후보: 68개 단지
* 저장: GPKG/GeoJSON/CSV(utf-8-sig)

---

## Step 4-B. ITS 교통소통정보 — 파일데이터(5분 CSV)

### 개요

* API 키 대신 **5분 단위 CSV(예: 20250906\_5Min.csv, 559MB)** 활용
* 청크 단위 로드 → 출퇴근(07–09, 17–19) 평균속도, 혼잡레벨 집계
* 결과 CSV: `its_commute_summary_utf8sig.csv`

---

# 📦 산출물(파일) 데이터 사전(Data Dictionary)

## 1) TAGO BIS 정적 구조

* **stops\_std.csv**: 정류소 ID/이름/좌표
* **routes\_std.csv**: 노선 ID/번호/유형
* **route\_stops\_std.csv**: 노선별 경유정류소

## 2) 도착정보(동적 로그) & 품질 지표

* **arrivals\_raw\.csv**: 수집 원천 로그
* **headway\_station\_route\_bucketed.csv**: 노선×정류소 headway 통계
* **headway\_station.csv**: 정류소별 평균 headway, CV

## 3) 산업단지

* **industrial\_parks\_cw\_parklevel.geojson/gpkg**: 단지 경계/위치/면적/센트로이드
* **industrial\_parks\_cw\_parklevel\_utf8sig.csv**: CSV 버전

## 4) ITS 혼잡도 요약(예정)

* **its\_commute\_summary\_utf8sig.csv**: 링크별 출퇴근 평균속도/혼잡레벨

---

# 🔗 분석 활용 가이드

* **B-1** 버스 배차간격 vs 청년 밀집도
* **B-2** 정시성 vs 전월세 단가
* **B-4** 혼잡도 vs 청년 주거 선호
* **B-5** 환승거점 접근성 vs 임대료
* **B-6** 산업단지 접근성 vs 청년 거주율

---
