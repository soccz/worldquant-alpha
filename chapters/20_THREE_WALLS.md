# Chapter 20: 3 Great Walls

*"Theory says it should work. 425 simulations say otherwise."*

---

## 벽 앞에서

655개의 알파 아이디어. 34개의 IS Check 통과. 6개의 제출 가능. 3개의 실제 제출. 통과율 1.4%.

이 숫자들은 냉혹하다. 하지만 그 냉혹함 안에 패턴이 있다. 425개의 시뮬레이션이 부딪힌 벽은 무작위가 아니었다. 세 개의 구조적 벽이 있었고, 각 벽은 특정 유형의 알파를 체계적으로 거부했다.

이 장은 그 세 개의 벽에 대한 보고서다. 어떤 이론도, 어떤 직관도 포함하지 않는다. 오직 시뮬레이션 결과만 말한다.

---

## 벽 1: Fitness < 1.0 — "사후 Decay 조정은 작동하지 않는다"

Fitness는 알파의 종합 품질 지표다.

```
Fitness = Sharpe * sqrt(|Returns| / max(TVR, 0.125))
```

Sharpe가 높아도 TVR(Turnover)이 높으면 Fitness가 떨어진다. 제출 기준은 Fitness 1.0 이상(Average 등급).

425개 시뮬 중 가장 많은 실패 원인이 Fitness < 1.0이었다. 실패 통계에서 LOW_FITNESS는 34건으로 1위를 차지했다. Sharpe 1.3-1.5 범위의 알파들이 TVR 40-60% 때문에 Fitness 0.7-0.95에 머물렀다.

자연스러운 해결책은 Decay를 올려 TVR을 낮추는 것이다. TVR ~ C/(Decay+1) 관계에 따라, Decay를 4에서 6으로 올리면 TVR이 약 30% 감소한다.

이것을 실제로 시도했다. Fitness 근접 실패(0.8-0.99) 15개 알파에 대해 Decay +2와 +4를 적용하여 30개를 재시뮬레이션했다.

**결과: 전부 실패.**

Decay를 올리면 TVR이 내려가지만, Sharpe도 같이 내려갔다. 단기 반전 신호(lookback 3-5일)는 빠른 포지션 전환이 수익의 원천이다. Decay를 올려 포지션 전환을 늦추면, 그 수익 원천 자체가 약해진다. TVR이 30%에서 20%로 내려가도, Sharpe가 1.4에서 1.1로 내려가면 Fitness는 오히려 악화된다.

```
Before: Sharpe 1.4 * sqrt(R/0.30) = Fitness 0.93 (실패)
After:  Sharpe 1.1 * sqrt(R/0.20) = Fitness 0.87 (더 나빠짐)
```

이것이 첫 번째 교훈이다. **Fitness는 수식 구조에서 결정된다.** Decay 조정으로 구제할 수 없다. 처음부터 TVR이 낮은 구조를 가진 알파만이 Fitness 기준을 통과한다.

어떤 구조가 처음부터 낮은 TVR을 갖는가? 실전 데이터를 보면:

| 수식 유형 | 최적 Decay | TVR | Sharpe | Fitness |
|----------|-----------|-----|--------|---------|
| 반전 2-3일 단독 | 4 | 40-50% | 1.5-1.6 | 0.7-0.9 (실패) |
| 반전 x Amihud | 5-6 | 22-32% | 1.8-2.1 | 1.2-1.5 (통과) |
| Intraday x Score | 4 | 28-36% | 2.1-2.4 | 1.5-1.8 (통과) |
| 가산 조합 | 5-6 | 21-28% | 1.7-2.2 | 1.1-1.7 (통과) |

패턴이 보인다. **Amihud 승수**(`rank(ts_mean(abs(returns)/adv20, 21))`)와 **Score Derivative 상호작용**이 TVR을 구조적으로 낮춘다. Amihud는 유동성이 낮은 종목에 가중을 주어 포지션 변동을 완화한다. Score Derivative는 분기별로 천천히 변하는 신호를 곱하여, 가격 기반 신호의 빠른 회전을 억제한다.

**통과하는 알파는 처음부터 TVR이 낮은 구조를 갖고 있다. 실패하는 알파에 Decay를 올려 TVR을 낮추는 것은 작동하지 않는다.**

---

## 벽 2: Self-Correlation > 0.7 — "윈도우를 바꾸는 것은 아무것도 바꾸지 않는다"

Self-Correlation(SC)은 기존 제출 알파와의 상관도다. 0.7 이상이면 "이미 있는 것과 너무 비슷하다"는 판정을 받고 점수가 크게 감소한다.

SC 벽의 잔인함은 단순함에 있다. 같은 구조에서 파라미터만 바꾸면 SC가 0.85 이상이다.

