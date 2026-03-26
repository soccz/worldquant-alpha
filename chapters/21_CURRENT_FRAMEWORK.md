# Chapter 21: 현재의 프레임워크

*"This is not an ending. It's a waypoint."*

---

## 지도 위의 현재 위치

여기까지 오는 데 ICQ 6세대, 760개 이상의 알파 아이디어, 425번의 시뮬레이션, 그리고 셀 수 없는 실패가 필요했다. 이 장은 지금까지의 여정을 하나의 프레임워크로 압축하고, 아직 가지 않은 길을 표시하는 지도다.

---

## 현재 자산: 숫자로 보는 상태

| 항목 | 수치 |
|------|------|
| ICQ1-6 알파 아이디어 | 760+ |
| 시뮬레이션 실행 | 425+ |
| IS Check 전체 통과 | 34 |
| 제출 가능 (SC 포함) | 6+ |
| 실제 제출 | 3+ |
| IS Check 통과율 | 1.4% |
| 최고 Sharpe (B4_09) | 2.41 |
| 최고 Fitness (B4_09) | 1.76 |

34개의 통과 알파는 7개 그룹으로 분류된다. Intraday x Score(6개), Reversal x Amihud(5개), Reversal x Score(4개), group_rank Residual(5개), Additive 조합(4개), 3-Dataset(1개), Standalone(3개). 각 그룹에서 1-2개를 선택하면, SC 0.7 미만으로 8-12개의 독립 알파를 제출할 수 있다.

---

## 프레임워크의 핵심: 세 기둥

760개의 아이디어와 425번의 시뮬레이션에서 증류된 프레임워크는 세 개의 기둥으로 서 있다.

### 기둥 1: 단기 반전 베이스

BRAIN의 2019-2023 백테스트 환경에서, 수익의 가장 안정적인 원천은 2-5일 단기 반전이다. 모멘텀이 아니라 반전. 이것은 학술 문헌의 일반적 결론(모멘텀이 가장 robust한 anomaly)과 정면으로 충돌하지만, 425번의 시뮬레이션이 일관되게 확인한 사실이다.

반전이 작동하는 이유에 대한 가설: BRAIN 참가자의 대다수가 모멘텀을 사용하기 때문에 모멘텀 프리미엄이 이미 소진되었고, 반전은 상대적으로 덜 crowded하다. 또한 2019-2023 기간의 시장 구조(코로나 쇼크, 급격한 섹터 로테이션)가 반전에 유리했을 수 있다.

반전의 두 가지 형태:
- **가격 반전**: `-rank(ts_mean(returns, 3))` — 직접적, 높은 TVR
- **인트라데이 반전**: `-rank(ts_mean((close-open)/open, 5))` — 더 정밀하고, 약간 낮은 TVR

인트라데이 반전이 가격 반전보다 일관되게 높은 Sharpe를 보인다(2.1-2.4 vs 1.8-2.1). 장중 움직임은 소매 투자자의 과잉반응을 더 순수하게 포착하는 것으로 추정된다.

### 기둥 2: 교차 데이터셋 상호작용

단기 반전 단독은 Fitness 벽을 넘지 못한다. TVR이 너무 높기 때문이다. 반전 신호에 다른 데이터셋의 신호를 곱하면 두 가지 효과가 발생한다:

1. **TVR 감소**: Score Derivative나 Amihud 같은 느리게 변하는 신호가 빠른 반전의 회전을 억제한다.
2. **SC 분리**: 다른 데이터셋을 사용하면 기존 제출과의 상관이 구조적으로 낮아진다.

검증된 상호작용 파트너:
- **Amihud** (`rank(ts_mean(abs(returns)/adv20, 21))`): 비유동성 프리미엄. Sharpe 1.8-2.1.
- **Score Derivative** (growth_potential, multi_factor_acceleration 등): 펀더멘탈 개선 추세. Sharpe 2.1-2.4.
- **Volatility Regime** (`rank(ts_std_dev(returns, 5) / ts_std_dev(returns, 60))`): 변동성 체제 전환. Sharpe 1.5-1.8.
- **Sentiment** (`rank(ts_mean(scl12_sentiment, 63))`): 장기 감성 스무딩. 독립적 축.

