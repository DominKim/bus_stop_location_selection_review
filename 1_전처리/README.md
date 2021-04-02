## 전처리

### data set
- df_500(training data set)
    * 버스정류장 기준으로 500m 버퍼로 독립변수와 종속변수 데이터들을 뽑은 df
- df_100(수원시)
    * gid(100m x 100m)격자를 기준으로 독립변수 데이터들을 뽑은 df
- df_100_to_500(test data set)
    * gid격자의 중심을 기준으로 500m 버퍼로 독립변수 데이터들을 뽑은 df
    
### 전처리 순서
1. df_500 기준 data set 추출
2. 지하철 관련 data set 추출
3. 유동인구 - 인구정보 결합 data set 추출
4. 도로 관련 data set 추출
5. 토지이용특성 변수 도출
6. df_500 독립변수 종속변수 도출
7. df_100 독립변수 도출
8. df_100_to_500 독립변수 도출

### 코드 리뷰
- 동적변수 데이터 할당(globals())
    * 현재 global 변수들을 딕셔너리 형태로 return
``` python3
for idx in tqdm.tqdm(range(len(file_df_) -2)):
    file_name = ".".join(file_df.iloc[idx, :])
    globals()[f"df_{idx + 1}"] = gpd.read_file("./data/" + file_name)
``` 

- [geopandas.sjoin()](https://geopandas.org/docs/user_guide/mergingdata.html)
    * pandas의 merge와 같은 기능, 공간을 활용한 merge라는 차이가 있음
        ! op : 병합 방식 적용(contains, intersects etc..) [참고링크](https://geopandas.readthedocs.io/en/latest/docs/reference/api/geopandas.GeoSeries.intersects.html)
        ! how : join 방식(left, inner, right)
``` python3
bus_count = gpd.sjoin(bus_buffer, bus_point, "inner", op = "contains")["정류장ID_left"].value_counts()
```

- [geopandas.overlay()](https://geopandas.readthedocs.io/en/latest/docs/reference/api/geopandas.overlay.html?highlight=overlay)
    * 두개의 df를 받아 how에 따라 polygon형태를 포함한 df 리터
    * 이번 프로젝트에서는 버퍼 안에 속하는 건물 면적을 구하기 위해 사용
``` python3
df_1_build_pop_final_inter = gpd.overlay(df_1_build_crit, final_df, how = "intersection")
```

- [geoseries.centroid](https://geopandas.readthedocs.io/en/latest/docs/reference/api/geopandas.GeoSeries.centroid.html?highlight=centroid)
    * polyon의 중심 좌표 반환
    
- [GeoSeries.distance](https://geopandas.readthedocs.io/en/latest/docs/reference/api/geopandas.GeoSeries.distance.html?highlight=distance)
    * point에서 point의 거리 반환
``` python3
bus_with_build_area_df["center"].distance(bus_with_build_area_df["bus_center"])
```

- [GeoSeries.area](https://geopandas.readthedocs.io/en/latest/docs/reference/api/geopandas.GeoSeries.distance.html?highlight=distance)
    * polygon의 면적 반환(좌표계에 따라 크기가 다름)
``` python3
# 기준 면적(100mx100m 격자)
crit_area = final_df["geometry"].area
```

- [shapely.wkt.loads](https://stackoverflow.com/questions/51855917/shapely-polygon-from-string)
    * str 형태의 polygon 들을 polygon 형태로 변환
    * 주의점 : geopandas의 df의 컬럼의 값을 변경할 때 컬러명이 geometry가 아니면 위의 함수를 사용해도 변경이 안된다.
    * apply로 아래의 함수를 적용하고 dtype()으로 geometry로 변경
``` python3
def str_to_geo(x):
    a = shapely.wkt.loads(x)
    return 

df_gid_pop["gid_center"] = df_gid_pop["gid_center"].apply(lambda x: str_to_geo(x))
df_gid_pop["gid_center"] = df_gid_pop["gid_center"].astype("geometry")
```