# 창원시 교통 데이터 수집 파이프라인 정리 (Step0\~Step4 + 데이터 사전)
## https://janghjun.github.io/changwon_data_project/changwon_viz/index.html
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
# “소주제_교통(교통 서비스/접근성)” 분석을 본격 착수하기 전에, 우리가 이미 준비·정제·시각화한 내용 및 각 가설(B1~B5, B4·B6 제외) 정리

⸻

## 1) 데이터셋 인벤토리 (출처·위치·주요 컬럼·정제)

### 1-1. 정류소 위치(완료)
	•	출처/근거: 창원시 BIS OpenAPI/데이터포털(정류소/노선/경유정류소 등). 서비스 URL과 항목은 창원시 오픈API 문서·데이터포털에 명시.  ￼ ￼
	•	파일 위치: data_j/교통/경상남도 창원시_버스정류소 위치정보_20241231 (3).csv
	•	핵심 컬럼(정제 후):
	•	station_id(문자형), station_name, lon, lat, (기타 메타: 관할관청, 설치유형 등)
	•	정제 포인트: 좌표 표준화(x/y/경도/위도 → lon/lat), station_id 문자열 통일(머지 안정성)

### 1-2. Headway/정시성(완료)
	•	출처: 우리가 수집한 BIS 도착시계열 → 스텝3 가공 산출물
	•	파일 위치: data_j/step3_arrivals/headway_station_20250907_211949_utf8sig.csv (우선 사용)
	•	핵심 컬럼:
	•	station_id, headway_mean_s, headway_std_s, headway_cv, n_headways(신뢰도), 파생 bph = 3600 / headway_mean_s
	•	정제 포인트:
	•	n_headways ≥ 3 필터, cv 결측 시 std/mean 보정, station_id 문자열형으로 통일 후 정류소와 조인

### 1-3. 청년 인구(20–34세) 비중(완료)
	•	출처: KOSIS 읍면동 단위 연령/성별 인구(20–24/25–29/30–34 사용).  ￼
	•	파일 위치(파생): data_j/교통/derived/pop_emd_2024_youth_share.csv
	•	핵심 컬럼:
	•	emd_cd(법정동 10자리), youth_20_34, pop_total, youth_share=youth/pop
	•	정제 포인트:
	•	행정구역 문자열에서 괄호 내 10자리 법정동코드 추출, 쉼표 제거 후 숫자화, 2024년 계 기준 합산

### 1-4. 법정동 코드/경계(보강 필요: 경계 파일 or API)
	•	법정동코드: 행안부 행정표준코드 API(법정동). 코드/명세 공식.  ￼ ￼
	•	경계(폴리곤): 통계청 SGIS 경계 API(ADM/LEG 코드로 행정경계 조회 가능).  ￼
	•	현황: 코드값은 인구 파일에서 추출 완료. 경계는 지도·버퍼 가중 평균 계산용으로 추가 필요.

### 1-5. 전월세 실거래(수집/정제 진행 예정)
	•	출처: 국토부 전월세 실거래가 API (아파트/오피스텔/연립·다세대 등). 2025.07 최신 문서 갱신 확인.  ￼
	•	키 포인트:
	•	면적 기반 단가(㎡당 월세), 전세→월세 환산 시 전월세전환율 사용(한국부동산원 정의·시계열 참고).  ￼ ￼
	•	정제 계획:
	•	(1) 거래유형 파싱(월세/전세/보증부월세), (2) 전세·보증부월세는 전환율로 월세 환산, (3) 전용면적으로 ㎡당 단가 산출, (4) 주소→법정동 코드 매칭

⸻

## 2) 지금까지 시각화와 의도(독립/종속, 기대값)
	•	CV 히스토그램: 종속 없음(분포 확인).
	•	의도: 정시성 변동계수의 분포·상하위 분위 파악 → 소주제 B2 회귀 시 임계·완충 구간 가설 설정에 근거 제공.
	•	BPH 히스토그램: 종속 없음(빈도 분포 확인).
	•	의도: 정류소별 운행빈도(시간당) 분포·상하위 구간 탐색 → B1에서 독립변수 후보 분포 확인.
	•	mean vs CV 산점도: x=mean_s, y=cv.
	•	의도: 간헐/과밀 정류소 후보 식별 → B1·B2 대상 구간 선정 참고.
	•	허브 정류소 Top15(경유 노선 수): x=정류소, y=경유노선수.
	•	의도: 환승거점 후보군 추출 → B5(환승 접근성)에서 거점 정의에 활용.

⸻

## 3) 각 가설(B1·B2·B3·B5) 분석 설계

	(요청대로 B4(도로속도/혼잡)·B6(산업단지 접근성)은 보류/후순위)

