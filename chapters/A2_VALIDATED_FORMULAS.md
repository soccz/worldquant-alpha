# Appendix A2: 검증된 수식 카탈로그

> 기본 설정: USA / TOP3000 / Delay 1 / Neutralization SUBINDUSTRY / Truncation 0.08

---

## Pattern A: 단기 반전 x Amihud

```
-rank(ts_mean(returns, 3)) * rank(ts_mean(abs(returns) / adv20, 21))
```

| 항목 | 값 |
|------|-----|
| 예상 Sharpe | 1.8 - 2.1 |
| 예상 Fitness | 1.2 - 1.5 |
| 예상 TVR | 22 - 32% |
| 최적 Decay | 5 - 6 |
| 데이터셋 | PV 단독 |
| 변형 | ts_decay_linear(returns, 5), ts_zscore(returns, 5) |

**논리**: 2-5일 과잉반응 반전 + 비유동성 프리미엄(Amihud 2002). 유동성 낮은 종목에서 반전이 더 강하고, Amihud의 느린 변화가 TVR을 억제.

**대표 통과 알파**:
- PV04e: S=2.05, F=1.40, TO=30%, Decay 5
- PV01d: S=2.02, F=1.46, TO=27%, Decay 5 (ts_decay_linear 변형)
- PL02: S=2.08, F=1.39, TO=33%, Decay 4 (close/ts_mean(close,5) 변형)

---

## Pattern B: Intraday x Score Derivative

```
-rank(ts_mean((close - open) / open, 5)) * rank(growth_potential_rank_derivative)
```

| 항목 | 값 |
|------|-----|
| 예상 Sharpe | 2.1 - 2.4 |
| 예상 Fitness | 1.5 - 1.8 |
| 예상 TVR | 28 - 36% |
| 최적 Decay | 4 |
| 데이터셋 | PV x Fundamental Scores |
| Score 변형 | multi_factor_acceleration, relative_valuation, cashflow_efficiency |

**논리**: 인트라데이(장중) 과잉반응 반전 + 펀더멘탈 개선 추세. Score Derivative의 분기별 변화가 TVR 자연 억제.

**대표 통과 알파**:
- B4_09: **S=2.41**, F=1.76, TO=32%, Decay 4 (ts_decay_linear, **최고 성과**)
- B4_03: S=2.36, F=1.59, TO=36%, Decay 4 (ts_mean, lookback 3)
- R3_35: S=2.17, F=1.58, TO=28%, Decay 4 (ts_mean, lookback 5)
- B4_01: S=2.19, F=1.60, TO=28%, Decay 4 (multi_factor_acceleration)
- B4_02: S=2.17, F=1.58, TO=28%, Decay 4 (relative_valuation)
- B4_05: S=2.16, F=1.57, TO=28%, Decay 4 (cashflow_efficiency)

---

## Pattern C: Additive 조합 (A + B)

```
(-rank(ts_mean((close-open)/open, 5)) * rank(growth_potential_rank_derivative))
  + (-rank(ts_mean(returns, 3)) * rank(ts_mean(abs(returns)/adv20, 21)))
```

| 항목 | 값 |
|------|-----|
| 예상 Sharpe | 1.7 - 2.2 |
| 예상 Fitness | 1.1 - 1.7 |
| 예상 TVR | 21 - 28% |
| 최적 Decay | 5 - 6 |
| 데이터셋 | PV x Scores (혼합) |
| SC | < 0.5 (기존 단일 패턴 대비) |

**논리**: 두 독립 패턴의 OR 결합. 어느 한쪽만 신호를 내도 포지션 형성. TVR 자연 감소 + SC 감소.

**대표 통과 알파**:
- B4_18: S=2.24, **F=1.70**, TO=28%, Decay 5 (Intraday-Score + Reversal-Amihud)
- B4_20: S=2.20, F=1.68, TO=28%, Decay 5 (decay_linear-Amihud + Intraday-Score)

---

## Pattern D: group_rank Residual

```
(-group_rank(ts_mean((close-open)/open, 5), subindustry))
  * rank(growth_potential_rank_derivative)
```

| 항목 | 값 |
|------|-----|
| 예상 Sharpe | 1.6 - 2.0 |
| 예상 Fitness | 1.0 - 1.3 |
| 예상 TVR | 28 - 33% |
| 최적 Decay | 4 - 5 |
| 데이터셋 | PV x Scores (산업 중립) |
| SC | 구조적으로 낮음 (rank vs group_rank 차이) |

**논리**: 산업 내 상대 순위 기반 반전. Sub-Universe에 강하고, rank() 기반과 SC 분리.

**대표 통과 알파**:
- B4_16: S=2.19, F=1.54, TO=31%, Decay 4 (Intraday-GPD)
- B4_15: S=1.95, F=1.25, TO=33%, Decay 5 (Reversal-Amihud)

---

## ICQ7 Batch — Top 5

### 1. Dispersion Compression (A1)
```
-rank(ts_delta(earnings_per_share_standard_deviation
  / (abs(earnings_per_share_median_value) + 0.001), 63))
```
| Decay | Universe | 예상 SC vs 기존 |
|-------|----------|----------------|
| 6 | TOP3000 | 0.3 - 0.5 |

**논리**: EPS 추정 분산 축소 = 불확실성 해소 → 주가 상승 드리프트. returns/Amihud 미포함.

### 2. IV Slope Normalization (B1)
```
-rank(ts_delta(implied_volatility_call_30 - implied_volatility_call_90, 20))
```
| Decay | Universe | 예상 SC vs 기존 |
|-------|----------|----------------|
| 5 | TOP3000 | 0.4 - 0.6 |

**논리**: 단기 IV 급등 후 정상화 = 이벤트 공포 해소. 순수 비가격 신호.

### 3. Competitor Relative Reversal x Score (C1)
```
-rank(ts_mean(rel_ret_comp, 5)) * rank(growth_potential_rank_derivative)
```
| Decay | Universe | 예상 SC vs 기존 |
|-------|----------|----------------|
| 4 | TOP3000 | 0.4 - 0.6 |

**논리**: 경쟁사 대비 과매수 반전 + 성장 점수 개선. Relationship 데이터셋 최초 활용.

### 4. Beta Stability x Intraday (D3)
```
(-rank(ts_mean((close-open)/open, 5)))
  * (1 - rank(beta_last_30_days_spy / (beta_last_360_days_spy + 0.001)))
```
| Decay | Universe | 예상 SC vs 기존 |
|-------|----------|----------------|
| 4 | TOP3000 | 0.6 - 0.75 |

**논리**: 베타 안정(단기/장기 비율 낮음) 종목만 인트라데이 반전. Systematic Risk 데이터셋 활용.

### 5. Accrual x Dispersion Drop (E3)
```
(-rank(ts_delta((income - cashflow_op) / (assets + 0.001), 252)))
  * (-rank(ts_delta(earnings_per_share_standard_deviation
    / (abs(earnings_per_share_median_value) + 0.001), 63)))
```
| Decay | Universe | 예상 SC vs 기존 |
|-------|----------|----------------|
| 6 | TOP3000 | 0.3 - 0.5 |

**논리**: 이익 품질 개선(발생액 감소) + 불확실성 해소(추정 분산 감소). 2개 비가격 데이터셋 교차. SC 최저 예상.
