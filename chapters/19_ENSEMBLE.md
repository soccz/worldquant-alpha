# Chapter 19: 알파의 오케스트라

*"Individual alphas are instruments. The portfolio is the symphony."*

---

## 악기와 오케스트라

B4_09는 Sharpe 2.41, Fitness 1.76의 알파다. `-rank(ts_decay_linear((close - open) / open, 5)) * rank(growth_potential_rank_derivative)`. 이것만 보면 완벽해 보인다. 하지만 이것 하나만으로는 대회에서 이길 수 없다.

BRAIN의 점수 체계는 개별 알파의 품질만 보지 않는다. Self-Correlation을 측정한다. 같은 구조에서 파생된 알파 10개를 제출하면, SC가 0.85 이상으로 치솟고, 대부분이 점수를 잃는다. 포트폴리오의 다양성이 곧 점수다.

오케스트라에서 바이올린 20대는 바이올린 1대보다 크게 울릴 뿐, 새로운 소리를 내지 않는다. 바이올린, 첼로, 오보에, 팀파니가 함께 연주할 때 비로소 교향곡이 된다. 알파 포트폴리오도 마찬가지다.

이 장은 이전 17개 장에서 개발한 모든 기법들 -- 교차 데이터셋, 체제 전환, 잔차 추출, 비대칭 정보, 군집 역발상 -- 을 하나의 포트폴리오로 엮는 방법을 다룬다. 이론이 아니라, 34개의 IS Check 통과 알파와 425번의 시뮬레이션 경험에서 나온 실전 지식이다.

---

## 5축 직교화: 다양성의 설계도

알파 간 상관을 낮추려면 체계적인 분리가 필요하다. ICQ의 메타 전략 프레임워크는 5개의 직교 축을 정의한다. 이 5축은 "어떻게 하면 같은 데이터에서 다른 소리를 낼 수 있는가"에 대한 체계적 답이다.

**축 1: 데이터셋 분리.** 가장 강력한 분리 방법이다. PV 데이터로 만든 알파와 Analyst 데이터로 만든 알파는 구조적으로 다른 정보원을 사용한다. BRAIN에는 14개 데이터셋이 있고, 가능한 2-데이터셋 교차 조합만 91개다. 같은 데이터셋 조합에서 최대 4개까지가 효율적이고, 그 이상은 SC가 급등한다.

이것이 왜 가장 강력한지, 실전 데이터가 말해준다. 우리의 11개 제출 알파 전부가 PV 기반(returns, close, open, volume)이다. Score Derivative를 곱하긴 하지만, 핵심 신호는 PV다. ICQ7이 Analyst Dispersion(Family A), Options Term Structure(Family B), Relationship(Family D)을 개척하려는 이유가 여기에 있다. PV에서 벗어나지 않으면, SC 벽을 넘을 수 없다.

현재 우리가 확인한 데이터셋별 쿼타와 위험도를 정리하면:

| 데이터셋 조합 | 알파 수 상한 | SC 위험 | 현재 사용 |
|-------------|------------|---------|----------|
| PV 단독 | 3-4개 | 높음 (crowded) | 11개 (초과!) |
| Analyst 단독 | 2-3개 | 중간 | 0개 |
| Options 단독 | 2-3개 | 낮음 | 0개 |
| Analyst x PV | 3-4개 | 낮음 | 0개 |
| Analyst x Options | 3-4개 | 매우 낮음 | 0개 |
| 3중 교차 | 2-3개 | 사실상 제로 | 1개 (R3_17) |

현재 상태는 명백한 불균형이다. PV 11개, 나머지 거의 0개. 오케스트라로 치면 바이올린만 11대 있고 다른 악기가 없는 것이다.

**축 2: 시간 수평선 분리.** 5일 반전과 63일 모멘텀은 같은 returns 데이터를 사용하지만, 상관이 0.3-0.5 수준으로 낮다. 단기(2-5일, D0용)와 중기(21-63일, D1용)로 분리하는 것이 최적이다.

