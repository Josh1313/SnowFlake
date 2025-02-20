# Snowflake Continuation

## Introduction
This is the second part of the Snowflake guide, focusing on advanced COPY options for data loading. It provides detailed examples and explanations for various scenarios, including error handling, data validation, size limits, and file truncation. This document aims to enhance your understanding of Snowflake's COPY INTO command by exploring multiple use cases and configurations, making it easier to efficiently load and manage data in Snowflake.

---

## Overview
This document provides an overview of managing COPY options in Snowflake, including advanced techniques for validating, handling errors, and controlling data load behaviors. All SQL commands are presented in a ready-to-use format, allowing users to easily copy and paste them into their Snowflake environment for immediate execution.

---

## Copy Options

### VALIDATION_MODE
This section demonstrates how to use the VALIDATION_MODE option to test data loading without actually inserting the data into the target table. It helps ensure that the data meets all requirements before the actual load.

-- Prepare database & table
```sql
CREATE OR REPLACE DATABASE COPY_DB;
```
```sql
CREATE OR REPLACE TABLE  COPY_DB.PUBLIC.ORDERS (
    ORDER_ID VARCHAR(30),
    AMOUNT VARCHAR(30),
    PROFIT INT,
    QUANTITY INT,
    CATEGORY VARCHAR(30),
    SUBCATEGORY VARCHAR(30));
```    

-- Prepare stage object
```sql
CREATE OR REPLACE STAGE COPY_DB.PUBLIC.aws_stage_copy
    URL='s3://snowflakebucket-copyoption/size/';
```  

```sql  
LIST @COPY_DB.PUBLIC.aws_stage_copy;
```

-- Validate data without loading using VALIDATION_MODE = RETURN_ERRORS

```sql
COPY INTO COPY_DB.PUBLIC.ORDERS
    FROM @aws_stage_copy
    FILE_FORMAT= (TYPE = CSV FIELD_DELIMITER=',' SKIP_HEADER=1)
    PATTERN='.*Order.*'
    VALIDATION_MODE = RETURN_ERRORS;
```
```sql
SELECT * FROM ORDERS;  
```
--Load data using copy command VALIDATION_MODE  RETURN_5_ROWS 
```sql
COPY INTO COPY_DB.PUBLIC.ORDERS
    FROM @aws_stage_copy
    file_format= (type = csv field_delimiter=',' skip_header=1)
    pattern='.*Order.*'
   VALIDATION_MODE = RETURN_5_ROWS ; 
```

```sql
SELECT * FROM ORDERS;  
```

### Use files with errors 
This option allows you to return only the failed rows, which is useful for debugging data issues. Combining this with the ON_ERROR option helps you focus on problematic data.

--Create a stage
```sql
create or replace stage copy_db.public.aws_stage_copy
    url ='s3://snowflakebucket-copyoption/returnfailed/'; 
```
-- LISTING FILES
```sql
list @copy_db.public.aws_stage_copy; 
```

-- show all errors --
```sql
copy into copy_db.public.orders
    from @copy_db.public.aws_stage_copy
    file_format = (type=csv field_delimiter=',' skip_header=1)
    pattern='.*Order.*'
    validation_mode=return_errors;
```

-- validate first n rows --
```sql
copy into copy_db.public.orders
    from @copy_db.public.aws_stage_copy
    file_format = (type=csv field_delimiter=',' skip_header=1)
    pattern='.*error.*'
    validation_mode=return_1_rows;
```
-- Test do it yourself
```sql
CREATE OR REPLACE TABLE  EXERCISE_DB.PUBLIC.EMPLOYEES (
    customer_id INT,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    email VARCHAR(50),
    age INT,
    deparment VARCHAR(50));
```
--- Prepare stage object

```sql
CREATE OR REPLACE STAGE EXERCISE_DB.PUBLIC.aws_stage_copy
    url='s3://snowflake-assignments-mc/copyoptions/example1';
```

```sql
 list @EXERCISE_DB.public.aws_stage_copy;   
```

---Creating file format object

```sql
CREATE OR REPLACE FILE FORMAT EXERCISE_DB.FILE_FORMATS.MY_FILE_FORMAT
    TYPE = 'CSV'
    FIELD_DELIMITER = ','
    SKIP_HEADER = 1;   
```     

--Validation data using copy command  VALIDATION_MODE  RETURN_ERRORS will be none so is ready to load
```sql
COPY INTO EXERCISE_DB.PUBLIC.EMPLOYEES
    FROM  @EXERCISE_DB.PUBLIC.aws_stage_copy
    FILE_FORMAT = (FORMAT_NAME = EXERCISE_DB.FILE_FORMATS.MY_FILE_FORMAT)
    pattern='.*employees.*'
    VALIDATION_MODE = RETURN_ERRORS;
```    