```
# 원본: 3일 반전 x Amihud, Decay 5
-rank(ts_mean(returns, 3)) * rank(ts_mean(abs(returns)/adv20, 21))
→ SC vs 기존: 기준점

# 변형 1: 5일 반전 (윈도우만 변경)
-rank(ts_mean(returns, 5)) * rank(ts_mean(abs(returns)/adv20, 21))
→ SC: 0.88

# 변형 2: Decay 6 (Decay만 변경)
-rank(ts_mean(returns, 3)) * rank(ts_mean(abs(returns)/adv20, 21))
→ SC: 0.92

# 변형 3: 21일 Amihud (Amihud 윈도우 변경)
-rank(ts_mean(returns, 3)) * rank(ts_mean(abs(returns)/adv20, 29))
→ SC: 0.85
```

윈도우를 바꾸든, Decay를 바꾸든, Amihud의 기간을 바꾸든, SC는 0.7 아래로 내려가지 않는다. 같은 악기를 다른 키로 연주해도, 여전히 같은 악기 소리다.

SC를 0.7 미만으로 낮추려면 구조적으로 달라야 한다. 실전에서 확인된 SC 감소 전략:

**데이터셋 교체** — A 신호와 B 신호 중 하나를 완전히 다른 데이터셋으로 바꾼다. PV 반전을 Relationship 반전(`rel_ret_comp`)으로 교체하면 SC가 0.4-0.6으로 떨어진다.

**연산자 교체** — `rank()`를 `group_rank(subindustry)`로 바꾸면 SC가 약 0.15 감소한다. `ts_mean()`을 `ts_decay_linear()`로 바꾸면 추가 0.05-0.10 감소.

**가산 구조** — 곱셈(A*B)에서 가산(A+B)으로 바꾸면 SC가 크게 변한다. 특히 가산의 두 항이 서로 다른 패밀리에서 올 때.

**비가격 신호** — returns, close, open, volume을 전혀 사용하지 않는 알파는 가격 기반 알파와 SC가 낮다. Dispersion(`earnings_per_share_standard_deviation` 변화), IV Term Structure(`implied_volatility_call_30 - implied_volatility_call_90` 변화) 같은 순수 비가격 신호는 기존 제출과의 SC가 0.3-0.5 수준이다.

---

## 벽 3: Sub-Universe 실패 — "대형주에서만 작동하면 소용없다"

TOP3000에서 Sharpe 1.5, Fitness 1.2로 통과한 알파가 Sub-Universe 검증에서 탈락한다. TOP500이나 TOP200으로 축소했을 때 Sharpe가 기준 미만으로 떨어지는 것이다.

실패 통계에서 SUB_UNIVERSE FAIL은 21건이었다. 전체 실패의 약 20%.

원인은 구조적이다. TOP3000의 하위 2500개 종목(중소형주)은 유동성이 낮고, 정보 비효율성이 크고, 개별 종목 특성이 강하다. 이런 특성에 의존하는 신호는 대형주 500개로 축소하면 작동하지 않는다.

대표적인 실패 유형:
- **유동성 프리미엄 의존**: Amihud 기반 알파가 대형주에서는 유동성 차이가 작아 신호 강도가 약해짐
- **정보 비효율성 의존**: 소형주의 느린 가격 반영에 의존하는 반전 알파
- **커버리지 편향**: 애널리스트 커버리지가 낮은 소형주에서만 작동하는 리비전 알파

해결책은 벽 1과 유사하다. 사후 조정이 아니라 구조적 설계가 필요하다.

1. **TOP500/TOP200에서 직접 시뮬레이션**: 처음부터 작은 유니버스에서 설계하면 Sub-Universe 실패가 없다. 단, Sharpe가 낮아질 수 있다.

2. **group_rank 래핑**: `group_rank(signal, subindustry)`는 산업 내 상대 순위만 사용하므로, 유니버스 크기에 대한 의존도가 줄어든다. B4_15(`-group_rank(ts_mean(returns, 3), subindustry) * rank(ts_mean(abs(returns)/adv20, 21))`)는 Sharpe 1.95, Fitness 1.25로 Sub-Universe도 통과했다.

3. **교차 데이터셋 상호작용**: 특정 유니버스 크기에 의존하지 않는 신호. Analyst x Options, Fundamental x Sentiment 같은 교차 구조는 대형주와 소형주 모두에서 작동하는 경향이 있다.

---

## 작동하는 것과 작동하지 않는 것: 425회의 증거

9라운드, 425개 시뮬레이션의 결과를 두 테이블로 요약한다.

### 작동하는 것 (검증됨)

