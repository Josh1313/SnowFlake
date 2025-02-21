# Performance Optimization in Snowflake

## Key Points to Consider
- **Making Queries Run Faster:** Optimize query performance to reduce execution time.
- **Save Costs:** Efficient resource usage to minimize cost.
- **Automatically Managed Micro-Partitions:** Leverage Snowflake's automatic partitioning for efficient data storage and retrieval.
- **Assigning Appropriate Data Types:** Use the most suitable data types to optimize storage and performance.
- **Sizing Virtual Warehouses:** Right-size virtual warehouses to balance cost and performance.
- **Cluster Keys:** Use cluster keys for better query performance on large tables.

---

## Our Main Objective
Understand the needs of the company and create dedicated virtual warehouses for each group, separating them according to different workloads to optimize performance and cost efficiency.

![alt text](image-1.png)

### Scaling Strategies
- **Scaling Up:** For known patterns of high workloads, scale up the warehouse size to improve performance.
- **Scaling Out (Multi-Clusters):** Dynamically scale out to handle unpredictable workload patterns.

### Maximize Cache Usage
- **Automatic Caching:** Maximize Snowflake's automatic caching to improve query speed.

### Cluster Keys
- Use cluster keys for efficient querying on large tables.

### Dedicated Warehouses
- **Right Size & Type:** Choose the appropriate size and type for each group's workload.
- **Avoid Underutilization:** Correctly classify company needs to prevent underutilization and unnecessary costs.

### Refine Classification
Work patterns may change over time; revisit and refine classifications regularly.

---

## Snowflake SQL Commands
### Create Virtual Warehouses
We specified the creation command, then the name of the warehouse, and last, we define the properties of the warehouse
#### Data Scientists
```sql
CREATE WAREHOUSE DS_WH
  WITH 
  WAREHOUSE_SIZE = 'SMALL'
  WAREHOUSE_TYPE = 'STANDARD'
  AUTO_SUSPEND = 300
  AUTO_RESUME = TRUE
  MIN_CLUSTER_COUNT = 1
  MAX_CLUSTER_COUNT = 1
  SCALING_POLICY = 'STANDARD';
```

#### DBA
```sql
CREATE WAREHOUSE DBA_WH
  WITH 
  WAREHOUSE_SIZE = 'XSMALL'
  WAREHOUSE_TYPE = 'STANDARD'
  AUTO_SUSPEND = 300
  AUTO_RESUME = TRUE
  MIN_CLUSTER_COUNT = 1
  MAX_CLUSTER_COUNT = 1
  SCALING_POLICY = 'STANDARD';
```

### Create Roles and Grant Access
```sql
CREATE ROLE DATA_SCIENTIST;
GRANT USAGE ON WAREHOUSE DS_WH TO ROLE DATA_SCIENTIST;

CREATE ROLE DBA;
GRANT USAGE ON WAREHOUSE DBA_WH TO ROLE DBA;
```

### Setting Up Users
#### Data Scientists
```sql
CREATE USER DS1 PASSWORD = 'DS1' LOGIN_NAME = 'DS1' DEFAULT_ROLE='DATA_SCIENTIST' DEFAULT_WAREHOUSE = 'DS_WH'  MUST_CHANGE_PASSWORD = FALSE;
CREATE USER DS2 PASSWORD = 'DS2' LOGIN_NAME = 'DS2' DEFAULT_ROLE='DATA_SCIENTIST' DEFAULT_WAREHOUSE = 'DS_WH'  MUST_CHANGE_PASSWORD = FALSE;
CREATE USER DS3 PASSWORD = 'DS3' LOGIN_NAME = 'DS3' DEFAULT_ROLE='DATA_SCIENTIST' DEFAULT_WAREHOUSE = 'DS_WH'  MUST_CHANGE_PASSWORD = FALSE;

GRANT ROLE DATA_SCIENTIST TO USER DS1;
GRANT ROLE DATA_SCIENTIST TO USER DS2;
GRANT ROLE DATA_SCIENTIST TO USER DS3;
```

#### DBAs
```sql
CREATE USER DBA1 PASSWORD = 'DBA1' LOGIN_NAME = 'DBA1' DEFAULT_ROLE='DBA' DEFAULT_WAREHOUSE = 'DBA_WH'  MUST_CHANGE_PASSWORD = FALSE;
CREATE USER DBA2 PASSWORD = 'DBA2' LOGIN_NAME = 'DBA2' DEFAULT_ROLE='DBA' DEFAULT_WAREHOUSE = 'DBA_WH'  MUST_CHANGE_PASSWORD = FALSE;

GRANT ROLE DBA TO USER DBA1;
GRANT ROLE DBA TO USER DBA2;
```

### Drop Objects
```sql
DROP USER DBA1;
DROP USER DBA2;

DROP USER DS1;
DROP USER DS2;
DROP USER DS3;

DROP ROLE DATA_SCIENTIST;
DROP ROLE DBA;

DROP WAREHOUSE DS_WH;
DROP WAREHOUSE DBA_WH;
```

---
## Scaling up
*- Increasing the size of a virtual warehouses
*- More Complex Query
Example how to alter the warehouse properties

```sql
ALTER WAREHOUSE COMPUTE_WH SUSPEND;

ALTER WAREHOUSE COMPUTE_WH RESUME;


ALTER WAREHOUSE COMPUTE_WH
    SET 
        WAREHOUSE_SIZE = 'SMALL',
        MIN_CLUSTER_COUNT = 1,
        MAX_CLUSTER_COUNT = 1,
        SCALING_POLICY = 'STANDARD',
        AUTO_SUSPEND = 600,
        AUTO_RESUME = TRUE;
```

## Increase the size of the warehouse
```sql
ALTER WAREHOUSE COMPUTE_WH
SET WAREHOUSE_SIZE = 'SMALL'

```
# OR use GUI
admin --> warehouses --> select the three dots next to the ware in question and edit

## Scaling out
*- Using addition warehouses / Multicluster warehouses
*- More Concurrent Users


## Notes
- Copy and paste the above commands directly into Snowflake's query editor.
- Adjust warehouse sizes and scaling policies based on your organization's requirements.