하지만 경고가 필요하다. 9라운드의 교훈: 5일과 10일의 SC는 0.88이었다. 3일과 7일도 0.85. 같은 시간대 범위 안에서 윈도우만 바꾸는 것은 SC를 낮추지 못한다. 2-5일 범위와 21-63일 범위 사이에서만 의미 있는 분리가 발생한다.

**축 3: 신호 유형 분리.** 모멘텀, 반전, 밸류, 리비전, 변동성, 퀄리티, 유동성. 7개의 anomaly 클러스터 각각에서 1-2개만 제출하고, 클러스터 간 상관을 0.3 미만으로 유지하는 것이 이상적이다.

425번의 시뮬레이션이 보여준 현실: 이 7개 중 BRAIN의 2019-2023 환경에서 실제로 작동하는 것은 반전(Sharpe 1.5-2.4)과 리비전(아직 미검증이지만 이론적 유망)뿐이다. 모멘텀은 음수 Sharpe. 밸류는 단독으로 약함. Options/Vol은 단독으로 약함. 이론적으로는 7개 클러스터가 있지만, 실전에서 쓸 수 있는 것은 2-3개다.

**축 4: 조합 방법 분리.** 같은 신호 쌍(A, B)이라도 fusion 방식에 따라 상관이 다르다. `rank(A) * rank(B)` (Late Fusion)와 `rank(A * B)` (Early Fusion)의 상관은 약 0.6이다. `rank(A) * rank(B)`와 `rank(A / (B + 0.001))` (Ratio Fusion)의 상관은 약 0.4다. 같은 재료로 다른 요리를 만들 수 있다.

실전 응용: B4_09(Late Fusion: `rank(intraday) * rank(score)`)와 동일 신호의 Early Fusion 버전은 SC가 약 0.6으로, 별개 제출이 가능할 수 있다. 이것은 ICQ 15장의 잔차/직교 알파 방법론에서 체계화한 개념이다.

**축 5: 유니버스/중립화 분리.** TOP3000과 TOP500은 다른 종목 집합이다. `rank()`와 `group_rank(subindustry)`는 다른 기준선을 사용한다. 같은 수식이라도 유니버스와 중립화를 바꾸면 상관이 낮아진다.

이 5축에서 각각 최소 2개의 선택지를 확보하면, 이론적으로 2^5 = 32개의 직교 슬롯이 생긴다. 실전에서는 15-20개 수준의 독립 알파 포트폴리오가 가능하다.

---

## 가산 vs 곱셈: AND 논리와 OR 논리

알파를 조합하는 방법은 크게 두 가지다. 이 선택이 포트폴리오의 성격을 결정한다.

**곱셈 조합** (`rank(A) * rank(B)`)은 AND 논리다. A 조건과 B 조건이 동시에 충족되는 종목만 강한 신호를 받는다. 높은 집중도, 높은 확신. 하지만 조건이 동시에 충족되는 종목이 적어 TVR이 높아질 수 있다.

곱셈의 문제를 구체적으로 보면: A가 좋고 B가 중립이면 결과는 중립이다 (0.5 x 0.5 = 0.25). 두 신호 모두 좋아야 작동하므로 "AND" 논리다. 3개 이상 신호를 곱하면 과적합의 영역에 진입한다. 4개 이상 곱셈은 CLAUDE.md에 Anti-Pattern으로 명시한 금기다.

**가산 조합** (`rank(A) + rank(B)`)은 OR 논리다. A 조건이든 B 조건이든 하나만 충족되어도 신호를 받는다. 넓은 커버리지, 낮은 TVR. 두 신호가 구조적으로 다르면 SC가 극적으로 낮아진다.

가산의 강점은 3개 이상 신호 통합에서 드러난다. 곱셈으로 4개를 합치면 anti-pattern이지만, 가산으로 4개를 합치면 안전하다. 각 신호가 독립적으로 기여하므로, 하나가 중립이어도 나머지가 포트폴리오를 지탱한다.

