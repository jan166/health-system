# 추출 데이터 (Apple Health export → 정제 CSV)

> 원본 `apple-health-export_2026-06-29.zip`(202MB, export.xml 3.4GB)은 GitHub 100MB 제한 때문에 커밋 못 함(.gitignore, 로컬만).
> 대신 분석에 실제로 쓰는 데이터만 스트리밍 파싱해 아래 CSV로 뽑음 (2026-07-02 생성). Claude 세션은 zip 대신 이 CSV를 읽으면 됨.
> 재생성 스크립트: 세션 스크래치패드 `extract_health.py` (원본 zip에서 iterparse 스트리밍).

## `inbody-timeseries.csv` — 체성분 시계열 (long format, 3,948행)
| 컬럼 | 설명 |
|---|---|
| `date` | 측정일 (YYYY-MM-DD) |
| `datetime` | 측정 시각 (타임존 포함) |
| `metric` | `body_mass_kg`(체중) / `body_fat_pct`(체지방률, **%단위**) / `lean_body_mass_kg`(제지방, lb→kg 변환됨) / `bmi` |
| `value` | 값 |
| `source` | 측정 기기 — **`InBody`**(=인바디, 판단 기준) / `PICOOC`(집 스마트체중계) / `단식 추적기` / `Mi Fit` 등 |

- ⚠️ **판단은 `source=InBody` 로 필터**해서 봐야 함 (CLAUDE.md 5번·11번: 효주는 인바디 체지방으로 판단). PICOOC은 참고용.
- 기간: 2017-09-08 ~ 2026-06-29. 최신 InBody(2026-06-29): 체중 64.1 / 체지방 29.9% / 제지방 44.9.

## `daily-activity.csv` — 일별 활동 합산 (wide format, 3,070일)
| 컬럼 | 설명 |
|---|---|
| `date` | 날짜 |
| `steps` | 걸음수(합) |
| `active_energy_kcal` | 활동 소모 에너지(합) |
| `exercise_min` | Apple 운동시간(분, 합) |
| `distance_km` | 걷기·달리기 거리(km) |

- 연말(Q4) 운동 붕괴 사이클 분석용 (CLAUDE.md 3번). 여러 소스가 겹치면 합산되므로 절대값보단 **추세**로 볼 것.

## `workouts.csv` — 운동 세션별 (786건)
| 컬럼 | 설명 |
|---|---|
| `date` | 날짜 |
| `type` | 운동 종류 (Pilates, Walking, Golf, TraditionalStrengthTraining 등) |
| `duration_min` | 세션 길이(분) |
| `source` | Apple Watch / Fleek 등 |
