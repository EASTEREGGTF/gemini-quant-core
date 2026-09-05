# [Project Handover] OKX SFP 퀀트 자동매매 시스템 마스터 인수인계서 (v2.3 Production Live)

## 1. 프로젝트 개요 및 운영 현황 (2026-09-04 배포 완료)
- **시스템 명칭**: OKX SFP + SMC Liquidity Sweep & Micro-structure Multi-Asset Live Bot
- **운영 인프라**: Vultr Cloud VPS (Ubuntu 22.04 LTS, Seoul Region) / `systemd` 24/7 무인 상주 (`okx-bot.service`)
- **운용 자본**: 512 USDT (초기 시드, 실시간 자본 연동 동적 사이징)
- **거래 대상**: OKX 무기한 선물 3개 페어 병렬 감시 (`BTC-USDT-SWAP`, `ETH-USDT-SWAP`, `SOL-USDT-SWAP`)
- **마진 모드**: 격리 마진(Isolated Margin) 10배 (단방향 Net Mode)
- **알림 채널**: Discord Webhook 실시간 텔레메트리 연동 완료
- **서비스 상태**: `active (running)` 정상 무인 가동 중 (PID: 2322)

---

## 2. 3-Room 워크플로우 분업 파이프라인 (운영 프로토콜)
본 프로젝트는 시스템의 무결성과 수학적 엣지를 보호하기 위해 3개의 독립된 세션으로 엄격히 분화하여 운영한다.
* **Room 1: 전략 기획 & 아키텍처 룸 (Strategy & Architecture)**
  - 마이크로스트럭처 분석, 수수료/손익비 수학적 검증, 파라미터 최적화, 기능 명세서(Spec Sheet) 발급
* **Room 2: 프로덕션 구현 룸 (Implementation & Production)**
  - 기획 스펙 기반 `config/`, `core/`, `main.py` 파이썬 실코드 구현 및 Vultr VPS 실서버 패치/배포
* **Room 3: 퀀트 백테스트 룸 (Quant Backtesting / Google Colab)**
  - 거래소 틱/캔들 데이터 기반 파라미터 스위핑, 수수료/슬리피지 차감 실질 승률, 거래 빈도, MDD 검증

---

## 3. 코어 전략 철학 및 수학적 엣지 (SPEC-003 고빈도 복리 엔진)
- **핵심 철학**: "대박 홈런(추세 맹신)이 아닌, 높은 승률과 양의 기댓값(Positive Expectancy)으로 수수료를 극복하고 복리를 가속화한다."
- **후행 지표 전면 배제**: RSI, MACD, 이동평균선 등 후행 지표를 일체 사용하지 않으며, **오더북 유동성 스윕(SFP) + 강제 청산(미결제약정 급감 $\Delta OI$)**의 물리적 흔적만을 타겟팅.
- **수수료 침식(Fee Erosion) 원천 방어**: 5분봉 등 극초단타 노이즈를 배제하고, 스탑 거리가 두터운 **15분봉(15m)** 타임프레임을 고수하여 왕복 수수료를 순익의 5% 미만으로 통제.
- **타임프레임 스윗스팟**:
  - 상위 추세 필터(HTF): **1시간봉(1h) SMC BOS** (기존 4h에서 경량화하여 진입 기회 3~4배 확대)
  - 진입 타임프레임(LTF): **15분봉(15m) SFP** + 반전 꼬리 $\ge 35\%$ + 거래량 20이평 대비 $\ge 1.2$배
  - 청산 폭발 필터: 15초 메모리 덱 기반 **$\Delta OI_{15m} \le -0.2\%$** (강제 청산 발생 필수 검증)
- **진입 방식 (38.2% 얕은 풀백 지정가)**:
  - 캔들 종가 추격 매매 금지 $\rightarrow$ 스윕 캔들의 실제 반등/하락 꼬리 진폭 기준 38.2% 되돌림 지정가(Post-Only) 체결.
  - $\text{Long: } P_{limit} = Close - 0.382 \times (Close - Low)$
  - $\text{Short: } P_{limit} = Close + 0.382 \times (High - Close)$

---

## 4. 자금 관리 및 청산 매트릭스 (소액 시드 512 USDT 최적화)
- **실시간 고정 비율 리스크 모델 (Fixed Fractional 1.0% Risk)**:
  - 1회 거래당 손실 허용액 = $\text{Total Equity} \times 0.01$ (512 USDT 기준 정확히 5.12 USDT 고정)
  - 진입 수량 공식: $\text{Contracts} = \frac{\text{Risk Amount}}{|P_{limit} - P_{SL}| \times \text{ContractSize}}$
  - 잔고 증가 시 복리 가속, 잔고 감소 시 판돈 자동 축소로 파산 확률(Risk of Ruin) 0% 수렴.
- **포트폴리오 리스크 캡**:
  - `MAX_CONCURRENT_POSITIONS = 2` (BTC/ETH/SOL 중 최대 2개까지만 동시 진입 허용)
  - 2개 종목 동시 손절 시에도 포트폴리오 총 손실은 정확히 **-2.0%(-10.24 USDT)**로 차단.
- **소액 계좌 분할 익절 분기 로직 (51001 에러 방어)**:
  - **1계약(`contracts == 1`) 체결 시**: 60% 분할 불가 $\rightarrow$ **1.2R 단일 TP(1계약 전량)**로 자동 전환 (+1.20R 확정).
  - **2계약 이상(`contracts >= 2`) 체결 시**:
    - **TP 1 (@ 1.0R)**: 물량 **60% 익절** (+0.60R 확정) 즉시 잔여 40% SL을 진입가(Break-Even)로 상향 이동 (Free-Roll 확정).
    - **TP 2 (@ 1.5R)**: 잔여 **40% 전량 청산** (+0.60R 추가 확보 $\rightarrow$ 총 **+1.20R 완주**).

---

## 5. 실서버 배포 파일 구조 및 아키텍처 명세
```text
/root/okx_sfp_bot/
├── .env                     # OKX API 키(Live), 암호화 패스프레이즈, Discord Webhook 격리
├── bot_state.db             # SQLite 멀티 심볼 포지션/주문 영속성 DB
├── config/
│   └── settings.py          # 멀티 심볼, HTF 1h/LTF 15m, 38.2% 풀백, 리스크 캡 설정
├── core/
│   ├── execution.py         # 1계약 단일 TP 분기, 포지션 레벨 Hard SL, 51008/51001 방어
│   ├── signal_engine.py     # 심볼별 독립 1h BOS 추세, 15m SFP 판정, 꼬리 진폭 38.2% 산출
│   ├── state_manager.py     # aiosqlite 비동기 기반 멀티 심볼 트레이드 상태 복구
│   └── ws_client.py         # 3개 심볼 15초 실시간 OI 폴링 루프, 캔들 콜드스타트 제거
├── logs/
│   ├── bot.log              # 봇 내부 실시간 텔레메트리 로그
│   └── systemd_err.log      # 서비스 표준 에러 로그
└── main.py                  # 비동기 멀티태스킹 오케스트레이터, 3중 원자적 취소-체결 방어선
