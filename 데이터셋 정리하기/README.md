# 데이터셋 인벤토리 (B1~B6 + 교통 그룹)

---

## B1. 정류장 × 청년 인구

### b1_analysis_panel_with_youth_density_filled.csv
- 역할: B1 개별분석 최종본 (정류장 밀도 + 청년 인구 패널, 결측치 보정 포함)
- 컬럼
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

### population_youth_panel.csv
- 역할: 청년 인구 패널 원본 (B1~B6 공통으로 활용 가능)

### population_youth_panel_dedup.csv
- 역할: 중복 제거된 청년 인구 패널

### stan_code_changwon.csv
- 역할: 행정구역 코드 매핑 표준화 테이블 (읍면동/법정동 변환용)

---

## B2. 아파트 임대료

### B2_molit_apt_rent_clean_20250913_035219.csv
- 역할: 국토부 아파트 임대차 정제 최종본 (단위 보정, B2 개별분석 핵심)

### B2_ready_for_corr_apt.csv
- 역할: 상관분석용 패널 (교통 + 청년 인구 + 임대료)

### B2_molit_apt_master_20250913_033922.csv
- 역할: 아파트 임대차 원본 마스터 (단위 정제 전)

### B2_molit_rent_master_20250913_034335.csv
- 역할: 주택 임대 전반 마스터 (연립, 다세대 포함)

---

## B3. 소형주택 수요–공급

### B3_ready_final_with_small_share_fix3.csv
- 역할: B3 개별분석 최종본 (소형주택 점유율 + 교통 + 청년 인구)

### B3_ready_for_corr_proxy.csv
- 역할: 상관분석용 경량화 패널 (소형주택 proxy + 교통 + 청년 인구)

### B3_ready_for_corr_small.csv
- 역할: 상관분석 대안본 (소형주택 점유율을 가구수/면적 기준으로 세분화)

### B3_small_proxy_by_emd.csv
- 역할: 읍면동 단위 소형주택 proxy 데이터 (보조용)

---

## B4. 교통 혼잡 × 청년 인구

### B4_congestion_youth_emd_2024.csv
- 역할: B4 개별분석 최종본 (교통 혼잡 × 청년 인구)

### B4_congestion_proxy_all_2024.csv
- 역할: 혼잡 프록시 확장 집계본 (구간·차종별, 연평균·피크 포함)

### B4_congestion_proxy.csv
- 역할: 단순 혼잡률 요약본

### B4_congestion_long.csv
- 역할: 월별 교통량 원본(long format)

### B4_corr_congestion_youth_2024.csv
- 역할: 혼잡 지표 vs 청년 인구 상관분석 결과

---

## B5. 교통 접근성 × 정시성

### B5_access_features.csv
- 역할: B5 개별분석 최종본 (정시성 포함 접근성 패널)

### B5_access_features_v2_no_headway.csv
- 역할: 정시성 데이터가 없는 대체본

### B5_bus_access_emd.csv
- 역할: 읍면동 단위 접근성 요약본

### B5_headway_by_stop_merged.csv
- 역할: 정류장 단위 배차간격 데이터

### analysis/B5_access_rent_merged.csv
- 역할: 교통 접근성 + 임대료 결합 (상관분석용)

### analysis/B5_summary_core.csv
- 역할: 분석 결과 요약 리포트

---

## B6. 산업단지 × 청년 인구

### analysis/B6_industry_youth_merged.csv
- 역할: B6 개별분석 최종본 (산업단지 + 청년 인구)

### analysis/B6_correlations.csv
- 역할: 산업 변수 vs 청년 비율 상관분석 결과

### analysis/B6_corr_candidates.csv
- 역할: 상관분석 후보 변수 목록

### B6_industry_emd_agg.csv
- 역할: 읍면동 단위 산업 집계 데이터

### B6_factories_matched_step1_2.csv
- 역할: 공장 원본 주소 매핑본 (정제 전 단계)

---

## 교통 그룹 (원본/마스터)

### stops_std.csv
- 역할: 정류장 마스터 (ID, 좌표 기준)

### routes_std.csv
- 역할: 노선 마스터

### route_stops_std.csv
- 역할: 노선–정류소 매핑

### headway_station.csv
- 역할: 정류장 단위 정시성 지표 (평균, 표준편차, CV)

### arrivals_by_stop.csv
- 역할: 정류장별 도착 로그 (Headway 산출 원시 데이터)

### bjd5_stops_routes.csv
- 역할: 법정동 단위 정류장·노선 집계

### bjd7_stops_routes.csv
- 역할: 읍면동 단위 정류장·노선 집계 (분석에서 가장 자주 활용)

### emd_stops_nearest.csv
- 역할: 읍면동 중심점 → 최근접 정류장 거리, 접근성 지표 생성용

---

## 청년 인구 (공통 종속변수)

### pop_emd_2024_youth_share.csv
- 역할: B1~B6 공통 종속변수 (청년 인구 비율)
- 컬럼
  - emd_cd : 법정동 코드
  - youth_20_34 : 청년 인구 수
  - pop_total : 총인구 수
  - youth_share : 청년 인구 비율

---

# 정리

- 개별분석 최종본
  - B1: b1_analysis_panel_with_youth_density_filled.csv
  - B2: B2_molit_apt_rent_clean_20250913_035219.csv
  - B3: B3_ready_final_with_small_share_fix3.csv
  - B4: B4_congestion_youth_emd_2024.csv
  - B5: B5_access_features.csv
  - B6: analysis/B6_industry_youth_merged.csv

- 상관분석용
  - B2_ready_for_corr_apt.csv
  - B3_ready_for_corr_proxy.csv
  - B4_corr_congestion_youth_2024.csv
  - analysis/B5_access_rent_merged.csv
  - analysis/B6_correlations.csv
  - 공통 종속변수: pop_emd_2024_youth_share.csv

- 보조/원본
  - population_youth_panel.csv, stan_code_changwon.csv
  - B2_molit_apt_master, B2_molit_rent_master
  - B3_ready_for_corr_small.csv, B3_small_proxy_by_emd.csv
  - B4_congestion_proxy_all_2024.csv, B4_congestion_long.csv
  - B5_access_features_v2_no_headway.csv, B5_headway_by_stop_merged.csv
  - B6_industry_emd_agg.csv, B6_factories_matched_step1_2.csv
  - 교통 그룹 마스터 (stops_std, routes_std, route_stops_std, headway_station, arrivals_by_stop, bjd7_stops_routes, emd_stops_nearest)
