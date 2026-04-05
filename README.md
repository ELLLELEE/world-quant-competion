# world-quant-competion

- 첫번째 알파 만드는 과정
  - 내가 선택한 초기 가설 : 리버젼 방식 -> 평균으로 회귀할 것이다.
  - 리버젼 방식 : 너무 과열된 애들은 다시 내릴 것이고 급락한 애는 다시 오를것

1) clv = ((close-low)-(high-close))/(high-low);
over_vol = volume > ts_mean(volume, 252);
-zscore(clv) * rank(over_vol)
