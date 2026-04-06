# world-quant-competion

## 첫번째 알파 만드는 과정
  - 내가 선택한 초기 가설 : 리버젼 방식 -> 평균으로 회귀할 것이다.
  - 리버젼 방식 : 너무 과열된 애들은 다시 내릴 것이고 급락한 애는 다시 오를것

1) clv = ((close-low)-(high-close))/(high-low);
   
    over_vol = volume > ts_mean(volume, 252);
    -zscore(clv) * rank(over_vol)

- result : 턴오버 135%
- reason :
  - clv 너무 단기적이고 noisy clv는 하루짜리 캔들 위치만 봄
  - over_vol = volume > ts_mean(volume, 252); -> 연속 값이 아닌 트루 폴스값으로 나옴

2) vol_surprise = volume / ts_mean(volume, 252);
   
    -zscore(clv) * rank(vol_surprise)

- 개선방향 : 거래량이 평균 대비 얼마나 큰지를 연속적으로 반영

3) rank(ts_mean(close, 20) - close)

- 현재 종가가 20일 평균보다 너무 위면 숏 너무 아래면 롱

  
4) event = volume>adv20;
   
   alpha = (-ts_delta(close,5));
   
    trade_when(event,alpha,-1)

- 거래량과 가격 차이를 기준으로 거래
- 거래량이 20일 평균보다 크면 최근 5일간의 가격 차이를 기준으로 거래

5) vwap_dev = (close - vwap) / vwap;
   
    vol_shock = ts_zscore(volume, 20);
   
    -rank(vwap_dev) * rank(vol_shock)

- 종가가 VWAP위로 많이 뜬 종목
- 거래량도 비정상적으로 많음
- 다음날 되돌림 기대

6) -rank((close - vwap) / vwap)

- VWAP 위에서 과열 마감한 종목 숏

### 첫번째 알파의 핵심 아이디어
7) -ts_zscore(enterprise_value/ebitda, 63)

- EX/EBITDA 가 최근 기준으로 상대적으로 높으면 비싸다
- 고밸류 종목은 숏 / 저밸류 종목은 롱

- result
  - 1) decay 8 industry -> 1.17, 12.02, 0.97
    2) decay 8 subindustry -> 1.21, 12.65, 0.96

- 결과 분석
  1) return이 너무 낮아 fitness가 1.0을 넘지 못함
  2) 너무 안정적인 코드

- 개선방향
  - 리버젼 + 리버젼

8) z1 = (close - ts_mean(close, 10)) / ts_std_dev(close, 10);
   
    z2 = (close - vwap) / vwap;

    -(rank(z1) + rank(z2))

- 최근 평균 대비 과열 + 당일 VWAP 대비 과열 -> 과열이면 숏 과매도면 롱
- z1 = (close - ts_mean(close, 10)) / ts_std_dev(close, 10);
  - 최근 10일 평균 기준으로 지금 가격이 얼마나 떨어져/올라 있는지
  - z1 > 0  -> 평균 보다 위 -> 과열
  - z1 < 0 -> 평균 보다 아래 -> 과매도
  - 중기 기준 리버전 신호
- z2 = (close - vwap) / vwap;
  - 당일 거래 평균 가격(vwap) 대비 현재 종가 어디있는지
  - z2 > 0 -> 오늘 거래보다 위 -> 매수 쏠림
  - z2 < 0 -> 오늘 거래보다 아래 -> 매도 쏠림
  - 단기 기준 리버전 신호

-> 최근에도 과열 + 오늘도 과열 -> 진짜 과열

- rank(z1) , rank(z2) -> 모든 종목을 상대적 순위로 변환
  - 괄호값이 크면 매우 큰 과열 -> 음수 붙여서 숏

-result 
  - sharpe : 1.72 turnover : 55% fitness : 1.0

### 최종 첫번째 알파 
9)
    z1 = (close - ts_mean(close, 10)) / ts_std_dev(close, 10);
  
     z2 = ts_mean((close - vwap) / vwap, 2);
  
     -(rank(z1) + rank(z2))
   
- setting : decay :4 , industry
- z2에 ts_mean 추가
- z2 = ts_mean((close - vwap) / vwap, 2);
  - VWAP 괴리를 2일 평균으로 부드럽게
  - 원래 코드는 하루를 기준으로 값을 측정하기때문에 불안정 하고 노이즈 많음
  - 2일 평균으로 하면 오늘과열했고 어제도 과열 했다 -> 신뢰도 높음
 
  - result
    - Sharpe : 1.62
    - turnover : 49%
    - Fitness : 1.02
  

   

  




  
