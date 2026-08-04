# 추출 데이터 (Apple Health export → 정제 CSV)

> 원본 `apple-health-export_2026-06-29.zip`(202MB, export.xml 3.4GB)은 GitHub 100MB 제한 때문에 커밋 못 함(.gitignore, 로컬만).
> 대신 분석에 실제로 쓰는 데이터만 스트리밍 파싱해 아래 CSV로 뽑음 (2026-07-02 생성). Claude 세션은 zip 대신 이 CSV를 읽으면 됨.
> 재생성 스크립트: 세션 스크래치패드 `extract_health.py` (원본 zip에서 iterparse 스트리밍).

## `inbody-timeseries.csv` — 체성분 시계열 (long format, 3,948행)
| 컬럼 | 설명 |
|---|---|
| `date` | 측정일 (YYYY-MM-DD) |
| `datetime` | 측정 시각 (타임존 포함) |
| `metric` | `body_mass_kg`(체중) / `body_fat_pct`(체지방률, **%단위**) / `lean_body_mass_kg`(제지방, lb→kg 변환됨) / `bmi` / `waist_cm`(허리둘레, 수기측정) |
| `value` | 값 |
| `source` | 측정 기기 — **`InBody`**(=인바디, 판단 기준) / `PICOOC`(집 스마트체중계) / `단식 추적기` / `Mi Fit` / `manual`(효주 수기 입력) 등 |

- ⚠️ **판단은 `source=InBody` 로 필터**해서 봐야 함 (CLAUDE.md 5번·11번: 효주는 인바디 체지방으로 판단). PICOOC은 참고용.
- 기간: 2017-09-08 ~ (계속 갱신 중, 세션에서 스크린샷 확인될 때마다 수동 추가). 2026-06-30~08-05 구간은 여름 컷 세션 대화 중 스크린샷으로 수동 반영(자동 재추출 아님 — 원본 export.xml에 아직 없음).

## `2026-cut-daily-intake.csv` — 2026 여름 컷 일별 섭취 칼로리 (27행)
> 출처: 세션 대화 중 다이어리앱 스크린샷 수동 반영. 매크로(탄단지) 세부는 CSV에 없음 — 필요하면 대화 로그 참고.

| 컬럼 | 설명 |
|---|---|
| `date` | 날짜 (2026-07-01~) |
| `calories_kcal` | 일별 총 섭취 kcal |
| `source` | 다이어리앱 |

- 미기록일(7/14·15·25·27~31 등)은 스크린샷에 없어서 CSV에도 없음. 7/27~31은 생리 기간.

## `daily-activity.csv` — 일별 활동 합산 (wide format, 3,070일)
| 컬럼 | 설명 |
|---|---|
| `date` | 날짜 |
| `steps` | 걸음수(합) |
| `active_energy_kcal` | 활동 소모 에너지(합) |
| `exercise_min` | Apple 운동시간(분, 합) |
| `distance_km` | 걷기·달리기 거리(km) |

- 연말(Q4) 운동 붕괴 사이클 분석용 (CLAUDE.md 3번). 여러 소스가 겹치면 합산되므로 절대값보단 **추세**로 볼 것.

## `2024-cut-diary-daily.csv` — 2024 바디프로필 컷 일별 섭취·소모 (86행)
> 출처: 다이어리 앱 캡처 `IMG_4925~4930.PNG`를 손으로 디지털화(2026-07-02). **Apple Health엔 2024 4~5월이 비어 있어(당시 워치 미착용) 이 컷의 일별 실측은 이게 유일한 소스.**

| 컬럼 | 설명 |
|---|---|
| `date` | 날짜 (2024-04-01 ~ 2024-06-30) |
| `intake_kcal` | "먹었어요" 일별 섭취 kcal |
| `burn_kcal` | "태웠어요" 일별 **운동소모** kcal (기초대사 제외, 활동분만) |
| `net_intake_kcal` | 섭취 − 운동소모 (⚠️ TDEE 적자 아님. 운동소모만 뺀 값) |

- 월평균: 4월 섭취1427/소모617, 5월 1379/572, 6월 1238/745 → CLAUDE.md 3번 요약과 일치(검증됨).
- 이상치: **5/28 섭취 3123**(폭식/치팅데이), 6/1 소모 2483(대운동일). 6월 18~22일·후반부는 미기록(컷 마무리로 트래킹 축소).

## `workouts.csv` — 운동 세션별 (786건)
| 컬럼 | 설명 |
|---|---|
| `date` | 날짜 |
| `type` | 운동 종류 (Pilates, Walking, Golf, TraditionalStrengthTraining 등) |
| `duration_min` | 세션 길이(분) |
| `source` | Apple Watch / Fleek 등 |
