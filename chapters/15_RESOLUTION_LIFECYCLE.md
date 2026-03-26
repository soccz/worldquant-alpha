# 15장: 해소 촉매와 수명 사이클 — "언제"가 "무엇"보다 중요하다

---

## 서문: 비대칭의 존재 vs 비대칭의 해소

12_information_cascade_speed.md의 구조는 이랬다:

```
선행 신호 높음 x 가격 미반영 → 매수
```

이건 맞다. 하지만 불완전하다.

"가격이 아직 안 반영됐다"는 것만으로는 충분하지 않다.
가격이 안 반영된 상태가 6개월째 지속될 수 있다.
Decay 4~6인 알파로 6개월을 기다릴 수는 없다.

비대칭이 존재한다는 것과, 그 비대칭이 지금 해소되려 한다는 것은
완전히 다른 문제다.

진짜 알파는 비대칭의 존재가 아니라
**해소 촉매(Resolution Trigger)**를 포착하는 것이다.

---

## Resolution Trigger: 5가지 촉매 유형

### Type 1: 정보 임계값 돌파

정보는 선형으로 전파되지 않는다.
기관 투자자는 신호가 충분히 강해질 때까지 기다린다.

- 애널리스트 수정 1번: 노이즈로 처리
- 3번 연속: 포지션 검토 시작
- 5번 연속 + 크기 증가: 실제 매수 실행

```
# 수정의 누적 강도가 역사적 상위 20%에 처음 진입하는 순간
cumulative_revision = ts_sum(
  ts_delta(earnings_per_share_median_value, 23) > 0,
  131
)

rank(cumulative_revision)
  * (ts_rank(cumulative_revision, 233)
     - ts_lag(ts_rank(cumulative_revision, 233), 23))
```

돌파 직후 23일이 가장 강한 알파 구간.
이미 오래 상위에 있던 것은 이미 반영됨 → 자동 제외.

### Type 2: 불확실성 해소

정보가 있어도 불확실성이 높으면 기관은 포지션을 안 잡는다.
IV(내재변동성)가 높으면 헤지 비용이 높다. 진입 장벽이다.
IV가 하락하기 시작하면 = 헤지 비용 감소 = 진입 장벽 제거.

대기 중이던 자금이 동시에 진입하는 순간.

```
# IV가 역사적 고점에서 하락 시작 + 펀더멘탈 좋음
iv_was_high = ts_rank(implied_volatility_call_30, 131) > 0.75
iv_declining = ts_delta(implied_volatility_call_30, 13) < 0
fundamental_signal = rank(cashflow_op / (assets + 0.001))

fundamental_signal * (iv_was_high * iv_declining)
```

### Type 3: 관심 촉매

정보가 있어도 시장이 주목하지 않으면 반영되지 않는다.
Buzz가 급증하면 개인 투자자가 유입되고,
개인 유입이 가격을 움직이면 기관도 주목하게 된다.

오랫동안 무시되던 종목에 갑자기 관심이 쏠리는 순간.

```
# buzz가 역사적 저점에서 급등 + 펀더멘탈 좋음
quality_signal = rank(cashflow_op / (abs(income) + 0.001))

quality_signal
  * rank(ts_delta(scl12_buzz, 11))
  * (ts_rank(scl12_buzz, 131) < 0.4)
```

23_DEEP_RETHINK의 "침묵 후 폭발" 패턴의 직접 구현.

### Type 4: 구조적 전환점 (2차 미분)

신호가 방향을 바꾸는 순간이 아니라,
신호의 변화 속도가 바뀌는 순간 = 2차 미분의 부호 전환.

악화되던 것이 악화 속도가 줄어드는 순간 = 바닥 형성.

```
first_derivative = ts_delta(cashflow_op / (abs(income) + 0.001), 47)
second_derivative = ts_delta(first_derivative, 47)

# 1차 미분 음수(악화 중) + 2차 미분 양수(악화 속도 감소)
rank(second_derivative) * (first_derivative < 0) * (second_derivative > 0)
```

