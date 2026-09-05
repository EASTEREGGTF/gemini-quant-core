# [Project Handover] OKX SFP 퀀트 자동매매 시스템 마스터 인수인계서 (v2.3.2 Production Live)

## 1. 프로젝트 개요 및 운영 현황 (2026-09-05 긴급 핫패치 배포 완료)
- 시스템 명칭: OKX SFP + SMC Liquidity Sweep & Micro-structure Multi-Asset Live Bot
- 운영 인프라: Vultr Cloud VPS (Ubuntu 22.04 LTS, Seoul Region) / systemd 24/7 무인 상주 (okx-bot.service)
- 운용 자본: 512 USDT (초기 시드, 실시간 자본 연동 동적 사이징)
- 거래 대상: OKX 무기한 선물 3개 페어 병렬 감시 (BTC/USDT:USDT, ETH/USDT:USDT, SOL/USDT:USDT)
- 마진 모드: 격리 마진(Isolated Margin) 10배 (단방향 Net Mode)
- 알림 채널: Discord Webhook 실시간 텔레메트리 연동 완료
- 서비스 상태: active (running) 정상 무인 가동 중 (PID: 19460 / Cold-Start 캔들 수신 및 6단계 퍼널 정상 작동 확인)

---

## 2. 3-Room 워크플로우 분업 파이프라인 (운영 프로토콜)
본 프로젝트는 시스템의 무결성과 수학적 엣지를 보호하고 AI 컨텍스트 오염을 방지하기 위해 3개의 독립된 세션으로 엄격히 분화하여 운영한다.

1. Room 1: 전략 기획 & 아키텍처 룸 (Strategy & Architecture)
   - 마이크로스트럭처 분석, 수수료/손익비 수학적 검증, 파라미터 최적화, 기능 명세서(Spec Sheet) 발급
2. Room 2: 프로덕션 구현 룸 (Implementation & Production)
   - 기획 스펙 기반 config/, core/, main.py 파이썬 실코드 구현 및 Vultr VPS 실서버 패치/배포 형상 관리
3. Room 3: 퀀트 백테스트 룸 (Quant Backtesting / Google Colab)
   - 거래소 틱/캔들 데이터 기반 파라미터 스위핑, 수수료/슬리피지 차감 실질 승률, 거래 빈도, MDD 검증

---

## 3. 코어 전략 철학 및 수학적 엣지 (v2.3.2 최적화 엔진)
- 핵심 철학: "대박 홈런(추세 맹신)이 아닌, 높은 승률과 양의 기댓값(Positive Expectancy)으로 수수료를 극복하고 복리를 가속화한다."
- 후행 지표 전면 배제: RSI, MACD, 이동평균선 등 후행 지표를 일체 사용하지 않으며, 오더북 유동성 스윕(SFP) + 강제 청산(미결제약정 급감 delta OI)의 물리적 흔적만을 타겟팅.
- 수수료 침식(Fee Erosion) 원천 방어: 5분봉 등 극초단타 노이즈를 배제하고, 스탑 거리가 두터운 15분봉(15m) 타임프레임을 고수하여 왕복 수수료를 순익의 5% 미만으로 통제.
- 타임프레임 스윗스팟 및 파라미터 명세:
  - 상위 추세 필터(HTF): 1시간봉(1h) SMC BOS (HTF_PIVOT_WINDOW = 3, 7시간 윈도우로 추세 전환 랙 최소화)
  - 진입 타임프레임(LTF): 15분봉(15m) SFP (PIVOT_WINDOW = 3, 좌우 3봉/총 7봉 프랙탈 스윙 레벨 탐색)
  - 캔들 꼬리 거부 비율: 진폭 대비 25% 이상 (WICK_RATIO_MIN = 0.25, 급반등 몸통 흡수 수용)
  - 거래량 필터: 20이평 대비 1.0배 이상 (VOLUME_MULT = 1.0, 평균 볼륨 충족 시 통과)
  - 청산 폭발 필터: 15초 메모리 덱 기반 delta OI(15m) <= -0.2% (강제 청산 발생 필수 검증)
