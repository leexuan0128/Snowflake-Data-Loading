# Snowflake Data Loading
Load Kaggle Crime Report Dataset into Snowflake through Snowsql CLI

The basic steps to load local data into snowflake:
1. Check the input source type.
2. Create & Run target table SQL Command.
3. OPEN CLI / Web interface to load data.
4. Select database, schema and table
5. Create File Format
6. Create Stage
7. Put file into Stage
8. Copy data into table

After we have the source input, let us check the data first:
![screenshot](https://tva1.sinaimg.cn/large/e6c9d24egy1h0hbwx57zej21a40u0nd0.jpg)
We should notice that there is a redundant header in the first line we will not use it. Also, the data format needs to be processed carefully, as there is no second suffix. The comma in a inner sentence should be isolated, it is not a delimiter anymore. All these works need to be done before we load data to table of database.

The next step is to create the target table to Snowflake. We need to choose which one database and schema we want to upload. The constraints can be roughly used:
```SQL
CREATE OR REPLACE TABLE CRIME_LITE(
    INCIDENT_NUMBER VARCHAR(15) NOT NULL,
    OFFENSE_CODE VARCHAR(10) NOT NULL,
    OFFENSE_CODE_GROUP VARCHAR(255) NOT NULL,
    OFFENSE_DESCRIPTION VARCHAR(255),
    DISTRICT VARCHAR(10),
    SHOOTING VARCHAR(2),
    OCCURRED_ON_DATE TIMESTAMP_NTZ(9) NOT NULL,
    YEAR NUMBER(38,0) NOT NULL,
    MONTH NUMBER(38,0) NOT NULL,
    DAY_OF_WEEK VARCHAR(20) NOT NULL,
    HOUR NUMBER(38,0) NOT NULL,
    UCR_PART VARCHAR(10),
    STREET VARCHAR(255)
);
```
The empty table has been created in the Snowflake:
![empty_table](https://tva1.sinaimg.cn/large/e6c9d24egy1h0hc797j43j21we05y76j.jpg)

We also need to create a File Format for Snowflake to recognize the data:
```SQL
CREATE OR REPLACE FILE FORMAT "DE_DEMO".PUBLIC.CRIME_DEMO
    TYPE = 'CSV'
    COMPRESSION = 'GZIP'
    FIELD_DELIMITER = ','
    RECORD_DELIMITER = '\n'
    SKIP_HEADER = 1
    FIELD_OPTIONALLY_ENCLOSED_BY = '\042'
    TRIM_SPACE = TRUE
    ERROR_ON_COLUMN_COUNT_MISMATCH = TRUE
    ESCAPE = 'NONE'
    ESCAPE_UNENCLOSED_FIELD = '\134'
    DATE_FORMAT = 'AUTO'
    TIMESTAMP_FORMAT = 'dd/mm/yy HH24:MI'
    NULL_IF = ('NULL');
```
After we create the file format in the command line, we can see the output in the web end.
![file_format](https://tva1.sinaimg.cn/large/e6c9d24egy1h0hcp1hbr2j21280f80uh.jpg)

The next step is to create the Stage, as snowflake will put all temporary files into Stage.
```SQL
CREATE OR REPLACE STAGE "DE_DEMO"."PUBLIC"."MY_CSV_STAGE"
    FILE_FORMAT = CRIME_DEMO;
```
![stage](https://tva1.sinaimg.cn/large/e6c9d24egy1h0hcx1dgy7j20zy07m0ty.jpg)

Then we put files into Stage:
```SQL
put file:///Users/xuan/Downloads/crime2.csv @MY_CSV_STAGE auto_compress = TRUE;
```
The local file crime.csv has been uploaded to cloud Stage:
![put_files](https://tva1.sinaimg.cn/large/e6c9d24egy1h0hd4x6s71j21bk06ogng.jpg)

The final step is to copy all data of Crime.csv from Stage to Table:
```SQL
COPY INTO CRIME_LITE
    FROM @MY_CSV_STAGE/crime2.csv.gz
    FILE_FORMAT = CRIME_DEMO
    ON_ERROR = 'skip_file';
```
![](https://tva1.sinaimg.cn/large/e6c9d24egy1h0hdgq7mghj21xk09eacs.jpg)

Let's have a test:
```SQL
SELECT * FROM CRIME_LITE LIMIT 5;
```
![](https://tva1.sinaimg.cn/large/e6c9d24egy1h0hdlqry8uj22by0i843f.jpg)

That's All. We managed to load the data from the local to Snowflake.