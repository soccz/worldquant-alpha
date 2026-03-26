# Chapter 19: 알파의 오케스트라

*"Individual alphas are instruments. The portfolio is the symphony."*

---

## 악기와 오케스트라

B4_09는 Sharpe 2.41, Fitness 1.76의 알파다. `-rank(ts_decay_linear((close - open) / open, 5)) * rank(growth_potential_rank_derivative)`. 이것만 보면 완벽해 보인다. 하지만 이것 하나만으로는 대회에서 이길 수 없다.

BRAIN의 점수 체계는 개별 알파의 품질만 보지 않는다. Self-Correlation을 측정한다. 같은 구조에서 파생된 알파 10개를 제출하면, SC가 0.85 이상으로 치솟고, 대부분이 점수를 잃는다. 포트폴리오의 다양성이 곧 점수다.

오케스트라에서 바이올린 20대는 바이올린 1대보다 크게 울릴 뿐, 새로운 소리를 내지 않는다. 바이올린, 첼로, 오보에, 팀파니가 함께 연주할 때 비로소 교향곡이 된다. 알파 포트폴리오도 마찬가지다.

---

## 5축 직교화: 다양성의 설계도

알파 간 상관을 낮추려면 체계적인 분리가 필요하다. ICQ의 메타 전략 프레임워크는 5개의 직교 축을 정의한다.

**축 1: 데이터셋 분리.** 가장 강력한 분리 방법이다. PV 데이터로 만든 알파와 Analyst 데이터로 만든 알파는 구조적으로 다른 정보원을 사용한다. BRAIN에는 14개 데이터셋이 있고, 가능한 2-데이터셋 교차 조합만 91개다. 같은 데이터셋 조합에서 최대 4개까지가 효율적이고, 그 이상은 SC가 급등한다.

**축 2: 시간 수평선 분리.** 5일 반전과 63일 모멘텀은 같은 returns 데이터를 사용하지만, 상관이 0.3-0.5 수준으로 낮다. 단기(2-5일, D0용)와 중기(21-63일, D1용)로 분리하는 것이 최적이다.

**축 3: 신호 유형 분리.** 모멘텀, 반전, 밸류, 리비전, 변동성, 퀄리티, 유동성. 7개의 anomaly 클러스터 각각에서 1-2개만 제출하고, 클러스터 간 상관을 0.3 미만으로 유지하는 것이 이상적이다.

**축 4: 조합 방법 분리.** 같은 신호 쌍(A, B)이라도 fusion 방식에 따라 상관이 다르다. `rank(A) * rank(B)` (Late Fusion)와 `rank(A * B)` (Early Fusion)의 상관은 약 0.6이다. `rank(A) * rank(B)`와 `rank(A / (B + 0.001))` (Ratio Fusion)의 상관은 약 0.4다. 같은 재료로 다른 요리를 만들 수 있다.

**축 5: 유니버스/중립화 분리.** TOP3000과 TOP500은 다른 종목 집합이다. `rank()`와 `group_rank(subindustry)`는 다른 기준선을 사용한다. 같은 수식이라도 유니버스와 중립화를 바꾸면 상관이 낮아진다.

이 5축에서 각각 최소 2개의 선택지를 확보하면, 이론적으로 2^5 = 32개의 직교 슬롯이 생긴다. 실전에서는 15-20개 수준의 독립 알파 포트폴리오가 가능하다.

---

## 가산 vs 곱셈: AND 논리와 OR 논리

알파를 조합하는 방법은 크게 두 가지다.

**곱셈 조합** (`rank(A) * rank(B)`)은 AND 논리다. A 조건과 B 조건이 동시에 충족되는 종목만 강한 신호를 받는다. 높은 집중도, 높은 확신. 하지만 조건이 동시에 충족되는 종목이 적어 TVR이 높아질 수 있다.

**가산 조합** (`rank(A) + rank(B)`)은 OR 논리다. A 조건이든 B 조건이든 하나만 충족되어도 신호를 받는다. 넓은 커버리지, 낮은 TVR. 두 신호가 구조적으로 다르면 SC가 극적으로 낮아진다.

실전 검증 결과가 이를 확인한다:

```
# 곱셈: Intraday x Score (단일 패턴)
-rank(ts_mean((close-open)/open, 5)) * rank(growth_potential_rank_derivative)
→ Sharpe 2.1~2.4, 하지만 기존 제출과 SC 높음

# 가산: (Intraday x Score) + (Reversal x Amihud) (앙상블)
(-rank(ts_mean((close-open)/open, 5)) * rank(growth_potential_rank_derivative))
  + (-rank(ts_mean(returns, 3)) * rank(ts_mean(abs(returns)/adv20, 21)))
→ Sharpe 2.0~2.3, SC < 0.5
```

가산 조합은 Sharpe가 약간 희석되지만, SC를 극적으로 낮춘다. B4_18(가산 조합)은 Sharpe 2.24, Fitness 1.70으로, 개별 패턴보다 Fitness가 높았다. 두 패턴이 서로 다른 시점에 신호를 내기 때문에 TVR이 자연스럽게 낮아지기 때문이다.

3-Way, 4-Way 가산으로 확장하면 SC는 더 낮아진다:
```
# 4-Way: Intra-GPD + Rev-AMI + Vol Regime + Sentiment
→ 예상 SC < 0.3, Sharpe 1.6~1.9
```

Sharpe 1.6은 개별 패턴의 2.4보다 낮지만, SC 0.3은 대회 점수 체계에서 훨씬 유리하다. 낮은 SC로 제출 가능한 알파 수가 늘어나기 때문이다.

