# Chapter 05: 8가지 다른 시선 — 같은 시장을 여덟 개의 렌즈로 보다

---

같은 시장을 여덟 개의 렌즈로 봤다. 각각이 다른 것들이 보였다.

Chapter 04에서 우리는 신호 라이브러리를 만들고 조합 규칙을 정했다. 그런데 조합 방식 자체가 `rank(A) * rank(B)` 하나뿐이었다. 블록은 다양해졌지만 조립법이 하나라면 결국 비슷한 구조의 알파만 쏟아낼 뿐이었다. Self-correlation 0.7 벽에 걸려 대량 탈락하는 참사가 그 증거였다.

방법론을 다양화해야 했다. 같은 EPS revision과 같은 IV 데이터를 써도, 그것을 바라보는 수학적 관점을 바꾸면 완전히 다른 알파가 나온다. 우리는 여덟 가지 관점을 체계화했다.

---

## M1: Early vs Late Fusion — 곱하기의 순서가 바꾸는 것

기존 방식(Late Fusion)은 각각을 먼저 rank한 뒤 곱한다.

```
rank(ts_delta(est_eps, 20)) * (-rank(ts_delta(iv_30d, 20)))
```

Early Fusion은 하나의 rank() 안에서 상호작용시킨다.

```
rank(ts_delta(est_eps, 20) * (-ts_delta(iv_30d, 20)))
```

차이가 미묘해 보이지만 본질적이다. Late Fusion에서 A가 극단값이고 B가 중간값이면 곱이 약해진다. 두 신호의 channel coupling이 끊어지는 것이다. Early Fusion에서는 A×B 자체가 하나의 값으로 ranking된다. 극단적 조합이 살아남는다.

우리 연구에서 concat_a(여러 채널을 합친 모형)의 IC가 0.057이었고, 개별 채널은 0.007~0.02에 불과했다. 공유 projection 안에서 channel coupling이 발생했기 때문이다. BRAIN에서의 Early Fusion은 이 원리를 번역한 것이다.

비율(Ratio Fusion) `rank(A / (B + 0.001))`도 같은 맥락에서 시도했다. 분모가 0에 가까울 때의 위험만 관리하면, 곱셈보다 경제적 의미가 명확한 경우가 많았다. EPS 변화를 PER로 나누면 "밸류에이션 대비 개선 속도"가 되는데, 이건 곱셈으로는 포착되지 않는 관계다.

---

## M2: Signal Surprise — 순위 자체가 뛰었다

절대 수준이 아니라 상대적 위치의 변화. `ts_delta(rank(est_eps_ntm), 20)` — 20일 전 대비 이 주식의 EPS 추정치 순위가 얼마나 올랐는가.

하위 20%에서 상위 20%로 점프한 종목은, 처음부터 상위 20%에 있던 종목보다 정보가 많다. 수준이 아니라 변화, 변화가 아니라 변화의 변화. Cross-sectional rank의 delta는 이중 차분 효과를 내장한다. 절대 수준의 노이즈가 자동으로 제거되는 구조.

IV 순위 하락 `-ts_delta(rank(iv_30d), 10)`과 곱하면 "추정치 순위 상승 + 불확실성 순위 하락"이라는 복합 개선을 포착한다. 이건 기존 level 기반 알파로는 잡을 수 없는 신호였다.

---

## M3: Disagreement — 불일치 자체가 신호다

기존에는 A와 B가 같은 방향일 때만 신호로 삼았다. M3에서는 반대를 본다. A와 B가 **다른** 방향인 것 자체가 기회다.

EPS 추정치는 오르는데 가격은 빠지고 있다(DIS-2). 교과서적 해석: 시장이 아직 추정 상향을 반영하지 못했다. 이건 PEAD(Post-Earnings Announcement Drift)의 확장판이다. 실적 서프라이즈뿐 아니라 ongoing revision까지 포함하는, 더 넓은 정보 비대칭 포착.

```
rank(ts_delta(est_eps_ntm, 20)) * (-rank(ts_mean(returns, 10)))
```

가격은 오르는데 거래량이 빠지고 있다(DIS-4). 약한 랠리, 지지 없는 상승. 센티먼트는 긍정인데 IV가 올라간다(DIS-3). 표면적 낙관과 심층적 불안의 괴리. 이런 불일치는 시장의 균열이고, 균열에서 정보가 새어나온다.

3개 소스를 합산하는 Multi-Source Disagreement Score도 시도했다. 추정, 센티먼트, 가격 세 개를 더한다. 합이 극단(모두 긍정 또는 모두 부정)이면 강한 신호, 합이 중간이면 약한 신호. 일치의 강도를 연속적으로 측정하는 것이다.

---

## M4: Multi-Scale — 2차 미분은 남들이 안 본다

같은 신호를 여러 lookback으로 분해하면 스케일 간 관계가 알파가 된다.

`ts_mean(returns, 21) - ts_mean(returns, 5)` — 중기 평균에서 단기 평균을 빼면 모멘텀 가속도가 나온다. 중기가 단기보다 크면 가속 중이고, 작으면 과열 중이다. 이건 1차 미분(변화)이 아니라 2차 미분(변화의 변화)이다.

Revision Acceleration은 더 강력했다. `ts_delta(est_eps_ntm, 10) - ts_delta(est_eps_ntm, 30)` — 최근 10일 수정 속도가 30일 수정 속도보다 빠르면 개선이 가속하는 중이다. 레벨이 높은 것보다 레벨이 올라가고 있고 그 속도마저 빨라지는 것이 훨씬 강한 신호다.