실전 검증 결과가 이를 확인한다:

```
# 곱셈: Intraday x Score (단일 패턴)
-rank(ts_mean((close-open)/open, 5)) * rank(growth_potential_rank_derivative)
  Sharpe 2.1~2.4, 하지만 기존 제출과 SC 높음

# 가산: (Intraday x Score) + (Reversal x Amihud) (앙상블)
(-rank(ts_mean((close-open)/open, 5)) * rank(growth_potential_rank_derivative))
  + (-rank(ts_mean(returns, 3)) * rank(ts_mean(abs(returns)/adv20, 21)))
  Sharpe 2.0~2.3, SC < 0.5
```

가산 조합은 Sharpe가 약간 희석되지만, SC를 극적으로 낮춘다. B4_18(가산 조합)은 Sharpe 2.24, Fitness 1.70으로, 개별 패턴보다 Fitness가 높았다. 두 패턴이 서로 다른 시점에 신호를 내기 때문에 TVR이 자연스럽게 낮아지기 때문이다. 이것은 결정적 발견이었다. TVR 감소가 Sharpe 감소보다 크면, Fitness가 오히려 올라간다.

가산 vs 곱셈의 선택 기준을 정리하면:

- 두 신호의 상관이 높으면 -> 곱셈 (interaction 강화)
- 두 신호의 상관이 낮으면 -> 가산 (독립 정보 합산)
- A 또는 B의 coverage가 70% 미만이면 -> 가산 (결측 방어)
- 3개 이상 신호 -> 가산 필수 (곱셈은 anti-pattern)

3-Way, 4-Way 가산으로 확장하면 SC는 더 낮아진다:
```
# 4-Way: Intra-GPD + Rev-AMI + Vol Regime + Sentiment
  예상 SC < 0.3, Sharpe 1.6~1.9
```

Sharpe 1.6은 개별 패턴의 2.4보다 낮지만, SC 0.3은 대회 점수 체계에서 훨씬 유리하다. 낮은 SC로 제출 가능한 알파 수가 늘어나기 때문이다. 대회의 본질은 하나의 최고 알파가 아니라, 독립적인 다수의 양질 알파다.

---

## 잔차의 힘: 팩터를 빼면 알파가 보인다

ICQ 15장의 잔차 알파 방법론은 앙상블 구성에서 또 다른 차원을 열어준다. 알려진 팩터의 노출을 제거한 후 남는 잔차(residual)가 순수 알파다.

BRAIN에는 ts_regression이 없으므로 프록시 잔차를 사용한다:

```
residual_proxy = rank(signal) - rank(known_factor)
```

또는 group_rank을 활용한 조건부 잔차:

```
# RES-2: 잔차 모멘텀 (업종 효과 제거)
rank(ts_mean(returns, 21)) - group_rank(ts_mean(returns, 21), subindustry)
```

잔차가 raw 신호보다 나은 이유는 세 가지다. 첫째, crowding 감소. raw momentum은 수만 명이 제출한다. momentum - industry factor는 거의 아무도 제출하지 않는다. 둘째, 직교성. 기존 팩터와 상관이 구조적으로 낮으므로, 포트폴리오에서 추가적 알파를 제공한다. 셋째, OS 안정성. 팩터 노출이 없으므로 시장 체제 변화에 강건하다.

실전에서 이것이 어떻게 작동했는가? B4_16은 `(-group_rank(ts_mean((close-open)/open, 5), subindustry)) * rank(growth_potential_rank_derivative)`이다. 핵심은 `group_rank`. 이것이 B4_09(rank 사용)과의 SC를 구조적으로 낮춘다. B4_16은 Sharpe 2.19, Fitness 1.54. B4_09보다 약간 낮지만, SC가 다르기 때문에 별개 제출이 가능하다.

이중 잔차(Double Residual)는 더 극단적이다:

```
# RES-7: 잔차 모멘텀 x 잔차 Revision
(rank(ts_mean(returns, 21)) - group_rank(ts_mean(returns, 21), subindustry))
  * (rank(ts_delta(earnings_per_share_median_value, 20))
     - group_rank(ts_delta(earnings_per_share_median_value, 20), subindustry))
```

두 신호 모두 팩터를 제거한 잔차끼리 교차. 가장 순수한 종목 고유 알파. Crowding은 사실상 제로에 가깝다.

---

## TVR-Decay-Fitness: 숨겨진 삼각관계

Turnover(TVR)와 Decay의 관계는 대략적으로 다음을 따른다:

```
Fitness = Sharpe * sqrt(|Returns| / max(TVR, 0.125))
```

이 수식이 포트폴리오 구성의 핵심 제약이다. 같은 Sharpe에서 TVR이 낮을수록 Fitness가 높다.

LESSONS_LEARNED에서 확인된 핵심 교훈을 다시 한 번 강조한다: **Decay를 사후에 올려서 Fitness를 구제하는 것은 불가능하다.** Fitness 근접 실패(0.8-0.99) 15개에 대해 Decay +2/+4로 30개를 재시뮬레이션한 결과, 전부 실패했다. Decay를 올리면 TVR이 내려가지만, Sharpe도 같이 내려가기 때문이다.

이것이 앙상블 구성에서 의미하는 바: 가산 조합이 Fitness 측면에서 유리하다. 두 개의 독립적 신호를 더하면, 두 신호가 서로 다른 시점에 활성화되기 때문에 포지션 전환이 느려진다. TVR이 자연스럽게 낮아진다. B4_18(가산 조합)의 TVR 28%는 B4_09(단일 곱셈)의 TVR 32%보다 낮다. 이 4%p 차이가 Fitness에서 의미 있는 개선을 만든다.

실전 데이터를 테이블로 정리하면:

| 수식 유형 | 최적 Decay | TVR | Sharpe | Fitness |
|----------|-----------|-----|--------|---------|
| 반전 2-3일 단독 | 4 | 40-50% | 1.5-1.6 | 0.7-0.9 (실패) |
| 반전 x Amihud | 5-6 | 22-32% | 1.8-2.1 | 1.2-1.5 (통과) |
| Intraday x Score | 4 | 28-36% | 2.1-2.4 | 1.5-1.8 (통과) |
| 가산 조합 | 5-6 | 21-28% | 1.7-2.2 | 1.1-1.7 (통과) |

패턴이 보인다. **Amihud 승수**와 **Score Derivative 상호작용**이 TVR을 구조적으로 낮춘다. Amihud는 유동성이 낮은 종목에 가중을 주어 포지션 변동을 완화한다. Score Derivative는 분기별로 천천히 변하는 신호를 곱하여, 가격 기반 신호의 빠른 회전을 억제한다.

가산 조합은 이 효과를 더 강화한다. (빠른 신호) + (느린 신호)를 더하면, 전체 회전 속도가 두 신호의 중간 어딘가에 안착한다.

---

## D0 vs D1: 두 개의 전쟁터

BRAIN의 점수 체계는 두 개의 Delay 설정을 허용한다:

```
IS Score = D1_Score + D0_Score / 3
Merged = 25% * IS + 75% * OS
```

D1(Delay 1)이 주된 점수원이고, D0(Delay 0)는 보너스(1/3 가중)다. 하지만 OS가 75%이므로, D1에서 robust하게 통과하는 것이 최우선이다.

**D1에 적합한 알파**: 느린 정보 반영(PEAD, Revision), 중기 lookback(21-63일), 교차 데이터셋 상호작용. 하루의 정보 지연을 견디는 신호. Lookback 최소 10일 이상. 5일 이하 단기 신호는 하루 지연으로 크게 약화된다.

