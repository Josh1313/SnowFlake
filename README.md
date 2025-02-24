# Loading Data from AWS S3 to Snowflake

## Overview
This document provides a step-by-step guide on how to load data from an AWS S3 bucket into Snowflake. We will cover the creation of an S3 bucket, configuring the necessary IAM roles and policies, setting up Snowflake integration, and updating the trust policy for seamless data loading.

## Index
- [Step 1: Creating an S3 Bucket](#step-1-creating-an-s3-bucket)
- [Step 2: Configuring IAM Policies](#step-2-configuring-iam-policies)
- [Step 3: Setting Up Snowflake Integration](#step-3-setting-up-snowflake-integration)
- [Step 4: Updating AWS Trust Policy](#step-4-updating-aws-trust-policy)
- [Conclusion](#conclusion)

---

## Step 1: Creating an S3 Bucket
1. Go to **S3** → **Create bucket**.
2. Choose the same region as your Snowflake account to avoid additional expenses.
3. Enter a **Bucket Name** (e.g., `my-snowflake-bucket`).
4. Choose **General Purpose** and click **Create Bucket**.
5. After creating the bucket, select it and create two folders:
    - One for JSON files
    - One for CSV files
6. Upload your JSON and CSV files to the corresponding folders.

You have now successfully created an S3 bucket.

---

## Step 2: Configuring IAM Policies
This step establishes a connection between the S3 bucket and Snowflake by creating a new role in AWS IAM.

1. Go to **IAM** in AWS.
2. On the left, select **Roles**.
3. Click **Create Role**.
4. Choose **AWS Account** → **This Account**.
5. Select **Require External ID** and enter `00000` as a temporary value.
6. Click **Next**.
7. Search for **S3** and select **AmazonS3FullAccess**.
8. Click **Next**.
9. Name the role as `snowflake-access-role`.
10. Add a description: "This role is used to grant access to Snowflake".
11. Click **Create Role**.

Configuration is done, but we need to update it with information from Snowflake.

---

## Step 3: Setting Up Snowflake Integration
1. Log in to Snowflake and open a new worksheet.
2. Retrieve the S3 bucket name and file names, then use them in the Snowflake object creation.
3. In AWS, go to **IAM** and select the role created for Snowflake.
4. Copy the **ARN Number** of the role.
5. In Snowflake, create a stage u:
6. Note the following values:
    - `STORAGE_AWS_IAM_USER_ARN` → (1)
    - `STORAGE_AWS_EXTERNAL_ID` → (2)

---

## Step 4: Updating AWS Trust Policy
1. In AWS, select the role created for Snowflake.
2. Go to **Trusted Relationships** → **Edit Trust Policy**.
3. Update the following values:
```json
"AWS": "(1) ",
"sts:ExternalId": "(2)"
```
4. Click **Update Policy**.

With these steps, the S3 bucket and Snowflake integration are successfully configured, enabling seamless data loading.

5. ## Code Storage Integration Object

 Create storage integration object

```sql
create or replace storage integration s3_int
  TYPE = EXTERNAL_STAGE
  STORAGE_PROVIDER = S3
  ENABLED = TRUE 
  STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::110991281769:role/snowflake-acces-role'
  STORAGE_ALLOWED_LOCATIONS = ('s3://snowflake1985es/csv/', 's3://snowflake1985es/json/')
   COMMENT = 'This an optional comment' ;
```   
   
   
-- See storage integration properties to fetch external_id so we can update it in S3

```sql
DESC integration s3_int;
```


-- We have full integrated and stablish conection we load the data from s3

-- Create table first
```sql
CREATE OR REPLACE TABLE OUR_FIRST_DB.PUBLIC.movie_titles (
  show_id STRING,
  type STRING,
  title STRING,
  director STRING,
  cast STRING,
  country STRING,
  date_added STRING,
  release_year STRING,
  rating STRING,
  duration STRING,
  listed_in STRING,
  description STRING )
  ```
  
  

-- Create file format object here we have a intentional issue where we see a few thing how to fixed

```sql
CREATE OR REPLACE file format MANAGE_DB.file_formats.csv_fileformat
    type = csv
    field_delimiter = ','
    skip_header = 1
    null_if = ('NULL','null')
    empty_field_as_null = TRUE;
```    
    
    
--  Create stage object with integration object & file format object

```sql
CREATE OR REPLACE stage MANAGE_DB.external_stages.csv_folder
    URL = 's3://snowflake1985es/csv/'
    STORAGE_INTEGRATION = s3_int
    FILE_FORMAT = MANAGE_DB.file_formats.csv_fileformat
```    



--  Use Copy command 

```sql
COPY INTO OUR_FIRST_DB.PUBLIC.movie_titles
    FROM @MANAGE_DB.external_stages.csv_folder
```    
    
    
    
    
    
--  Create file format object // already fixed

```sql
CREATE OR REPLACE file format MANAGE_DB.file_formats.csv_fileformat
    type = csv
    field_delimiter = ','
    skip_header = 1
    null_if = ('NULL','null')
    empty_field_as_null = TRUE    
    FIELD_OPTIONALLY_ENCLOSED_BY = '"'    
```    
    
```sql    
SELECT * FROM OUR_FIRST_DB.PUBLIC.movie_titles
```
## successfully Done with CSV files from AWS


-- Taming the JSON file// we need to create a format and the appropiate stage for json files
## Now  We Start with JSON files

-- Create file format object // Update for json
```sql 
CREATE OR REPLACE file format MANAGE_DB.file_formats.json_fileformat
    type = JSON  
```    

--  Create stage object with integration object & json file format object

```sql 
CREATE OR REPLACE stage MANAGE_DB.external_stages.json_folder
    URL = 's3://snowflake1985es/json/'
    STORAGE_INTEGRATION = s3_int
    FILE_FORMAT = MANAGE_DB.file_formats.json_fileformat
```


--  First query from S3 Bucket   // if we dont know the schema we can query

```sql 
SELECT * FROM @MANAGE_DB.external_stages.json_folder;
```



-- Introduce columns 

```sql 
SELECT 
$1:asin,
$1:helpful,
$1:overall,
$1:reviewText,
$1:reviewTime,
$1:reviewerID,
$1:reviewTime,
$1:reviewerName,
$1:summary,
$1:unixReviewTime
FROM @MANAGE_DB.external_stages.json_folder;
```

-- Format columns & use DATE function

```sql 
SELECT 
$1:asin::STRING as ASIN,
$1:helpful as helpful,
$1:overall as overall,
$1:reviewText::STRING as reviewtext,
$1:reviewTime::STRING,
$1:reviewerID::STRING,
$1:reviewTime::STRING,
$1:reviewerName::STRING,
$1:summary::STRING,
DATE($1:unixReviewTime::int) as Revewtime
FROM @MANAGE_DB.external_stages.json_folder;
```

-- Format columns & handle custom date // template

```sql 
SELECT 
$1:asin::STRING as ASIN,
$1:helpful as helpful,
$1:overall as overall,
$1:reviewText::STRING as reviewtext,
DATE_FROM_PARTS( <year>, <month>, <day> )
$1:reviewTime::STRING,
$1:reviewerID::STRING,
$1:reviewTime::STRING,
$1:reviewerName::STRING,
$1:summary::STRING,
DATE($1:unixReviewTime::int) as Revewtime
FROM @MANAGE_DB.external_stages.json_folder;
```

-- Use DATE_FROM_PARTS and see another difficulty// intentionally error because the substring might not always contain a proper number

```sql 
SELECT 
$1:asin::STRING as ASIN,
$1:helpful as helpful,
$1:overall as overall,
$1:reviewText::STRING as reviewtext,
DATE_FROM_PARTS( RIGHT($1:reviewTime::STRING,4), LEFT($1:reviewTime::STRING,2), SUBSTRING($1:reviewTime::STRING,4,2) ),
$1:reviewerID::STRING,
$1:reviewTime::STRING,
$1:reviewerName::STRING,
$1:summary::STRING,
DATE($1:unixReviewTime::int) as unixRevewtime
FROM @MANAGE_DB.external_stages.json_folder;
```


-- Use DATE_FROM_PARTS and handle the case difficulty// fix used case

```sql 
SELECT 
$1:asin::STRING as ASIN,
$1:helpful as helpful,
$1:overall as overall,
$1:reviewText::STRING as reviewtext,
DATE_FROM_PARTS( 
  RIGHT($1:reviewTime::STRING,4), 
  LEFT($1:reviewTime::STRING,2), 
  CASE WHEN SUBSTRING($1:reviewTime::STRING,5,1)=',' 
        THEN SUBSTRING($1:reviewTime::STRING,4,1) ELSE SUBSTRING($1:reviewTime::STRING,4,2) END),
$1:reviewerID::STRING,
$1:reviewTime::STRING,
$1:reviewerName::STRING,
$1:summary::STRING,
DATE($1:unixReviewTime::int) as UnixRevewtime
FROM @MANAGE_DB.external_stages.json_folder;
```


-- Create destination table

```sql 
CREATE OR REPLACE TABLE OUR_FIRST_DB.PUBLIC.reviews (
asin STRING,
helpful STRING,
overall STRING,
reviewtext STRING,
reviewtime DATE,
reviewerid STRING,
reviewername STRING,
summary STRING,
unixreviewtime DATE
);
```

--  Copy transformed data into destination table
```sql 
COPY INTO OUR_FIRST_DB.PUBLIC.reviews
    FROM (SELECT 
$1:asin::STRING as ASIN,
$1:helpful as helpful,
$1:overall as overall,
$1:reviewText::STRING as reviewtext,
DATE_FROM_PARTS( 
  RIGHT($1:reviewTime::STRING,4), 
  LEFT($1:reviewTime::STRING,2), 
  CASE WHEN SUBSTRING($1:reviewTime::STRING,5,1)=',' 
        THEN SUBSTRING($1:reviewTime::STRING,4,1) ELSE SUBSTRING($1:reviewTime::STRING,4,2) END),
$1:reviewerID::STRING,
$1:reviewerName::STRING,
$1:summary::STRING,
DATE($1:unixReviewTime::int) Revewtime
FROM @MANAGE_DB.external_stages.json_folder);
```  
    
--  Validate results

```sql 
SELECT * FROM OUR_FIRST_DB.PUBLIC.reviews ;
```sql 
    
    
   

---

## Conclusion
You have now successfully:
- Created an S3 bucket in the correct region
- Configured IAM roles and policies
- Set up Snowflake integration
- Updated the AWS trust policy

You can now continue working with your data in Snowflake efficiently.

