# Loading Data from Azure to Snowflake

## Overview
This document provides a step-by-step guide on how to load data from an Azure Storage Account into Snowflake. We will cover the creation of a storage account, setting up containers, uploading files, and configuring Snowflake integration for seamless data loading.

## Index
- [Step 1: Creating an Azure Storage Account](#step-1-creating-an-azure-storage-account)
- [Step 2: Creating and Uploading to Containers](#step-2-creating-and-uploading-to-containers)
- [Step 3: Collecting Required Azure Information](#step-3-collecting-required-azure-information)
- [Step 4: Integrating with Snowflake](#step-4-integrating-with-snowflake)
- [Conclusion](#conclusion)

---

## Step 1: Creating an Azure Storage Account
1. Ensure you have an **Azure account**.
2. In the **Search Bar**, type **Storage Account** and select the **+ Create** button.
3. Enter the following details:
    - **Resource Group**: Use an existing one or create a new one (e.g., `snowflake`).
    - **Storage Account Name**: Choose a unique name (e.g., `snowflakeawsgcp`).
    - **Region**: Select the same region as your Snowflake account.
    - **Redundancy**: Choose **Locally Redundant Storage (LRS)** for testing purposes.
4. Click **Review + Create**, verify the details, and select **Create**.

At this point, your Azure Storage Account is successfully created.

---

## Step 2: Creating and Uploading to Containers
1. Go to **Storage Accounts** and select your newly created storage account.
2. Navigate to **Data Storage** → **Containers**.
3. Click **+ Container** and create two containers:
    - One for CSV files (`csv`)
    - One for JSON files (`json`)
4. After creation, select a container and click **+ Upload**.
5. Click **Browse File**, select the files to upload, and click **Upload**.
6. Repeat the process for all files in both containers.

Your data is now stored and ready for integration with Snowflake.

---

## Step 3: Collecting Required Azure Information
Before integrating with Snowflake, collect the following details from Azure:
1. **Tenant ID**:
    - Search for **Tenant Properties** in the Azure search bar.
    - Copy the **Tenant ID** from the **Tenant Properties** page.
2. **Storage Account Name** (e.g., `snowflakeawsgcp`).
3. **Container Names** (`csv`, `json`).

---

## Step 4: Integrating with Snowflake
Use the collected Azure details in the following Snowflake commands to create the integration.

```sql
USE DATABASE DEMO_DB;

CREATE OR REPLACE STORAGE INTEGRATION azure_integration
  TYPE = EXTERNAL_STAGE
  STORAGE_PROVIDER = AZURE
  ENABLED = TRUE
  AZURE_TENANT_ID = '<tenant_id>'
  STORAGE_ALLOWED_LOCATIONS = ('azure://your-storage-account-name.blob.core.windows.net/csv',
  'azure://your-storage-account-name.blob.core.windows.net/json');
```

###  Describe integration object to provide access
```sql
DESC STORAGE integration azure_integration;
```
---
### Granting Access after STORAGE integration azure_integration 

- In the Snowflake description output, look for `AZURE_CONSENT_URL`.
- Copy the **AZURE_CONSENT_URL** and paste it into a new browser window where you are logged into your Azure account to grant authorization.
- After granting permission, you will receive an **Application ID**. Copy and save this ID for later use.

---
---
## Step 5: Granting Permissions in Azure
1. Go to **Storage Account** → **Your Container** → **Access Control (IAM)**.
2. Click on the **+ Add** icon → **Add role assignment**.
3. In the **Search Bar**  == **Storage Blob Data Reader** and select it.
4. Click **Next**.
5. Click on **+ Select Members**.
6. In the **Search Bar** on the right-side pop-up window, paste the **Application ID** copied from the Azure consent step.
7. Select the application and click **Continue**.

Permissions are now successfully granted to Snowflake.
---

### Create Database File Format & Stage Objects
```sql
CREATE OR REPLACE DATABASE DEMO_DB;
```

```sql
CREATE OR REPLACE FILE FORMAT demo_db.public.fileformat_azure
    TYPE = CSV
    FIELD_DELIMITER = ','
    SKIP_HEADER = 1;
```
```sql
CREATE OR REPLACE STAGE demo_db.public.stage_azure
    STORAGE_INTEGRATION = azure_integration
    URL = 'azure://your-storage-account-name.blob.core.windows.net/csv'
    FILE_FORMAT = fileformat_azure;
```

### Query & Load Data
```sql
LIST @demo_db.public.stage_azure;
```

---- Query files & Load data ----

--query files

```sql
SELECT 
$1,
$2,
$3,
$4,
$5,
$6,
$7,
$8,
$9,
$10,
$11,
$12,
$13,
$14,
$15,
$16,
$17,
$18,
$19,
$20
FROM @demo_db.public.stage_azure;
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
FROM @demo_db.public.stage_azure;
```


```sql
SELECT * FROM HAPPINESS;
```

--- Load JSON ----

```sql 
create or replace file format demo_db.public.fileformat_azure_json
    TYPE = JSON;
```
```sql  
create or replace stage demo_db.public.stage_azure
    STORAGE_INTEGRATION = azure_integration
    URL = 'azure://snowflakeawsazure.blob.core.windows.net/json'
    FILE_FORMAT = fileformat_azure_json; 
```    
```sql
 DESC INTEGRATION azure_integration; 
```

-- THIS IS JUST IN CASE WE NNED TO ALTE BY SET THE LOCATION OF OUR CONTAINERS
-- ALTER INTEGRATION azure_integration
-- SET STORAGE_ALLOWED_LOCATIONS =('azure://snowflakeawsazure.blob.core.windows.net/csv','azure://snowflakeawsazure.blob.core.windows.net/json')

```sql  
LIST  @demo_db.public.stage_azure;
```

-- Query from stage  

```sql
SELECT * FROM @demo_db.public.stage_azure; 
``` 


-- Query one attribute/column

```sql
SELECT $1:"Car Model" FROM @demo_db.public.stage_azure; 
```
  
-- Convert data type  
```sql
SELECT $1:"Car Model"::STRING FROM @demo_db.public.stage_azure; 
```

-- Query all attributes  

```sql
SELECT 
$1:"Car Model"::STRING, 
$1:"Car Model Year"::INT,
$1:"car make"::STRING, 
$1:"first_name"::STRING,
$1:"last_name"::STRING
FROM @demo_db.public.stage_azure;   
```
  
-- Query all attributes and use aliases 

```sql
SELECT 
$1:"Car Model"::STRING as car_model, 
$1:"Car Model Year"::INT as car_model_year,
$1:"car make"::STRING as "car make", 
$1:"first_name"::STRING as first_name,
$1:"last_name"::STRING as last_name
FROM @demo_db.public.stage_azure;     
```

```sql
Create or replace table car_owner (
    car_model varchar, 
    car_model_year int,
    car_make varchar, 
    first_name varchar,
    last_name varchar);
```    
```sql 
COPY INTO car_owner
FROM
(SELECT 
$1:"Car Model"::STRING as car_model, 
$1:"Car Model Year"::INT as car_model_year,
$1:"car make"::STRING as "car make", 
$1:"first_name"::STRING as first_name,
$1:"last_name"::STRING as last_name
FROM @demo_db.public.stage_azure);
```

```sql
SELECT * FROM CAR_OWNER;
```


-- Alternative: Using a raw file table step

```sql
truncate table car_owner;
select * from car_owner;
```

```sql
create or replace table car_owner_raw (
  raw variant);
```

```sql
COPY INTO car_owner_raw
FROM @demo_db.public.stage_azure;
```

```sql
SELECT * FROM car_owner_raw;
```
```sql    
INSERT INTO car_owner  
(SELECT 
$1:"Car Model"::STRING as car_model, 
$1:"Car Model Year"::INT as car_model_year,
$1:"car make"::STRING as car_make, 
$1:"first_name"::STRING as first_name,
$1:"last_name"::STRING as last_name
FROM car_owner_raw)  ;
```
  
```sql  
select * from car_owner;
```

---

## Conclusion
You have now successfully:
- Created an Azure Storage Account
- Configured and uploaded files into Azure containers
- Retrieved the necessary details for integration
- Set up Snowflake integration for data loading

Your data is now ready to be accessed and processed in Snowflake.