**D0에 적합한 알파**: 단기 반전(5일 이하), 뉴스/센티먼트 즉각 반응, IV 스파이크 반전. 당일 진입이 필요한 빠른 신호. D0 Sharpe 기준(2.0 이상)은 D1(1.25)보다 절대값은 높지만, 당일 반영이라 구조적으로 Sharpe가 높게 나오므로 달성 가능하다.

D1에서 D0로 변환하는 공식:

```
# 방법 1: Lookback 절반
alpha_d0 = alpha_d1(lookback / 2)

# 방법 2: Decay 증가 (+2~4)
alpha_d0 = alpha_d1(decay + 2)

# 방법 3: 단기 확인 신호 추가
alpha_d0 = alpha_d1 * rank(ts_mean(returns, 5))
```

실전 운용 전략: D1에서 Sharpe 1.5 이상인 알파를 찾고, 그것의 D0 버전을 별도 제출한다. D1과 D0 버전은 Delay 차이로 인해 상관이 낮아 직교 보너스를 받을 수 있다. 이론적으로 5개의 D1 코어 알파에서 5개의 D0 변환 = 총 10개. 이것이 ICQ 13장의 메타 전략에서 제시한 10일 제출 플랜의 핵심이다.

하지만 솔직히, 우리는 아직 D0를 체계적으로 공략하지 않았다. 이것은 미탐색 영역이다.

---

## Sub-Universe 테스트의 수학과 실전

TOP3000에서 통과한 알파가 하위 유니버스(TOP500, TOP200)에서 실패하는 것은 세 번째 벽이다. 이 실패의 수학적 원인을 보면:

TOP3000은 대형주 500개 + 중소형주 2500개로 구성된다. 중소형주에서만 작동하는 신호(예: 유동성 프리미엄)는 TOP3000 전체에서는 좋은 성과를 보이지만, TOP500으로 축소하면 그 신호가 사라진다.

앙상블 구성에서 이것이 의미하는 바: 특정 유니버스에 의존하지 않는 신호만이 robust한 포트폴리오의 재료가 된다. 교차 데이터셋 상호작용은 대형주와 소형주 모두에서 작동하는 경향이 있다. Analyst 데이터(EPS revision)는 대형주에서 coverage가 높으므로 TOP200에서 오히려 강할 수 있다. Options 데이터도 마찬가지다.

group_rank 래핑은 Sub-Universe 문제의 가장 실전적 해결책이다. `group_rank(signal, subindustry)`는 산업 내 상대 순위만 사용하므로, 유니버스 크기에 대한 의존도가 줄어든다. B4_15의 사례가 이를 증명한다: Sharpe 1.95, Fitness 1.25로 Sub-Universe도 통과했다.

---

## 통합 포트폴리오: 네 개의 독립 알파

ICQ2 31장의 통합 포트폴리오 설계는 "다른 시장 상태에서 작동하는 알파들의 조합"이라는 원칙을 세웠다.

상관이 낮은 알파 N개의 포트폴리오 Sharpe:

```
Sharpe_portfolio = Sharpe_avg * sqrt(N) / sqrt(1 + (N-1) * avg_correlation)
```

N=4, avg_correlation=0.1이면: Sharpe_portfolio는 개별 평균의 약 1.9배. 개별 Sharpe 1.0짜리 4개를 상관 0.1로 조합하면 포트폴리오 Sharpe가 약 1.9에 도달한다. 다양성의 수학적 보상이다.

이 원칙에 따라 설계된 4개 알파의 시장 상태별 작동 매트릭스:

| 시장 상태 | Alpha 1 (구조적) | Alpha 2 (quiet 모멘텀) | Alpha 3 (이벤트) | Alpha 4 (volatile 반전) |
|---------|---------|---------|---------|---------|
| quiet + 관심 없음 | 강 | 강 | 약 | 약 |
| quiet + 관심 있음 | 강 | 강 | 약 | 약 |
| volatile + 이벤트 | 강 | 약 | 강 | 강 |
| volatile + 이벤트 없음 | 강 | 약 | 약 | 강 |