--copy data regardells of the errors command  ON_ERROR  CONTINUE
```sql
COPY INTO EXERCISE_DB.PUBLIC.EMPLOYEES
    FROM  @EXERCISE_DB.PUBLIC.aws_stage_copy
    FILE_FORMAT = (FORMAT_NAME = EXERCISE_DB.FILE_FORMATS.MY_FILE_FORMAT)
    pattern='.*employees.*'
    ON_ERROR = 'CONTINUE';
```    
    
    
```sql    
SELECT* FROM EXERCISE_DB.PUBLIC.EMPLOYEES 
```

-- more validation methos more advance
   
---- Use files with errors ----
```sql 
CREATE OR REPLACE STAGE COPY_DB.PUBLIC.aws_stage_copy
    url='s3://snowflakebucket-copyoption/returnfailed/';
```

```sql 
LIST @COPY_DB.PUBLIC.aws_stage_copy;  
```  


```sql 
COPY INTO COPY_DB.PUBLIC.ORDERS
    FROM @aws_stage_copy
    file_format= (type = csv field_delimiter=',' skip_header=1)
    pattern='.*Order.*'
    VALIDATION_MODE = RETURN_ERRORS;
```    


```sql 
COPY INTO COPY_DB.PUBLIC.ORDERS
    FROM @aws_stage_copy
    file_format= (type = csv field_delimiter=',' skip_header=1)
    pattern='.*Order.*'
    VALIDATION_MODE = RETURN_1_rows;
```    
    



-------------- Working with error results -----------

---- 1) Saving rejected files after VALIDATION_MODE ---- 

```sql 
CREATE OR REPLACE TABLE  COPY_DB.PUBLIC.ORDERS (
    ORDER_ID VARCHAR(30),
    AMOUNT VARCHAR(30),
    PROFIT INT,
    QUANTITY INT,
    CATEGORY VARCHAR(30),
    SUBCATEGORY VARCHAR(30));
```    

```sql 
COPY INTO COPY_DB.PUBLIC.ORDERS
    FROM @aws_stage_copy
    file_format= (type = csv field_delimiter=',' skip_header=1)
    pattern='.*Order.*'
    VALIDATION_MODE = RETURN_ERRORS;
```
// WE COPY THE QUERY IF FROM THE GUI    

  --  '01ba8495-030c-6f01-000b-a75f0001c006'

-- Storing rejected /failed results in a table

```sql 
CREATE OR REPLACE TABLE rejected AS 
select rejected_record from table(result_scan(last_query_id()));
```
-- or we can use the query id number

```sql 
select rejected_record from table(result_scan('01ba8495-030c-6f01-000b-a75f0001c006'));
```




-- Adding additional records --

```sql 
INSERT INTO rejected
select rejected_record from table(result_scan(last_query_id()));
```
```sql 
SELECT * FROM rejected;
```

---- 2) Saving rejected files without VALIDATION_MODE ---- 

```sql 
COPY INTO COPY_DB.PUBLIC.ORDERS
    FROM @aws_stage_copy
    file_format= (type = csv field_delimiter=',' skip_header=1)
    pattern='.*Order.*'
    ON_ERROR=CONTINUE;
```    
  
```sql   
select * from table(validate(orders, job_id => '_last'));
```


---- 3) Working with rejected records ---- 


```sql 
SELECT REJECTED_RECORD FROM rejected;
```

```sql 
CREATE OR REPLACE TABLE rejected_values as
SELECT 
SPLIT_PART(rejected_record,',',1) as ORDER_ID, 
SPLIT_PART(rejected_record,',',2) as AMOUNT, 
SPLIT_PART(rejected_record,',',3) as PROFIT, 
SPLIT_PART(rejected_record,',',4) as QUATNTITY, 
SPLIT_PART(rejected_record,',',5) as CATEGORY, 
SPLIT_PART(rejected_record,',',6) as SUBCATEGORY
FROM rejected; 
```

```sql 
SELECT * FROM rejected_values;
```

-- Size limit in bites the first  file always will uploads regardless of the number of files



---- SIZE_LIMIT ----

-- Prepare database & table

```sql 
CREATE OR REPLACE DATABASE COPY_DB;
```

```sql 
CREATE OR REPLACE TABLE  COPY_DB.PUBLIC.ORDERS (
    ORDER_ID VARCHAR(30),
    AMOUNT VARCHAR(30),
    PROFIT INT,
    QUANTITY INT,
    CATEGORY VARCHAR(30),
    SUBCATEGORY VARCHAR(30));
```    
    
