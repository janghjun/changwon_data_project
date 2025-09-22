최종 분석안(확정) — 중주제 3

생활여건(QOL: 환경·문화·교통) × 주거비 → 청년 순유입

1) 연구질문(재확인·정밀화)

Q1. QOL이 높을수록 청년(20–39세) 순유입률(EMD×월)이 커지는가? (기대부호 +)

Q2. **주거비(Cost: ㎡임대료 로그 / RIR)**가 높을수록 QOL 효과가 약화/역치를 보이는가? (QOL×Cost < 0 기대)

2) 데이터셋 매핑(최종) — first_data 기준

키·단위: EMD×월 패널(가능 범위 최대), 좌표 WGS84→EPSG:5186(거리/버퍼), 면적보정(밀도).

(A) 필수(내부)
| source\_path                                                     | target\_path                                                                            | purpose                                                                                 |
| ---------------------------------------------------------------- | --------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| first\_data/B2/B2\_molit\_apt\_rent\_clean\_20250913\_035219.csv | first\_data/master\_sec/M3/dataset/B2/B2\_molit\_apt\_rent\_clean\_20250913\_035219.csv | **Cost**(㎡임대료): log + 1% winsor, `yyyymm` 월 패널                                          |
| first\_data/B5/B5\_access\_features.csv                          | first\_data/master\_sec/M3/dataset/B5/B5\_access\_features.csv                          | **QOL\_transit**: `n_stops`, `n_routes_mean`, `walk_min_to_stop`, `headway_mean_s_mean` |
| (원본) `/mnt/data/202012_202212_연령별인구현황_연간.csv`                    | first\_data/master\_sec/M3/dataset/common/raw\_age\_2020\_2022.csv                      | 연령별 인구(연간) 원본                                                                           |
| (원본) `/mnt/data/202212_202412_연령별인구현황_연간.csv`                    | first\_data/master\_sec/M3/dataset/common/raw\_age\_2022\_2024.csv                      | 연령별 인구(연간) 원본                                                                           |
| (원본) `/mnt/data/202012_202412_주민등록인구및세대현황_월간.csv`                | first\_data/master\_sec/M3/dataset/common/raw\_pop\_household\_2020\_2024\_monthly.csv  | 총인구/세대(월간) 원본                                                                           |
| first\_data/common/pop\_emd\_2024\_youth\_share.csv              | first\_data/master\_sec/M3/dataset/common/pop\_emd\_2024\_youth\_share\_old.csv         | **기존 20–34세** 지표(아카이브)                                                                  |
| *(산출)* — 20–39세 재정제                                              | first\_data/master\_sec/M3/dataset/common/pop\_emd\_youth20\_39.csv                     | **재정제본**: 청년(20–39) 인구·비율(EMD×연/월)                                                      |

(B) 권장(보조)
| source\_path                                        | target\_path                                                                  | purpose          |
| --------------------------------------------------- | ----------------------------------------------------------------------------- | ---------------- |
| first\_data/B6/B6\_industry\_emd\_agg.csv           | first\_data/master\_sec/M3/dataset/derived/B6\_industry\_emd\_agg.csv         | **통제**: 취업/산업 밀도 |
| first\_data/derived/emd\_transport\_with\_youth.csv | first\_data/master\_sec/M3/dataset/derived/emd\_transport\_with\_youth.csv    | 키 정합 보조(교통+청년)   |
| (중주제3 폴더) `최종_청년인구이동 X ... RIR ... .xlsx`           | first\_data/master\_sec/M3/dataset/common/youth\_RIR\_netmig\_2020\_2023.xlsx | RIR, 순유입률(구×연)   |
| (중주제3 폴더) `인구_청년_순이동률_2020.01_2025.06_clean.xlsx`   | first\_data/master\_sec/M3/dataset/common/youth\_netmig\_202001\_202506.xlsx  | 순유입률(월/분기)       |

3) 변수·지수화(버전 A/B 병행)

QOL 합성지수(0–100): 표준화 z 후 방향성 정규화(역지표: PM, 범죄, 헤드웨이 등은 부호 반전)

레이어: 환경(–PM, +공원/녹지) + 문화(+시설/좌석/운영시간) + 교통(+접근성/빈도)

A안: 동일가중(커뮤니케이션 강점)

B안: PCA 가중(데이터 주도·강건성)

Cost: log(㎡임대료) 또는 RIR(가능 시 병기; 한국 전월세 혼재 한계 주석)

통제: employment_density(B6), univ_access(추가), small_housing_share(B3), new_apt_share(신축), 시간·구 FE

4) 분석 단계(모형 사다리)

EDA & 역치 탐색

분위(p20↔p80) 순유입률 차이, Cost 분위별 단순기울기

GAM로 비선형/포화점 확인(예: QOL p70↑ 한계효과 둔화)

패널 고정효과(FE) + 상호작용(본선)

표준오차: HAC(Newey–West) 또는 Driscoll–Kraay

공간 의존: Moran’s I → 유의 시 SAR/SEM 보조

비선형·이질성 강화

GBM/RandomForest → PDP/ICE로 QOL×Cost 상호작용/역치 시각화

(선택) Causal Forest / DML로 조건부 효과(HTE) 확인

준실험(가능 시)

DID/이벤트 스터디(문화시설 개관, 공원 리모델링, BRT 정류장 신설 등)

Sun–Abraham 이벤트 스터디, Synthetic Control(사례분석)

민감도/강건성

QOL 가중(A vs B), Cost(임대료 vs RIR), 라깅(1–2개월)

표준오차(HAC vs DK), 시계열 롤링 검증

5) “청년 = 20–39세” 재정제 (확정)

원본 연령대 파일(연간)을 이용해 20–24/25–29/30–34/35–39 합산 → 20–39세 재계산

최초 산출은 region×year 기준 파일을 만들고, EMD×월 자료가 확보되는 대로 키를 세분화

기존 20–34세 지표는 .../pop_emd_2024_youth_share_old.csv 로 보관(재현성)

실행 스크립트(스켈레톤) — 바로 사용 가능

기능: (1) 20–39세 재정제(region×year), (2) QOL/패널 결합 스켈레톤

위치: Download: m3_pipeline_skeleton.py

사용 전 준비: 아래 원본을 매핑 경로로 복사

/mnt/data/202012_202212_연령별인구현황_연간.csv → first_data/master_sec/M3/dataset/common/raw_age_2020_2022.csv

/mnt/data/202212_202412_연령별인구현황_연간.csv → first_data/master_sec/M3/dataset/common/raw_age_2022_2024.csv

실행(Colab):

!python /content/mnt/data/m3_pipeline_skeleton.py
# 산출: first_data/master_sec/M3/dataset/common/pop_youth20_39_region_year.csv


이후 stan_code_changwon.csv(행정동 코드 매핑)와 월별 전입·전출이 들어오면 region → emd_cd7 매핑 및 EMD×월 보정 로직을 이어서 붙여줄게요(가중: 인구/세대/사업체 등 합리적 분배 기준).

6) 산출물(확정)

QOL 합성지수 지도(EMD, 버전 A/B)

QOL×Cost 상호작용 곡선(PDP/ICE, 단순기울기)

유입 잠재 랭킹(QOL 상위·Cost 중간, 신축 비중 보정)

본선 회귀표(FE + 상호작용, 표준오차 2종 병기)

민감도 표(가중·라깅·표준오차·RIR 대체 등)
