# Apache Iceberg on AWS

## Table of contents

- [What's included](#whats-included)
- [Set Up Work](#set-up)
- [Main Tutorial](#main-tutorial)
- [Useful Links](#useful-link)
- [Creators](#creators)

## What's included

The repo is to supplement the [youtube video](https://youtu.be/iGvj1gjbwl0) on Iceberg in AWS.

## Set up

1. Run the cloud formation template. This will create;
- The S3 bucket
- Glue database
- Athena work group

2. Upload the data from the `data` folder


## Main Tutorial
1. Upload the CSV data from `./data/csv`
2. Create the CSV table using athena
    ```
    CREATE EXTERNAL TABLE
      iceberg_tutorial_db.nyc_taxi_csv
      (
            vendorid bigint,
            tpep_pickup_datetime timestamp,
            tpep_dropoff_datetime timestamp,
            passenger_count double,
            trip_distance double,
            ratecodeid double,
            store_and_fwd_flag string,
            pulocationid bigint,
            dolocationid bigint,
            payment_type bigint,
            fare_amount double,
            extra double,
            mta_tax double,
            tip_amount double,
            tolls_amount double,
            improvement_surcharge double,
            total_amount double,
            congestion_surcharge double,
            airport_fee double
      )
      ROW FORMAT DELIMITED
      FIELDS TERMINATED BY ','
      STORED AS TEXTFILE
      LOCATION 's3://<s3-bucket-name>/csv/';
    ```
3. Create an Apache Iceberg table
    ```
    CREATE TABLE
      iceberg_tutorial_db.nyc_taxi_iceberg 
      (
            vendorid bigint,
            tpep_pickup_datetime timestamp,
            tpep_dropoff_datetime timestamp,
            passenger_count double,
            trip_distance double,
            ratecodeid double,
            store_and_fwd_flag string,
            pulocationid bigint,
            dolocationid bigint,
            payment_type bigint,
            fare_amount double,
            extra double,
            mta_tax double,
            tip_amount double,
            tolls_amount double,
            improvement_surcharge double,
            total_amount double,
            congestion_surcharge double,
            airport_fee double
      )
      PARTITIONED BY (day(tpep_pickup_datetime))
      LOCATION 's3://<s3-bucket-name>/nyc_taxi_iceberg/'
      TBLPROPERTIES ( 'table_type' ='ICEBERG'  );
    ```
4. Create a 2nd Apache Cceberg table for updating data
    ```
    CREATE TABLE
      iceberg_tutorial_db.nyc_taxi_iceberg_data_manipulation 
      (
            vendorid bigint,
            tpep_pickup_datetime timestamp,
            tpep_dropoff_datetime timestamp,
            passenger_count double,
            trip_distance double,
            ratecodeid double,
            store_and_fwd_flag string,
            pulocationid bigint,
            dolocationid bigint,
            payment_type bigint,
            fare_amount double,
            extra double,
            mta_tax double,
            tip_amount double,
            tolls_amount double,
            improvement_surcharge double,
            total_amount double,
            congestion_surcharge double,
            airport_fee double
      )
      PARTITIONED BY (day(tpep_pickup_datetime))
      LOCATION 's3://<s3-bucket-name>/nyc_taxi_iceberg_data_manipulation/'
      TBLPROPERTIES ( 'table_type' ='ICEBERG'  );
    ```
5. Insert from CSV table to Iceberg format
    ```
    INSERT INTO iceberg_tutorial_db.nyc_taxi_iceberg
    SELECT 
    *
    FROM "iceberg_tutorial_db"."nyc_taxi_csv" ;
    ```
6. Run a query to look at the Day partition to see how Iceberg works
    ```
    SELECT * FROM nyc_taxi_iceberg WHERE day(tpep_pickup_datetime) =  5 limit 20;
    ```
7. Insert into another Iceberg table for data manipulation
    ```
    INSERT INTO nyc_taxi_iceberg_data_manipulation
    SELECT * FROM nyc_taxi_iceberg;
    ```
8. Update some data to see the change take place 
    ```
    UPDATE nyc_taxi_iceberg_data_manipulation SET passenger_count = 4.0 WHERE vendorid = 2 AND year(tpep_pickup_datetime) =2022;
    ```
9. Select the updated data
    ```
    SELECT * FROM nyc_taxi_iceberg_data_manipulation WHERE vendorid = 2 and year(tpep_pickup_datetime) =2022 limit 10;
    ```
10. Time travel query
    ```
    SELECT * FROM nyc_taxi_iceberg_data_manipulation FOR SYSTEM_TIME AS OF TIMESTAMP '2022-11-01 22:00:00' WHERE vendorid = 2 and year(tpep_pickup_datetime)= 2022 limit 10; 
    ```
11. Delete from iceberg table
    ```
    DELETE FROM nyc_taxi_iceberg_data_manipulation WHERE year(tpep_pickup_datetime) != 2008; 
    ```

## Creators

**Johnny Chivers**

- <https://github.com/johnny-chivers/>

## Useful Links

- [youtube video](https://youtu.be/iGvj1gjbwl0) 
- [website](https://www.johnnychivers.co.uk)
- [buy me a coffee](https://www.buymeacoffee.com/johnnychivers)


Enjoy :metal:
