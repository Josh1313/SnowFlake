# Snowflake Git Integration

## Overview
This guide provides step-by-step instructions for integrating a GitHub repository with Snowflake using API integration and secrets. By following these steps, you can securely connect your GitHub repository to Snowflake, fetch updates, and manage repository files directly within Snowflake.

## Setup Instructions

### 1. Create Secrets
```sql
CREATE OR REPLACE SECRET git_secret_integration
 TYPE=password
 USERNAME= your_gith_username
 PASSWORD= "yoursecrets";
```

### 2. Create API Integration
```sql
CREATE OR REPLACE API INTEGRATION git_api_integration
  API_PROVIDER = git_https_api
  API_ALLOWED_PREFIXES= ("https://github.com/yourusername")
  ALLOWED_AUTHENTICATION_SECRETS = (git_secret_integration)
  ENABLED= TRUE;
```

### 3. Create Git Repository in Snowflake
```sql
CREATE OR REPLACE git repository SnowFlake
    api_integration = git_api_integration
    git_credentials = git_secret_integration
    origin = 'https://github.com/Josh1313/repositoryname';
```

### 4. Show Repositories and Branches
```sql
-- Show repos added to Snowflake.
SHOW GIT REPOSITORIES;

-- Show branches in the repository.
SHOW GIT BRANCHES IN GIT REPOSITORY SnowFlake;
```

### 5. List and View Files
```sql
-- List files in the main branch.
LS @SnowFlake/branches/main;

-- Show code in README.md file.
SELECT $1 FROM @SnowFlake/branches/main/README.md;
```

### 6. Fetch Repository Updates
```sql
ALTER GIT REPOSITORY SnowFlake FETCH;
```

### 7. Store README.md Content in a Table
```sql
CREATE TABLE MANAGE_DB.PUBLIC.README_CONTENT (
    FILE_NAME STRING,
    CONTENT STRING
);

INSERT INTO MANAGE_DB.PUBLIC.README_CONTENT (FILE_NAME, CONTENT)
SELECT 'README.md', $1
FROM @SnowFlake/branches/main/README.md;
```

### 8. Retrieve Stored Content
```sql
SELECT * FROM MANAGE_DB.PUBLIC.README_CONTENT;
```

## Conclusion
By following this guide, you have successfully integrated a GitHub repository with Snowflake. You can now fetch updates, browse files, and manage repository content directly within Snowflake. This integration enables efficient version control and seamless collaboration for Snowflake-based development workflows.

