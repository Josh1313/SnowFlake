### Snowflake Database Management and Data Loading

### Introduction

This document provides a comprehensive guide on managing external stages, creating tables, and loading data into Snowflake from an AWS S3 bucket. It includes step-by-step SQL commands for seamless execution. Additionally, this guide covers file format objects, which are essential for defining the structure of the data being imported, such as CSV or JSON formats. By leveraging Snowflake's powerful data loading and transformation capabilities, you can efficiently manage large datasets.

### Overview

This document provides an overview of managing external stages, creating tables, and loading data into Snowflake from an AWS S3 bucket. All SQL commands are included for easy execution.
---

## Database and Schema Setup

```sql
-- Create a database to manage stages, file formats, etc.
CREATE OR REPLACE DATABASE MANAGE_DB;

-- Create a schema for external stages
CREATE OR REPLACE SCHEMA external_stages;
```

---

## Creating and Managing External Stages

### Create an External Stage (AWS S3 Connection)
An external stage is a connection to an AWS S3 bucket where files are stored in the below example with credentials.

```sql
CREATE OR REPLACE STAGE MANAGE_DB.external_stages.aws_stage
    URL='s3://bucketsnowflakes3'
    CREDENTIALS=(AWS_KEY_ID='ABCD_DUMMY_ID' AWS_SECRET_KEY='1234abcd_key');
```

### Describe the External Stage

```sql
DESC STAGE MANAGE_DB.external_stages.aws_stage;
```

### Altering External Stage Credentials
```sql
ALTER STAGE MANAGE_DB.external_stages.aws_stage
    SET CREDENTIALS=(AWS_KEY_ID='XYZ_DUMMY_ID' AWS_SECRET_KEY='987xyz');
```

### Public External Stage (No Credentials Needed)
```sql
CREATE OR REPLACE STAGE MANAGE_DB.external_stages.aws_stage
    URL='s3://bucketsnowflakes3';
```

### List Files in the Stage 
Meaning in the s3 bucket and also be mindfull to use @ before your stage-name @aws_stage

```sql
LIST @aws_stage;
```

---

## Creating Orders Tables
Remenber at this stage we know the schema 

```sql
CREATE OR REPLACE TABLE OUR_FIRST_DB.PUBLIC.ORDERS (
    ORDER_ID VARCHAR(30),
    AMOUNT INT,
    PROFIT INT,
    QUANTITY INT,
    CATEGORY VARCHAR(30),
    SUBCATEGORY VARCHAR(30)
);
```

### Verify Table Creation
```sql
SELECT * FROM OUR_FIRST_DB.PUBLIC.ORDERS;
```

---

## Copying Data into Snowflake Tables

### Basic Copy Command
Without declaring the database assuming you do so at the beggining of the workflow

```sql
COPY INTO OUR_FIRST_DB.PUBLIC.ORDERS
    FROM @aws_stage
    FILE_FORMAT = (TYPE = CSV FIELD_DELIMITER=',' SKIP_HEADER=1);
```

### Copy Command with Fully Qualified Stage Object
Declaring the database
```sql
COPY INTO OUR_FIRST_DB.PUBLIC.ORDERS
    FROM @MANAGE_DB.external_stages.aws_stage
    FILE_FORMAT = (TYPE = CSV FIELD_DELIMITER=',' SKIP_HEADER=1);
```

### List Files in S3 Bucket
```sql
LIST @MANAGE_DB.external_stages.aws_stage;
```

### Copy Specific File
```sql
COPY INTO OUR_FIRST_DB.PUBLIC.ORDERS
    FROM @MANAGE_DB.external_stages.aws_stage
    FILE_FORMAT = (TYPE = CSV FIELD_DELIMITER=',' SKIP_HEADER=1)
    FILES = ('OrderDetails.csv');
```

### Copy Using a Pattern (Wildcard)
```sql
COPY INTO OUR_FIRST_DB.PUBLIC.ORDERS
    FROM @MANAGE_DB.external_stages.aws_stage
    FILE_FORMAT = (TYPE = CSV FIELD_DELIMITER=',' SKIP_HEADER=1)
    PATTERN = '.*Order.*';
```
---

