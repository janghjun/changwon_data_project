# 창원시 교통 데이터 수집 파이프라인 정리 (Step0\~Step4 + 데이터 사전)

> 목적: 공모전 제출/재현을 위해 **Colab 노트북** 흐름을 Step별로
> 정리하고, 각 단계의 **핵심 코드 스니펫**과 **산출물(파일) 사전**을
> 함께 제공합니다. 실제 실행은 노트북에서 셀 단위로 복사·실행하세요.

------------------------------------------------------------------------

## Step 0. 환경 준비 & 비밀키 관리

### 개요

-   Google Drive 마운트, 프로젝트 디렉토리/출력 경로 생성
-   **.env** 에 API 키(공공데이터포털 일반 인증키; URL-Encoded) 저장 후
    환경 변수로 로드
-   키는 **URL-Encoded(...%3D%3D)** 형태를 `.env`에 저장, 코드에서는
    `unquote()` 로 Decoding

### 코드 스니펫

``` python
from google.colab import drive
drive.mount('/content/drive')

import os, textwrap
DRIVE = "/content/drive/MyDrive"
SECRET_DIR = f"{DRIVE}/.secrets/changwon"
DATA_DIR   = f"{DRIVE}/data_j"
os.makedirs(SECRET_DIR, exist_ok=True)
os.makedirs(DATA_DIR, exist_ok=True)
```

(이후 dotenv 로드 및 KEY 확인 코드 포함)

------------------------------------------------------------------------

## Step 1. TAGO BIS --- 도시코드 조회 (자동)

(코드 및 설명 포함)

------------------------------------------------------------------------

## Step 2. TAGO BIS --- 정류소/노선/경유정류소 전수 수집

(전체 코드 및 FK 무결성 검증 포함)

------------------------------------------------------------------------

## Step 3. 도착정보 샘플링 → 헤드웨이/정시성(CV)

(파일럿 루프, 이벤트 검출, 버킷법, 정류소 레벨 집계 코드 포함)

------------------------------------------------------------------------

## Step 4-A. 산업단지 경계도면(SHP) → 창원 추출

(강건 로더, 좌표계 변환, BBox 필터, dissolve, 면적/센트로이드 계산 코드
포함)

------------------------------------------------------------------------

## Step 4-B. ITS 교통소통정보 --- 파일데이터(5분 CSV) 처리 (예정)

(청크 단위 로드, 출퇴근 시간대 집계 코드 포함)

------------------------------------------------------------------------

# 📦 산출물(파일) **데이터 사전(Data Dictionary)**

## 1) TAGO BIS 정적 구조

-   stops_std.csv: 정류소 ID/이름/좌표
-   routes_std.csv: 노선 ID/번호/유형
-   route_stops_std.csv: 노선별 경유정류소

## 2) 도착정보(동적 로그) & 품질 지표

-   arrivals_raw.csv: 원천 로그
-   headway_station_route_bucketed.csv: 노선×정류소 헤드웨이 통계
-   headway_station.csv: 정류소별 요약

## 3) 산업단지

-   industrial_parks_cw_parklevel.geojson/gpkg: 단지
    경계/위치/면적/센트로이드
-   industrial_parks_cw_parklevel_utf8sig.csv: CSV 버전

## 4) ITS 혼잡도 요약(예정)

-   its_commute_summary_utf8sig.csv: 링크별 출퇴근 평균속도/혼잡레벨

------------------------------------------------------------------------

# 🔗 분석 활용 가이드

-   B-1 버스 배차간격 vs 청년 밀집도
-   B-2 정시성 vs 전월세 단가
-   B-4 혼잡도 vs 청년 주거 선호
-   B-5 환승거점 접근성 vs 임대료
-   B-6 산업단지 접근성 vs 청년 거주율