시장은 "아직 악화 중"이라는 1차 정보만 보고 매도한다.
하지만 2차 정보(속도 감소)는 바닥 신호다. 미반영.

### Type 5: 교차 확인 수렴

여러 독립 소스가 같은 방향으로 수렴하는 순간.
하나의 소스가 움직이는 것은 노이즈.
독립적인 3개 소스가 동시에 같은 방향 = 진짜 정보.

```
analyst_positive = ts_delta(earnings_per_share_median_value, 23) > 0
options_positive = ts_delta(implied_volatility_call_30, 13) < 0
micro_positive = ts_zscore((close - vwap) / (close + 0.001), 47) > 0

convergence = analyst_positive + options_positive + micro_positive

rank(convergence) * (convergence >= 2) * (-rank(ts_mean(returns, 23)))
```

3개 독립 채널 중 2개 이상이 같은 방향 = 정보의 신뢰도 극대.
가격이 아직 안 움직였다면 = 해소 직전.

---

## 치명적 실패: 원래 수명 사이클 이론의 순환 논리

28_RESOLUTION_TRIGGER와 별개로,
초기 수명 사이클 이론(30_ALPHA_LIFECYCLE_THEORY)에는
**치명적인 논리적 결함**이 있었다.

### 문제: 자기 참조

```
나쁜 구조: rank(momentum) x 조건(momentum의 ts_rank)
→ 모멘텀으로 모멘텀을 예측 = 순환 논리
```

모멘텀의 수명 사이클을 모멘텀의 ts_rank로 판단하고 있었다.
"모멘텀이 역사적 고점에 있으면 모멘텀이 곧 반전한다"
— 이건 동어반복이다.

이 순환 논리를 발견한 것은 30_INDEPENDENT_LIFECYCLE을 쓸 때였다.
460개의 알파를 다시 검토하다가, 수명 사이클 관련 알파의 절반이
같은 데이터로 방향과 타이밍을 동시에 판단하고 있었다.

### 수정: 독립 신호 기반 수명 사이클

핵심 원칙: **방향 신호와 타이밍 신호는 독립이어야 한다.**

| 방향 신호 | 독립 타이밍 신호 | 왜 독립인가 |
|-----------|----------------|-----------|
| 모멘텀 (returns) | 거래량 패턴 (volume) | 가격과 거래량은 다른 프로세스 |
| EPS revision | IV 변화 | 애널리스트와 옵션 시장은 다른 참여자 |
| 펀더멘탈 quality | Sentiment buzz | 재무제표와 뉴스는 다른 채널 |

### 수명 사이클의 3단계

**단계 1: 탄생 (Smart Money 진입)**

```
# ILC-1: 가격 상승 + 거래량 낮음 = quiet accumulation
rank(ts_mean(returns, 21)) * (1 - rank(volume / adv20))
```

소수의 informed trader만 진입 중. 대중은 모른다.
거래량이 동반되지 않는 가격 상승 = 기관의 조용한 매집.

**단계 2: 성숙 (대중 유입)**

가격도 높고 거래량도 높다. 모든 사람이 알고 있다.
알파가 이미 반영되고 있다.
→ 이 단계에서는 포지션을 잡지 않는다. Null Zone.

**단계 3: 과열/소멸 (반전)**

```
# ILC-2: 거래량 극대에서 모멘텀 반전
-rank(ts_mean(returns, 21))
  * rank(volume / adv20)
  * rank(ts_rank(volume / adv20, 126))
```

거래량이 역사적 고점 = 마지막 매수자 진입 완료 = 매수자 부족.

### v1의 3가지 치명적 버그

독립 수명 사이클 이론의 첫 버전에도 버그가 있었다:

**버그 1**: ILC-4(과반영 반전)에서 3개 신호를 곱해 신호가 너무 희소했다.
revenue x IV x momentum 세 가지 조건을 동시에 만족하는 종목이
TOP3000에서 50개 미만이었을 가능성.

