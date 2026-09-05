content = """# OKX Micro-Liquidity Reverse-Psychology Architecture (v3.1.0-UNIVERSAL)

**문서 상태:** Production Master Specification  
**시스템 분류:** 24/7 Direction-Agnostic Micro-Liquidity Dynamic Scalping Engine  
**운영 환경:** OKX Perpetual Futures (REST + WebSocket v5)  
**핵심 특성:** 완전 자본 독립형(Scale-Invariant), 시장 변동성 적응형(Volatility-Adaptive), 지연 필터 배제

---

## 1. 코어 철학: 대중 심리의 역이용과 복리 (Core Philosophy)

### 1.1 개미 90%의 체계적 패배 패턴 역이용 (Anti-Retail Exploitation)
선물 시장에서 일반 개인 투자자(Retail Traders)의 90% 이상이 손실을 보는 이유는 지표의 부재가 아니라, **극단적 탐욕(FOMO)과 공포(Panic)로 인해 특정 캔들 구조에서 동일한 행동을 반복**하기 때문이다. 본 시스템은 이 대중 심리를 정밀하게 역이용한다.

1. **상단 돌파 찌르기 (FOMO Trap ➜ SHORT):**
   - 캔들이 직전 캔들의 고점을 갱신할 때, 개미들은 "상방 돌파"로 착각하고 시장가 롱 매수를 추격한다.
   - 스마트 머니(MM)는 바로 그 유동성을 이용해 물량을 정리하고 가격을 직전 고점 아래로 누른다.
   - **시스템 행동:** 직전 고점을 찔렀으나 종가가 고점 아래로 마감(High SFP)하는 순간, 물린 개미들의 패닉 손절을 예상하고 **즉시 숏(SHORT) 진입**.
2. **하단 이탈 찌르기 (Panic Trap ➜ LONG):**
   - 캔들이 직전 캔들의 저점을 깰 때, 개미들은 "지지선 붕괴"의 공포에 질려 시장가 투매를 하거나 숏을 추격한다.
   - 스마트 머니는 개미들이 쏟아낸 손절 물량을 바닥에서 전량 흡수(Absorption)하고 가격을 들어 올린다.
   - **시스템 행동:** 직전 저점을 깼으나 종가가 저점 위로 회복(Low SFP)하는 순간, 세력의 V자 반등에 편승하여 **즉시 롱(LONG) 진입**.

### 1.2 거시 추세 맹신 탈피와 작은 복리의 연속 (Continuous Micro-Compounding)
- 며칠에 한 번 올까 말까 한 거창한 스윙 추세나 완벽한 타이밍을 기다리지 않는다.
- 시장은 24시간 내내 캔들 단위에서 크고 작은 미세 유동성 페이크아웃(Micro-Sweep)을 발생시킨다.
- **수익과 손실은 필연적으로 반복된다.** 본 시스템은 손실을 $1R$로 철저히 제한하고, 수익을 $1.5R$ 이상(또는 반대편 캔들 유동성)으로 확보하여 **수수료와 펀딩비를 압도하는 양의 기댓값(Positive Expectancy)을 24시간 연속 누적**하는 것을 유일한 목표로 한다.

---

## 2. 3-Session 분업 운영 파이프라인 (Operational Protocol)

인지적 편향과 코드 오염을 방지하고, 기획-구현-검증의 객관성을 보장하기 위해 3개의 전문 세션을 분리 운영한다.

```
┌─────────────────────────────────────────────────────────────┐
│ 1. 전략기획방 (Strategy Planning & Architecture)             │
│    - 시장 미세구조 및 개미 심리 역이용 메커니즘 수립        │
│    - 자본 규모 불변성(Scale-Invariance) 및 손익비 수학 검증 │
│    - 프로덕션 기능 명세서(Architecture Spec) 발급/형상 관리 │
└──────────────────────────────┬──────────────────────────────┘
                               │ [Handoff: Architecture Spec]
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. 코드구현방 (Production Implementation & Ops)              │
│    - 비동기(Asyncio) 주문 엔진 및 웹소켓 마감 감지기 구현    │
│    - 거래소 메타데이터 동적 파싱, 레이스 컨디션/슬리피지 방어│
│    - Vultr Cloud VPS 실서버 systemd 데몬 24/7 무인 운영    │
└──────────────────────────────┬──────────────────────────────┘
                               │ [Handoff: Production Code]
                               ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. 구글코랩 백테스트방 (Quantitative Backtesting Lab)         │
│    - CCXT 페이징 데이터 인제스천 파이프라인 구축            │
│    - Lookahead Bias 배제 시뮬레이션, 수수료 차감 후 성과 검증│
│    - 파라미터 민감도 및 시드별(소액 vs 대형) 왜곡 검증      │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. 시그널 생성 및 진입 엔진 (Dynamic Signal Engine)

### 3.1 방향 비의존성 (Direction Agnostic)
- 상위 타임프레임(HTF)의 고정 편향(BOS 필터 등)을 배제한다.
- 캔들 레벨에서 개미 털기(SFP)가 발생하면 **상승/하락 추세와 무관하게 롱/숏 양방향을 24시간 유연하게 진입**한다.
- **기준 타임프레임 (`TIMEFRAME`):** 5분봉(`5m`) 또는 15분봉(`15m`) 동적 적용.
- **참조 윈도우 (`LOOKBACK_BARS`):** 직전 $N$개 봉 ($N \ge 1$, 기본값 $N=1$: 직전 캔들).

### 3.2 캔들 단위 미세 유동성 스윕 (Zero-Lag Micro-SFP)
*지연 지표(MA, RSI 등) 및 인위적 필터(고정 꼬리 비율, 고정 ATR 진폭 등)는 일체 배제한다.*

#### [SHORT 진입 규칙 - 개미 FOMO 박멸]
1. **스윕 (Sweep):** 현재 캔들의 고가(`High`)가 직전 $N$개 봉의 최고가(`Recent_High`)를 상향 돌파.
2. **거부 (Reject):** 현재 캔들의 종가(`Close`)가 직전 $N$개 봉의 최고가(`Recent_High`) 아래로 내려와 마감 (`Close < Recent_High`).
3. **진입 체결:** 캔들이 확정 마감되는 즉시(다음 캔들 Open 0ms) **SHORT 진입**.

#### [LONG 진입 규칙 - 개미 Panic 흡수]
1. **스윕 (Sweep):** 현재 캔들의 저가(`Low`)가 직전 $N$개 봉의 최저가(`Recent_Low`)를 하향 이탈.
2. **복귀 (Reclaim):** 현재 캔들의 종가(`Close`)가 직전 $N$개 봉의 최저가(`Recent_Low`) 위로 말아 올려 마감 (`Close > Recent_Low`).
3. **진입 체결:** 캔들이 확정 마감되는 즉시(다음 캔들 Open 0ms) **LONG 진입**.

### 3.3 무-풀백 즉시 체결 원칙 (No-Pullback Policy)
- 세력이 개미를 털고 방향을 틀 때는 기회를 주지 않고 V자로 쏘아 올린다. 눌림목을 기다리는 지정가 주문은 세력이 힘없이 무너지는 패배 거래만 체결시키는 **역선택(Adverse Selection)**을 유발한다.
- **실행 프로토콜:** 캔들 마감 확인 즉시 최유리 BBO(Best Bid/Offer) 지정가 또는 허용 슬리피지(Slippage Tolerance) 범위 내 시장가로 즉각 체결하여 모멘텀을 선점한다.

---

## 4. 자본 규모 불변 자금 관리 (Scale-Invariant Risk Model)

### 4.1 동적 자본 연동 리스크 (Fixed Fractional Model)
본 시스템은 특정 자본금(예: 512 USDT)이나 특정 목표 달러에 의존하지 않으며, 계좌 잔고의 변동에 완벽히 연동된다.

1. **거래당 허용 리스크 (`RISK_FRACTION`):**
   $$\text{Risk Amount} = \text{Current Total Equity} \times \text{RISK_FRACTION}$$
   *(기본 설정: 계좌 자본의 $1.0\% \sim 2.0\%$ 동적 할당)*
2. **거래소 메타데이터 런타임 동적 로드:**
   - 종목별 계약 크기(`contractSize`), 최소 주문 단위(`minSz`), 수량 정밀도(`lotSz`)를 CCXT `load_markets()`로부터 런타임에 동적으로 주입받는다.
3. **스케일러블 주문 수량 산출 수식:**
   $$\text{Contracts} = \text{round\_down}\left( \frac{\text{Risk Amount}}{|\text{Entry Price} - \text{Stop Loss}| \times \text{Contract Size}}, \text{lotSz} \right)$$
   - 계산된 수량이 거래소 최소 수량(`minSz`) 미만일 경우:
     - 1계약 진입 시의 리스크가 설정 리스크의 $1.3$배 이내라면 진입 허용 (소액 계좌의 기회 상실 방지).
     - $1.3$배를 초과하면 시스템 안전을 위해 주문을 건너뛰고 자본 보호 로그를 기록.

---

## 5. 동적 청산 및 손익비 매트릭스 (Dynamic Exit Matrix)

특정 고정 달러($) 익절은 엄격히 금지하며, **시장 변동성(ATR) 및 손익비(R-Multiple)**에 의해서만 유기적으로 청산한다.

### 5.1 스탑로스 (Dynamic Hard SL)
- **손절 기준:** 세력이 개미를 털어낸 꼬리의 극값(Extreme Wick)에 시장의 최소 변동폭 버퍼를 가산.
  - 롱(LONG) SL: `Sweep_Candle_Low - Dynamic_Buffer`
  - 숏(SHORT) SL: `Sweep_Candle_High + Dynamic_Buffer`
  - *Dynamic Buffer:* `max(Tick_Size, ATR(14) * 0.1)`
- **원리:** 세력조차 지켜내야 하는 마지노선이 무너지면, 해당 셋업은 유동성 스윕이 아닌 추세 지속이므로 즉시 $1R$의 정해진 손실로 방어 탈출한다.

### 5.2 익절 프로토콜 (Dual Dynamic TP)
시장의 흐름과 포지션 규모에 맞춰 아래 2가지 중 하나로 유연하게 작동한다:

1. **상대적 손익비 모델 (R-Multiple Exit):**
   - **타겟:** **$1 : 1.5R$** (또는 시장 변동성에 따라 $1.3R \sim 1.8R$ 동적 설정).
   - 진입가와 손절가의 거리($1R$) 대비 $1.5$배 가격 도달 시 전량 청산.
   - 계좌 크기와 무관하게 항상 일관된 통계적 기대값 형성.
2. **미세구조 반대편 유동성 모델 (Opposing Micro-Liquidity Exit):**
   - 롱(LONG) 포지션: 진입 파동 형성 중 발생한 **직전 캔들의 최고점(Opposing High)**에 도달 시 청산.
   - 숏(SHORT) 포지션: 진입 파동 형성 중 발생한 **직전 캔들의 최저점(Opposing Low)**에 도달 시 청산.
   - *원리: 반대편 개미들이 손절하거나 돌파 매수로 덤벼드는 유동성 집중 지점에 물량을 넘기고 퇴장.*

### 5.3 수수료 방어 및 양의 기댓값 증명
- 1회 거래당 왕복 수수료(Maker/Taker 혼합 약 $0.04\% \sim 0.07\%$) 대비, $1.5R$ 타겟은 최소 수수료의 6~10배 이상의 변동폭을 취한다.
- 따라서 승률이 **$42\%$ 이상**만 유지되면 수수료와 펀딩비를 완전히 상쇄하고 계좌 잔고는 수학적으로 우상향(Positive Mathematical Expectancy)한다.

---

## 6. 시스템 예외 처리 및 런타임 안정성 (Failsafe & Execution)

1. **동시 보유 포지션 제어 (`MAX_CONCURRENT_POSITIONS`):**
   - 기본값 2개 (설정 파일에서 자본 규모에 따라 동적 조절 가능).
   - 동시 손절 발생 시에도 전체 계좌 낙폭은 `MAX_CONCURRENT_POSITIONS * RISK_FRACTION` 이내로 강제 제한.
2. **보수적 체결 판정 (Pessimistic Backtest Matching):**
   - 동일 캔들 내에서 고가/저가가 SL과 TP를 동시에 건드렸을 경우, 과적합을 원천 차단하기 위해 **스탑로스(SL) 우선 체결**로 정산.
3. **웹소켓 0ms 마감 캡처:**
   - 캔들 수량 리스트 의존성을 제거하고, `candle_start_timestamp > current_bar_timestamp` 조건을 통해 캔들이 마감되는 순간 지연 없이 다음 캔들 시가에 주문을 송신.

---

## 7. 향후 세션별 실행 과제 (Next Steps)

* **전략기획방:** 본 v3.1.0-UNIVERSAL 문서를 마스터 형상으로 확정하고, 실서버 텔레메트리 로그 구조 정의.
* **코드구현방:** 
  1. `settings.py`에서 하드코딩된 `CONTRACT_SIZES` 삭제 및 `load_markets()` 연동.
  2. 기존 1H BOS 및 38.2% 풀백 로직을 전면 제거하고, 본 문서 3절의 **'직전 봉 스윕 즉시 진입 엔진'**으로 리팩토링 및 Vultr 실서버 핫패치 배포.
* **구글코랩 백테스트방:** 본 동적 엔진을 기반으로 과거 6개월 5m/15m 실데이터를 수집하여 자본 규모별(소액 vs 대형) 동일 승률/PF 도출 여부 최종 검증.
"""

with open("ARCHITECTURE.md", "w", encoding="utf-8") as f:
    f.write(content.strip())

import os
print("생성 완료:", os.path.abspath("ARCHITECTURE.md"))
