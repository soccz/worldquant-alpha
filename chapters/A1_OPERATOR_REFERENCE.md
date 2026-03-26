# Appendix A1: BRAIN 연산자 및 필드 레퍼런스

---

## 사용 가능 연산자

### 순위/정규화
| 연산자 | 구문 | 설명 |
|--------|------|------|
| `rank` | `rank(x)` | 횡단면 순위, 0-1 범위 |
| `group_rank` | `group_rank(x, grouping)` | 그룹 내 순위. grouping: subindustry, industry, sector |
| `group_neutralize` | `group_neutralize(x, grouping)` | 그룹 평균 차감 |
| `bucket` | `bucket(x, n)` | n개 구간으로 분할 |

### 시계열
| 연산자 | 구문 | 설명 |
|--------|------|------|
| `ts_mean` | `ts_mean(x, N)` | N일 이동평균 |
| `ts_delta` | `ts_delta(x, N)` | N일 전 대비 변화 (x - x_N일전) |
| `ts_zscore` | `ts_zscore(x, N)` | (x - ts_mean(x,N)) / ts_std_dev(x,N) |
| `ts_rank` | `ts_rank(x, N)` | N일 시계열 내 순위 (0-1) |
| `ts_std_dev` | `ts_std_dev(x, N)` | N일 표준편차. **ts_std 아님!** |
| `ts_decay_linear` | `ts_decay_linear(x, N)` | 선형 가중 이동평균 (최근 높은 가중) |
| `ts_lag` | `ts_lag(x, N)` | N일 전 값 |

### 수학
| 연산자 | 구문 | 설명 |
|--------|------|------|
| `abs` | `abs(x)` | 절대값 |
| `sign` | `sign(x)` | 부호 (-1, 0, 1) |
| `log` | `log(x)` | 자연로그 |
| `power` | `power(x, n)` | x의 n승 |

---

## 금지된 연산자

| 연산자 | 대체 | 이유 |
|--------|------|------|
| `ts_max` | `ts_rank(x, N)` | BRAIN 미지원. ts_rank로 상위 위치 근사 |
| `ts_min` | `1 - ts_rank(x, N)` | BRAIN 미지원. ts_rank 반전으로 하위 위치 근사 |
| `delay` | `ts_lag(x, N)` | BRAIN에서 delay 대신 ts_lag 사용 |
| `ts_std` | `ts_std_dev(x, N)` | 올바른 함수명은 ts_std_dev |

---

## 알려진 오류 패턴

| 수식 | 오류 | 원인 |
|------|------|------|
| `ts_delta(implied_volatility_call_30, N)` | CONCENTRATED_WEIGHT | IV의 ts_delta가 극단값 생성 |
| `pcr_vol` (단독) | 시뮬 오류 | 필드 미지원 또는 NaN 과다 |
| `fscore_quality` (단독) | 낮은 커버리지 | 30% 커버리지 |
| `rank(returns)` 단독 | 극도로 crowded | 수만 명 사용 |

---

## 핵심 필드 매핑 (Top 20)

### Price Volume (pv1) — 커버리지 100%
| 필드 | 설명 | 용도 |
|------|------|------|
| `close` | 종가 | Intraday 계산 |
| `open` | 시가 | Intraday 계산 |
| `high` | 고가 | Range 계산 |
| `low` | 저가 | Range 계산 |
| `volume` | 거래량 | 거래량 이상 탐지 |
| `vwap` | 거래량 가중 평균가 | 기관 매집 탐지 |
| `returns` | 수익률 | 반전/모멘텀 |
| `adv20` | 20일 평균 거래량 | Amihud 계산 |
| `cap` | 시가총액 (millions) | FCF Yield 등 |
| `sharesout` | 발행주식수 | 자사주매입 탐지 |

### Fundamental (fundamental6) — 커버리지 50%
| 필드 | 설명 | 용도 |
|------|------|------|
| `assets` | 총자산 | Accrual, CF 효율 |
| `cashflow_op` | 영업현금흐름 | CF 품질, Accrual |
| `income` | 순이익 | Accrual |
| `revenue` | 매출 | 성장률 |
| `cogs` | 매출원가 | 총이익률 |
| `ebitda` | EBITDA | 밸류에이션 |

### Analyst Estimate (analyst4) — 커버리지 70-100%
| 필드 | 설명 | 용도 |
|------|------|------|
| `earnings_per_share_median_value` | EPS 추정 중앙값 | Revision |
| `actual_sales_value_quarterly` | 실제 분기 매출 | Sales Surprise |

### Fundamental Scores (model16) — 커버리지 30-100%
| 필드 | 설명 | 용도 |
|------|------|------|
| `growth_potential_rank_derivative` | 성장 잠재력 변화 | 최고 Sharpe 파트너 |
| `cashflow_efficiency_rank_derivative` | CF 효율 변화 | 대체 Score |
| `composite_factor_score_derivative` | 종합 점수 변화 | 범용 |
| `analyst_revision_rank_derivative` | 리비전 순위 변화 | 빌트인 리비전 |

### Options/Volatility (option8) — 커버리지 70%
| 필드 | 설명 | 용도 |
|------|------|------|
| `implied_volatility_call_30` | 30일 콜 IV | VRP 계산 |
| `historical_volatility_30` | 30일 실현 변동성 | VRP 계산 |

### Sentiment (socialmedia12) — 커버리지 99%
| 필드 | 설명 | 용도 |
|------|------|------|
| `scl12_sentiment` | 센티먼트 지수 | 장기 스무딩 |

---

## 올바른 구문 체크리스트

```
ts_std_dev  (O)    ts_std     (X)
ts_lag      (O)    delay      (X)
ts_rank     (O)    ts_max     (X)
ts_rank     (O)    ts_min     (X)
subindustry (O)    sub_industry (X)
adv20       (O)    adv_20     (X)
cashflow_invst (O) cashflow_inv (X)  — 투자활동 CF
```
