# 데이터셋 인벤토리 + 중주제 합산·고도화 설계서

---

## Ⅰ. 데이터셋 인벤토리

### B1. 정류장 × 청년 인구

#### b1_analysis_panel_with_youth_density_filled.csv
역할: B1 개별분석 최종본 (정류장 밀도 + 청년 인구 패널, 결측치 보정 포함)  
컬럼
- adm_cd7 : 법정동 코드(7자리)
- adm_nm : 행정구역명
- year : 연도
- stops_count : 정류장 수
- stops_per_km2 : 면적당 정류장 수
- coverage_pct : 커버리지 비율
- stops_per_10k_youth : 청년 1만 명당 정류장 수
- pop_total : 총인구
- youth_20_34 : 청년 인구(20–34세)
- youth_share : 청년 비율
- pop_fill_source : 인구 보정 출처

#### population_youth_panel.csv
역할: 청년 인구 패널 원본 (B1~B6 공통 활용 가능)

#### population_youth_panel_dedup.csv
역할: 중복 제거된 청년 인구 패널

#### stan_code_changwon.csv
역할: 행정구역 코드 매핑 표준화 테이블

---

### B2. 아파트 임대료

#### B2_molit_apt_rent_clean_20250913_035219.csv
역할: 국토부 아파트 임대차 정제 최종본 (단위 보정, B2 개별분석 핵심)

#### B2_ready_for_corr_apt.csv
역할: 상관분석용 패널 (교통 + 청년 인구 + 임대료)

#### B2_molit_apt_master_20250913_033922.csv
역할: 아파트 임대차 원본 마스터 (단위 정제 전)

#### B2_molit_rent_master_20250913_034335.csv
역할: 주택 임대 전반 마스터 (연립, 다세대 포함)

---

### B3. 소형주택 수요–공급

#### B3_ready_final_with_small_share_fix3.csv
역할: B3 개별분석 최종본 (소형주택 점유율 + 교통 + 청년 인구)

#### B3_ready_for_corr_proxy.csv
역할: 상관분석용 경량화 패널 (소형주택 proxy + 교통 + 청년 인구)

#### B3_ready_for_corr_small.csv
역할: 상관분석 대안본 (소형주택 점유율을 가구수/면적 기준으로 세분화)

#### B3_small_proxy_by_emd.csv
역할: 읍면동 단위 소형주택 proxy 데이터

---

### B4. 교통 혼잡 × 청년 인구

#### B4_congestion_youth_emd_2024.csv
역할: B4 개별분석 최종본 (교통 혼잡 × 청년 인구)

#### B4_congestion_proxy_all_2024.csv
역할: 혼잡 프록시 확장 집계본

#### B4_congestion_proxy.csv
역할: 단순 혼잡률 요약본

#### B4_congestion_long.csv
역할: 월별 교통량 원본(long format)

#### B4_corr_congestion_youth_2024.csv
역할: 혼잡 지표 vs 청년 인구 상관분석 결과

---

### B5. 교통 접근성 × 정시성

#### B5_access_features.csv
역할: B5 개별분석 최종본 (정시성 포함 접근성 패널)

#### B5_access_features_v2_no_headway.csv
역할: 정시성 데이터 없는 대체본

#### B5_bus_access_emd.csv
역할: 읍면동 단위 접근성 요약본

#### B5_headway_by_stop_merged.csv
역할: 정류장 단위 배차간격 데이터

#### analysis/B5_access_rent_merged.csv
역할: 교통 접근성 + 임대료 결합 (상관분석용)

#### analysis/B5_summary_core.csv
역할: 분석 결과 요약 리포트

---

### B6. 산업단지 × 청년 인구

#### analysis/B6_industry_youth_merged.csv
역할: B6 개별분석 최종본 (산업단지 + 청년 인구)

#### analysis/B6_correlations.csv
역할: 산업 변수 vs 청년 비율 상관분석 결과

#### analysis/B6_corr_candidates.csv
역할: 상관분석 후보 변수 목록

