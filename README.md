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

## Step 4: Creating Pipe
```sql
CREATE OR REPLACE PIPE MANAGE_DB.pipes.employee_pipe
  AUTO_INGEST = TRUE
AS
  COPY INTO OUR_FIRST_DB.PUBLIC.employees
  FROM @MANAGE_DB.external_stages.csv_folder;

DESC PIPE MANAGE_DB.pipes.employee_pipe;
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
7. **Destination**: Select **SQS**
8. **SQS ARN**: Paste the value from Snowflake's Notification Channel
9. Click **Save**.

---

## Step 6: Testing and Error Handling
```sql
SELECT SYSTEM$PIPE_STATUS('MANAGE_DB.pipes.employee_pipe');

SELECT * FROM TABLE(VALIDATE_PIPE_LOAD(
  PIPE_NAME => 'MANAGE_DB.pipes.employee_pipe',
  START_TIME => DATEADD(HOUR,-2,CURRENT_TIMESTAMP())));

SELECT * FROM TABLE (INFORMATION_SCHEMA.COPY_HISTORY(
  TABLE_NAME  =>  'OUR_FIRST_DB.PUBLIC.EMPLOYEES',
  START_TIME => DATEADD(HOUR,-2,CURRENT_TIMESTAMP())));
```

---

## Step 7: Managing Pipes
```sql
SHOW PIPES;
SHOW PIPES LIKE '%employee%';
SHOW PIPES IN DATABASE MANAGE_DB;
SHOW PIPES IN SCHEMA MANAGE_DB.pipes;

ALTER PIPE MANAGE_DB.pipes.employee_pipe SET PIPE_EXECUTION_PAUSED = TRUE;
SELECT SYSTEM$PIPE_STATUS('MANAGE_DB.pipes.employee_pipe');

ALTER PIPE MANAGE_DB.pipes.employee_pipe SET PIPE_EXECUTION_PAUSED = FALSE;
```

---

## Conclusion
You have now successfully:
- Created a storage integration for S3
- Configured Snowpipe with S3 event notifications
- Created and tested the pipe for auto-ingestion
- Managed and monitored Snowpipe status

Your data is now continuously loaded into Snowflake using Snowpipe.

