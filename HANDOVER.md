# [Project Handover] OKX Micro-Liquidity Reverse-Psychology Bot (v3.1.0 Master)

## 1. 운영 인프라 및 실서버 현황
- **시스템 명칭**: OKX Micro-Liquidity 24/7 Dynamic Scalping Bot (v3.1.0)
- **운영 인프라**: Vultr Cloud VPS (Ubuntu 22.04 LTS, Seoul Region) / systemd (okx-bot.service)
- **운용 자본**: 계좌 총자본 연동 완전 동적 사이징 (Scale-Invariant)
- **대상 페어**: OKX 무기한 선물 (`BTC/USDT:USDT`, `ETH/USDT:USDT`, `SOL/USDT:USDT`)
- **마진 모드**: 격리 마진(Isolated Margin) 10배 (단방향 Net Mode)
- **알림 채널**: Discord Webhook 실시간 텔레메트리 연동

---

## 2. 3-Session 분업 파이프라인
1. **전략기획방**: 시장 미세구조 분석, 수수료/손익비 수학적 검증, 아키텍처 명세서(ARCHITECTURE.md) 발급.
2. **코드구현방**: Python 비동기 주문 엔진, WebSocket 실시간 캔들 마감 감지, Vultr 실서버 형상 관리.
3. **구글코랩 백테스트방**: 과거 실데이터 수집, Lookahead Bias 배제 시뮬레이션, 수수료 차감 후 실질 엣지 검증.

---

## 3. 코어 전략 핵심 사양 (v3.1.0-UNIVERSAL)
- **철학**: 개미 90%의 FOMO/Panic 심리 역이용 + 24시간 캔들 단위 미세 유동성 사냥.
- **방향성**: 상위 프레임 족쇄 해제, 롱/숏 양방향 24시간 무제한 진입 (Direction Agnostic).
- **타임프레임**: 5분봉(`5m`) 및 15분봉(`15m`).
- **진입 트리거 (Zero-Lag Micro-SFP)**:
  - Short: 현재 캔들 High가 직전 N개 봉 High를 찌르고 종가가 직전 High 아래로 마감 ➜ 다음 봉 시가 즉시 SHORT.
  - Long: 현재 캔들 Low가 직전 N개 봉 Low를 깨고 종가가 직전 Low 위로 마감 ➜ 다음 봉 시가 즉시 LONG.
- **체결 방식**: 풀백 대기 전면 폐기, 캔들 마감 즉시 최유리 BBO 지정가 또는 시장가 즉각 체결.
- **자금 관리 및 청산**:
  - 거래당 리스크: `Total Equity * RISK_FRACTION` (동적 계약수 산출).
  - 손절(SL): 스윕 캔들 꼬리 끝 + Dynamic ATR 버퍼.
  - 익절(TP): 고정 **$1.5R$** 또는 반대편 직전 캔들 극값 도달 시 전량 청산.
  - 포트폴리오 캡: 동시 최대 2개 포지션 제한.

---

## 4. 세션별 차기 작업 가이드
- **코드구현방**:
  1. `settings.py`에서 하드코딩된 계약 크기 제거 및 `load_markets()` 동적 바인딩.
  2. 1H BOS 추세 필터 및 38.2% 풀백 지정가 대기 로직 완전 삭제.
  3. 직전 봉 스윕 마감 즉시 진입하는 v3.1.0 엔진으로 리팩토링 후 Vultr 실서버 핫패치 배포.
- **구글코랩 백테스트방**:
  1. v3.1.0 엔진 기준 과거 6개월 5m/15m 실데이터 백테스트 단일 셀 스크립트 작성 및 Net PnL 검증.
