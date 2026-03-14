### Context
KODEX ETF를 활용한 섹터 로테이션 전략은 시장의 주도 테마를 선점하여 초과 수익(Alpha)을 추구하는 퀀트 매매 방식입니다. 본 문서는 특정 기간 수익률이 높은 섹터를 식별하고, 해당 섹터 내 대장주를 종가에 매수한 뒤, 중기 추세 지표인 MACD(20, 84, 7)를 활용하여 매도 시점을 포착하는 자동화된 투자 로직을 다룹니다.

### Core
이 전략의 핵심은 섹터 모멘텀 측정과 EMA(지수이동평균) 기반의 추세 이탈 감지입니다. 아래는 파이썬(Python)을 활용한 기술적 지표 계산 예시입니다.

```python
import pandas as pd
import talib

def get_sector_momentum(etf_prices):
    # 각 섹터 ETF의 최근 N일 수익률 계산 (예: 5일 또는 20일)
    returns = etf_prices.pct_change(periods=20).iloc[-1]
    return returns.sort_values(ascending=False)

def calculate_macd_custom(df, fast=20, slow=84, signal=7):
    # MACD(20, 84, 7) 계산: 중기 추세 추종용 설정
    macd, macdsignal, macdhist = talib.MACD(df['close'], 
                                            fastperiod=fast, 
                                            slowperiod=slow, 
                                            signalperiod=signal)
    return macd, macdsignal

def check_exit_signal(macd, signal):
    # Dead Cross 발생 여부 확인 (매도 시그널)
    if macd.iloc[-2] > signal.iloc[-2] and macd.iloc[-1] < signal.iloc[-1]:
        return True
    return False
```

### Insight
* MACD(20, 84, 7) 주기의 기술적 의미: 표준 설정인 (12, 26, 9)보다 주기가 훨씬 길어 단기 변동성(Noise)에 의한 가짜 신호를 억제합니다. 20일은 약 1개월, 84일은 약 4개월(1분기 이상)의 영업일을 의미하며, 주 추세가 완전히 꺾일 때 매도하여 수익을 극대화하는 설정입니다.
* 종가 베팅(Closing Auction Betting)의 강점: 장 마감 시점의 확정된 가격으로 진입함으로써 장중 변동성 리스크를 회피하고, 익일 시초가 갭 상승(Gap Up)을 노릴 수 있습니다. 특히 주도 섹터의 1순위 종목은 시장 자금이 집중되므로 익일 상승 확률이 상대적으로 높습니다.
* 연 수익률 20% 달성을 위한 전제 조건: 본 전략은 강세장과 순환매 장세에서 강력하지만, 지수가 우하향하는 하락장에서는 섹터 전체가 동반 하락할 위험이 있습니다. 따라서 시장 전체의 이동평균선(예: 코스피 200일선) 위에 지수가 위치할 때만 진입하는 필터를 병행하는 것이 권장됩니다.
* 리스크 관리: 종목 선정 시 거래대금이 해당 섹터 내에서 압도적인지 확인하여 환금성 리스크를 방지해야 합니다.

**출처:** [MACD Technical Indicator Guide](https://www.investopedia.com/terms/m/macd.asp), [KODEX Sector ETF List](https://www.kodex.com/product/sector.do)