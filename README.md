# Snowflake Warehouse Cheat Sheet

## Overview
This guide will help you get started quickly with Snowflake Warehouse by providing a cheat sheet for basic operations. It includes instructions on creating a table, checking its contents, and loading data from an S3 bucket.

---

## Getting Started

### 1. Creating the Table (Metadata Definition)
To create the `LOAN_PAYMENT` table, copy and run the following SQL command:

```sql
CREATE TABLE "OUR_FIRST_DB"."PUBLIC"."LOAN_PAYMENT" (
  "Loan_ID" STRING,
  "loan_status" STRING,
  "Principal" STRING,
  "terms" STRING,
  "effective_date" STRING,
  "due_date" STRING,
  "paid_off_time" STRING,
  "past_due_days" STRING,
  "age" STRING,
  "education" STRING,
  "Gender" STRING
);
```

---

### 2. Checking if the Table is Empty
Before loading data, ensure the table exists and is empty by running:

```sql
USE DATABASE OUR_FIRST_DB;

SELECT * FROM LOAN_PAYMENT;
```

---

### 3. Loading Data from an S3 Bucket
To load data from an S3 bucket into the `LOAN_PAYMENT` table, execute the following command:

```sql
COPY INTO LOAN_PAYMENT
    FROM s3://bucketsnowflakes3/Loan_payments_data.csv
    file_format = (type = csv
                   field_delimiter = ','
                   skip_header=1);
```

---

### 4. Validating the Data Load
After loading the data, confirm that it was inserted successfully by running:

```sql
SELECT * FROM LOAN_PAYMENT;
```

---

## Notes
- Ensure you have the necessary permissions to create tables and load data.
- Replace `s3://bucketsnowflakes3/Loan_payments_data.csv` with the correct S3 path.
- Modify field types as necessary based on the actual data format.

This cheat sheet provides a quick start to working with Snowflake Warehouse. Happy querying! 🚀


