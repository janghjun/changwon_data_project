좋습니다 👍 방금 올려주신 **중주제 문서**를 읽어보고, 이전에 쓴 데이터셋과 대주제 기획안에서 요구하는 변경점을 비교했습니다. 정리하면, **Day1 대주제 진행을 위한 데이터셋**은 크게 두 가지로 나눌 수 있습니다:

---

# ✅ Day1 필요 데이터셋 정리

## 1. 중주제 때 이미 확보·정제된 데이터 (재사용 가능)

* **교통 (B5)**

  * `/first_data/derived/B5_qol_transit_final.csv`
    → 정류장 접근성, 노선 다양성, 평균 headway (정시성 CV는 보조)

    * 대주제 Day1에서는 여기서 **stop\_density, headway\_mean, headway\_cv**를 그대로 활용 가능.
    * 다만 대주제에서는 **500m 접근성(stop\_density\_500m)** 지표 추가 산출 필요.

* **인구**

  * `/first_data/derived/POP_TOTAL_MONTHLY_adm9.csv`
  * `/first_data/derived/POP_YOUTH20_39_ANNUAL_adm9.csv`
    → 청년 기준이 중주제 당시 20–34세였는데, 대주제에서는 **20–39세로 확정 변경**됨.

    * 이미 20–39세 버전을 정제한 파일이 있으므로, 이를 주 데이터로 사용.

* **주거 (임대료)**

  * `/first_data/derived/cost_emdm_panel_v4_2_winsor.csv`
    → m²당 임대료, 윈저라이즈, log 변환본

    * 대주제에서는 이 값을 그대로 쓰되, \*\*기간을 2020–2023 → 최근 10년(2015\~)\*\*으로 확장 필요.

* **청년 순유입률 (Y)**

  * `/first_data/derived/YOUTH_NET_RATE_EMD_MONTHLY_from_GU_ANNUAL.csv`
    → 구×연 단위를 EMD×월로 downscale한 버전

    * 중주제 한계: 2020–2023까지만, 연→월 상수화라 변동성 낮음
    * 대주제에서는 **KOSIS 월 전입·전출 데이터 확보 → 월 패널 보강** 필요.

* **산업/고용 (B6)**

  * `/first_data/derived/B6_adm9_full.csv`
    → 취업밀도, 산업 구조, 면적 보정 통제변수

---

## 2. 대주제에서 새로 추가/수집해야 하는 데이터

* **건축 인허가 데이터 (공급 예측용)**

  * 현재 first\_data에는 없음 → 신규 수집 필요
  * 후보 출처:

    * KOSIS → 건축 인허가 현황 (시군구, 용도별/세대수별)
    * 국토부 건축 인허가 통계/오픈API
  * 저장 경로(예정):

    * `/first_data/master_sec/permit/building_permits_monthly.parquet`

* **청년 인구 이동 (전입·전출, 수요 추정용)**

  * 중주제는 구×연 수준만 확보했음
  * 대주제에서는 **20–39세 전입·전출(월 단위, 구→EMD 분배)** 데이터가 필요.
  * 신규 수집: KOSIS 주민등록 인구이동통계 (연령·성별·시군구 단위)

* **장기 인구 패널 (10년치)**

  * 중주제: 2020–2023 4년만 활용
  * 대주제: 최근 10년치(예: 2015–2024)를 확보해야 함

* **RIR(주거비 부담률) 계산용 소득 데이터**

  * 중주제에서는 평균 소득 대신 단순 비율만 사용
  * 대주제에서는 **청년 소득 추정치(분위/중위) 보강** 필요 → KOSIS 가계동향조사, 통계청 소득분포 자료

* **현행/계획 공공주택·LH 공고 좌표화 데이터**

  * 비교·검증용: 공공임대 공급 예정지, 시/LH 계획 자료
  * 별도 크롤링/공문 확보 필요

---

# ⚠️ 대주제 진행 시 수정·보완 포인트 (중주제 대비 변경 사항)

1. **청년 인구 정의**

   * 중주제: 20–34세 사용
   * 대주제: 20–39세로 확정
   * → `pop_emd_youth20_39.csv`를 표준으로 사용, 기존 20–34 자료는 폐기.

2. **기간 확장**

   * 중주제: 2020–2023 (결측 때문에 2024 제외)
   * 대주제: 최근 10년치(2015\~2024) 사용
   * → 인구, 임대료, 교통 모두 과거 연장 데이터 확보 필요.

3. **RIR 계산식**

   * 중주제: `RIR = monthly_rent_equiv / youth_income` 단순 산출
   * 대주제: 월세환산(전세→월세) 반영 + 청년 소득 분위/중위값 기반 계수 적용
   * → 청년 소득 자료 수집 및 RIR 재계산 필요.

4. **교통 지표**

   * 중주제: 정시성 CV는 데이터 품질 문제로 제외/민감도용
   * 대주제: 정시성 CV, stop\_density\_500m, transit\_score까지 본선 반영
   * → 기존 B5에서 확장 필요.

5. **Gap 산출**

   * 중주제: 공급 측면 고려 X
   * 대주제: 건축 인허가 → 준공 라그 적용해 12/18개월 공급 예측
   * → 인허가 데이터 신규 수집 및 준공 캘린더 생성 필요.

---

# 📂 최종 정리

## Day1 진행 시 사용할 기존 데이터셋 (first\_data 기준)

* `/derived/B5_qol_transit_final.csv` (교통 접근성/정시성)
* `/derived/POP_TOTAL_MONTHLY_adm9.csv`, `/derived/POP_YOUTH20_39_ANNUAL_adm9.csv` (인구)
* `/derived/cost_emdm_panel_v4_2_winsor.csv` (임대료)
* `/derived/YOUTH_NET_RATE_EMD_MONTHLY_from_GU_ANNUAL.csv` (순유입률, 보완 필요)
* `/derived/B6_adm9_full.csv` (산업/고용 통제)

## Day1에서 새로 수집·보완해야 할 데이터

* `/master_sec/permit/building_permits_monthly.parquet` (인허가 → 준공 라그)
* `/master_sec/pop/youth_migration_monthly.parquet` (20–39세 전입/전출, KOSIS 기반)
* `/master_sec/pop/youth_income_distribution.csv` (청년 소득 추정, RIR 재계산용)
* `/master_sec/housing/public_housing_plan.csv` (LH·시 공급 계획, 좌표화)

---

👉 이렇게 정리하면 Day1에서 **“어떤 건 그대로 쓰고, 어떤 건 새로 수집/수정해야 하는지”** 명확해집니다.

형준님, 다음 단계로 제가 \*\*Day1 ETL 설계 플로우(코드 뼈대)\*\*를 먼저 짜드릴까요, 아니면 지금은 \*\*신규 수집 대상(인허가·전입전출·소득·LH 계획)**에 대한 수집 경로·API/다운로드 방법**부터 정리해드릴까요?