핵심 통찰: 2차 미분은 crowding이 극히 적다. 대부분의 참가자가 1차 미분(ts_delta, ts_mean)에서 멈추기 때문이다. "변화의 변화"를 보는 사람은 거의 없었다.

---

## M5: Cross-Sectional Relative Position — 업종 안에서 어디 서 있는가

`group_rank(ts_mean(returns, 21), subindustry)` — 전체 시장에서의 순위가 아니라 같은 업종 안에서의 순위. 이건 neutralization과 다르다. Neutralization은 보정(correction)이지만, group_rank는 신호(signal)다.

가장 흥미로운 발견은 Cross-Level Divergence였다. 업종 내 순위와 전체 순위의 차이 `group_rank(..., subindustry) - rank(...)`. 업종 내에서는 1등인데 전체에서는 중간이라면? 그 업종 자체가 저평가되었을 수 있다. 업종 로테이션의 선행 지표.

Industry Leader × Laggard Spread까지 갔다. 업종 내 EPS 수정 1등인데 가격은 최근 가장 안 오른 종목. 업종 내 정보 확산 지연을 직접 포착하는 구조다.

---

## M6: Information Environment Meta-Signals — 관심의 부재가 기회

여기서 시야가 한 단계 올라갔다. 신호의 내용이 아니라 신호의 환경을 본다.

`(-rank(num_analysts)) * rank(ts_delta(option_volume, 10))` — 애널리스트가 적은데 옵션 거래가 급증. 이건 정보 비대칭의 가장 순수한 측정이다. Easley, O'Hara & Srinivas(1998)가 이론으로 세웠고, 우리가 BRAIN 수식으로 번역했다. 소수의 informed trader가 옵션 시장에서 먼저 움직이고 있다.

Neglected Firm + Quality도 효과적이었다. 무시받는 고수익 기업. Arbel(1985)의 neglected firm effect를 운영이익/총자산으로 직접 구현한 것이다. 시장의 관심이라는 것은 그 자체로 편향되어 있고, 관심의 빈자리가 알파의 온상이다.

---

## M7: Strategy Switching — 약화가 아니라 전환

기존 Regime 처리 방식은 scaling이었다. `rank(momentum) * (1 - rank(vol))` — 고변동성에서 모멘텀을 약화시킨다. 문제: 신호가 0에 수렴하면 아무것도 하지 않는다.

M7의 제안: 약화가 아니라 전환.

```
rank(momentum) * (2 * rank(vol_ratio) - 1)
```

`vol_ratio`가 중간보다 크면(조용한 시장) 양수, 즉 모멘텀. 중간보다 작으면(변동성 시장) 음수, 즉 반전. 단일 수식으로 두 가지 전략이 체제에 따라 자동 전환된다. Scaling이 "볼륨을 줄이는 것"이라면, Switching은 "채널을 바꾸는 것"이다.

Asness(2013)의 "Value and Momentum Everywhere"를 조건부로 구현한 Value-Momentum Switch도 만들었다. 저변동성에서는 가치 전략, 고변동성에서는 모멘텀 전략. 하나의 수식 안에서.

---

## M8: Decay Structure — 신호마다 수명이 다르다

단기 반전은 3~5일 안에 사라진다. PEAD 드리프트는 60~90일 지속한다. Revision 반영은 20~40일이 걸린다. 같은 Decay 5를 일률적으로 적용하면, 반전에게는 느리고 PEAD에게는 빠르다.

Decay를 신호 유형에 매칭시키는 것. 단기 반전에는 2~3, PEAD에는 8~12, Revision에는 4~6, Value에는 6~10. 이 매핑만으로 Fitness가 0.2~0.3 포인트 올라가는 경우가 있었다. TVR을 최적화하면서 신호를 보존하는 유일한 방법이었다.

---

## Gate Mixing: 체제 가중 혼합의 연속체

Strategy Switching(M7)의 확장이 Gate Mixing이다. Mixture-of-Experts에서 영감을 받았다.

```
gate × signal_A + (1 - gate) × signal_B
```

gate는 `rank(ts_std_dev(returns, 60))` 같은 연속 변수다. 0에서 1 사이. 고변동성이면 gate가 1에 가까워지면서 signal_A에 비중이 실린다. 저변동성이면 signal_B로 옮겨간다. 이산적 ON/OFF가 아니라 연속적 가중 전환.

이 구조의 진짜 장점은 gate가 0.5 근처(체제가 불분명한 구간)에서 두 신호가 상쇄되면서 자연스럽게 포지션이 0에 수렴한다는 것이다. **Null Zone의 자동 생성.** "모르겠을 때는 아무것도 하지 않는다"가 수식에 내장되어 있다.

IV를 gate로 쓰면 forward-looking 체제 판단이 된다. Intraday range를 gate로 쓰면 미시구조 기반 체제 판단이 된다. 같은 모멘텀-반전 전환이라도 gate의 출처가 다르면 상관이 0.3~0.4까지 떨어진다. 독립적인 알파가 된다.

---

여덟 개의 렌즈는 각각 시장의 다른 면을 드러냈다. Early Fusion은 신호 간 결합의 순서를, Signal Surprise는 상대 위치의 점프를, Disagreement는 불일치의 정보를, Multi-Scale은 가속도의 crowding 우위를, Cross-Sectional은 업종 내 위치를, Information Environment는 관심의 부재를, Strategy Switching은 체제 전환의 자동화를, Decay Structure는 시간 매칭의 정밀함을 보여줬다.

같은 EPS revision 데이터로 우리는 여덟 가지 서로 다른 알파를 만들 수 있었다. 데이터를 바꾸지 않았다. 관점을 바꿨다. 그리고 관점의 다양성이 Self-correlation 벽을 무너뜨렸다.
