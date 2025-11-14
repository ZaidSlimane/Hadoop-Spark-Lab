# Sensor Analytics Project

## Overview
A comprehensive big data analytics project demonstrating both **batch processing** and **stream processing** capabilities using Apache Spark on a Hadoop cluster. The project processes sensor data to calculate temperature/humidity statistics and comfort indices.

---

## Project Structure

```
sensor-analytics/
├── batch-processing/
│   ├── data/
│   │   └── sensor_data.json
│   ├── src/
│   │   └── simple_batch.py
│   └── requirements.txt
└── stream-processing/
    ├── src/
    │   └── simple_stream.py
    └── requirements.txt
```

---

## Technology Stack

| Technology | Version | Purpose |
|------------|---------|---------|
| **Apache Spark** | 3.2.1 | Distributed data processing engine |
| **Apache Hadoop** | - | Distributed file storage (HDFS) |
| **Scala** | 2.12.15 | Spark interactive shell scripting |
| **Python** | - | PySpark application development |
| **Java** | 1.8.0_312 | JVM runtime environment |
| **YARN** | - | Cluster resource management |
| **Docker** | - | Container orchestration |

---

## Environment Setup

### System Resources
- **Memory:** 7750 MB RAM
- **Swap:** 2048 MB
- **Container:** hadoop-master
- **Network:** 172.18.0.5

### Services Running
- NameNode (port 9000)
- DataNode
- Secondary NameNode
- ResourceManager
- NodeManager
- Spark UI (port 4040)
- SSH Service

---

## Installation & Configuration

### 1. Access Hadoop Cluster
```bash
docker exec -it hadoop-master bash
```

### 2. Start Hadoop Services
```bash
./start-hadoop.sh
```

**Expected Output:**
```
Starting namenodes on [hadoop-master]
Starting datanodes
Starting secondary namenodes [hadoop-master]
Starting resourcemanager
Starting nodemanagers
```

### 3. Verify Spark Installation
```bash
cd /usr/local/spark
spark-shell --version
```

**Expected Output:**
```
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 3.2.1
      /_/
```

---

## Data Preparation

### Upload Sensor Data to HDFS
```bash
cd ~
hdfs dfs -put sensor_data.json /data
```

### Verify Upload
```bash
hdfs dfs -ls /data/
```

**Expected Output:**
```
Found 2 items
drwxr-xr-x   - root supergroup          0 2025-10-18 13:30 /data/bigdata_ilsi
-rw-r--r--   2 root supergroup         40 2025-11-08 12:11 /data/testing_spark.txt
```

---

## Batch Processing

### Description
Performs aggregation and statistical analysis on sensor data, calculating:
- Total record count
- Average temperature
- Average humidity
- Sensor with maximum temperature

### Execution
```bash
cd ~/sensor-analytics/batch-processing/src
/usr/local/spark/bin/spark-submit simple_batch.py
```

### Results
```
Total records: 5
Average temperature: 23.72°C
Average humidity: 61.0%
Sensor with max temp: A2
```

### Key Operations
- Read JSON data from HDFS
- Aggregate temperature and humidity values
- Calculate statistical metrics
- Identify maximum temperature sensor

---

## Stream Processing

### Description
Simulates real-time data processing by calculating a comfort index for each sensor reading based on temperature and humidity values.

**Comfort Index Formula:**
```
Comfort Index = Temperature - (0.55 - 0.0055 × Humidity) × (Temperature - 14.5)
```

### Execution
```bash
cd ~/sensor-analytics/stream-processing/src
$SPARK_HOME/bin/spark-submit simple_stream.py
```

### Results
```
Total records: 5
Comfort index per record:
('A1', 17.98)
('A2', 18.62)
('A3', 17.2)
('A1', 18.17)
('A2', 18.8)
```

### Key Operations
- Read sensor data from HDFS
- Apply comfort index calculation formula
- Process records individually (simulating stream)
- Output sensor ID with comfort value

---

## Interactive Spark Shell Demo

### WordCount Example

#### 1. Launch Spark Shell
```bash
spark-shell
```

#### 2. Execute WordCount
```scala
// Load text file
val lines = sc.textFile("/data/testing_spark.txt")

// Split into words
val words = lines.flatMap(_.split("\\s+"))

// Count word occurrences
val wc = words.map(w => (w, 1)).reduceByKey(_ + _)

// Save results
wc.saveAsTextFile("testing_spark.wordcount")
```

#### 3. Retrieve Results
```bash
hdfs dfs -get /user/root/testing_spark.wordcount
cat testing_spark.wordcount/part-00000
```

**Output:**
```
(Hello,2)
(,1)
(Wordcount!,1)
```

---

## System Monitoring

### Check Running Processes
```bash
ps -aux
```

### Monitor System Resources
```bash
top
```

**System Performance:**
```
Tasks:   5 total,   1 running,   4 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.9 sy,  0.0 ni, 99.1 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   7750.2 total,   6839.1 free,    609.9 used,    301.2 buff/cache
```

---

## HDFS Operations

### Common Commands

| Command | Description |
|---------|-------------|
| `hdfs dfs -put <local> <hdfs>` | Upload file to HDFS |
| `hdfs dfs -get <hdfs> <local>` | Download file from HDFS |
| `hdfs dfs -ls <path>` | List directory contents |
| `hdfs dfs -cat <file>` | Display file contents |
| `hdfs dfs -rm <file>` | Delete file |

---

## Troubleshooting

### Issue: HDFS Connection Refused
**Error:**
```
put: Call From hadoop-master/172.18.0.5 to hadoop-master:9000 failed on connection exception: java.net.ConnectException: Connection refused
```

**Solution:**
```bash
./start-hadoop.sh
```

### Issue: File Already Exists
**Error:**
```
get: `testing_spark.wordcount/_SUCCESS': File exists
```

**Solution:**
```bash
rm -rf testing_spark.wordcount
hdfs dfs -get /user/root/testing_spark.wordcount
```

### Issue: Hadoop Services Already Running
**Message:**
```
namenode is running as process 1064. Stop it first
```

**Action:** Services are already active, no action needed.

---

## Performance Metrics

### Batch Processing Job
- **Application Name:** batch_processing
- **Spark Context:** Initialized successfully
- **Memory Allocated:** 2004.6 MiB
- **Execution Time:** ~2-3 seconds
- **Records Processed:** 5

### Stream Processing Job
- **Application Name:** batch_processing (streaming simulation)
- **Memory Allocated:** 2004.6 MiB
- **Execution Time:** ~5-6 seconds
- **Records Processed:** 5
- **Comfort Index Calculations:** 5

---

## Key Achievements

✅ **Environment Setup**
- Successfully configured Hadoop/Spark cluster
- Docker container deployment
- Service orchestration

✅ **HDFS Operations**
- File upload and retrieval
- Directory management
- Data persistence

✅ **Batch Processing**
- Statistical aggregations
- Temperature/humidity analysis
- Maximum value identification

✅ **Stream Processing**
- Real-time simulation
- Custom formula implementation
- Per-record processing

✅ **Interactive Shell**
- Scala-based WordCount
- RDD transformations
- Result verification

---

## Web Interfaces

| Service | URL | Description |
|---------|-----|-------------|
| **Spark UI** | http://hadoop-master:4040 | Job monitoring and execution details |
| **NameNode** | http://hadoop-master:9870 | HDFS status and file browser |
| **ResourceManager** | http://hadoop-master:8088 | YARN application tracking |

---

## Future Enhancements

- [ ] Implement real-time streaming with Apache Kafka
- [ ] Add data visualization dashboard
- [ ] Integrate machine learning models for anomaly detection
- [ ] Implement time-series analysis
- [ ] Add unit tests for Spark jobs
- [ ] Create automated deployment scripts
- [ ] Add data quality validation
- [ ] Implement alerting system for threshold violations

---

## License

This project is for educational and demonstration purposes.

---

## Contact & Support

For questions or issues, please refer to the official documentation:
- [Apache Spark Documentation](https://spark.apache.org/docs/3.2.1/)
- [Apache Hadoop Documentation](https://hadoop.apache.org/docs/)
- [PySpark API Reference](https://spark.apache.org/docs/3.2.1/api/python/)

---

**Last Updated:** November 8, 2025  
**Project Status:** ✅ Completed and Tested