모든 시장 상태에서 최소 2개가 작동한다. Alpha 1(항상 작동하는 구조적 특성)이 포트폴리오의 안정적 기저 수익을 제공하고, 나머지가 체제에 따라 추가 수익을 올린다. Alpha 2와 Alpha 4는 반대 조건에서 작동하는 쌍이다. quiet 체제에서는 모멘텀(Alpha 2), volatile 체제에서는 반전(Alpha 4). 둘을 합산하면 체제에 따라 자동으로 방향이 바뀌는 Convexity 구조가 된다.

이것은 아름다운 이론이다. 하지만 425번의 시뮬레이션이 보여준 현실은 이보다 거칠다. 체제 전환 자체를 시뮬에서 정확히 포착하는 것이 어렵고, Gate Mixing 방식의 체제전환 알파는 대부분 과적합으로 실패했다(Sharpe -0.5 ~ 0.5). 이론과 실전의 간극이 여기에도 있다.

---

## 앙상블의 실전 레시피

34개 IS Check 통과 알파를 7개 그룹으로 분류한 결과:

| 그룹 | 대표 | 알파 수 | Sharpe 범위 | 특성 |
|------|------|---------|------------|------|
| A: Intraday x Score | B4_09 | 6 | 2.16-2.41 | 최고 Sharpe, SC 높음 |
| B: Reversal x Amihud | PV04e | 5 | 1.59-2.08 | 가장 안정적 |
| C: Reversal x Score | SC04a | 4 | 1.77-2.15 | A와 B의 중간 |
| D: group_rank Residual | B4_16 | 5 | 1.64-2.19 | SC 구조적으로 낮음 |
| E: Additive 조합 | B4_18 | 4 | 1.69-2.24 | SC 낮고 Fitness 높음 |
| F: 3-Dataset | R3_17 | 1 | 1.92 | SC 가장 낮을 확률 |
| G: Standalone | PV10a | 3 | 1.67-1.82 | 독립 |

각 그룹 안에서의 SC는 0.85 이상이다. 그룹 A 안의 6개 알파는 score derivative만 다를 뿐 구조가 같다(growth_potential vs multi_factor_acceleration vs relative_valuation vs cashflow_efficiency). 이 6개 중 제출할 수 있는 것은 1-2개뿐이다.

하지만 그룹 간 SC는 다르다. A와 B의 SC는 약 0.65-0.70. A와 D의 SC는 약 0.55-0.65(group_rank 효과). A와 E의 SC는 약 0.45-0.55(가산 구조 효과). A와 F의 SC는 약 0.40-0.50(3-dataset 효과).

최적 제출 전략: 각 그룹에서 1-2개만 선택.

1. **B4_09** (A그룹) -- 바이올린: 최고 Sharpe 2.41, Intraday x Score
2. **B4_18** (E그룹) -- 화성: 가산 조합, Sharpe 2.24, Fitness 1.70
3. **B4_16** (D그룹) -- 오보에: group_rank, Sharpe 2.19, SC 낮음
4. **PL02** (B그룹) -- 첼로: 가격 수준 기반, 안정적
5. **R3_17** (F그룹) -- 팀파니: 3-dataset, SC 가장 낮을 확률

총 5-8개의 구조적으로 독립적인 알파 포트폴리오. 이것이 현재 가능한 최선의 오케스트라다.

하지만 이 오케스트라에는 근본적 약점이 있다. 모든 악기가 같은 음계(단기 가격 반전)에서 연주하고 있다는 것이다. Score Derivative, Amihud, group_rank 같은 "반주"는 다르지만, "멜로디"는 하나다. 진정한 교향곡이 되려면, ICQ7의 새로운 패밀리 -- Analyst Dispersion, IV Term Structure, Relationship, Beta Structure, Fundamental Repair -- 에서 완전히 다른 멜로디를 가진 악기가 필요하다.

---

## 조건부 앙상블: 체제에 따라 다른 악기를