### Test  Do it yourself

-- Create stage object, first step here database , public and the name that we assign the stage

```sql
CREATE OR REPLACE STAGE EXERCISE_DB.public.aws_stage
    url='s3://snowflake-assignments-mc/loadingdata';
```
-- List files in stage
```sql
LIST @EXERCISE_DB.public.aws_stage;
```

```sql
LIST @aws_stage;
```

--Describe the stage 

```sql
DESC STAGE EXERCISE_DB.public.aws_stage; 
```
### Create a table in the same database to copy everything from aws, we need to know the schema

```sql
CREATE OR REPLACE TABLE EXERCISE_DB.PUBLIC.CUSTOMER (
    ID VARCHAR,
    first_name VARCHAR,
    last_name VARCHAR,
    email VARCHAR,
    age INT,
    city VARCHAR)
```    
-- Verify Table Creation
```sql 
SELECT * FROM EXERCISE_DB.PUBLIC.CUSTOMER;
``` 
-- Copy the data 
```sql
COPY INTO EXERCISE_DB.PUBLIC.CUSTOMER
    FROM @aws_stage
    file_format= (type = csv field_delimiter=';' skip_header=1)
```
-- Check how many rows have been loaded
```sql
SELECT COUNT(*) FROM EXERCISE_DB.PUBLIC.CUSTOMER;
```
--drop unwanted stages in case errors
```sql
DROP STAGE IF EXISTS EXERCISE_DB.PUBLIC.EXERCISE_DB;
```

---

## Data Transformation Using SQL Functions

### Create a New Table with a Subset of Columns
Example 1
```sql
CREATE OR REPLACE TABLE OUR_FIRST_DB.PUBLIC.ORDERS_EX (
    ORDER_ID VARCHAR(30),
    AMOUNT INT
);
```

### Copy with a SQL Transformation
```sql
COPY INTO OUR_FIRST_DB.PUBLIC.ORDERS_EX
    FROM (
        SELECT s.$1, s.$2 FROM @MANAGE_DB.external_stages.aws_stage s
    )
    FILE_FORMAT = (TYPE = CSV FIELD_DELIMITER=',' SKIP_HEADER=1)
    FILES = ('OrderDetails.csv');
```

```sql
SELECT * FROM OUR_FIRST_DB.PUBLIC.ORDERS_EX;
```

### Create a Table with Profitability Flag
Example 2
Reset Table

```sql
CREATE OR REPLACE TABLE OUR_FIRST_DB.PUBLIC.ORDERS_EX (
    ORDER_ID VARCHAR(30),
    AMOUNT INT,
    PROFIT INT,
    PROFITABLE_FLAG VARCHAR(30)
);
```

### Copy with Profitability Calculation
```sql
COPY INTO OUR_FIRST_DB.PUBLIC.ORDERS_EX
    FROM (
        SELECT s.$1, s.$2, s.$3,
        CASE WHEN CAST(s.$3 AS INT) < 0 THEN 'Not Profitable' ELSE 'Profitable' END
        FROM @MANAGE_DB.external_stages.aws_stage s
    )
    FILE_FORMAT = (TYPE = CSV FIELD_DELIMITER=',' SKIP_HEADER=1)
    FILES = ('OrderDetails.csv');
```
```sql
SELECT * FROM OUR_FIRST_DB.PUBLIC.ORDERS_EX;
```

### Extracting a Substring from a Column
Example 3
Reset Table
```sql
CREATE OR REPLACE TABLE OUR_FIRST_DB.PUBLIC.ORDERS_EX (
    ORDER_ID VARCHAR(30),
    AMOUNT INT,
    PROFIT INT,
    CATEGORY_SUBSTRING VARCHAR(5)
  
    );
```
### Copy with Substring from a Column

```sql
COPY INTO OUR_FIRST_DB.PUBLIC.ORDERS_EX
    FROM (
        SELECT s.$1, s.$2, s.$3, SUBSTRING(s.$5, 1, 5)
        FROM @MANAGE_DB.external_stages.aws_stage s
    )
    FILE_FORMAT = (TYPE = CSV FIELD_DELIMITER=',' SKIP_HEADER=1)
    FILES = ('OrderDetails.csv');
```
```sql
SELECT * FROM OUR_FIRST_DB.PUBLIC.ORDERS_EX;
```