| 패턴 | 대표 알파 | Sharpe | Fitness | 특성 |
|------|----------|--------|---------|------|
| 반전 2-5일 x Amihud | PV04e | 1.8-2.1 | 1.2-1.5 | 가장 안정적 |
| Intraday x Score Derivative | B4_09 | 2.1-2.4 | 1.5-1.8 | 최고 Sharpe |
| 가산 (패턴A + 패턴B) | B4_18 | 1.7-2.2 | 1.1-1.7 | SC 낮음 |
| group_rank 잔차 | B4_16 | 1.6-2.0 | 1.0-1.3 | Sub-Universe 강함 |

### 작동하지 않는 것 (검증됨)

| 시도 | Sharpe 범위 | 실패 이유 |
|------|-----------|----------|
| 모멘텀 21일+ 단독 | -0.7 ~ -0.1 | 2019-2023은 반전장 |
| 펀더멘탈 252일 단독 | 0.2 ~ 0.9 | 신호가 너무 느림 |
| Options/Vol 단독 | -0.5 ~ 0.4 | 신호가 약함 |
| Sentiment 단독 | -0.3 ~ 0.5 | 신호가 약함 |
| Relationship 단독 | -0.2 ~ 0.7 | Coverage 문제 |
| ts_corr 기반 | -0.7 ~ 0.7 | 불안정 |
| Gate Mixing 체제전환 | -0.5 ~ 0.5 | 과적합 |
| Perception Gap | 0.2 ~ 0.5 | 신호 약함 |
| Balance Sheet 변화 | 0.2 ~ 0.5 | 너무 느림 |

"모멘텀이 작동한다"는 금융학의 가장 robust한 발견 중 하나다. 하지만 425개 시뮬레이션은 21일 이상 모멘텀의 Sharpe가 음수라고 말한다. 이론과 실전의 간극이 여기에 있다. BRAIN의 2019-2023 백테스트 기간, TOP3000 유니버스, Subindustry Neutralization이라는 구체적 조건에서, 모멘텀은 작동하지 않는다.

반대로, Intraday 반전(종가-시가 차이의 단기 반전)은 학술 문헌에서 거의 다루지 않는 신호다. 하지만 B4_09의 Sharpe 2.41, Fitness 1.76은 425개 중 최고 성과다. 이론이 아니라 실전이 답을 준 사례다.

---

## 최고의 알파: B4_09

```
-rank(ts_decay_linear((close - open) / open, 5)) * rank(growth_potential_rank_derivative)
```

- Sharpe: 2.41
- Fitness: 1.76
- TVR: 32%
- Decay: 4

이 수식을 해부하면:

`(close - open) / open`은 인트라데이 수익률이다. 장중 상승한 종목은 다음 날 하락하는 경향이 있다(인트라데이 반전). `ts_decay_linear(x, 5)`는 5일간의 가중 이동평균으로, 최근에 더 높은 가중을 준다. `-rank()`는 인트라데이 수익률이 높은 종목일수록 낮은 순위(숏 방향).

`growth_potential_rank_derivative`는 성장 잠재력 점수의 변화율이다. 이 값이 높은 종목은 펀더멘탈 개선이 진행 중이다. `rank()`는 개선이 큰 종목일수록 높은 순위.

두 신호를 곱하면: **단기적으로 과매수(인트라데이 상승)되었지만 펀더멘탈이 개선 중인 종목을 숏하고, 단기적으로 과매도(인트라데이 하락)되었지만 펀더멘탈이 악화 중인 종목을 롱한다.**

이것이 작동하는 이유: 인트라데이 상승은 단기 과잉반응이고, 성장 잠재력 개선은 중기 추세다. 두 시간대의 정보가 교차하면서, 단기 노이즈와 중기 신호를 분리한다. Score Derivative의 느린 변화가 TVR을 자연스럽게 억제하여 Fitness도 높다.

---

## 벽 너머의 풍경

세 개의 벽은 무자비하다. 하지만 벽이 있다는 것은 벽 너머에 무언가가 있다는 뜻이기도 하다.

1.4%의 통과율은 낮아 보이지만, 이는 동시에 이 벽을 넘는 알파가 진짜라는 뜻이기도 하다. 98.6%의 실패가 필터링한 후 남은 것이기 때문이다. IS Check를 통과한 34개, 그중 제출 가능한 6개는 우연의 산물이 아니라 구조적으로 robust한 신호다.

425번의 시뮬레이션이 가르쳐준 것: Fitness는 구조로 해결하라. SC는 데이터셋을 바꿔라. Sub-Universe는 group_rank로 중립화하라. 그리고 무엇보다, 모멘텀 21일을 포기하고 인트라데이 반전 x Score Derivative를 찾아라.

이론은 작동해야 한다고 말한다. 425번의 시뮬레이션은 그렇지 않다고 말한다. 우리는 시뮬레이션을 믿는다.
