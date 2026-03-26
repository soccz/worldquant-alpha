# Chapter 02: Crowding의 벽

## 순진한 시작

BRAIN 플랫폼에 처음 로그인했을 때, 우리가 가장 먼저 시도한 건 당연히 가장 단순한 것이었다.

```python
-rank(returns)
```

전일 수익률이 높은 종목을 매도하고, 낮은 종목을 매수한다. 단기 반전. 행동재무학 교과서 1장에 나오는 과잉반응 효과다. 시뮬레이터를 돌렸더니 IS에서 Sharpe가 나쁘지 않았다. "이거 되는 거 아닌가?"

하지만 제출 버튼을 누르자 현실이 왔다. Self-Correlation 0.85. 이미 수천 명이 같은 걸 제출한 뒤였다.

---

## rank() 래핑의 늪

그 다음 시도도 크게 다르지 않았다. 우리는 Price Volume 데이터셋의 23개 필드를 이리저리 조합했다.

```python
# 모멘텀
rank(ts_mean(returns, 21))

# 변동성 조정 모멘텀
rank(ts_mean(returns, 21) / ts_std(returns, 21))

# 12-1 모멘텀
rank(ts_mean(returns, 252) - ts_mean(returns, 21))

# 거래량 서프라이즈
rank(volume / ts_mean(volume, 20))

# 저변동성
-rank(ts_std(returns, 20))
```

패턴이 보이는가? 전부 `rank()` 하나로 단일 신호를 래핑한 구조다. 데이터셋은 항상 Price Volume. 바뀌는 건 안에 들어가는 필드와 lookback window뿐이었다.

이 구조로 수십 개를 만들었다. 간혹 Sharpe 1.25를 넘기는 것도 있었지만, 대부분 세 가지 이유 중 하나로 탈락했다:

1. **Self-Correlation > 0.7**: 이미 누가 제출함
2. **LOW_FITNESS**: TVR이 너무 높아서 Fitness가 1.0 미달
3. **LOW_SHARPE**: OS에서 1.25 미달

특히 모멘텀 21일 이상 단독 사용은 현재 시장에서 거의 음의 Sharpe를 보였다. 2019-2023 구간에서 모멘텀 크래시가 반복되면서, 교과서적 모멘텀은 죽어 있었다. 우리가 배운 금융 이론이 실전에서 통하지 않는 첫 번째 경험이었다.

---

## 460개의 교훈

솔직하게 말하자. 우리는 처음 460개 가까운 알파를 만들었고, 그 대부분이 본질적으로 같은 실수를 반복하고 있었다.

돌이켜 보면, 그 460개 알파의 공통점은 이랬다:

- Price Volume 데이터셋만 사용 (coverage 100%니까 편하므로)
- `rank()` 하나에 하나의 신호 (구조적 복잡성 제로)
- lookback window만 바꿔서 변형 제출 (5일을 10일로, 10일을 21일로)
- 같은 데이터셋 안에서의 조합 (returns와 volume은 같은 PV 데이터셋이다)

이건 알파 연구가 아니라 파라미터 스캔이었다. 그리고 파라미터 스캔은 crowding의 정의 그 자체다.

실패 통계를 정리했을 때 현실이 명확해졌다:

| 실패 유형 | 건수 | 핵심 원인 |
|-----------|------|-----------|
| LOW_FITNESS | 34 | TVR 과다 (Decay 부족) |
| SELF_CORRELATION | 31 | 같은 구조의 파라미터 변형 |
| LOW_SHARPE | 22 | 단일 PV 신호의 한계 |
| SUB_UNIVERSE FAIL | 21 | TOP3000 통과했으나 소형 유니버스 탈락 |

108개의 실패가 기록되어 있었다. 그리고 그 108개 중 거의 전부가 "Price Volume 단일 데이터셋"이라는 같은 병에 걸려 있었다.

---

## 벽의 정체

어느 시점에서, 우리는 왜 이런 일이 벌어지는지 이해하기 시작했다.

BRAIN 플랫폼에는 수만 명의 사용자가 있다. 그리고 Price Volume 데이터는 coverage 100%, 필드 수 23개다. 23개 필드로 만들 수 있는 합리적인 수식의 수는 유한하다. 수만 명이 수년간 그 유한한 공간을 탐색해 왔다.

우리가 30분 고민해서 만든 수식은, 누군가 3년 전에 이미 제출한 것이었다.

이것이 crowding이다. 너무 많은 사람이 같은 신호를 쫓으면, 그 신호의 수익성은 사라진다. 시장에서든 플랫폼에서든 원리는 같다. 정보가 가격에 반영되는 속도가 빨라지고, alpha decay가 가속되고, 남는 건 거래비용뿐이다.

구체적으로 어떤 일이 벌어지는지 보자:

```python
# 이것은 crowded다 (PV 단일 데이터셋)
-rank(ts_mean(returns, 5))
# → Sharpe ~0.8, SC > 0.7

# 이것도 crowded다 (같은 PV, window만 다름)
-rank(ts_mean(returns, 3))
# → 위 알파와 Self-Correlation 0.85+

# 이것도 crowded다 (PV끼리의 조합)
rank(returns) - rank(volume / ts_mean(volume, 20))
# → 여전히 PV 1개 데이터셋. 경쟁자 수천 명.
```

문제는 수식의 복잡도가 아니었다. 데이터셋의 다양성이었다.

---

## 수정의 한계

처음에는 실패한 알파를 "수정"하려고 했다. Decay를 올려서 TVR을 낮추고, Fitness를 개선하는 식이다.

```python
# 원본: TVR 과다로 LOW_FITNESS
-rank(ts_mean(returns, 3))
# Decay 4 → TVR 45%, Fitness 0.85

# 수정: Decay 올림
-rank(ts_mean(returns, 3))
# Decay 8 → TVR 28%, Fitness 1.05
```

이런 식의 파라미터 조정으로 LOW_FITNESS 일부는 해결할 수 있었다. Decay를 2~3 올리면 TVR이 내려가고, Fitness가 1.0을 넘기는 경우가 있었다.

하지만 SELF_CORRELATION과 LOW_SHARPE는 Decay 조정으로 해결되지 않았다. 구조가 같으면 상관관계가 높고, 신호 자체가 약하면 파라미터를 아무리 바꿔도 Sharpe는 오르지 않는다. TVR을 낮추면 Sharpe도 같이 내려가는 경우가 많았다 -- Fitness 공식의 함정이었다.

Amihud 유동성 승수를 곱하거나, `group_rank()`으로 업종 내 상대화를 하는 등의 기법도 시도했다:

```python
# Amihud 유동성 필터 추가
-rank(ts_mean(returns, 5)) * rank(ts_mean(abs(returns) / adv20, 21))

# 업종 내 상대 랭킹
group_rank(-rank(ts_mean(returns, 5)), subindustry)
```

이런 수정들이 일부 알파를 살려내기는 했다. 하지만 근본적인 문제 -- 단일 데이터셋의 crowding -- 는 해결하지 못했다.

---

## 전환점

전환점은 우리 자신의 연구 노트를 다시 읽었을 때 왔다.

모멘텀 팩터의 시계열 변동을 연구하면서, 우리는 하나의 중요한 발견을 했었다. 단일 채널의 Information Coefficient(IC)는 0.01 수준이었다. 모멘텀 하나, 변동성 하나, 거래량 하나 -- 각각은 거의 예측력이 없었다.

하지만 두 신호를 곱해서 interaction term을 만들면, IC가 0.06으로 뛰었다. 여섯 배.

그리고 BRAIN에서의 crowding 문제와 연결했을 때, 모든 것이 맞아떨어졌다:

- 단일 데이터셋 알파: 수만 명이 탐색한 공간. Sharpe < 1.0이 정상.
- 2개 데이터셋 교차: 탐색한 사람이 급감. Sharpe 1.0~1.5 가능.
- 3개 데이터셋 교차: 거의 아무도 안 해봤다. Sharpe 1.5+ 가능.

수식의 복잡도를 높이는 게 아니다. **데이터의 교차점**에서 edge가 생긴다.

460개의 실패 끝에, 우리는 완전히 다른 방향을 향해야 한다는 걸 인정했다. `rank()` 래핑을 멈추고, 데이터셋의 벽을 넘어야 했다. Analyst Estimate의 653개 필드, Options Analytics의 74개 필드, Relationship Data의 165개 필드 -- 이 미개척지들이 기다리고 있었다.

좌절은 끝났다. 다음 단계가 시작됐다.