---

## Additional Table Transformations

### Reset Table with Subset of columns Only the ones that we Needed
Example 4 
```sql
CREATE OR REPLACE TABLE OUR_FIRST_DB.PUBLIC.ORDERS_EX (
    ORDER_ID VARCHAR(30),
    AMOUNT INT,
    PROFIT INT,
    PROFITABLE_FLAG VARCHAR(30)
  
    );
```
### Copy a subset of columns
```sql
COPY INTO OUR_FIRST_DB.PUBLIC.ORDERS_EX (ORDER_ID,PROFIT)
    FROM (select 
            s.$1,
            s.$3
          from @MANAGE_DB.external_stages.aws_stage s)
    file_format= (type = csv field_delimiter=',' skip_header=1)
    files=('OrderDetails.csv'); 
```
```sql
SELECT * FROM OUR_FIRST_DB.PUBLIC.ORDERS_EX;
```

### Reset Table with Auto-Increment ID
Example 5
```sql
CREATE OR REPLACE TABLE OUR_FIRST_DB.PUBLIC.ORDERS_EX (
    ORDER_ID NUMBER AUTOINCREMENT START 1 INCREMENT 1,
    AMOUNT INT,
    PROFIT INT,
    PROFITABLE_FLAG VARCHAR(30)
);
```

### Copy Data with Auto-Increment ID
```sql
COPY INTO OUR_FIRST_DB.PUBLIC.ORDERS_EX (PROFIT, AMOUNT)
    FROM (
        SELECT s.$2, s.$3
        FROM @MANAGE_DB.external_stages.aws_stage s
    )
    FILE_FORMAT = (TYPE = CSV FIELD_DELIMITER=',' SKIP_HEADER=1)
    FILES = ('OrderDetails.csv');
```

### Verify Data
```sql
SELECT * FROM OUR_FIRST_DB.PUBLIC.ORDERS_EX;
SELECT * FROM OUR_FIRST_DB.PUBLIC.ORDERS_EX WHERE ORDER_ID > 15;
```

### Drop the Table if Needed
```sql
DROP TABLE OUR_FIRST_DB.PUBLIC.ORDERS_EX;
```

---

## Cleanup
If needed, remove an unwanted stage:

```sql
DROP STAGE IF EXISTS EXERCISE_DB.PUBLIC.EXERCISE_DB;
```

---

## File formats Objects


 
// Specifying file_format in Copy command

```sql
COPY INTO OUR_FIRST_DB.PUBLIC.ORDERS_EX
    FROM @MANAGE_DB.external_stages.aws_stage_errorex
    file_format = (type = csv field_delimiter=',' skip_header=1)
    files = ('OrderDetails_error.csv')
    ON_ERROR = 'SKIP_FILE_3'; 
```    
    
    

// Creating table

```sql
CREATE OR REPLACE TABLE OUR_FIRST_DB.PUBLIC.ORDERS_EX (
    ORDER_ID VARCHAR(30),
    AMOUNT INT,
    PROFIT INT,
    QUANTITY INT,
    CATEGORY VARCHAR(30),
    SUBCATEGORY VARCHAR(30));  
```      
    
// Creating schema to keep things organized

```sql
CREATE OR REPLACE SCHEMA MANAGE_DB.file_formats;
```

// Creating file format object

```sql
CREATE OR REPLACE file format MANAGE_DB.file_formats.my_file_format;
```

// See properties of file format object

```sql
DESC file format MANAGE_DB.file_formats.my_file_format;
```

// Using file format object in Copy command  

```sql
COPY INTO OUR_FIRST_DB.PUBLIC.ORDERS_EX
    FROM @MANAGE_DB.external_stages.aws_stage_errorex
    file_format= (FORMAT_NAME=MANAGE_DB.file_formats.my_file_format)
    files = ('OrderDetails_error.csv')
    ON_ERROR = 'SKIP_FILE_3'; 
```    


// Altering file format object

