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
# ▶ Colab: Drive 마운트
from google.colab import drive
drive.mount('/content/drive')

# ▶ 경로 세팅
import os, textwrap
DRIVE = "/content/drive/MyDrive"
SECRET_DIR = f"{DRIVE}/.secrets/changwon"
DATA_DIR   = f"{DRIVE}/data_j"
os.makedirs(SECRET_DIR, exist_ok=True)
os.makedirs(DATA_DIR, exist_ok=True)
```

(중략)

------------------------------------------------------------------------

# 📦 산출물(파일) **데이터 사전(Data Dictionary)**

> 실제 저장 경로 기준(예시): `/content/drive/MyDrive/data_j/...`

## 1) TAGO BIS 정적 구조

-   `raw/tago/stops_std.csv`
    -   **station_id**: 정류소 ID (문자열; 예: CWB3790...)
    -   **station_name**: 정류소명 (한글)
    -   **lat, lon**: WGS84 좌표 (float)
    -   활용: 정류소 위치(버퍼/격자), 접근성 계산, 산업단지/단지 접근성
        매칭 \`\`\`