작동하지 않는 상호작용:
- 펀더멘탈 252일 변화: 너무 느려 D1에서도 약함
- Options 단독: 신호 강도 부족
- 4개 이상 신호 곱셈: 과적합

### 기둥 3: 연산자 다양성

같은 신호 쌍이라도 연산자를 바꾸면 다른 알파가 된다. 이것이 SC를 관리하면서 제출 수를 늘리는 핵심 기법이다.

- `ts_mean()` → `ts_decay_linear()`: 최근 데이터에 더 높은 가중. SC 0.05-0.10 감소.
- `rank()` → `group_rank(subindustry)`: 산업 내 상대 순위. SC 0.10-0.15 감소.
- `rank()` → `ts_zscore()`: z-score 정규화. SC 0.05-0.10 감소.
- 곱셈 → 가산: 조합 논리 변경. SC 0.15-0.25 감소.

이 세 기둥의 조합 공식:

```
Alpha = [단기 반전 변형] * [교차 데이터셋 파트너] (+ [추가 축 가산])
         ↓                    ↓                        ↓
    ts_mean/decay_linear   Amihud/Score/Vol        Sentiment/다른패밀리
    rank/group_rank        다른 데이터셋             SC 감소
    2~5일                  21~63일
```

---

## 다음 전선: ICQ7 패밀리

34개의 통과 알파는 모두 같은 패밀리에서 왔다. 반전 x Amihud, 반전 x Score. 구조적으로 다른 패밀리를 개척하지 않으면, SC 벽에 부딪혀 제출 수를 늘릴 수 없다.

ICQ7은 5개의 새로운 패밀리를 설계했다.

### Family A: Analyst Dispersion

EPS 추정의 표준편차 변화를 핵심 신호로 사용한다. returns나 Amihud를 전혀 포함하지 않으므로, 기존 9개 제출과의 SC가 0.3-0.5로 예상된다.

```
# A1: Dispersion Compression — 불확실성 해소 드리프트
-rank(ts_delta(earnings_per_share_standard_deviation
  / (abs(earnings_per_share_median_value) + 0.001), 63))
```
EPS 추정의 분산이 축소되면 불확실성이 해소된 것이고, 이는 주가 상승으로 이어진다는 논리다.

### Family B: IV Term Structure

옵션 내재변동성의 기간 구조(30일 vs 90일 IV 차이)의 변화를 사용한다. 순수 비가격 신호.

```
# B1: IV Slope Normalization — 이벤트 공포 해소
-rank(ts_delta(implied_volatility_call_30 - implied_volatility_call_90, 20))
```
단기 IV가 장기 IV보다 급등한 후 정상화되는 과정을 포착한다.

### Family C: Relationship Signals

경쟁사/고객 대비 상대 수익률(`rel_ret_comp`, `rel_ret_cust`)을 사용한다. 공급망 정보 전파.

```
# C1: Competitor Relative Reversal x Score
-rank(ts_mean(rel_ret_comp, 5)) * rank(growth_potential_rank_derivative)
```
경쟁사 대비 과매수 상태에서 반전. Cohen & Frazzini(2008)의 공급망 모멘텀을 BRAIN으로 번역.

### Family D: Beta/Risk Structure

베타의 안정성(단기/장기 베타 비율)을 필터로 사용한다.

```
# D3: Beta Stability x Intraday Reversal
(-rank(ts_mean((close-open)/open, 5)))
  * (1 - rank(beta_last_30_days_spy / (beta_last_360_days_spy + 0.001)))
```
베타가 안정적인 종목에서만 인트라데이 반전을 실행. 기존 패스들과 SC 분리.

### Family E: Fundamental Repair

장기 펀더멘탈 개선(Accrual 변화, 자사주 매입)을 사용한다.

