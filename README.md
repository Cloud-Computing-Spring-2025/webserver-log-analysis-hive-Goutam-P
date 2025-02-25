# Hive Web Server Log Analysis

## Project Overview
This project analyzes web server logs using Apache Hive to extract meaningful insights about website traffic patterns. The dataset consists of CSV-formatted logs with fields such as IP address, timestamp, URL, HTTP status, and user agent.

## Implementation Approach
- Create a Hive external table to store web server logs.
- Load CSV data into the Hive table.
- Execute HiveQL queries to analyze the dataset.
- Implement partitioning for optimized query performance.
- Extract results and export them for reporting.

## Execution Steps
### 1. Setting Up Hive Environment
```bash
# Start Docker containers (if applicable)
docker-compose up -d

# Access Hive CLI
docker exec -it hive-server /bin/bash
hive
```

### 2. Creating Hive Database and External Table
```sql
CREATE DATABASE IF NOT EXISTS web_logs;
USE web_logs;

CREATE EXTERNAL TABLE IF NOT EXISTS server_logs (
    ip STRING,
    timestamp STRING,
    url STRING,
    status INT,
    user_agent STRING
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE;
```

### 3. Loading Data into Hive Table
```bash
hdfs dfs -mkdir -p /user/hive/logs
hdfs dfs -put web_server_logs.csv /user/hive/logs/

LOAD DATA INPATH '/user/hive/logs/web_server_logs.csv' INTO TABLE server_logs;
```

### 4. Running Analysis Queries
#### a) Count Total Web Requests
```sql
SELECT COUNT(*) AS total_requests FROM server_logs;
```

#### b) Analyze Status Codes
```sql
SELECT status, COUNT(*) AS frequency FROM server_logs GROUP BY status;
```

#### c) Identify Most Visited Pages
```sql
SELECT url, COUNT(*) AS visit_count FROM server_logs GROUP BY url ORDER BY visit_count DESC LIMIT 3;
```

#### d) Traffic Source Analysis
```sql
SELECT user_agent, COUNT(*) AS count FROM server_logs GROUP BY user_agent ORDER BY count DESC;
```

#### e) Detect Suspicious Activity (IPs with >3 failed requests)
```sql
SELECT ip, COUNT(*) AS failed_requests FROM server_logs WHERE status IN (404, 500) GROUP BY ip HAVING COUNT(*) > 3;
```

#### f) Traffic Trends Over Time (Requests per Minute)
```sql
SELECT SUBSTR(timestamp, 0, 16) AS minute, COUNT(*) AS request_count FROM server_logs GROUP BY minute ORDER BY minute;
```

### 5. Implementing Partitioning for Performance Optimization
```sql
CREATE TABLE partitioned_logs (
    ip STRING,
    timestamp STRING,
    url STRING,
    user_agent STRING
)
PARTITIONED BY (status INT)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE;

SET hive.exec.dynamic.partition = true;
SET hive.exec.dynamic.partition.mode = nonstrict;

INSERT OVERWRITE TABLE partitioned_logs PARTITION (status) 
SELECT ip, timestamp, url, user_agent, status FROM server_logs;
```

### 6. Exporting Output to Local Machine
```bash
hdfs dfs -mkdir -p /user/hive/output
hdfs dfs -cp /user/hive/logs /user/hive/output/

# Copy data from HDFS to local system
docker exec -it namenode hdfs dfs -get /user/hive/output ~/hive_outputs/

# Verify output in local directory
ls ~/hive_outputs/
```

## Challenges Faced
- Data formatting issues were handled by ensuring CSV file consistency.
- Partitioning required enabling dynamic partitions in Hive.
- Exporting results from HDFS to local required correct path references.

## Sample Input and Output
**Sample Input (CSV file):**
```
192.168.1.1,2024-02-01 10:15:00,/home,200,Mozilla/5.0
192.168.1.2,2024-02-01 10:16:00,/products,200,Chrome/90.0
192.168.1.3,2024-02-01 10:17:00,/checkout,404,Safari/13.1
```

**Sample Output:**
```
Total Requests: 100
Status Code Analysis:
200: 80
404: 10
500: 10
Most Visited Pages:
/home: 50
/products: 30
/checkout: 20
```

    

