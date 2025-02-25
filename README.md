# Setting up Snowpipe with Azure
![alt text](image.png)
![alt text](image-1.png)

## Overview
This document provides a step-by-step guide on how to set up Snowpipe in Snowflake using Azure Blob Storage for continuous data loading. Snowpipe automatically loads data as soon as files appear in the Azure container, ideal for scenarios where data must be immediately available for analysis. This guide covers creating storage accounts, setting up containers, collecting necessary Azure information, configuring Snowflake integration, and granting permissions.

## Index
- [Prerequisites](#prerequisites)
- [Step 1: Creating a Storage Account](#step-1-creating-a-storage-account)
- [Step 2: Creating and Uploading to Containers](#step-2-creating-and-uploading-to-containers)
- [Step 3: Collecting Required Azure Information](#step-3-collecting-required-azure-information)
- [Step 4: Creating Snowflake Integration](#step-4-creating-snowflake-integration)
- [Step 5: Granting Permissions in Azure](#step-5-granting-permissions-in-azure)
- [Conclusion](#conclusion)

---

## Prerequisites
Ensure you have an **Azure account**.

---

## Step 1: Creating a Storage Account
1. In the **Search Bar**, type **Storage Account** and select the **+ Create** button.
2. Enter the following details:
    - **Resource Group**: Use an existing one or create a new one (e.g., `snowflake`).
    - **Storage Account Name**: Choose a unique name (e.g., `snowflakeawsgcp`).
    - **Region**: Select the same region as your Snowflake account.
    - **Redundancy**: Choose **Locally Redundant Storage (LRS)** for testing purposes.
3. Click **Review + Create**, verify the details, and select **Create**.

At this point, your Azure Storage Account is successfully created.

---

## Step 2: Creating and Uploading to Containers
1. Go to **Storage Accounts** and select your newly created storage account.
2. Navigate to **Data Storage** → **Containers**.
3. Click **+ Container** and create one container for Snowpipe:
    - **snowpipecsv** for CSV files
4. After creation, select the container and click **+ Upload**.
5. Click **Browse File**, select the files to upload, and click **Upload**.

Your data is now stored and ready for integration with Snowflake.

---

## Step 3: Collecting Required Azure Information
Before integrating with Snowflake, collect the following details from Azure:
1. **Tenant ID**:
    - Search for **Tenant Properties** in the Azure search bar.
    - Copy the **Tenant ID** from the **Tenant Properties** page.
2. **Storage Account Name** (e.g., `snowflakeawsgcp`).
3. **Container Name** (`snowpipecsv`).

---

## Step 4: Creating Snowflake Integration
```sql
CREATE OR REPLACE DATABASE SNOWPIPE;

CREATE OR REPLACE STORAGE INTEGRATION azure_snowpipe_integration
  TYPE = EXTERNAL_STAGE
  STORAGE_PROVIDER = AZURE
  ENABLED = TRUE
  AZURE_TENANT_ID = 'dae2c9e3-5d25-4628-a32f-6077ffaede10'
  STORAGE_ALLOWED_LOCATIONS = ('azure://snowflakeawsgcp.blob.core.windows.net/snowpipecsv');
```
-- -- Describe integration object to provide access
```sql
DESC STORAGE INTEGRATION azure_snowpipe_integration;
```

### Granting Access
- In the Snowflake description output, look for `AZURE_CONSENT_URL`.
- Copy the **AZURE_CONSENT_URL** and paste it into a new browser window where you are logged into your Azure account to grant authorization.
- After granting permission, you will receive an **Application ID**. Copy and save this ID for later use.

---

## Step 5: Granting Permissions in Azure
1. Go to **Storage Account** → **Your Container** → **Access Control (IAM)**.
2. Click on the **+ Add** icon → **Add role assignment**.
3. In the **Search Bar**, type **Storage Blob Data Reader** and select it.
4. Click **Next**.
5. Click on **+ Select Members**.
6. In the **Search Bar** on the right-side pop-up window, paste the **Application ID** copied from the Azure consent step.
7. Select the application and click **Continue**.

Permissions are now successfully granted to Snowflake.


---

## After Successful Connection: Setting up Queue and Notifications

To fully set up Snowpipe, you need to complete two additional steps:
- **Queue Setup**
- **Notifications**

---

## Step 6: Setting up Queue in Azure
1. Go to **Storage Account** → **Your Container** → **Data Storage** → **Queue**.
2. Click on the **+ Queue** icon.
3. **Queue Name**: Enter a unique name and click **OK**.

---

## Step 7: Configuring Event Notifications
1. In the left-hand side menu, look for **⚡Events** (⚡) and select it.
2. Click on **+ Event Subscription**.
3. Enter the following details:
    - **Name**: Unique name for the event
    - **Event Schema**: Leave as default
    - **System Topic**: Same as the name
    - **Filter Events Type**: Select **Only Blob Created**
    - **End Point Type**: Choose **Storage Queue**
4. Click **Configure End Point**.
5. In the right-side pop-up window:
    - **Storage Account**: Select your storage account
    - **Queue**: Select the queue previously created
6. Click **OK**.

**Note:** You may encounter an error, but don't worry. Continue with the following steps.

---

## Step 8: Registering Event Grid in Azure
1. Copy your Azure URL and paste it into a new browser window:  
   `https://portal.azure.com/?quickstart=true#home`
2. In the **Search Bar**, type **Subscription** and select your subscription.
3. In **Settings**, go to **Resource Providers**.
4. In the **Search Bar**, type **Event**.
5. Select **Microsoft.EventGrid**.
6. In the upper left-hand side, click **Register**.

Once registered, go back to the **Event Subscription** window, and the error should be resolved.

---

## Conclusion
You have now successfully:
- Created an Azure Storage Account and container for Snowpipe
- Uploaded data to the Azure container
- Configured Snowflake integration with Azure
- Granted necessary permissions in Azure

Your data is now ready to be ingested into Snowflake using Snowpipe.