- 진입 방식 (38.2% 얕은 풀백 지정가):
  - 캔들 종가 추격 매매 금지 -> 스윕 캔들의 실제 반등/하락 진폭 기준 38.2% 되돌림 지정가(Post-Only) 체결.
  - Long: Limit = Close - 0.382 * (Close - Low)
  - Short: Limit = Close + 0.382 * (High - Close)

---

## 4. 자금 관리 및 청산 매트릭스 (소액 시드 512 USDT 최적화)
- 실시간 고정 비율 리스크 모델 (Fixed Fractional 1.0% Risk):
  - 1회 거래당 손실 허용액 = Total Equity * 0.01 (512 USDT 기준 정확히 5.12 USDT 고정)
  - 진입 수량 산정 공식: Contracts = floor( Risk Amount / (|Limit - SL| * ContractSize) )
  - 잔고 증가 시 복리 가속, 잔고 감소 시 판돈 자동 축소로 파산 확률(Risk of Ruin) 0% 수렴.
- 포트폴리오 리스크 캡:
  - MAX_CONCURRENT_POSITIONS = 2 (BTC/ETH/SOL 중 최대 2개까지만 동시 진입 허용)
  - 2개 종목 동시 손절 시에도 포트폴리오 총 손실은 정확히 -2.0%(-10.24 USDT)로 차단.
- 소액 계좌 분할 익절 분기 로직 (51001 에러 방어):
  - 1계약(contracts == 1) 체결 시: 60% 분할 불가 -> 1.2R 단일 TP(1계약 전량)로 자동 전환 (+1.20R 확정).
  - 2계약 이상(contracts >= 2) 체결 시:
    - TP 1 (@ 1.0R): 물량 60% 익절 (+0.60R 확정) 즉시 잔여 40% SL을 진입가(Break-Even)로 상향 이동 (Free-Roll 확정).
    - TP 2 (@ 1.5R): 잔여 40% 전량 청산 (+0.60R 추가 확보 -> 총 +1.20R 완주).

---

## 5. 실서버 배포 파일 구조 및 아키텍처 명세
/root/okx_sfp_bot/
├── .env                     # OKX API 키(Live), 암호화 패스프레이즈, Discord Webhook 격리
├── bot_state.db             # SQLite 멀티 심볼 포지션/주문 영속성 DB
├── config/
│   └── settings.py          # 멀티 심볼, v2.3.2 완화 파라미터(k=3, wick=0.25, vol=1.0), 리스크 캡 설정
├── core/
│   ├── execution.py         # 1계약 단일 TP 분기, 포지션 레벨 Hard SL, 51008/51001 방어
│   ├── signal_engine.py     # v2.3.2 6단계 투명 퍼널 로깅, 심볼별 1h BOS 추세, 15m SFP 판정
│   ├── state_manager.py     # aiosqlite 비동기 기반 멀티 심볼 트레이드 상태 복구
│   └── ws_client.py         # v2.3.2 타임스탬프 전환 감지 + Cold-Start 마감봉 즉시 로드 하이브리드 엔진
├── logs/
│   ├── systemd.log          # 서비스 표준 출력 로그 (6단계 퍼널 판정 및 캔들 마감 실시간 기록)
│   └── systemd_err.log      # 서비스 표준 에러 로그
└── main.py                  # 비동기 멀티태스킹 오케스트레이터, 3중 원자적 취소-체결 방어선

---

## 6. 프로덕션 트러블슈팅 및 핫픽스 기록 (Crucial Fixes)
1. OKX Error 51001 (Parameter sz error) 핫픽스: 
   - 512 USDT 소액 환경에서 비트코인 1계약 체결 시 int(1 * 0.6) = 0 수량 주문으로 거부되던 버그 해결.
   - contracts == 1일 때는 분할 주문을 생략하고 1.2R 단일 익절(1계약 전량)로 자동 분기 처리.
