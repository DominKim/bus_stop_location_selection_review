## 전처리

### data set
- df_500(training data set)
    * 버스정류장 기준으로 500m 버퍼로 독립변수와 종속변수 데이터들을 뽑은 df
- df_100(수원시)
    * gid(100m x 100m)격자를 기준으로 독립변수 데이터들을 뽑은 df
- df_100_to_500(test data set)
    * gid격자의 중심을 기준으로 500m 버퍼로 독립변수 데이터들을 뽑은 df
    
### 모델링 순서
1. df_500 교차검증 실시
2. df_500 이상치 제거
3. df_500 cleansing data 교차검증 실시
4. df_100_to_500 독립변수 5점 척도화
5. df_100_to_500 지수 도출, 광고 Tareget 추출


### 코드 리뷰
- cross_val_score(model, x, y, cv = kfold) : 교차검증
``` python3
from sklearn.model_selection import KFold, train_test_split, cross_val_score
def check_score(df, kfold, x_col, y_col, model):
    x = df[x_col]
    y = df[y_col]
    print("교차 검증 점수 : ", cross_val_score(model, x, y, cv = kfold))
```

- folium을 통한 시각화
    * style
        - fillColor : RGB값 입력, 버퍼내 색상
        - color : 경계선 색삭
        - opacity : 경계선 투명도
        - fillOpacity : 버퍼내 색상 투명도
    * m = folum.map([lat, lon], zoom_start = 7)
        - 기준 점
    * df.crs
        - 데이터 프레임에 좌표계 설정
        
    * folum.GeoJson
        - data : 시각화할 데이터 입력
        - style_function : 시각화 스타일 입력
    
``` python 3
# air 시각화
style0 = {'fillColor': '#228B22', 'color': 'black', "opacity":"0.3", "fillOpacity":"0"}
style1 = {'fillColor': '#228B22', 'color': 'blue', "fillOpacity":"0","opacity":"1"}
style2 = {'fillColor': '#00FFFFFF', 'color': 'red', "fillOpacity":"0","opacity":"1"}
style3 = {'fillColor': '#00FFFFFF', 'color': 'red', "opacity":"0.5"}
m = folium.Map([37,127], zoom_start=7)

# df_100_0.crs = "epsg:4326"
df_plot.crs = "epsg:4326"
df_plot_bus_geo.crs = "epsg:4326"
df_500.crs = "epsg:4326"
road_geo.crs = "epsg:4326"

m = folium.Map(max_bounds = True,
               zoom_start=12,
               location=[37.277896,127.016544])


folium.GeoJson("./data/31.수원시_행정경계(읍면동).geojson", name="geojson", style_function = lambda x: style0).add_to(m)
folium.GeoJson(data = df_plot.loc[:, "geometry"], style_function = lambda x:style1).add_to(m)
folium.GeoJson(data = df_plot_bus_geo.loc[:, "geometry"], style_function = lambda x:style2).add_to(m)
folium.GeoJson(data = road_geo.loc[:, "geometry"], style_function = lambda x:style3).add_to(m)
m

```