```sql
ALTER file format MANAGE_DB.file_formats.my_file_format
    SET SKIP_HEADER = 1;
```
    
// Defining properties on creation of file format object  

```sql 
CREATE OR REPLACE file format MANAGE_DB.file_formats.my_file_format
    TYPE=JSON,
    TIME_FORMAT=AUTO;  
```      
    
// See properties of file format object  

```sql
DESC file format MANAGE_DB.file_formats.my_file_format; 
```  

  
// Using file format object in Copy command 

```sql
COPY INTO OUR_FIRST_DB.PUBLIC.ORDERS_EX
    FROM @MANAGE_DB.external_stages.aws_stage_errorex
    file_format= (FORMAT_NAME=MANAGE_DB.file_formats.my_file_format)
    files = ('OrderDetails_error.csv')
    ON_ERROR = 'SKIP_FILE_3'; 
```    


// Altering the type of a file format is not possible

```sql
ALTER file format MANAGE_DB.file_formats.my_file_format
SET TYPE = CSV;
```


// Recreate file format (default = CSV)

```sql
CREATE OR REPLACE file format MANAGE_DB.file_formats.my_file_format;
```


// See properties of file format object 

```sql   
DESC file format MANAGE_DB.file_formats.my_file_format;  
``` 



// Truncate table == Deleting values from the table
```sql
TRUNCATE table OUR_FIRST_DB.PUBLIC.ORDERS_EX;
```


// Overwriting properties of file format object 

```sql     
COPY INTO OUR_FIRST_DB.PUBLIC.ORDERS_EX
    FROM  @MANAGE_DB.external_stages.aws_stage_errorex
    file_format = (FORMAT_NAME= MANAGE_DB.file_formats.my_file_format  field_delimiter = ',' skip_header=1 )
    files = ('OrderDetails_error.csv')
    ON_ERROR = 'SKIP_FILE_3'; 
```    

```sql
DESC STAGE MANAGE_DB.external_stages.aws_stage_errorex;
```
---

// Test Do it yourself

-- Create stage object first step here database , public and the name that we asign to the stage

```sql
CREATE OR REPLACE STAGE EXERCISE_DB.public.aws_stage
    url='s3://snowflake-assignments-mc/fileformat';
```    

    -- List files in stage

```sql    
LIST @EXERCISE_DB.public.aws_stage;
```

```sql
DESC STAGE EXERCISE_DB.public.aws_stage; 
```

// Creating schema to keep things organized For my file format

```sql
CREATE OR REPLACE SCHEMA EXERCISE_DB.file_formats;
```

// Creating file format object

```sql
CREATE OR REPLACE FILE FORMAT EXERCISE_DB.FILE_FORMATS.MY_FILE_FORMAT
    TYPE = 'CSV'
    FIELD_DELIMITER = '|'
    SKIP_HEADER = 1;
```    
    
// See properties of file format object

```sql
DESC file format EXERCISE_DB.file_formats.my_file_format;
```



--Reset -table

```sql
CREATE OR REPLACE TABLE EXERCISE_DB.PUBLIC.CUSTOMER (
    ID VARCHAR,
    first_name VARCHAR,
    last_name VARCHAR,
    email VARCHAR,
    age INT,
    city VARCHAR)
```    

    
    -- We verify the table creation

```sql    
SELECT * FROM EXERCISE_DB.PUBLIC.CUSTOMER;
```


-- Load the data 

```sql
COPY INTO EXERCISE_DB.PUBLIC.CUSTOMER
FROM @EXERCISE_DB.PUBLIC.AWS_STAGE
FILE_FORMAT = (FORMAT_NAME = EXERCISE_DB.FILE_FORMATS.MY_FILE_FORMAT)
FILES = ('customers4.csv')
ON_ERROR = 'SKIP_FILE_3%';  
```

   

## Conclusion
This document serves as a detailed guide for setting up external stages, managing data in Snowflake, and transforming data using SQL functions. By using file format objects, the process of data loading becomes more efficient and less error-prone. The step-by-step SQL commands provided here facilitate ease of execution. Copy and execute them as needed to streamline your Snowflake data management tasks. Happy querying!