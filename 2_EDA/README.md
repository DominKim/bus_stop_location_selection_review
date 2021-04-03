## 전처리

### data set
- df_500(training data set)
    * 버스정류장 기준으로 500m 버퍼로 독립변수와 종속변수 데이터들을 뽑은 df
- df_100(수원시)
    * gid(100m x 100m)격자를 기준으로 독립변수 데이터들을 뽑은 df
- df_100_to_500(test data set)
    * gid격자의 중심을 기준으로 500m 버퍼로 독립변수 데이터들을 뽑은 df
    
### 전처리 순서
1. df_500 실루엣 계수를 통한 군집수 선정
2. df_500 군집 별 평균차이 검정
3. df_100 실루엣 계수를 통한 군집수 선정
4. df_100 군집 별 평균차이 검정
5. df_100_to_500 실루엣 계수를 통한 군집수 선정
6. df_100_to_500 군집 별 평균차이 검정
7. df_500 상관분석을 통해 상관성이 높은 변수 중 변수제거 (why? 다중공선성 사전 제거)
8. df_500 vif 다중공선성을 통해 10이상의 값을 보이는 변수 제거
9. 공간회귀 분석을 통해 유의한 변수 추출(queen 가중치 사용)

### 코드 리뷰
- geopandas.points_from_xy(lon, lat) : 위동와 경도로 geometry 타입의 point 추출
``` python3

# geometry poin 생성
geo = gpd.points_from_xy(df_500.lon, df_500.lat)
```
- Kmeans(n_clusters = i, init = "k-means++", max_iter = 300) : kmeans 군집분석
``` python3
from sklearn.cluster import KMeans, DBSCAN
kmeans = KMeans(n_clusters=i, init="k-means++", max_iter=300)
kmeans.fit(df[col])
df["label"] = kmeans.labels_
```

- silhouette_samples(독립변수, 종속변수(군집)) : 군집분석을 평가하기 위한 실루엣계수 점수 추출
``` python3
from sklearn.metrics import silhouette_samples, silhouette_score
score_samples = silhouette_samples(df[col], df["label"])
df["silhouette_coeff"] = score_samples
```

- silhouette_score(독리변수, 종속변수) : 실루엣계수 평균 추출
``` python3
from sklearn.metrics import silhouette_samples, silhouette_score
average_score = silhouette_score(df[col], df["label"])
score_lst.append(average_score)
```

- sns.lineplot(x = , y, ...) : lineplot
    * plt.xlabel() : x축의 레이블 입력
    * plt.ylabel() : y축의 레이블 입력
    * ply.ylim() : y축의 범위
    * plt.grid() : plot의 grid 설정
``` python3
sns.lineplot(x = range(2, 6), y = score_lst, markers=True)
plt.xlabel("군집 수")
plt.ylabel("실루엣 계수")
plt.ylim(0,1)
plt.grid(True, axis = "y", alpha = 0.5, linestyle = "--")
```

- stats.ttest_ind(df[col1], df[col2]) : ttest
``` python3
from scipy import stats
stats, pvalue = stats.ttest_ind(df_500_0[col], df_500_1[col])
```

- sns.heatmap() : 상관분석 히트맵 사용시 주로 사용
``` python3
sns.heatmap(df_500[x_col].corr(), annot=True)
```
- [variance_inflation_factor(value, i)](https://www.statsmodels.org/stable/generated/statsmodels.stats.outliers_influence.variance_inflation_factor.html) : 다중공선성 확인
``` python3
#다중공선성 체크
from statsmodels.stats.outliers_influence import variance_inflation_factor

# 다중공선성 확인 "버스정류장수" 제일 높음
vif = pd.DataFrame()
vif["vif_value"] = [variance_inflation_factor(df_500[x_col].values, i) for i in range(len(x_col))]
vif["columns"] = x_col

vif
```
#











###
