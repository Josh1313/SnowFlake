# Loading and Unloading Data from GCP to Snowflake

## Overview
This document provides a step-by-step guide on how to load and unload data between Google Cloud Platform (GCP) and Snowflake. We will cover creating buckets in GCP, configuring permissions, setting up Snowflake integration, and performing data loading and unloading.

## Index
- [Step 1: Creating GCP Buckets](#step-1-creating-gcp-buckets)
- [Step 2: Uploading Data to Buckets](#step-2-uploading-data-to-buckets)
- [Step 3: Setting up Snowflake Integration](#step-3-setting-up-snowflake-integration)
- [Step 4: Granting Permissions in GCP](#step-4-granting-permissions-in-gcp)
- [Step 5: Loading Data into Snowflake](#step-5-loading-data-into-snowflake)
- [Step 6: Unloading Data from Snowflake](#step-6-unloading-data-from-snowflake)
- [Conclusion](#conclusion)

---

## Step 1: Creating GCP Buckets
1. Sign in to your **GCP account**.
2. In the top left, click on the **Menu (stack of pancakes)**.
3. Navigate to **Cloud Storage** → **Buckets**.
4. Click on **Get Started** → **Create a Bucket**.
5. Enter a **Unique Name** (e.g., `csv-bucket` or `json-bucket`).
6. **Location Type**: Choose **Multi-region** → **Continue**.
7. **Storage Class**: Leave as default → **Continue**.
8. **Access Control**: Leave as default → **Continue**.
9. **Protection**: Leave as default → **Create**.

Repeat the above steps to create additional buckets as needed for JSON or other file types.

---

## Step 2: Uploading Data to Buckets
1. Go to the **Bucket List** and select the bucket to upload files.
2. Click **Upload** → **Files or Folders**.
3. Browse and select the files to upload.
4. Click **Upload**.

You have now successfully created the buckets and uploaded data to each bucket.

---

## Step 3: Setting up Snowflake Integration
Use the bucket names from GCP in the following Snowflake commands.

```sql
CREATE STORAGE INTEGRATION gcp_integration
  TYPE = EXTERNAL_STAGE
  STORAGE_PROVIDER = GCS
  ENABLED = TRUE
  STORAGE_ALLOWED_LOCATIONS = ('gcs://Bucket-name/path', 'gcs://Bucket-name/path2');

-- Describe integration object to get the required info
DESC STORAGE INTEGRATION gcp_integration;
```

---

## Step 4: Granting Permissions in GCP
1. Go to **Storage Account** → **Buckets**.
2. Select the buckets created earlier.
3. Click on **Permissions**.
4. Click **Add Principal**.
5. **New Principal**: Paste the value from `STORAGE_GCP_SERVICE_ACCOUNT` from the Snowflake `DESC` output.
6. **Role**: Select **Storage Admin**.
7. Click **Save**.

Permissions are now successfully granted to Snowflake.

---

## Step 5: Loading Data into Snowflake
```sql
CREATE OR REPLACE FILE FORMAT demo_db.public.fileformat_gcp
    TYPE = CSV
    FIELD_DELIMITER = ','
    SKIP_HEADER = 1;
```

```sql
CREATE OR REPLACE STAGE demo_db.public.stage_gcp
    STORAGE_INTEGRATION = gcp_integration
    URL = 'gcs://snowflakegcp1985'
    FILE_FORMAT = fileformat_gcp;
```
```sql
LIST @demo_db.public.stage_gcp;
```
---- Query files & Load data ----

--query files
```sql
SELECT 
$1,$2,$3,$4,$5,$6,$7,$8,$9,$10,$11,
$12,$13,$14,$15,$16,$17,$18,$19,$20
FROM @demo_db.public.stage_gcp;
```

```sql
create or replace table happiness (
    country_name varchar,
    regional_indicator varchar,
    ladder_score number(4,3),
    standard_error number(4,3),
    upperwhisker number(4,3),
    lowerwhisker number(4,3),
    logged_gdp number(5,3),
    social_support number(4,3),
    healthy_life_expectancy number(5,3),
    freedom_to_make_life_choices number(4,3),
    generosity number(4,3),
    perceptions_of_corruption number(4,3),
    ladder_score_in_dystopia number(4,3),
    explained_by_log_gpd_per_capita number(4,3),
    explained_by_social_support number(4,3),
    explained_by_healthy_life_expectancy number(4,3),
    explained_by_freedom_to_make_life_choices number(4,3),
    explained_by_generosity number(4,3),
    explained_by_perceptions_of_corruption number(4,3),
    dystopia_residual number (4,3));
```    
    
```sql 
COPY INTO HAPPINESS
FROM @demo_db.public.stage_gcp;
```
```sql
SELECT * FROM HAPPINESS;
```

## ------- Unload data -----
--  In case you are not using it   
```sql
USE ROLE ACCOUNTADMIN;
USE DATABASE DEMO_DB;
```


-- create integration object that contains the access information we have done this already
```sql
CREATE STORAGE INTEGRATION gcp_integration
  TYPE = EXTERNAL_STAGE
  STORAGE_PROVIDER = GCS
  ENABLED = TRUE
  STORAGE_ALLOWED_LOCATIONS = ('gcs://Bucket-name', 'gcs://Bucket-name');
```
  
  
-- Create file format  we have done this already
```sql
create or replace file format demo_db.public.fileformat_gcp
    TYPE = CSV
    FIELD_DELIMITER = ','
    SKIP_HEADER = 1;
```

-- Create stage object we start here again where we are cerating the subfolder csv_happines
```sql
create or replace stage demo_db.public.stage_gcp
    STORAGE_INTEGRATION = gcp_integration
    URL = 'gcs://snowflakegcp1985/csv_happiness'
    FILE_FORMAT = fileformat_gcp;
```    

-- In case we need to modifi the locations we can do that  using alter
```sql
ALTER STORAGE INTEGRATION gcp_integration
SET  storage_allowed_locations=('gcs://snowflakebucketgcp', 'gcs://snowflakebucketgcpjson');
```
```sql
SELECT * FROM HAPPINESS;
```

-- into the destination the stage
```sql
COPY INTO @stage_gcp
FROM
HAPPINESS;
```

---

## Conclusion
You have now successfully:
- Created GCP buckets and uploaded data
- Configured Snowflake integration with GCP
- Loaded data from GCP into Snowflake
- Unloaded data from Snowflake to GCP

Your data is now seamlessly connected between GCP and Snowflake.