ICQ9의 100개 앙상블 수식 중, 가장 정교한 카테고리는 조건부 앙상블(Conditional Ensemble)이다. 시장 상태에 따라 다른 알파에 가중을 주는 구조다.

```
# 변동성 체제 전환: 저변동 -> Score 강화, 고변동 -> Amihud 강화
rank(ts_std_dev(returns, 5)) * (reversal x Amihud)
  + rank(-ts_std_dev(returns, 5)) * (intraday x Score)
```

저변동성 환경에서는 Score Derivative의 느린 신호가 유효하고, 고변동성 환경에서는 Amihud의 유동성 필터가 더 중요하다는 논리다.

이 접근의 매력은 직관적이다. 하지만 LESSONS_LEARNED의 "Gate Mixing 체제전환: Sharpe -0.5 ~ 0.5"라는 기록이 경고를 보낸다. 조건부 전환은 이론적으로 아름답지만, 실전에서는 전환 시점의 불확실성이 신호를 잠식한다. bucket()을 사용한 이산 전환은 연속 스케일링보다 명확한 threshold를 제공하지만, 그만큼 과적합 위험도 높다.

결론적으로, 조건부 앙상블은 "고급 무기"다. 기본 가산 앙상블이 통과한 후에만 시도할 가치가 있다. 먼저 단순한 것을 작동시키고, 그 다음에 복잡한 것을 시도하라.

---

## 제출 순서의 전략

ICQ 13장의 메타 전략은 10일 제출 플랜을 제시했다. 그 핵심 원칙들을 정리하면:

1. **확실한 것부터**: PV 단독 알파(필드 확인 불필요, 즉시 제출)
2. **높은 기대값부터**: S-Tier -> A-Tier -> B-Tier
3. **직교성 높은 것부터**: 다른 데이터셋/신호 유형 우선
4. **D1 먼저, D0 다음**: D1이 점수 기여 3배
5. **TOP3000 먼저, TOP200 다음**: 안전 확보 후 점수 극대화

하루 최대 2,000점. 하루에 점수 상한에 도달하면 추가 제출은 다음 날로 이월된다. 따라서 매일 꽉 채우는 것이 최적이다. "아끼는 것"은 비효율이다.

각 알파 제출 전 체크리스트:
- Sharpe >= 1.25 (D1) / >= 2.00 (D0)
- Fitness >= 1.0 (Average)
- 1% < TVR < 70%
- 기존 제출 알파와 SC < 0.7
- Decay 적정 여부
- Sub-Universe Test (TOP200 제출 시)

---

## 아직 연주되지 않은 음표

이 오케스트라에는 빈 의자가 많다. 아직 연주되지 않은 악기들:

**Analyst Dispersion (ICQ7 Family A)**: returns도 Amihud도 사용하지 않는 순수 analyst 신호. `-rank(ts_delta(earnings_per_share_standard_deviation / (abs(earnings_per_share_median_value) + 0.001), 63))`. 기존 11개 제출과 SC 0.3-0.5 예상. 이것이 통과하면, 오케스트라에 완전히 새로운 음색이 추가된다.

**IV Term Structure (ICQ7 Family B)**: `-rank(ts_delta(implied_volatility_call_30 - implied_volatility_call_90, 20))`. 현물 가격 정보가 전혀 없는 순수 options 신호. 또 다른 새로운 악기.

**Relationship Signals (ICQ7 Family D)**: `-rank(ts_mean(rel_ret_comp, 5)) * rank(growth_potential_rank_derivative)`. 경쟁사 대비 상대 수익률. returns가 아니라 rel_ret_comp를 사용하므로, 같은 "반전"이라도 다른 미시구조를 본다.

이 빈 의자들이 채워질 때, 바이올린만의 독주에서 진정한 교향곡으로 전환될 수 있다.

다음 장에서는 이 오케스트라가 부딪히는 세 개의 거대한 벽을 다룬다. 이론이 아니라 425번의 시뮬레이션이 증명한, 진짜 벽이다.
