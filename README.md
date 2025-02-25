# Setting up Snowpipe with AWS

## Overview
This document provides a step-by-step guide on how to set up Snowpipe in Snowflake using AWS S3 for continuous data loading. Snowpipe automatically loads data as soon as files appear in the S3 bucket, ideal for scenarios where data must be immediately available for analysis. This guide covers creating stages, testing with COPY commands, creating the pipe, and setting up S3 notifications.

## Index
- [What is Snowpipe?](#what-is-snowpipe)
- [Step 1: Creating the Storage Integration](#step-1-creating-the-storage-integration)
- [Step 2: Creating Table and File Format](#step-2-creating-table-and-file-format)
- [Step 3: Creating Stage](#step-3-creating-stage)
- [Step 4: Creating Pipe](#step-4-creating-pipe)
- [Step 5: Setting up S3 Event Notification](#step-5-setting-up-s3-event-notification)
- [Step 6: Testing and Error Handling](#step-6-testing-and-error-handling)
- [Step 7: Managing Pipes](#step-7-managing-pipes)
- [Conclusion](#conclusion)

---

## What is Snowpipe?
Snowpipe is a continuous data loading feature in Snowflake that automatically loads files as they appear in an S3 bucket. It is designed for real-time data ingestion and is not intended for batch processing. Snowpipe uses event notifications from S3 to trigger data loading into Snowflake tables.

![alt text](image.png)

![alt text](image-1.png)

---

## Step 1: Creating the Storage Integration
```sql
CREATE OR REPLACE STORAGE INTEGRATION s3_int
  TYPE = EXTERNAL_STAGE
  STORAGE_PROVIDER = S3
  ENABLED = TRUE
  STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::110991281769:role/snowflake-access-role'
  STORAGE_ALLOWED_LOCATIONS = ('s3://snowflakeaws1985/csv/', 's3://snowflakeaws1985/json/', 's3://snowflakeaws1985/snowpipe/')
  COMMENT = 'This an optional comment';
```
## Describe  in case you havent configure your role
```sql
DESC INTEGRATION s3_int;
```

---

## Step 2: Creating Table and File Format
```sql
CREATE OR REPLACE TABLE OUR_FIRST_DB.PUBLIC.employees (
  id INT,
  first_name STRING,
  last_name STRING,
  email STRING,
  location STRING,
  department STRING);
```  
```sql
CREATE OR REPLACE FILE FORMAT MANAGE_DB.file_formats.csv_fileformat
  TYPE = CSV
  FIELD_DELIMITER = ','
  SKIP_HEADER = 1
  NULL_IF = ('NULL','null')
  EMPTY_FIELD_AS_NULL = TRUE;
```

---

## Step 3: Creating Stage
```sql
CREATE OR REPLACE STAGE MANAGE_DB.external_stages.csv_folder
  URL = 's3://snowflakeaws1985/csv/snowpipe'
  STORAGE_INTEGRATION = s3_int
  FILE_FORMAT = MANAGE_DB.file_formats.csv_fileformat;

LIST @MANAGE_DB.external_stages.csv_folder;
```

---

## Step 4: Creating Pipe & schema to keep things organized

-- Create schema to keep things organized
```sql
CREATE OR REPLACE SCHEMA MANAGE_DB.pipes;
```

-- Define pipe// at this stage you should test before creating the pipe to check if everyting work perfectly copy into == select
```sql
CREATE OR REPLACE PIPE MANAGE_DB.pipes.employee_pipe
AUTO_INGEST = TRUE
AS
COPY INTO OUR_FIRST_DB.PUBLIC.employees
FROM @MANAGE_DB.external_stages.csv_folder;
```  
-- Describe pipe copy the value of notification_channel after this go into step 5 
```sql
DESC pipe employee_pipe;
DESC PIPE MANAGE_DB.pipes.employee_pipe;
```
-- 
```sql
SELECT COUNT(*) FROM OUR_FIRST_DB.PUBLIC.employees;
```

---

## Step 5: Setting up S3 Event Notification
1. Go to **AWS Console** → **S3** → **Bucket** → **Properties**.
2. Scroll to **Event Notifications** and click **Create Event Notification**.
3. **Event Name**: Unique-name
4. **Prefix**: `csv/snowpipe`
5. **Suffix**: (Optional)
6. **Event Type**: Select **All Object Create Events**
7. **Destination**: Select **SQS queue**
8. **Enter SQS queue ARN**: Paste the value from Snowflake's Notification Channel
9. Click **Save**.

---

## Step 6: Testing and Error Handling

-- Create file format object field_delimiter = '|' TO CREATE AND  ERROR IN DATA 3

```sql
CREATE OR REPLACE file format MANAGE_DB.file_formats.csv_fileformat
    type = csv
    field_delimiter = ','
    skip_header = 1
    null_if = ('NULL','null')
    empty_field_as_null = TRUE;
``` 
```sql   
SELECT COUNT(*) FROM OUR_FIRST_DB.PUBLIC.employees ; 
``` 
```sql
ALTER PIPE employee_pipe refresh;
```

-- Validate pipe is actually working
```sql
SELECT SYSTEM$PIPE_STATUS('MANAGE_DB.pipes.employee_pipe');
```
-- Snowpipe error message
```sql
SELECT * FROM TABLE(VALIDATE_PIPE_LOAD(
  PIPE_NAME => 'MANAGE_DB.pipes.employee_pipe',
  START_TIME => DATEADD(HOUR,-2,CURRENT_TIMESTAMP())));
```

--COPY command history from table to see error massage

```sql
SELECT * FROM TABLE (INFORMATION_SCHEMA.COPY_HISTORY(
  TABLE_NAME  =>  'OUR_FIRST_DB.PUBLIC.EMPLOYEES',
  START_TIME => DATEADD(HOUR,-2,CURRENT_TIMESTAMP())));
```

---

## Step 7: Managing Pipes
```sql
-- SHOW  ALL PIPES IN A LIST
SHOW PIPES;
-- SEARCHING FOR EMPLOYEES USING WILDCARDS
SHOW PIPES LIKE '%employee%';
-- SEARCHING FOR PIPE IN A UNIQUE DATABASE
SHOW PIPES IN DATABASE MANAGE_DB;
-- SEARCHING FOR SCHEMA 
SHOW PIPES IN SCHEMA MANAGE_DB.pipes;
-- ALSO COMBINING THE SEARCHING METHODS
SHOW PIPES like '%employee%' in Database MANAGE_DB;
```
## Changing pipe (alter stage or file format) --
```sql
-- Preparation table first
CREATE OR REPLACE TABLE OUR_FIRST_DB.PUBLIC.employees2 (
  id INT,
  first_name STRING,
  last_name STRING,
  email STRING,
  location STRING,
  department STRING
  );
```  
-- Pause pipe// BASE PRACTICES PAUSE
```sql
ALTER PIPE MANAGE_DB.pipes.employee_pipe SET PIPE_EXECUTION_PAUSED = TRUE;
```

-- Verify pipe is paused and has pendingFileCount 0 
```sql
SELECT SYSTEM$PIPE_STATUS('MANAGE_DB.pipes.employee_pipe');
```


-- Recreate the pipe to change the COPY statement in the definition

```sql
CREATE OR REPLACE pipe MANAGE_DB.pipes.employee_pipe
auto_ingest = TRUE
AS
COPY INTO OUR_FIRST_DB.PUBLIC.employees2
FROM @MANAGE_DB.external_stages.csv_folder ;
```
```sql
ALTER PIPE  MANAGE_DB.pipes.employee_pipe refresh;
```

-- List files in stage

```sql
LIST @MANAGE_DB.external_stages.csv_folder ;
```
```sql
SELECT * FROM OUR_FIRST_DB.PUBLIC.employees2;
```

-- Reload files manually that where aleady in the bucket
```sql
COPY INTO OUR_FIRST_DB.PUBLIC.employees2
FROM @MANAGE_DB.external_stages.csv_folder;  
```


-- Resume pipe
```sql
ALTER PIPE MANAGE_DB.pipes.employee_pipe SET PIPE_EXECUTION_PAUSED = false;
```

-- Verify pipe is running again
```sql
SELECT SYSTEM$PIPE_STATUS('MANAGE_DB.pipes.employee_pipe') ;
```


---

## Conclusion
You have now successfully:
- Created a storage integration for S3
- Configured Snowpipe with S3 event notifications
- Created and tested the pipe for auto-ingestion
- Managed and monitored Snowpipe status

Your data is now continuously loaded into Snowflake using Snowpipe.

