![](https://cdn.qwiklabs.com/l0sFLZS%2BO9dGHcecgA2MrRf7u2BRusBkSVPzUDf8AUg%3D)
# Engineer Data in Google Cloud: Challenge Lab

**Overview**

할당 된 시간 내에 일련의 작업을 완료해야 합니다. 단계별 지침을 따르는 대신 시나리오와 일련의 작업이 제공됩니다. 직접 완료하는 방법을 알아내야 합니다!

Challenge Lab을 수강하더라도 BigQuery 또는 데이터 엔지니어링 개념을 배우지 않습니다. 제시된 과제에 대한 솔루션을 구축하려면 과제 랩이 속한 퀘스트에서 배운 기술을 사용해야 합니다. 

이 실습은 데이터 엔지니어링 퀘스트의 실습을 완료한 학생에게만 권장됩니다.

**Objectives**

Topics tested:

- 기존의 데이터로부터 BigQuery 테이블 생성
- BigQuery, Dataprep 혹은 Dataflow를 사용하여 ML Model 생성을 위한 데이터 정제
- BQML에서 모델 생성 및 최적화
- BQML에서 예측된 데이터 생성

---

## 도전 과제 시나리오

책상에 앉자마자 첫 번째 과제를 받게 됩니다. 요금을 예측하기 위한 BQML을 생성하려고 합니다. 데이터를 가져오고, 정제한 다음 새 데이터로 모델을 생성하고 배치 예측을 수행하여 모델 성능을 검토하고 애플리케이션 배포에 대한 진행 여부 결정을 내릴 수 있도록 합니다.

## Task 1: 훈련 데이터의 정제

랩이 시작될 때 데이터 집합을 만들고 데이터를 `taxirides`와 `historical_taxi_rides_raw` 테이블에 가져왔습니다. 

이 작업을 마무리 하기 위해 다음을 수행합니다.

- `historical_taxi_rides_raw` 데이터를 정제하고 `taxi_training_data`를 같은 데이터셋에 생성합니다. BigQuery, DataPrep, DataFlow등을 사용하여 데이터를 정제할 수 있습니다. 학습하고 하는 대상 속성은 `fare_amount`입니다.

힌트

- BQ UI에서 소스 데이터 집합을 볼 수 있으며, 먼저 서스 스키마를 숙지합니다.
- 예측 시 확인할 수 있는 데이터에 대한 힌트로서, 예측 시간에 도착한다는 것을 보여주는 `taxirides.report_prediction_data` 데이터를 숙지합니다.

데이터 정제 작업:

- `trip_distance`가 0보다 커야 합니다.
- `fare_amount` 금액이 작은 데이터는 제거합니다($2.5 미만).
- 위도와 경도가 사용 사례에 적합한지 확인합니다.
- `passenger_count`가 0보다 커야 합니다.
- `total_amount`에는 팁이 포함되므로 `tolls_amount` 및 `fare_amount`를 대상 변수로 추가합니다.
- 소스 데이터 집합이 크기 때문에(>1억행), 데이터 집합을 100만건 미만으로 샘플링합니다.
- 모델에서 사용할 필드만 복사합니다(`report_prediction_data`는 좋은 예제입니다).

```sql
CREATE OR REPLACE TABLE
    taxirides.taxi_training_data AS
SELECT
    (tolls_amount + fare_amount) AS fare_amount,
    pickup_datetime,
    pickup_longitude,
    pickup_latitude,
    dropoff_longitude,
    dropoff_latitude,
    passenger_count,
FROM
    taxirides.historical_taxi_rides_raw
WHERE
    RAND() < 0.001
    AND trip_distance > 0
    AND fare_amount >= 2.5
    AND pickup_longitude > -78
    AND pickup_longitude < -70
    AND dropoff_longitude > -78
    AND dropoff_longitude < -70
    AND pickup_latitude > 37
    AND pickup_latitude < 45
    AND dropoff_latitude > 37
    AND dropoff_latitude < 45
    AND passenger_count > 0
```

## Task2. BQML 모델 생성 (taxirides.fare_model)

`taxirides.taxi_training_data`의 데이터를 기반으로 `fare_amount`을 예측하는 BQML 모델을 생성합니다. 모델의 이름은 `taxirides.fare_model`로 설정하며, 모델의 성능은 10이하의 RMSE가 되도록 합니다.

힌트:

- TRANSFORM() 절을 사용하여 데이터 변환에 대한 부분을 캡슐화 합니다.
- TRANSFORM() 절의 특징만 모델에 전달된다는 점에 유의합니다. * EXCEPT(feature_to_leave_out)을 사용하여 명시적으로 호출하지 않고 일부 또는 모든 기능을 전달할 수 있습니다.
- BigQuery에 있는 ST_distance()와 ST_GeogPoint() GIS 함수를 사용하여 유클리드 거리(즉, 택시가 얼마나 멀리 주행했는가)를 쉽게 계산할 수 있습니다.

```sql
ST_Distance(ST_GeogPoint(pickuplon, pickuplat), ST_GeogPoint(dropofflon, dropofflat)) AS euclidean
```


```sql
CREATE OR REPLACE MODEL taxirides.fare_model
TRANSFORM(
  * EXCEPT(pickup_datetime),
  ST_Distance(ST_GeogPoint(pickuplon, pickuplat), 
  ST_GeogPoint(dropofflon, dropofflat)) AS euclidean,
  CAST(EXTRACT(DAYOFWEEK FROM pickup_datetime) AS STRING) AS dayofweek,
  CAST(EXTRACT(HOUR FROM pickup_datetime) AS STRING) AS hourofday
)
OPTIONS(input_label_cols=['fare_amount'], model_type='linear_reg') 
AS
SELECT * FROM taxirides.taxi_training_data
```

## Task3. 새로운 데이터에 대한 배치형태의 예측

2015년에 수집한 모든 데이터를 얼마나 잘 예측하는지 궁금합니다. taxirides.report_prediction_data에 데이터가 존재합니다. 알려진 예측 시간이 해당 테이블에 존재합니다.

ML.PREDICT를 사용하여 fare_amount를 예측하고 결과를 2015_fare_amount_predictions 테이블에 저장합니다.

```sql
CREATE OR REPLACE TABLE taxirides.2015_fare_amount_predictions
  AS
SELECT * FROM ML.PREDICT(MODEL taxirides.fare_model,(
  SELECT * FROM taxirides.report_prediction_data)
)​
```


### 축하합니다!!!