### B1. 배차빈도(BPH) ↔ 청년 밀집도
	•	질문: 버스 빈도가 촘촘할수록(=BPH↑) 정류장 주변 청년 비중이 높나?
	•	데이터: df_station(정류소+headway) + pop_emd_2024_youth_share + EMD 경계/버퍼
	•	변수 정의:
	•	독립: BPH(정류소 단위), 보조: headway_mean_s(음의 방향)
	•	종속: 정류소 400~500m 버퍼 내 가중 청년비중 youth_share_stop
	•	가중 방법: 버퍼∩EMD 면적비로 youth_share 면적가중 평균
	•	모형/검정:
	•	상관(스피어만/켄달) → 구간별(피크/오프) 비교; 공간가중 회귀(GWR) 또는 공간오차/공간시차(SAR/SEM) 보정
	•	기대 부호: BPH↑ → youth_share_stop (+ )

### B2. 정시성(CV) ↔ ㎡당 전월세 단가
	•	질문: 도착 변동이 작을수록(=CV↓) 임대료 프리미엄이 붙는가?
	•	데이터: 전월세 실거래(아파트·오피스텔·연립다세대) + 전환율로 월세 환산 + 면적 기준 rent_per_m2; 정류소/EMD 조인
	•	전환율 정의·적용 근거(한국부동산원 지표 정의/시계열): 전월세전환율(%) = 월세×12 / (전세가격-월세보증금). 이 지표로 전세를 월세로 환산해 ㎡당 단가 산출.  ￼
	•	변수 정의:
	•	독립: 정시성 headway_cv(낮을수록 정시성↑)
	•	종속: rent_per_m2(거래 기준 또는 월 집계 중앙값)
	•	통제: BPH, CBD거리, 역세권/상업지 비율, 준공연도, 면적구간, 월/계절 더미 등
	•	모형:
	•	로그-선형 회귀: ln(rent_per_m2) ~ cv + bph + controls
	•	기대 부호: CV↓ → 임대료↑ ⇒ cv (–)

### B3. 정류장/노선 밀도 ↔ 소형주택 재고
	•	질문: 대중교통 밀도가 높을수록 소형주택이 많이 분포하는가?
	•	데이터: 정류장/노선 밀도(격자 250~500m), 건축물대장 또는 GIS건물통합정보의 전유면적·주용도(소형 정의 ≤40㎡ 또는 ≤60㎡)
	•	참고: 이 파트의 원자료는 외부 수집 필요(건축HUB/건물통합정보).
	•	변수 정의:
	•	독립: stop_density/route_density, BPH
	•	종속: share_small_units(소형 전유면적 비중)
	•	모형/검정:
	•	격자 단위 상관/회귀 + 공간 자기상관(Moran’s I) 보정
	•	기대 부호: 밀도↑ → 소형주택비중 (+ )

### B5. 환승거점 접근성 ↔ 임대료/입주율
	•	질문: 환승 접근성이 좋은 곳일수록 임대료나(가능하면) 입주율이 높나?
	•	데이터: 허브 정류소(경유노선 수·BPH 상위), 도보망 기반 접근시간(네트워크, 간이 버전은 거리/버퍼 대체), 전월세 실거래 ㎡단가, (가능하면) 공공임대/LH/마이홈 공고·단지정보
	•	마이홈/LH API는 공고 스냅샷 수집이 필요(시계열성 보완)
	•	변수 정의:
	•	독립: access_index = z(BPH)+z(n_routes)−z(CV) 또는 허브까지의 접근시간
	•	종속: rent_per_m2(민간), (선택) 공공임대 경쟁률 proxy
	•	모형:
	•	단면/패널 회귀 + 공간클러스터링(핫스팟)
	•	기대 부호: 접근성↑ → 임대료↑ (+ )

⸻

## 4) 데이터별 컬럼 간단 설명(요약)
	•	정류소: station_id(키), station_name, lon/lat, (보조) 관할관청/유형
	•	Headway: headway_mean_s(평균 배차, s), headway_std_s, headway_cv(정시성), n_headways(신뢰도), bph
	•	인구(EMD): emd_cd, youth_20_34, pop_total, youth_share
	•	전월세(거래): 전월세구분, 전세보증금, 월세, 보증부월세, 전용면적, 지번/법정동코드, 파생 rent_per_m2
	•	전세→월세 환산은 한국부동산원 전월세전환율을 활용(지역·유형 기준 시계열 참조).  ￼

⸻