2. 풀백 공식 미세구조 왜곡 교정: 
   - 기준선과의 단순 거리가 아닌, 스윕 캔들의 물리적 반등폭(close - low, high - close)을 기준으로 38.2% 되돌림을 산출하도록 교정하여 추격 매수 위험 제거.
3. 스탑로스(Hard SL) 증발 레이스 컨디션 방어: 
   - 15분 봉 마감 시 미체결 취소 요청 전/후 3중 원자적 대조(_handle_order_filled)를 구축하여 무방비 포지션 방치 사고 100% 차단.
4. OKX Error 50119 (API key doesn't exist) 해결: 
   - IP 미지정 API 키의 자동 만료 방지 및 Vultr VPS 고정 IP 기반 신규 키 발급/반영.
   - settings.py에서 프로젝트 루트의 .env를 절대 경로(BASE_DIR / ".env")로 강제 바인딩.
5. [v2.3.2 핵심] WebSocket Cold-Start & Length Trap 무한 침묵 버그 박멸:
   - 문제: OKX 웹소켓은 캔들 스트림 수신 시 현재 형성 중인 캔들 1개(len == 1)만 전달함. 기존의 if len(candles) >= 2: 조건문이 최초 기동 및 소켓 재연결 시 영원히 False가 되어 24시간 동안 봉 마감 이벤트가 0회 발생하는 치명적 교착 상태(Deadlock) 유발.
   - 해결:
     1) Cold Start 도입: 기동 1초 만에 REST API로 직전 확정 마감봉을 즉시 yield하여 대기 시간 0초 실현.
     2) 타임스탬프 전환 엔진: 캔들 개수(len) 의존성을 폐기하고, 캔들 시작 타임스탬프가 다음 15분으로 넘어가는 순간(candle_ts > current_forming_ts)을 0ms 오차로 감지하여 확정 마감봉 방출.
6. [v2.3.2 핵심] 파라미터 과최적화(Overfitting) 해제 및 투명 퍼널 로깅:
   - 15m 좌우 8봉(k=8, 4.25시간 윈도우)의 과도하게 좁은 스윙 인식을 k=3(1.75시간 윈도우)로 현실화하고, 꼬리 거부 기준을 25%, 거래량을 1.0배로 완화.
   - core/signal_engine.py에 6단계 퍼널(SFP -> 1H 추세 -> 꼬리 -> 거래량 -> OI -> 타점 산출)의 실시간 통과/탈락 사유를 INFO/DEBUG로 출력하여 블랙박스 해소.

---

## 7. 차기 마일스톤 및 작업 가이드 (Next Steps)
- Phase 1 (현재 상태 - 실서버 무인 가동 모니터링):
  - Vultr VPS 실서버 24시간 실가동 모니터링 (15분 봉 마감 정상 수신 및 6단계 퍼널 로그 관찰).
  - 서비스 모니터링 명령어:
    systemctl status okx-bot.service --no-pager
    tail -f /root/okx_sfp_bot/logs/systemd.log
- Phase 2 (Room 3 퀀트 백테스트 검증):
  - Google Colab 백테스트: 최근 6개월 3개 심볼(BTC, ETH, SOL) 1h BOS + 15m SFP(v2.3.2 완화 스펙) 통합 백테스트 리포트 산출 (승률, 손익비, 최대 낙폭 MDD 검증).
- Phase 3 (Room 1 전략 확장 - 차기 v3.0 기획):
  - 추세 지속형 듀얼 모듈 도입: 강한 원웨이 빔 장세에서 유동성 스윕(SFP)을 기다리지 않고, 1H BOS 확정 직후 15m FVG(Fair Value Gap) 또는 Order Block 되돌림을 타는 지속형 진입 엔진 기획.
