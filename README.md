# Alpha Research Journal

**알파를 찾는 여정** — 퀀트 알파 연구의 사고 과정, 실패, 발견을 기록한 연구 저널.

> Cross-dataset interaction부터 425개 시뮬레이션의 교훈까지.
> 760개 알파 아이디어, 1.4% 통과율, 그리고 그 과정에서 배운 모든 것.

## Overview

IQC 2026 (International Quant Championship)에 참가하면서 축적한 알파 연구 방법론을 정리한 프로젝트입니다. 단순한 수식 목록이 아니라, **왜 그 접근을 했고, 뭐가 실패했고, 어떻게 생각이 바뀌었는지**를 기록합니다.

## Structure

```
worldquant-alpha/
├── index.html              # GitHub Pages 사이트 (연구 저널)
├── README.md               # 이 파일
└── chapters/               # 마크다운 챕터 파일
    ├── 01_WHAT_IS_ALPHA.md
    ├── 02_CROWDING_WALL.md
    ├── ...
    ├── 21_CURRENT_FRAMEWORK.md
    ├── A1_OPERATOR_REFERENCE.md
    ├── A2_VALIDATED_FORMULAS.md
    └── A3_REFERENCES.md
```

## Chapters

### Part I: 출발점
| Ch | Title | Description |
|----|-------|-------------|
| 01 | 알파란 무엇인가 | BRAIN 플랫폼, 제약 조건, 점수 체계 |
| 02 | Crowding의 벽 | 왜 단순한 접근은 전부 실패하는가 |

### Part II: 핵심 원리
| Ch | Title | Description |
|----|-------|-------------|
| 03 | 데이터셋이 충돌할 때 | Cross-Dataset Interaction, IC 6배 |
| 04 | 방향 x 확인 x 체제 | Signal Architecture, 조합 매트릭스 |
| 05 | 8가지 다른 시선 | Fusion, Surprise, Disagreement, Gate Mixing |
| 06 | 같은 데이터, 다른 수학 | Operator Innovation, 소수 윈도우 |

### Part III: 도메인 탐험
| Ch | Title | Description |
|----|-------|-------------|
| 07 | 가격이 '어떻게' 움직이는가 | Microstructure, Volume, VWAP |
| 08 | 공포의 곡선과 변동성 체제 | Options, IV, VRP, Macro Regime |
| 09 | 정보는 누구에게 먼저 도달하는가 | Analyst, Sentiment, Information Cascade |
| 10 | 숫자 뒤의 진실 | Fundamentals, Earnings Forensics |
| 11 | 편향을 거래하기 | Behavioral Finance |
| 12 | 보이지 않는 연결 | Network, Supply Chain, Industry Rotation |

### Part IV: 전환점
| Ch | Title | Description |
|----|-------|-------------|
| 13 | 자기 비판: 모든 것이 정적이었다 | 460개 알파의 솔직한 평가 |
| 14 | 전환점을 포착하는 5가지 방법 | Threshold, Pattern, Divergence, Velocity |
| 15 | Resolution, Lifecycle, Asymmetry | 해소 촉매, 알파 수명, 볼록성 |
| 16 | ML 우회로 | Conditioning Interface 연구가 가르쳐 준 것 |
| 17 | 3중 진화의 655개 알파 | Multi-Agent Evolution (ICQ3-6) |

### Part V: 통합과 실전
| Ch | Title | Description |
|----|-------|-------------|
| 18 | 군중을 읽다 | Crowding Detection, Contrarian |
| 19 | 알파의 오케스트라 | Ensemble, Meta-Strategy, 5축 직교화 |
| 20 | 3 Great Walls | 425개 시뮬, 1.4% 통과율의 교훈 |
| 21 | 현재의 프레임워크, 그리고 다음 | 760개 알파의 전체 조감 |

### Appendices
| | Title | Description |
|---|-------|-------------|
| A1 | Operator & Field Reference | BRAIN 연산자, 필드 매핑, 에러 패턴 |
| A2 | 검증된 패턴 수식집 | 실측 Sharpe/Fitness 포함 |
| A3 | 학술 참고 문헌 | 인용 논문 목록 |

## Key Findings

- **Cross-Dataset Interaction**: 단일 채널 IC ~0.01 vs 쌍 interaction IC ~0.06 (6배)
- **Operator > Field**: 같은 데이터에서 연산자만 바꿔도 SC < 0.5 달성 가능
- **실전 통과율 1.4%**: 425개 시뮬 중 6개 IS Check 통과
- **최고 실측 Sharpe 2.41**: Intraday x growth_potential_rank_derivative (Decay 4)
- **Fitness는 구조가 결정**: 사후 Decay 조정으로는 Fitness 교정 불가

## Tech Stack

- Single-page HTML (inline CSS + JS)
- Markdown chapters loaded from GitHub raw
- [marked.js](https://github.com/markedjs/marked) for markdown rendering
- [KaTeX](https://katex.org/) for math notation

## License

This research journal is shared for educational purposes. The methodologies described are based on publicly available academic literature.