## 5) 부족/보강 데이터 & 수집 체크리스트
	•	전월세 실거래 API 호출: 아파트/오피스텔/연립다세대(2024–2025, 창원시 코드). 데이터포털 OpenAPI 최신 문서 확인됨.  ￼
	•	법정동 경계(폴리곤): SGIS 경계 API 또는 별도 SHP 확보(면적가중·지도 시각화용).  ￼
	•	전월세 전환율 시계열: 한국부동산원 R-ONE 통계(지역별·유형별). 정의·산식 공식 확인.  ￼
	•	(선택) 허브 정의 보강: 창원 BIS에서 경유정류소목록/노선목록 API로 노선수 갱신·검증.  ￼

⸻

## 6) 바로 실행할 작업 순서(To-Do)
	1.	정류소×인구 결합 준비
	•	SGIS 경계 API 혹은 EMD SHP 추가 → 정류소 500m 버퍼 ∩ EMD 면적가중으로 youth_share_stop 산출.  ￼
	2.	전월세 실거래 수집/정제
	•	아파트/오피스텔/연립다세대 API 호출(창원) → 전세·보증부월세를 전환율로 월세 환산 → rent_per_m2 집계(EMD·월).  ￼ ￼
	3.	B1 분석/시각화
	•	산점: BPH vs youth_share_stop (+ 상관계수)
	•	분위 비교(상·하위 20%): T검정/비모수 검정
	4.	B2 분석/시각화
	•	산점: headway_cv vs rent_per_m2(로그 변환 권장)
	•	회귀: ln(rent_per_m2) ~ cv + bph + 통제 (월/지역 더미 포함)
	5.	B5 접근성 지수/거점 분석
	•	허브(노선수/BPH 상위) 정의 → access_index 산출 → rent_per_m2와 상관·회귀
	6.	B3(필요 데이터 확보 시)
	•	건물 전유면적 기반 share_small_units 만들고, stop_density/route_density와 상관/회귀

⸻

## 참고: 공식 근거/문서
	•	창원 BIS 오픈API 및 데이터포털 등록(노선·정류소·경유정류소 등 상세):  ￼ ￼ ￼
	•	KOSIS 읍면동 단위 연령·성별 인구(20–24/25–29/30–34):  ￼
	•	행안부 행정표준코드(법정동 코드/API·목록):  ￼ ￼
	•	전월세 실거래 API(아파트/연립다세대 등):  ￼
	•	전월세 전환율 정의·시계열(한국부동산원 R-ONE):  ￼

⸻
# 데이터셋 정리
⸻

## 지금 당장 쓰는 핵심셋 (B1·B2·B5 공통 베이스)
### 1.	정류소 위치(공식 CSV)

	•	경로(압축 내): [교통]/경상남도 창원시_버스정류소 위치정보_20241231 (3).csv (이름이 깨져 보이지만 실제로 존재)
	•	행/컬럼(샘플): 3526행 / 정류소ID, 서비스ID, 정류소명, X좌표(=경도), Y좌표(=위도), …
	•	메모: 우리는 이걸 표준화해 station_id(str), station_name, lon, lat로 써왔음.
	•	권장 표준 파일명: transport/stops_cw_20241231.csv

### 2.	Headway(정시성/빈도) – 정류소 단위

	•	경로: step3_arrivals/headway_station_20250907_211949_utf8sig.csv
	•	행/컬럼: 20행 / station_id, headway_mean_s, headway_cv, n_routes, n_pairs
	•	메모: 관측이 20개 정류소로 아주 작음. B1·B2 분석에는 그대로 쓰기 부족 → 아래 2-보조의 “bucketed”로 보완 권장.
	•	권장 표준 파일명: transport/headway_station_latest.csv

### 3.	청년 인구(EMD 2024, 파생)

	•	경로: [교통]/derived/pop_emd_2024_youth_share.csv
	•	행/컬럼: 61행 / emd_cd, youth_20_34, pop_total, youth_share
	•	메모: EMD 경계와 면적가중 조인용으로 이미 완성형.
	•	권장 표준 파일명: transport/derived/pop_emd_2024_youth_share.csv

### 4.	정류소–노선 매핑(허브/노선수 계산용)

	•	경로: raw/tago/route_stops_std_utf8sig.csv
	•	행/컬럼: 16,745행 / station_id, route_id, nodenm, nodeord, updowncd, gpslong, gpslati …
	•	메모: 정류소별 n_routes 계산, 허브 정의(B5) 등에 필요.
	•	권장 표준 파일명: transport/route_stops_std.csv

### 5.	정류소 표준(대안 소스)

	•	경로: raw/tago/stops_std_utf8sig.csv
	•	행/컬럼: 2,725행 / lat, lon, station_id, station_name, nodeno
	•	메모: ①과 중복 역할. 최신 관제 CSV(①)이 있으면 ① 우선, 없을 때 보조로 사용.
	•	권장 표준 파일명: transport/stops_std.csv

