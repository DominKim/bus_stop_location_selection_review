# bus_stop_location_selection_review
- [수원시_스마트_버스정류장_우선_설치위치_선정](https://compas.lh.or.kr/subj/past/info?subjNo=SBJ_2102_002)
  * 이 레포지토리는 약 20일간 대회를 참여하면서 사용한 코드와 지식들을 리뷰용임.
  
| 구분  | 내용 | 
| -------------| ------------- | 
| 수행기간 | 2021.03.01 ~ 2021.04.01 | 
| 수행인원 | 2명 |
| 사용언어 및 도구 | Python |
| 주요 분석 기법 | 군집분석, T-test, SGD Regressor, 지수 산출, 공간회귀 |
| 주요 패키지 | Geopandas, Pandas, Sklearn, numpy, Seaborn |

## 주의사항
- 비공개 데이터의 사용
  * 비공개들의 데이터들을 직접 사용해서 스크립트를 실행하기 위해서는 위의 링크로 들어가서 compas 서버상에서 직접 다운로드 후 실행해야 된다.

## 분석방법
1. TOD와 공공시설 입지선정 변수들(독립변수)을 활용하면 버스 정류장 승하차건수(종속변수)예측 한다.
2. 격자별 대기오염지수 변수들을 추출하여 승하차건수와 대기오염지수를 고려하여 버스정류장 우선 설치위치를 선정한다.
    * 설치위치 지수 식
        * 지수 = 승하차건수 * 0.8 + 대기오염지수 * 0.2
3. 모델링 결과 시 성능이 안좋으면 새로운 지수를 생성하여 우선 설치위치를 선정한다.
    * 선형회귀 시 유의한 독립변수들과 대기오염지수를 5점 척도로 변환 후 합하여 지수 산출
        * 지수(n + 1 ~ (n + 1) * 5) = 독립변수_1(1 ~ 5) + ... + 독립변수_1(1 ~ 5) + 대기오염지수(1 ~ 5)
        
        
## 분석순서
<img src="https://github.com/DominKim/bus_stop_location_selection_review/blob/main/image/분석순서.png" width="80%" height="50%"></img>

## 공간분석
- python 공갑분석 패키지 : [Geographic Data Sceience with Python](https://geographicdata.science/book/notebooks/04_spatial_weights.html)

