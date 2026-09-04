# [시스템 명세서] Smart Money Liquidity Sweep Algorithm (v1.0)

- **타겟 거래소**: OKX 선물 (USDT 무기한 스왑: BTC-USDT-SWAP, ETH-USDT-SWAP 등)
- **타임프레임**: 
  - HTF (상위 추세): 4시간봉 (4H)
  - LTF (진입 및 패턴 감지): 15분봉 (15m)
- **코어 철학**: 후행성 보조지표(EMA, RSI 등) 배제. 스마트머니(세력)의 개미 스탑로스/청산 유동성 사냥(Liquidity Sweep) 역이용.

---

## 1. HTF 구조적 추세 필터 (4H Market Structure)
일반 지표 대신 4H 캔들의 SMC 구조(Swing High/Low 돌파)만으로 추세를 판별한다.

- **피봇 기준**: 좌우 5개 봉 기준 최고점($SH_{HTF}$), 최저점($SL_{HTF}$)
- **추세 정의**:
  - **상승 추세 (Bullish, +1)**: 최근 4H 종가가 직전 $SH_{HTF}$ 상방 돌파 AND 직전 $SL_{HTF}$ 하방 이탈 없음
  - **하락 추세 (Bearish, -1)**: 최근 4H 종가가 직전 $SL_{HTF}$ 하방 돌파 AND 직전 $SH_{HTF}$ 상방 돌파 없음
  - **중립 (Neutral, 0)**: 위 조건 미충족 (횡보장, 신규 진입 차단)
- **원칙**: Bullish일 때는 '롱 SFP'만, Bearish일 때는 '숏 SFP'만 탐색.

---

## 2. 15m SFP (Swing Failure Pattern) 진입 조건
개미들의 돌파 추격 물량과 손절(SL)을 터트리고 즉시 박스 안으로 되돌아오는 움직임을 포착.

- **주요 스윙 레벨 ($P_{key}$)**: 15분봉 피봇($k=8$, 최소 20봉 이상 숙성된 고점 $H_{swing}$ 또는 저점 $L_{swing}$)
- **꼬리 비율 (Wick Ratio)**:
  - 하단 꼬리: $(\min(Open, Close) - Low) / (High - Low)$
  - 상단 꼬리: $(High - \max(Open, Close)) / (High - Low)$

### [롱 SFP (Bullish)]
1. $Trend_{HTF} == +1$ (4H 상승 추세)
2. $Low < L_{swing}$ (직전 저점 하방 이탈로 Sell-Stop 유도)
3. $Close > L_{swing}$ (15분봉 마감 시 저점 위로 즉각 가격 복귀: Reclaim 성공)
4. 하단 꼬리 비율 $\ge 0.35$ (캔들의 35% 이상이 밑꼬리)
5. 거래량 $\ge SMA(Volume, 20) \times 1.5$ (평균 대비 1.5배 이상 유동성 흡수)

### [숏 SFP (Bearish)]
1. $Trend_{HTF} == -1$ (4H 하락 추세)
2. $High > H_{swing}$ (직전 고점 상방 돌파로 Buy-Stop 유도)
3. $Close < H_{swing}$ (15분봉 마감 시 고점 아래로 즉각 가격 복귀: Reclaim 성공)
4. 상단 꼬리 비율 $\ge 0.35$ (캔들의 35% 이상이 윗꼬리)
5. 거래량 $\ge SMA(Volume, 20) \times 1.5$ (평균 대비 1.5배 이상 흡수 거래량)

---

## 3. Confluence 필터 (OI 및 펀딩비)
진짜 세력의 스윕인지, 강력한 추세 돌파인지 구별하는 필수 안전장치.

- **롱 진입 컨펌**:
  - $\Delta OI_{15m} \le -1.5\%$ (스윕 시 롱 손절/청산으로 미결제약정 급감)
  - 실시간 펀딩비 $\le -0.02\%$ (하방 숏 과열 상태)
- **숏 진입 컨펌**:
  - $\Delta OI_{15m} \le -1.5\%$ (스윕 시 숏 스퀴즈/청산으로 미결제약정 급감)
  - 실시간 펀딩비 $\ge +0.02\%$ (상방 롱 과열 상태)

---

## 4. 리스크 관리 및 주문 파라미터
어떤 경우에도 1회 거래당 계좌 리스크는 1%로 고정하며, 손익비 1:2 미만 타점은 버린다.

- **1회 손실 허용 한도 ($Risk$)**: 계좌 자산($Equity$)의 1% 고정
- **손절가 (SL)**:
  - 롱: $SL = Low_{sweep} - \max(Entry \times 0.0005, 0.2 \times ATR_{14})$
  - 숏: $SL = High_{sweep} + \max(Entry \times 0.0005, 0.2 \times ATR_{14})$
- **포지션 크기 (Position Size)**:
  - 진입 수량 = $Risk / |Entry - SL|$
- **익절가 (TP) 및 R:R 필터**:
  - 반대편 미체결 유동성 풀(직전 주요 고/저점)까지의 거리 기준 기대 손익비 계산
  - $\text{Expected R:R} = |Target - Entry| / |Entry - SL| \ge 2.0$ 필수
  - **TP 1**: 진입가 대비 $2.0 \times \text{Risk}$ 도달 시 물량 50% 익절 + 남은 물량 본절(BE) 이동
  - **TP 2**: 반대편 유동성 도달 시 전량 청산

---

## 5. 실전 예외 방어 로직 (Micro-Structure Safeguards)
1. **스윕 깊이 제한 (가짜 스윕 / 칼날 잡기 방지)**:
   - $0.1 < (|Low_{sweep} - L_{swing}| / ATR_{14}) \le 1.5$ 범위만 승인. 
   - $1.5 \times ATR$을 초과해 찢고 내려간 것은 강한 추세 돌파이므로 진입 무효화.
2. **슬리피지 방지 주문 실행**:
   - 봉마감 즉시 시장가 추격 매수 금지.
   - 스윕 캔들의 Reclaim 레벨($P_{key}$) 또는 0.382 되돌림에 Limit 지정가 주문 발주.
   - 2개 캔들(30분) 내 미체결 시 주문 자동 취소(Kill).