### 지금 없는 것(필요): EMD 경계(폴리곤) 파일/SGIS API. B1의 “정류소 500m 버퍼 ∩ EMD 면적가중”을 돌리려면 꼭 필요해.

⸻

### 보조/대안으로 쓰는 것(필요시)
	•	Headway(정류소×노선: bucketed)
	•	경로: step3_arrivals/headway_station_route_bucketed_20250907_211949_utf8sig.csv
	•	행/컬럼: 223행 / station_id, route_id, n_headways, headway_mean_s, headway_std_s, headway_cv
	•	용도: 정류소 단위 headway가 희소(20행)하므로, bucketed를 정류소 단위로 집계해서 bph, cv 보강 → “station_filled”를 만든 뒤 B1/B2에 투입 권장.
	•	권장 표준 파일명(집계 후): transport/headway_station_filled.csv
	•	예) groupby(station_id)로 bph=3600/mean_s, cv는 관측수 가중평균 또는 중앙값
	•	Headway(완화법 relaxed)
	•	경로: step3_arrivals/headway_station_route_relaxed_20250907_211949_utf8sig.csv
	•	행/컬럼: 3행 / 매우 빈약 → 분석에는 사용하지 않기 (진단/테스트용만)
	•	산업단지 GeoJSON(후순위 B6)
	•	경로: industrial_parks/industrial_parks_cw_parklevel.geojson
	•	피처수/속성: 68개 / park_id, …
	•	메모: 현재 범위(B1·B2·B5)에서는 사용 안 함(B6 보류 정책에 따라)
	•	기타 외부 SHP
	•	external/dam_yuch.shp 등 → 교통 분석 범위 밖, 지금은 미사용

⸻

### 소주제_교통 진행상황 점검 (맞게 가는지?)
	•	B1(빈도 BPH ↔ 청년밀집):
	•	정류소/Headway/인구 파생 모두 준비됨.
	•	단, headway_station이 20행이라 희소 → bucketed 집계로 보강한 headway_station_filled.csv를 먼저 만들고 진행하면 안정적.
	•	EMD 경계가 아직 없으므로, 경계 파일/SGIS 호출을 받아 정류소 500m 버퍼 ∩ EMD 면적가중 youth_share_stop 생성 필요.
	•	B2(정시성 CV ↔ ㎡당 전월세 단가):
	•	교통 쪽 변수(BPH/CV)는 위와 동일.
	•	전월세 실거래 API 수집 → 전세 월세환산 → ㎡당 단가 파이프라인이 필요(아파트/오피스텔/연립다세대).
	•	행안부 법정동 코드 매칭으로 emd_cd 기준 집계(월×EMD 중앙값)까지 만들면 회귀 투입 가능.
	•	B5(허브 접근성 ↔ 임대료):
	•	route_stops_std로 정류소별 경유 노선 수(n_routes) 계산, BPH와 합쳐 허브 후보군 선정 → 접근지수 access_index = z(BPH)+z(n_routes)-z(CV) 산출.
	•	임대료는 B2에서 만든 rent_per_m2 결과 재사용.
	•	(도보망은 간이로 거리/버퍼부터, 추후 네트워크 가능)

결론: 방향 맞음. 지금 압축에 든 데이터로 B1은 ‘경계’만 보강하면 즉시, B2/B5는 전월세 수집 파이프라인 추가 후 착수 가능.

⸻

### “실제로 사용하는 파일명” 리스트(정리용 제안)

	•	transport/stops_cw_20241231.csv  ← 기존 공식 정류소 CSV(정규화: station_id, station_name, lon, lat)
	•	transport/route_stops_std.csv    ← 노선-정류소 매핑
	•	transport/headway_station_latest.csv    ← 원본 station headway (희소)
	•	transport/headway_station_filled.csv    ← bucketed 집계로 보강한 station headway(권장 투입본)
	•	transport/derived/pop_emd_2024_youth_share.csv ← EMD 청년비중
	•	(추가 예정) transport/derived/station_youthshare_500m.csv ← 정류소 500m 버퍼∩EMD 면적가중 결과(B1 종속)
	•	(추가 예정) housing/derived/rent_per_m2_emd_month.csv ← 전월세 ㎡단가(월×EMD 중앙값, B2/B5 종속)

⸻

### 바로 다음 액션
	1.	bucketed→station 집계로 headway_station_filled.csv 생성
	2.	EMD 경계 파일(SHP/GeoJSON) 제공 또는 SGIS에서 내려받아 폴더에 넣기
	3.	전월세 API 수집 파이프라인 실행 → rent_per_m2_emd_month.csv 만들기
