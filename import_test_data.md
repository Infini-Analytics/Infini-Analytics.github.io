# 导入测试数据

导入一部分测试数据，以方便之后验证 Infini 图形渲染引擎和前端可视化组件的安装。测试数据抽取自 Uber 开放的纽约出租车订单数据，记录数 100,000 条。

1. 创建一个新表并安装FDW插件。
   ```
   postgres=# drop table if exists nyc_taxi;
   postgres=# create table nyc_taxi(
    vendor_id text,
    tpep_pickup_datetime timestamp,
    tpep_dropoff_datetime timestamp,
    passenger_count int,
    trip_distance float,
    pickup_longitute float,
    pickup_latitute float,
    dropoff_longitute float,
    dropoff_latitute float,
    fare_amount float,
    tip_amount float,
    total_amount float
    );
   postgres=# drop extension zdb_fdw;
   postgres=# create extension zdb_fdw;
   ```    

2. 获取测试数据。

   ```
   $ wget https://raw.githubusercontent.com/Infini-Analytics/infini/master/sample_data/nyc_taxi_data.csv
   $ docker cp nyc_taxi_data.csv <CONTAINER ID>:/megawise/
   ```

3. 向表中插入测试数据。

   ```
   postgres=# copy nyc_taxi from '/megawise/nyc_taxi_data.csv' WITH DELIMITER ',' csv header;
   ```
