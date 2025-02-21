
### Data Engineering Snowflake Loading Unstructured data

## Overview

This document provides a comprehensive guide for handling raw and structured data in Snowflake using various stages and file formats. It outlines a step-by-step process for loading, parsing, transforming, and querying raw data, both from JSON and Parquet formats. It includes SQL queries for each stage of the data pipeline, which can be copied and executed directly in Snowflake.

---

### Table of Contents
1. [Loading Raw JSON Data](#loading-raw-json-data)
2. [Parsing and Analyzing Raw JSON Data](#parsing-and-analyzing-raw-json-data)
3. [Handling Nested Data](#handling-nested-data)
4. [Working with Arrays](#working-with-arrays)
5. [Dealing with Hierarchy and Aggregation](#dealing-with-hierarchy-and-aggregation)
6. [Querying Parquet Data](#querying-parquet-data)
7. [Data Transformation and Metadata](#data-transformation-and-metadata)
8. [Final Data Insert and Table Creation](#final-data-insert-and-table-creation)
9. [Conclusion](#conclusion)

---

## 1. Loading Raw JSON Data

### Step 1: Load Raw JSON from an External Stage

```sql
CREATE OR REPLACE stage MANAGE_DB.EXTERNAL_STAGES.JSONSTAGE
     url='s3://bucketsnowflake-jsondemo';
```

### Step 1.2: Define a File Format for JSON

```sql
CREATE OR REPLACE file format MANAGE_DB.FILE_FORMATS.JSONFORMAT
    TYPE = JSON;
```

### Step 2: Create a Table to Store the Raw Data, flexible load type variant work with many formats 

```sql
CREATE OR REPLACE table OUR_FIRST_DB.PUBLIC.JSON_RAW (
    raw_file variant);
```

### Step 3: Load Data into the Table

```sql
COPY INTO OUR_FIRST_DB.PUBLIC.JSON_RAW
    FROM @MANAGE_DB.EXTERNAL_STAGES.JSONSTAGE
    file_format= MANAGE_DB.FILE_FORMATS.JSONFORMAT
    files = ('HR_data.json');
```

### Step 4: View Loaded Data

```sql
SELECT * FROM OUR_FIRST_DB.PUBLIC.JSON_RAW;
```

---

## 2. Parsing and Analyzing Raw JSON Data

### Step 1: Select Specific Attributes/Columns from Raw JSON

```sql
SELECT RAW_FILE:city FROM OUR_FIRST_DB.PUBLIC.JSON_RAW;
```

### Step 2: Select Attribute by Column Number

```sql
SELECT $1:first_name FROM OUR_FIRST_DB.PUBLIC.JSON_RAW;
```

### Step 3: Format Columns to Appropriate Data Types

```sql
SELECT RAW_FILE:first_name::string as first_name  FROM OUR_FIRST_DB.PUBLIC.JSON_RAW;
SELECT RAW_FILE:id::int as id  FROM OUR_FIRST_DB.PUBLIC.JSON_RAW;
```

### Step 4: Select Multiple Attributes with filename RAW_FILE

```sql
SELECT 
    RAW_FILE:id::int as id,  
    RAW_FILE:first_name::STRING as first_name,
    RAW_FILE:last_name::STRING as last_name,
    RAW_FILE:gender::STRING as gender
FROM OUR_FIRST_DB.PUBLIC.JSON_RAW;
```

---

## 3. Handling Nested Data

### Step 1: Access Nested Attributes

```sql
SELECT RAW_FILE:job.salary::INT as salary
FROM OUR_FIRST_DB.PUBLIC.JSON_RAW;
```

### Step 2: Select Multiple Nested Attributes with filename RAW_FILE

```sql
SELECT 
    RAW_FILE:first_name::STRING as first_name,
    RAW_FILE:job.salary::INT as salary,
    RAW_FILE:job.title::STRING as title
FROM OUR_FIRST_DB.PUBLIC.JSON_RAW;
```

---

## 4. Working with Arrays

### Step 1: Access Array Data

```sql
SELECT
    RAW_FILE:prev_company[1]::STRING as prev_company
FROM OUR_FIRST_DB.PUBLIC.JSON_RAW;
```

### Step 2: Select Array Size

```sql
SELECT
    ARRAY_SIZE(RAW_FILE:prev_company) as prev_company
FROM OUR_FIRST_DB.PUBLIC.JSON_RAW;
```

### Step 3: Combine Array Data with Other Columns

```sql
SELECT 
    RAW_FILE:id::int as id,  
    RAW_FILE:first_name::STRING as first_name,
    RAW_FILE:prev_company[0]::STRING as prev_company
FROM OUR_FIRST_DB.PUBLIC.JSON_RAW
UNION ALL 
SELECT 
    RAW_FILE:id::int as id,  
    RAW_FILE:first_name::STRING as first_name,
    RAW_FILE:prev_company[1]::STRING as prev_company
FROM OUR_FIRST_DB.PUBLIC.JSON_RAW
ORDER BY id;
```

---

## 5. Dealing with Hierarchy and Aggregation

### Step 1: Query Nested Arrays and Aggregate Data

```sql
SELECT 
     array_size(RAW_FILE:spoken_languages) as spoken_languages
FROM OUR_FIRST_DB.PUBLIC.JSON_RAW;
```

## Combining firstname and index but not good because we still have  nested
```sql
SELECT 
    RAW_FILE:first_name::STRING as first_name,
    RAW_FILE:spoken_languages[0] as First_language
FROM OUR_FIRST_DB.PUBLIC.JSON_RAW;
```

## Here we fixed by accesing through index and comibining again with the first name
```sql
SELECT 
    RAW_FILE:first_name::STRING as First_name,
    RAW_FILE:spoken_languages[0].language::STRING as First_language,
    RAW_FILE:spoken_languages[0].level::STRING as Level_spoken
FROM OUR_FIRST_DB.PUBLIC.JSON_RAW;
```


## Fix approach not really good

```sql
SELECT 
    RAW_FILE:id::int as id,
    RAW_FILE:first_name::STRING as First_name,
    RAW_FILE:spoken_languages[0].language::STRING as First_language,
    RAW_FILE:spoken_languages[0].level::STRING as Level_spoken
FROM OUR_FIRST_DB.PUBLIC.JSON_RAW
UNION ALL 
SELECT 
    RAW_FILE:id::int as id,
    RAW_FILE:first_name::STRING as First_name,
    RAW_FILE:spoken_languages[1].language::STRING as First_language,
    RAW_FILE:spoken_languages[1].level::STRING as Level_spoken
FROM OUR_FIRST_DB.PUBLIC.JSON_RAW
UNION ALL 
SELECT 
    RAW_FILE:id::int as id,
    RAW_FILE:first_name::STRING as First_name,
    RAW_FILE:spoken_languages[2].language::STRING as First_language,
    RAW_FILE:spoken_languages[2].level::STRING as Level_spoken
FROM OUR_FIRST_DB.PUBLIC.JSON_RAW
ORDER BY ID;
```


## Much better option by using the funtion flatten we use an alias and acces each value no more nulls
```sql
select
      RAW_FILE:first_name::STRING as First_name,
    f.value:language::STRING as First_language,
   f.value:level::STRING as Level_spoken
from OUR_FIRST_DB.PUBLIC.JSON_RAW, table(flatten(RAW_FILE:spoken_languages)) f;
```
## Insert final data

## Option 1: CREATE TABLE AS

```sql
CREATE OR REPLACE TABLE Languages AS
select
      RAW_FILE:first_name::STRING as First_name,
    f.value:language::STRING as First_language,
   f.value:level::STRING as Level_spoken
from OUR_FIRST_DB.PUBLIC.JSON_RAW, table(flatten(RAW_FILE:spoken_languages)) f;
```

```sql
SELECT * FROM Languages;
```
```sql
truncate table languages;
```

## Option 2: INSERT INTO

```sql
INSERT INTO Languages
select
      RAW_FILE:first_name::STRING as First_name,
    f.value:language::STRING as First_language,
   f.value:level::STRING as Level_spoken
from OUR_FIRST_DB.PUBLIC.JSON_RAW, table(flatten(RAW_FILE:spoken_languages)) f;
```
```sql
SELECT * FROM Languages;
```

## Querying parqet data

## Create file format and stage object

```sql    
CREATE OR REPLACE FILE FORMAT MANAGE_DB.FILE_FORMATS.PARQUET_FORMAT
    TYPE = 'parquet';
```    

```sql
CREATE OR REPLACE STAGE MANAGE_DB.EXTERNAL_STAGES.PARQUETSTAGE
    url = 's3://snowflakeparquetdemo'   
    FILE_FORMAT = MANAGE_DB.FILE_FORMATS.PARQUET_FORMAT;
```    
    
## Preview the data

```sql    
LIST  @MANAGE_DB.EXTERNAL_STAGES.PARQUETSTAGE; 
```
```sql      
SELECT * FROM @MANAGE_DB.EXTERNAL_STAGES.PARQUETSTAGE;
```
    


## File format in Queries

```sql
CREATE OR REPLACE STAGE MANAGE_DB.EXTERNAL_STAGES.PARQUETSTAGE
    url = 's3://snowflakeparquetdemo'  ;
```    
```sql    
SELECT * 
FROM @MANAGE_DB.EXTERNAL_STAGES.PARQUETSTAGE
(file_format => 'MANAGE_DB.FILE_FORMATS.PARQUET_FORMAT');
```

## Quotes can be omitted in case of the current namespace
```sql
USE MANAGE_DB.FILE_FORMATS;
```
```sql
SELECT * 
FROM @MANAGE_DB.EXTERNAL_STAGES.PARQUETSTAGE
(file_format => MANAGE_DB.FILE_FORMATS.PARQUET_FORMAT);
```

```sql
CREATE OR REPLACE STAGE MANAGE_DB.EXTERNAL_STAGES.PARQUETSTAGE
    url = 's3://snowflakeparquetdemo'   
    FILE_FORMAT = MANAGE_DB.FILE_FORMATS.PARQUET_FORMAT;
```    

## Syntax for Querying unstructured data with soem issues
```sql
SELECT 
$1:__index_level_0__,
$1:cat_id,
$1:date,
$1:"__index_level_0__",
$1:"cat_id",
$1:"d",
$1:"date",
$1:"dept_id",
$1:"id",
$1:"item_id",
$1:"state_id",
$1:"store_id",
$1:"value"
FROM @MANAGE_DB.EXTERNAL_STAGES.PARQUETSTAGE;
```

## Understanding Date conversion START IN JANUEARY 1970

```sql    
SELECT 1;


SELECT DATE(1);
SELECT DATE(60*60*24);

SELECT DATE(365*60*60*24);
SELECT DATE(365*60*60*24) AS DATE_COLUMN;
```

## Querying with conversions and aliases CONVERTING VALUE INTO THEIR PROPER FORMATS
## Assuming date still in a string first we need to convert it to integer

```sql    
SELECT 
$1:__index_level_0__::int as index_level,
$1:cat_id::VARCHAR(50) as category,
DATE($1:date::int ) as Date,
$1:"dept_id"::VARCHAR(50) as Dept_ID,
$1:"id"::VARCHAR(50) as ID,
$1:"item_id"::VARCHAR(50) as Item_ID,
$1:"state_id"::VARCHAR(50) as State_ID,
$1:"store_id"::VARCHAR(50) as Store_ID,
$1:"value"::int as value
FROM @MANAGE_DB.EXTERNAL_STAGES.PARQUETSTAGE;
```

## Adding metadata filename, row number and also time stamp because is useful toknow ehen this was created
```sql    
SELECT 
$1:__index_level_0__::int as index_level,
$1:cat_id::VARCHAR(50) as category,
DATE($1:date::int ) as Date,
$1:"dept_id"::VARCHAR(50) as Dept_ID,
$1:"id"::VARCHAR(50) as ID,
$1:"item_id"::VARCHAR(50) as Item_ID,
$1:"state_id"::VARCHAR(50) as State_ID,
$1:"store_id"::VARCHAR(50) as Store_ID,
$1:"value"::int as value,
METADATA$FILENAME as FILENAME,
METADATA$FILE_ROW_NUMBER as ROWNUMBER,
TO_TIMESTAMP_NTZ(current_timestamp) as LOAD_DATE
FROM @MANAGE_DB.EXTERNAL_STAGES.PARQUETSTAGE;
```

## Understanding timezone SQL funtion be mindfull in the time zones
```sql
SELECT (current_timestamp);
SELECT TO_TIMESTAMP_NTZ(current_timestamp);
ALTER SESSION SET TIMEZONE = 'Europe/Madrid';
ALTER SESSION SET TIMEZONE = 'UTC';
```

## Create destination table
```sql
CREATE OR REPLACE TABLE OUR_FIRST_DB.PUBLIC.PARQUET_DATA (
    ROW_NUMBER int,
    index_level int,
    cat_id VARCHAR(50),
    date date,
    dept_id VARCHAR(50),
    id VARCHAR(50),
    item_id VARCHAR(50),
    state_id VARCHAR(50),
    store_id VARCHAR(50),
    value int,
    Load_date timestamp default TO_TIMESTAMP_NTZ(current_timestamp));
```    

## Load the parquet data

```sql   
COPY INTO OUR_FIRST_DB.PUBLIC.PARQUET_DATA
    FROM (SELECT 
            METADATA$FILE_ROW_NUMBER,
            $1:__index_level_0__::int,
            $1:cat_id::VARCHAR(50),
            DATE($1:date::int ),
            $1:"dept_id"::VARCHAR(50),
            $1:"id"::VARCHAR(50),
            $1:"item_id"::VARCHAR(50),
            $1:"state_id"::VARCHAR(50),
            $1:"store_id"::VARCHAR(50),
            $1:"value"::int,
            TO_TIMESTAMP_NTZ(current_timestamp)
        FROM @MANAGE_DB.EXTERNAL_STAGES.PARQUETSTAGE);
```        
        
```sql    
SELECT * FROM OUR_FIRST_DB.PUBLIC.PARQUET_DATA;
```
---

## Conclusion

This document outlines the essential steps and SQL commands for efficiently managing raw and structured data within Snowflake. By following the steps provided, users can load raw JSON and Parquet data, parse and analyze it, handle nested data and arrays, aggregate hierarchical data, and query efficiently. Additionally, the metadata and transformation steps ensure that the data can be loaded into final destination tables for reporting or further analysis.

---
```

This `README.md` is structured to guide users through a data pipeline process for working with JSON and Parquet data in Snowflake, including SQL code for each step. It is designed for easy copy-paste execution without skipping any necessary details.