**버그 2**: ILC-5(숨겨진 보석)에서 (1 - rank(buzz))^2를 사용했는데,
rank(buzz)가 0이면 (1-0)^2 = 1, rank(buzz)가 0.5이면 0.25.
이 분포가 long-short 균형을 깨뜨릴 수 있었다.

**버그 3**: ILC-3(불안 속 좋은 뉴스)에서 IV/HV 비율을 곱했는데,
IV/HV가 1 미만(시장 안심)일 때도 revision과 곱해져
의도치 않은 방향의 포지션이 생길 수 있었다.

---

## 볼록성과 Null Zone: 수익의 "형태"를 설계하다

29_CONVEXITY_AND_NULL_ZONE에서 완전히 다른 질문을 했다:

> "어떤 종목을 살 것인가"가 아니라, "수익의 형태를 어떻게 설계할 것인가."

### 왜 볼록성이 중요한가

```
선형 알파:   맞으면 +1%, 틀리면 -1%  → 승률 55%면 E[R] = 0.1%
볼록 알파:   맞으면 +3%, 틀리면 -0.5% → 승률 40%면 E[R] = 0.9%
```

볼록 알파는 승률이 낮아도 기대수익이 높다.
Drawdown이 작다 (틀릴 때 적게 잃으므로).
Sharpe = mean/std에서 std가 작아지는 효과.

### 핵심 구조: 확신이 높을 때만 큰 포지션

```
alpha = direction_signal x conviction_weight

direction_signal = rank(primary_signal) - 0.5
# -0.5 ~ +0.5 범위. 중앙이 0.

conviction_weight = abs(rank(confirm_1) + rank(confirm_2) - 1.0)
# 확인 신호 2개의 합이 극단(0 또는 2)일 때 큰 값
# 중앙(1.0)일 때 0 → Null Zone
```

이 구조의 장점:
1. 자연스러운 볼록성: conviction이 높을 때만 큰 포지션
2. 자연스러운 Null Zone: conviction이 0이면 포지션 없음
3. Long-Short 대칭: direction이 양수/음수 다 가능
4. 조건 2~3개로 충분

### 같은 3개 신호에서 payoff 형태만 바꿔도 Sharpe가 달라진다

이것이 가장 놀라운 발견이었다.

```
# 기존: rank(A) * rank(B) * rank(C)
# 문제: 중간값 종목도 포지션을 잡음, 확신 구분 없음

# 제안: (rank(A) - 0.5) * abs(rank(B) + rank(C) - 1.0)
# 장점: 확신 높을 때만 포지션, 자연스러운 long/short/null
```

사실상 같은 3개 신호를 쓰면서 수익 구조만 바꾼 것인데,
Sharpe가 다를 수 있다. 그리고 기존보다 높을 가능성이 있다.

### 실패: abs() 함수의 BRAIN 지원 미확인

볼록성 알파의 핵심이 abs() 함수인데,
abs()가 BRAIN에서 지원되는지 확인하지 않았다.

abs() 없이는 conviction_weight를 계산할 수 없고,
conviction_weight 없이는 Null Zone이 작동하지 않는다.

이 하나의 함수 미확인이 볼록성 이론 전체를 위태롭게 했다.

대안: abs() 대신 rank(A)^2 같은 멱급수 구조로 대체 가능.
하지만 BRAIN에서 제곱 연산(* 자기 자신)도 검증이 필요했다.

---

## 신호 인터페이스: 6가지 결합 방법

27_SIGNAL_INTERFACE_THEORY에서 발견한 것:
같은 두 필드를 결합하는 방법이 6가지가 있고,
대부분의 사람은 하나(Late Multiplication)만 쓴다.

### 6가지 인터페이스

A = cashflow_op / (assets + 1), B = earnings_per_share_median_value / close로 예시:

| # | 인터페이스 | 수식 | 포착하는 것 |
|---|----------|------|-----------|
| 1 | Late Mult | rank(A) * rank(B) | 둘 다 좋은 종목 |
| 2 | Early Mult | rank(A * B) | 극단적 조합 |
| 3 | Ratio | rank(A / (B + 0.01)) | A의 B 대비 크기 |
| 4 | Difference | rank(A - B) | 두 신호의 괴리 |
| 5 | Additive | rank(A) + rank(B) | 하나만 좋아도 |
| 6 | Conditional | rank(A) * rank(B)^2 | B 극단에서만 A 유효 |

**이 6개는 같은 두 필드에서 6가지 서로 다른 알파를 만든다.**
서로 상관이 낮을 수 있다 — 다른 정보를 추출하므로.

### 실전 함의

10개 핵심 알파에 5개 대안 인터페이스를 적용하면 60개 변형이 가능하다.
상관 낮은 것들만 제출하면 알파 수가 크게 늘어난다.

하지만 이것도 "이론은 좋지만 실행은 별개" 문제에 부딪혔다.
60개를 다 시뮬레이션하려면 시간과 제출 크레딧이 엄청나게 들었다.

---

## 조건부 알파: "언제"가 "무엇"보다 중요하다

26_CONDITIONAL_ALPHA_THEORY의 핵심:

> 같은 신호가 상태에 따라 작동하거나 안 한다.
> "무엇을" 보는가보다 "언제" 보는가가 Sharpe를 결정한다.

### Sharpe에 미치는 영향

```
Sharpe = mean / std

기존: 같은 신호를 항상 적용
→ 어떤 체제에서는 맞고 어떤 체제에서는 틀림
→ std가 큼 → Sharpe 낮음

제안: 상태에 따라 신호를 켜고 끔
→ 맞는 체제에서만 적용
→ std가 작음 → Sharpe 높음
```

조건부 알파는 "더 많이 맞추는 것"이 아니라
"틀리는 횟수를 줄이는 것"이다.

mean이 같아도 std를 줄이면 Sharpe가 올라간다.

이것이 23_DEEP_RETHINK의 "안정성 > 강도" 원칙의 구체적 구현이다.

### 실패: boolean 연산자의 불확실성

조건부 알파의 핵심은 "이 상태일 때만" 활성화하는 것인데,
BRAIN에서 조건 분기를 어떻게 구현하는지 확실하지 않았다.

bucket() 함수의 == 연산자 지원 여부가 미확인.
비교 연산자(>, <)의 지원 여부가 미확인.

결국 곱셈 필터(rank(signal) * rank(state))로 우회했는데,
이것은 "이 상태일 때만 활성화"가 아니라
"이 상태일수록 더 강하게 활성화"가 된다.
의미가 미묘하게 다르다.

---

## 이 장의 실패 통계

| 이론 | 핵심 버그/실패 | 영향 |
|------|-------------|------|
| 원래 수명 사이클 | 순환 논리 (자기 참조) | v1 전체 무효화 |
| Resolution Trigger | boolean 합산 미확인 | Type 1, 5 구현 불가능 가능성 |
| 볼록성 이론 | abs() 미지원 가능성 | conviction weight 계산 불가 |
| 인터페이스 이론 | 60개 변형의 실행 비용 | 이론적 가능성만 확인 |
| 조건부 알파 | bucket() == 미지원 가능성 | 이산 필터 구현 불가 |
| 독립 수명 사이클 v1 | 3가지 치명적 버그 | 7개 알파 중 3개 수정 필요 |

실패율은 여전히 높았다.
하지만 실패의 원인이 "경제적 논리 부재"에서
"플랫폼 제약 확인 부족"으로 바뀌었다.

이것이 진보였다.

---

*다음 장에서는 수개월간의 딥러닝 실험이
어떻게 충격적인 결과를 낳았는지,
그리고 그것이 알파 생성에 어떤 교훈을 주었는지를 다룬다.*