#### B6_industry_emd_agg.csv
역할: 읍면동 단위 산업 집계 데이터

#### B6_factories_matched_step1_2.csv
역할: 공장 원본 주소 매핑본 (정제 전 단계)

---

### 교통 그룹 (원본/마스터)

#### stops_std.csv
역할: 정류장 마스터 (ID, 좌표 기준)

#### routes_std.csv
역할: 노선 마스터

#### route_stops_std.csv
역할: 노선–정류소 매핑

#### headway_station.csv
역할: 정류장 단위 정시성 지표 (평균, 표준편차, CV)

#### arrivals_by_stop.csv
역할: 정류장별 도착 로그 (Headway 산출 원시 데이터)

#### bjd5_stops_routes.csv
역할: 법정동 단위 정류장·노선 집계

#### bjd7_stops_routes.csv
역할: 읍면동 단위 정류장·노선 집계

#### emd_stops_nearest.csv
역할: 읍면동 중심점 → 최근접 정류장 거리

---

### 청년 인구 (공통 종속변수)

#### pop_emd_2024_youth_share.csv
역할: B1~B6 공통 종속변수 (청년 인구 비율)

---

## Ⅱ. 중주제 합산·고도화 설계서 (데이터셋 매핑)

### C1. 소형주택 수요–공급 갭 × 교통 접근성 → 청년 순유출
- Y: pop_emd_2024_youth_share.csv (청년 비율 기반, 순이동률 추가 수집 필요)
- X(주효과): B3_ready_final_with_small_share_fix3.csv, B3_small_proxy_by_emd.csv
- 조절: headway_station.csv, B5_access_features.csv
- 통제: B6_industry_emd_agg.csv, B2_ready_for_corr_apt.csv, supply_rate_vacancy.parquet(외부)
- 분석: 패널FE + 상호작용항, SAR/SEM, GWR

### C2. 소득 × 주거비 부담(RIR) → 청년 순유출
- Y: pop_emd 기반 순이동률
- X(주효과): RIR (외부), B2_molit_rent_master, 청년 소득 분위(외부)
- 상호작용: RIR×소득 분위
- 통제: headway_station.csv, B6_industry_emd_agg.csv
- 분석: 패널FE + 분위회귀

### C3. 가격 ‘충격’(Δ·변동성) vs 청년 이동
- Y: pop_emd 기반 순유출률
- X: B2_molit_apt_rent_clean, B2_ready_for_corr_apt (Δ%, 변동성 산출)
- 통제: headway_station.csv, B6_industry_emd_agg.csv
- 분석: 패널FE + 비선형 모형, 임계값 탐지

### C4. 임차화·소형주택 × 교통 인프라 → 청년 1인가구
- Y: 청년 1인가구 비중 (외부, KOSIS)
- X: B2_molit_rent_master, B3_ready_final_with_small_share_fix3.csv, bjd7_stops_routes.csv
- 방법: 군집분석 → 회귀분석

### C5. 주택공급 여건 × 인구·산업 → 청년 정착
- Y: pop_emd_2024_youth_share.csv 기반 청년 증감률
- X: supply_rate_vacancy.parquet(외부), B3_ready_for_corr_proxy.csv, B6_industry_emd_agg.csv
- 방법: 패널FE + 매개/조절효과 분석

---

## Ⅲ. 공통 분석 파이프라인
1. 데이터 적재 및 월 단위 리샘플링
2. 정제 및 합성지수 생성
3. 패널FE 및 공간모형 추정
4. 상호작용·비선형·임계값 검증
5. 지도/랭킹 산출

---

## Ⅳ. 핵심 산출물
- C1: 갭×정시성 상호작용 등고선 지도, 우선개입 후보지 랭킹
- C2: RIR×소득 상호작용 플롯, ICE 곡선
- C3: 가격 Δ·변동성 곡선, 임계값 전후 비교
- C4: 클러스터 맵, 회귀 영향도 바차트
- C5: 보급률×빈집률 사분면 지도, 경로도