-- Prepare stage object
```sql 
CREATE OR REPLACE STAGE COPY_DB.PUBLIC.aws_stage_copy
    url='s3://snowflakebucket-copyoption/size/';
```    
      
-- List files in stage
```sql 
LIST @aws_stage_copy;
```


-- Load data using copy command if the size exceedes the first file will upload only the first one and viceversa
```sql 
COPY INTO COPY_DB.PUBLIC.ORDERS
    FROM @aws_stage_copy
    file_format= (type = csv field_delimiter=',' skip_header=1)
    pattern='.*Order.*'
    SIZE_LIMIT=60000;// change 20000 size
```    

---- RETURN_FAILED_ONLY ----
```sql 
CREATE OR REPLACE TABLE  COPY_DB.PUBLIC.ORDERS (
    ORDER_ID VARCHAR(30),
    AMOUNT VARCHAR(30),
    PROFIT INT,
    QUANTITY INT,
    CATEGORY VARCHAR(30),
    SUBCATEGORY VARCHAR(30));
```    

-- Prepare stage object
```sql 
CREATE OR REPLACE STAGE COPY_DB.PUBLIC.aws_stage_copy
    url='s3://snowflakebucket-copyoption/returnfailed/';
```    

```sql   
LIST @COPY_DB.PUBLIC.aws_stage_copy;
```
  
    
--- Load data using copy command this is just a test when is not make any sense to use by itself
```sql 
COPY INTO COPY_DB.PUBLIC.ORDERS
    FROM @aws_stage_copy
    file_format= (type = csv field_delimiter=',' skip_header=1)
    pattern='.*Order.*'
    RETURN_FAILED_ONLY = TRUE;
```sql     
    
    
-- now here is good practice where we use the combination with continue    and we can focuse on the files that have errors
```sql 
COPY INTO COPY_DB.PUBLIC.ORDERS
    FROM @aws_stage_copy
    file_format= (type = csv field_delimiter=',' skip_header=1)
    pattern='.*Order.*'
    ON_ERROR =CONTINUE
    RETURN_FAILED_ONLY = TRUE;
```sql     

-- Default = FALSE

```sql 
CREATE OR REPLACE TABLE  COPY_DB.PUBLIC.ORDERS (
    ORDER_ID VARCHAR(30),
    AMOUNT VARCHAR(30),
    PROFIT INT,
    QUANTITY INT,
    CATEGORY VARCHAR(30),
    SUBCATEGORY VARCHAR(30));
```


--  This will gives us all files because we need to specified returnfailed to true because by default comes false
```sql 
COPY INTO COPY_DB.PUBLIC.ORDERS
    FROM @aws_stage_copy
    file_format= (type = csv field_delimiter=',' skip_header=1)
    pattern='.*Order.*'
    ON_ERROR =CONTINUE;
```    

-- Truncate columns

    ---- TRUNCATECOLUMNS ----
    // default = False
    // TRUE = string are automatically truncated to the target column lenght 
    // False = COPY produces an error if aloaded string exceeds the target column lenght


```sql 
CREATE OR REPLACE TABLE  COPY_DB.PUBLIC.ORDERS (
    ORDER_ID VARCHAR(30),
    AMOUNT VARCHAR(30),
    PROFIT INT,
    QUANTITY INT,
    CATEGORY VARCHAR(10),//cchanges the caracthers
    SUBCATEGORY VARCHAR(30));
```    


-- Prepare stage object
```sql 
CREATE OR REPLACE STAGE COPY_DB.PUBLIC.aws_stage_copy
    url='s3://snowflakebucket-copyoption/size/';
```    
```sql   
LIST @COPY_DB.PUBLIC.aws_stage_copy;
```
  
    
-- Load data using copy command this will give you an error

```sql 
COPY INTO COPY_DB.PUBLIC.ORDERS
    FROM @aws_stage_copy
    file_format= (type = csv field_delimiter=',' skip_header=1)
    pattern='.*Order.*';
```    

```sql 
COPY INTO COPY_DB.PUBLIC.ORDERS
    FROM @aws_stage_copy
    file_format= (type = csv field_delimiter=',' skip_header=1)
    pattern='.*Order.*'
    TRUNCATECOLUMNS = true; 
```    
    
```sql     
SELECT * FROM ORDERS;  
```

### FORCE
By default, Snowflake prevents duplicate data loads. The FORCE option allows you to override this behavior, which is useful for testing or reloading unchanged files.

// Force


---- FORCE ----
// Note that this option reload files , potentially duplicating data in a table 
// that is way by default comes false but we change that in case need it

