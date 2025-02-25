# Web Server Log Analysis with Apache Hive

## Project Overview
This project analyzes web server logs using Apache Hive to extract meaningful insights about website traffic patterns. The analysis includes total web requests, HTTP status code frequencies, most visited pages, traffic sources, suspicious activity detection, and traffic trends over time.

## Implementation Approach
1. **Setup Hive Environment**: Create a Hive database and external table.
2. **Load Data**: Upload the CSV log data to HDFS and load it into Hive.
3. **Execute Queries**: Perform analytical tasks using HiveQL.
4. **Implement Partitioning**: Optimize queries using partitioning.
5. **Export Output**: Save results to HDFS and copy them to the local machine.

## Execution Steps

### 1. Start Hadoop and Hive Services
```bash
start-dfs.sh
start-yarn.sh
hive --service metastore &
hive --service hiveserver2 &
```

### 2. Create Database and Table in Hive
```sql
CREATE DATABASE IF NOT EXISTS web_logs;
USE web_logs;

CREATE EXTERNAL TABLE IF NOT EXISTS web_server_logs (
    ip STRING,
    timestamp STRING,
    url STRING,
    status INT,
    user_agent STRING
) ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION '/user/hive/logs/';
```

### 3. Load Data into Hive
```bash
hdfs dfs -mkdir -p /user/hive/logs/
hdfs dfs -put web_server_logs.csv /user/hive/logs/
MSCK REPAIR TABLE web_server_logs;
```

### 4. Run Analysis Queries

#### 4.1 Count Total Web Requests
```sql
SELECT COUNT(*) AS total_requests FROM web_server_logs;
```

#### 4.2 Analyze Status Codes
```sql
SELECT status, COUNT(*) AS count FROM web_server_logs GROUP BY status;
```

#### 4.3 Identify Most Visited Pages
```sql
SELECT url, COUNT(*) AS visits FROM web_server_logs
GROUP BY url ORDER BY visits DESC LIMIT 3;
```

#### 4.4 Traffic Source Analysis
```sql
SELECT user_agent, COUNT(*) AS count FROM web_server_logs
GROUP BY user_agent ORDER BY count DESC;
```

#### 4.5 Detect Suspicious Activity (IP with >3 failed requests)
```sql
SELECT ip, COUNT(*) AS failed_requests FROM web_server_logs
WHERE status IN (404, 500)
GROUP BY ip HAVING failed_requests > 3;
```

#### 4.6 Analyze Traffic Trends (Requests per Minute)
```sql
SELECT SUBSTRING(timestamp, 1, 16) AS minute, COUNT(*) AS requests
FROM web_server_logs GROUP BY minute ORDER BY minute;
```

### 5. Implement Partitioning by Status Code
```sql
CREATE TABLE web_server_logs_partitioned (
    ip STRING,
    timestamp STRING,
    url STRING,
    user_agent STRING
) PARTITIONED BY (status INT)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
STORED AS TEXTFILE;

INSERT INTO TABLE web_server_logs_partitioned PARTITION (status)
SELECT ip, timestamp, url, user_agent, status FROM web_server_logs;
```

### 6. Export Query Results to HDFS
```sql
INSERT OVERWRITE DIRECTORY '/user/hive/output/total_requests'
SELECT COUNT(*) FROM web_server_logs;

INSERT OVERWRITE DIRECTORY '/user/hive/output/status_code_analysis'
SELECT status, COUNT(*) FROM web_server_logs GROUP BY status;
```

### 7. Copy Output to Local Machine
```bash
hdfs dfs -get /user/hive/output ~/hive_outputs/
```

## Challenges Faced
- **Data Formatting Issues**: Ensured CSV was correctly formatted and field delimiters matched.
- **HDFS Permission Errors**: Used `hdfs dfs -chmod` to set appropriate permissions.
- **Query Performance**: Used partitioning to optimize execution time.

## Sample Input (CSV Format)
```csv
ip,timestamp,url,status,user_agent
192.168.1.1,2024-02-01 10:15:00,/home,200,Mozilla/5.0
192.168.1.2,2024-02-01 10:16:00,/products,200,Chrome/90.0
192.168.1.3,2024-02-01 10:17:00,/checkout,404,Safari/13.1
```

## Sample Output
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

## Submission
Push your final code and results to GitHub Classroom and submit the repository link.