```
# E3: Accrual x Dispersion Drop (3-dataset)
(-rank(ts_delta((income - cashflow_op) / (assets + 0.001), 252)))
  * (-rank(ts_delta(eps_std / (abs(eps_median) + 0.001), 63)))
```
이익 품질 개선(발생액 감소) + 불확실성 해소(추정 분산 감소). 두 개의 비가격 데이터셋 교차로 SC 0.3-0.5 예상.

---

## 열린 질문들

이 프레임워크는 완성되지 않았다. 해결되지 않은 질문들이 있다.

**D0 전략의 최적화.** D0는 IS Score의 1/3을 차지하지만, 아직 체계적으로 공략하지 않았다. D1 알파의 D0 변환(lookback 절반 + Decay +2)이 이론적으로 유망하지만, 실전 검증이 부족하다.

**Sub-Universe 구조적 해결.** group_rank이 도움이 되지만, 근본적으로 대형주와 소형주 모두에서 작동하는 신호를 어떻게 설계할 것인가? 교차 데이터셋 상호작용이 유니버스 불변인 경향이 있다는 관찰은 있지만, 충분한 검증이 없다.

**Crowding의 동적 측정.** ICQ9의 군집 탐지 프록시는 이론적으로 설계되었지만, 실제 시뮬레이션은 아직이다. 횡단면 분산 수축이 정말로 군집 과열의 신뢰할 수 있는 지표인지는 실전 데이터가 답해야 한다.

**OS 성과의 불확실성.** IS Check를 통과한 34개 중 OS에서 살아남는 비율은 알 수 없다. IS의 25%와 OS의 75%가 합산되는 구조에서, IS 통과는 필요조건이지 충분조건이 아니다.

**새로운 데이터셋.** Report Footnotes, Systematic Risk Metrics, Ravenpack 등 아직 필드 확인이 되지 않은 데이터셋이 있다. 이 데이터셋들이 열리면 완전히 새로운 SC 슬롯이 생긴다.

---

## 프레임워크의 본질

모든 복잡성을 걷어내면, 이 프레임워크는 하나의 문장으로 요약된다:

**단기 과잉반응을 역이용하되, 교차 데이터셋으로 필터링하고, 연산자 다양성으로 SC를 관리하라.**

이것은 단순한 규칙이다. 하지만 이 규칙에 도달하기 위해 655개의 아이디어를 탐색하고, 425번의 시뮬레이션을 실행하고, 98.6%의 실패를 관찰해야 했다. 단순한 규칙의 뒤에는 복잡한 탐색의 잔해가 쌓여 있다.

---

## 이것은 끝이 아니다

이 저널의 17장에서 21장까지는 하나의 호(arc)를 이룬다. 다중 에이전트가 655개의 아이디어를 생산하고(17장), 군중을 읽는 메타-전략을 개발하고(18장), 개별 알파를 오케스트라로 조합하고(19장), 세 개의 벽에 부딪히고(20장), 현재의 프레임워크를 정리했다(21장).

하지만 이것은 종결이 아니라 경유지(waypoint)다. ICQ7의 5개 패밀리가 시뮬레이션을 기다리고 있다. 군집 탐지 프록시가 실전 검증을 기다리고 있다. 미확인 데이터셋이 탐사를 기다리고 있다.

760개의 아이디어 중 대부분은 실패할 것이다. 그것은 확실하다. 하지만 그 실패들이 남긴 패턴 — Fitness는 구조로 해결하라, SC는 데이터셋을 바꿔라, 모멘텀을 버리고 반전을 택하라 — 이 패턴들이 다음 세대의 알파를 만드는 나침반이 된다.

1.4%의 통과율은 낮다. 하지만 0%가 아니다. 그리고 그 1.4%의 알파들은 425번의 시련을 견딘 것들이다. 우리는 그것을 믿는다.

다음은 부록이다. 연산자, 수식, 참고문헌. 실전에서 필요한 레퍼런스들이다.