```sql 
CREATE OR REPLACE TABLE  COPY_DB.PUBLIC.ORDERS (
    ORDER_ID VARCHAR(30),
    AMOUNT VARCHAR(30),
    PROFIT INT,
    QUANTITY INT,
    CATEGORY VARCHAR(30),
    SUBCATEGORY VARCHAR(30));
```    

```sql 
// Prepare stage object
CREATE OR REPLACE STAGE COPY_DB.PUBLIC.aws_stage_copy
    url='s3://snowflakebucket-copyoption/size/';
```    
```sql   
LIST @COPY_DB.PUBLIC.aws_stage_copy;
```
  
    
-- Load data using copy command
```sql 
COPY INTO COPY_DB.PUBLIC.ORDERS
    FROM @aws_stage_copy
    file_format= (type = csv field_delimiter=',' skip_header=1)
    pattern='.*Order.*';
```    

-- Not possible to load file that have been loaded and data has not been modified
```sql 
COPY INTO COPY_DB.PUBLIC.ORDERS
    FROM @aws_stage_copy
    file_format= (type = csv field_delimiter=',' skip_header=1)
    pattern='.*Order.*';
```    
   
```sql 
SELECT * FROM ORDERS; 
```   


-- Using the FORCE option

```sql 
COPY INTO COPY_DB.PUBLIC.ORDERS
    FROM @aws_stage_copy
    file_format= (type = csv field_delimiter=',' skip_header=1)
    pattern='.*Order.*'
    FORCE = TRUE;
```    
-- load history

    
-- Query load history within a database --
```sql
USE COPY_DB;
```
```sql 
SELECT * FROM information_schema.load_history;
```





-- Query load history gloabally from SNOWFLAKE database --

```sql 
SELECT * FROM snowflake.account_usage.load_history;
```


-- Filter on specific table & schema
```sql 
SELECT * FROM snowflake.account_usage.load_history
  where schema_name='PUBLIC' and
  table_name='ORDERS';
```
  
  
-- Filter on specific table & schema

```sql 
SELECT * FROM snowflake.account_usage.load_history
  where schema_name='PUBLIC' and
  table_name='ORDERS' and
  error_count > 0;
 ``` 
  
  
-- Filter on specific table & schema
```sql 
SELECT * FROM snowflake.account_usage.load_history
WHERE DATE(LAST_LOAD_TIME) <= DATEADD(days,-1,CURRENT_DATE);
```


## Test do it yourself

```sql 
CREATE OR REPLACE TABLE  EXERCISE_DB.PUBLIC.EMPLOYEES (
    customer_id INT,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    email VARCHAR(50),
    age INT,
    deparment VARCHAR(50));
```    

-- Prepare stage object

```sql 
CREATE OR REPLACE STAGE EXERCISE_DB.PUBLIC.aws_stage_copy
    url='s3://snowflake-assignments-mc/copyoptions/example2';
```    
```sql 
 list @EXERCISE_DB.public.aws_stage_copy;  
```  

-- Creating file format object
```sql 
CREATE OR REPLACE FILE FORMAT EXERCISE_DB.FILE_FORMATS.MY_FILE_FORMAT
    TYPE = 'CSV'
    FIELD_DELIMITER = ','
    SKIP_HEADER = 1;    
```

-- Validation data using copy command  VALIDATION_MODE  RETURN_ERRORS will be none so is ready to load
```sql 
COPY INTO EXERCISE_DB.PUBLIC.EMPLOYEES
    FROM  @EXERCISE_DB.PUBLIC.aws_stage_copy
    FILE_FORMAT = (FORMAT_NAME = EXERCISE_DB.FILE_FORMATS.MY_FILE_FORMAT)
    pattern='.*employees.*'
    VALIDATION_MODE = RETURN_ERRORS;
```    

-- Copy data regardells of the errors command  TRUNCATE COLUMNS TRUE
```sql 
COPY INTO EXERCISE_DB.PUBLIC.EMPLOYEES
    FROM  @EXERCISE_DB.PUBLIC.aws_stage_copy
    FILE_FORMAT = (FORMAT_NAME = EXERCISE_DB.FILE_FORMATS.MY_FILE_FORMAT)
    pattern='.*employees.*'
    TRUNCATECOLUMNS = true
    ON_ERROR = 'CONTINUE';
```    
    
```sql    
SELECT* FROM EXERCISE_DB.PUBLIC.EMPLOYEES 
```


## Conclusion
This document explores advanced COPY options in Snowflake, enabling more control and flexibility in data loading. By understanding and using these options effectively, users can optimize data workflows, handle errors gracefully, and maintain data integrity. Copy and execute the SQL commands as needed to enhance your Snowflake data management strategy. Happy querying!