---

## TVR-Decay 관계: 숨겨진 지렛대

Turnover(TVR)와 Decay의 관계는 대략적으로 다음을 따른다:

```
TVR ~ C / (Decay + 1)
```

여기서 C는 신호의 고유 회전 상수다. 단기 반전(lookback 3일)은 C가 크고, 장기 밸류(lookback 252일)는 C가 작다.

이 관계가 중요한 이유는 Fitness 공식 때문이다:

```
Fitness = Sharpe * sqrt(|Returns| / max(TVR, 0.125))
```

같은 Sharpe 1.5에서:
- TVR 10% → Fitness = 1.5 * sqrt(R/0.125) (높음)
- TVR 30% → Fitness = 1.5 * sqrt(R/0.30) (낮음)

구체적으로 계산하면, Returns가 동일하다고 가정할 때:
- TVR 10%: Fitness ~ 1.5 * sqrt(1/0.125) = 1.5 * 2.83 = **4.24** (이론값)
- TVR 30%: Fitness ~ 1.5 * sqrt(1/0.30) = 1.5 * 1.83 = **2.74** (이론값)

실전에서는 Returns가 TVR에 비례하므로 이보다 완화되지만, TVR이 낮을수록 Fitness가 유리한 것은 변하지 않는다.

LESSONS_LEARNED에서 확인된 핵심 교훈: **Decay를 사후에 올려서 Fitness를 구제하는 것은 불가능하다.** Fitness 근접 실패(0.8-0.99) 15개에 대해 Decay +2/+4로 30개를 재시뮬레이션한 결과, 전부 실패했다. Decay를 올리면 TVR이 내려가지만, Sharpe도 같이 내려가기 때문이다.

결론: Fitness는 수식 구조에서 결정된다. Amihud 승수, Score Derivative 상호작용 같은 구조가 처음부터 TVR을 낮게 유지하는 알파만이 Fitness 기준을 통과한다. Decay 조정은 미세 조정이지 구조적 해결책이 아니다.

---

## D0 vs D1: 두 개의 전쟁터

BRAIN의 점수 체계는 두 개의 Delay 설정을 허용한다:

```
IS Score = D1_Score + D0_Score / 3
Merged = 25% * IS + 75% * OS
```

D1(Delay 1)이 주된 점수원이고, D0(Delay 0)는 보너스(1/3 가중)다. 하지만 OS가 75%이므로, D1에서 robust하게 통과하는 것이 최우선이다.

**D1에 적합한 알파**: 느린 정보 반영(PEAD, Revision), 중기 lookback(21-63일), 교차 데이터셋 상호작용. 하루의 정보 지연을 견디는 신호.

**D0에 적합한 알파**: 단기 반전(5일 이하), 뉴스/센티먼트 즉각 반응, IV 스파이크 반전. 당일 진입이 필요한 빠른 신호. D0 Sharpe 기준(2.0 이상)은 D1(1.25)보다 절대값은 높지만, 당일 반영이라 구조적으로 Sharpe가 높게 나오므로 달성 가능하다.

실전 운용 전략: D1에서 Sharpe 1.5 이상인 알파를 찾고, 그것의 D0 버전(lookback 절반 + Decay +2)을 별도 제출한다. D1과 D0 버전은 상관이 낮아 직교 보너스를 받을 수 있다.

---

## Sub-Universe 테스트의 수학

TOP3000에서 통과한 알파가 하위 유니버스(TOP500, TOP200)에서 실패하는 것은 세 번째 벽이다. 이 실패의 수학적 원인을 보면:

TOP3000은 대형주 500개 + 중소형주 2500개로 구성된다. 중소형주에서만 작동하는 신호(예: 유동성 프리미엄)는 TOP3000 전체에서는 좋은 성과를 보이지만, TOP500으로 축소하면 그 신호가 사라진다.

해결책은 세 가지다:

1. **직접 제출**: TOP500이나 TOP200에서 시뮬레이션하고 제출. 유니버스가 작을수록 종목 간 차이가 적어 Sharpe가 낮아지지만, Sub-Universe 검증을 통과할 확률이 높다.

2. **group_rank 래핑**: `group_rank(signal, subindustry)`로 산업 내 상대 순위를 사용하면, 소형주 노이즈가 제거된다. B4_16(group_rank Intraday x Score)은 Sharpe 2.19, Fitness 1.54로 높은 성과를 보였다.

3. **유니버스 불변 구조**: 교차 데이터셋 상호작용은 대형주와 소형주 모두에서 작동하는 경향이 있다. 특정 유니버스 크기에 의존하지 않는 신호가 가장 robust하다.

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

최적 제출 전략: 각 그룹에서 1-2개만 선택. A에서 B4_09, B에서 PL02, D에서 B4_16, E에서 B4_18, F에서 R3_17. 총 5-8개의 구조적으로 독립적인 알파 포트폴리오.

이것이 오케스트라다. B4_09의 바이올린(높은 Sharpe, 단기 intraday), PL02의 첼로(안정적 반전), B4_16의 오보에(group_rank의 독특한 음색), B4_18의 화성(가산 조합의 풍성함), R3_17의 팀파니(3-dataset의 독립성). 각각은 불완전하다. 함께 연주할 때 교향곡이 된다.

다음 장에서는 이 오케스트라가 부딪히는 세 개의 거대한 벽을 다룬다. 이론이 아니라 425번의 시뮬레이션이 증명한, 진짜 벽이다.
