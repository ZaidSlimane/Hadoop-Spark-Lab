# Lab 4: Apache Spark - Batch and Stream Processing

## Lab Information
**Course:** M2 ILSI - Big Data 2025–2026  
**University:** Constantine 2 - Abdelhamid Mehri  
**Faculty:** NTIC Faculty - TLSI Department  
**Duration:** 1 session  
**Instructor:** I. BOULEGHLIMAT

---

## Lab Objectives

✅ Load input data into HDFS  
✅ Gain practical experience with Spark programming  
✅ Work with structured data in batch mode  
✅ Handle data ingestion in real-time  
✅ Understand micro-batch processing  

---

## Overview

This lab demonstrates both **batch processing** for analyzing historical data and **stream processing** for handling real-time data flows using Apache Spark. The project processes sensor data to calculate temperature/humidity statistics and comfort indices.

---

## Technology Stack

| Technology | Version | Purpose |
|------------|---------|---------|
| **Apache Spark** | 3.2.1 | Distributed data processing engine |
| **Apache Hadoop** | 3.3.2 | Distributed file storage (HDFS) |
| **Scala** | 2.12.15 | Spark interactive shell scripting |
| **Python** | 3.x | PySpark application development |
| **Java** | 1.8.0_312 | JVM runtime environment |
| **YARN** | - | Cluster resource management |
| **Docker** | - | Container orchestration |
| **Netcat (nc)** | - | TCP/IP networking utility for streaming |

---

## Environment Setup

### System Resources
- **Memory:** 7750 MB RAM
- **Swap:** 2048 MB
- **Container:** hadoop-master (172.18.0.5)
- **Network:** Docker network

### Services Running
- NameNode (port 9000)
- DataNode
- Secondary NameNode
- ResourceManager
- NodeManager
- Spark UI (port 4040)
- SSH Service

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

## Part 1: Environment Preparation

### Step 1: Access Hadoop Cluster

```bash
docker exec -it hadoop-master bash
```

**Output:**
```
root@hadoop-master:~#
```

### Step 2: Start Hadoop Services

```bash
./start-hadoop.sh
```

**Expected Output:**
```
Starting namenodes on [hadoop-master]
hadoop-master: Warning: Permanently added 'hadoop-master,172.18.0.5' (ECDSA) to the list of known hosts.
Starting datanodes
Starting secondary namenodes [hadoop-master]
Starting resourcemanager
Starting nodemanagers
```

**Note:** If services are already running, you'll see:
```
namenode is running as process 1064. Stop it first
```
This means services are active - no action needed.

### Step 3: Verify Spark Installation

```bash
cd /usr/local/spark
spark-shell --version
```

**Output:**
```
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 3.2.1
      /_/

Using Scala version 2.12.15, OpenJDK 64-Bit Server VM, 1.8.0_312
Branch HEAD
Compiled by user hgao on 2022-01-20T19:26:14Z
```

### Step 4: Verify System Processes

```bash
ps -aux
```

**Output:**
```
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0   2616  1536 pts/0    Ss   11:37   0:00 sh -c service ssh start; bash
root        22  0.0  0.0  12180  3476 ?        Ss   11:37   0:00 sshd
root        24  0.0  0.0   6000  3712 pts/0    S+   11:37   0:00 bash
root        27  0.0  0.0   6000  3712 pts/1    Ss   11:39   0:00 bash
```

### Step 5: Check System Resources

```bash
top
```

**System Performance:**
```
Tasks:   5 total,   1 running,   4 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.9 sy,  0.0 ni, 99.1 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   7750.2 total,   6839.1 free,    609.9 used,    301.2 buff/cache
MiB Swap:   2048.0 total,   2048.0 free,      0.0 used.   6989.4 avail Mem
```

---

## Part 2: Spark Shell WordCount Demo

### Step 1: Create Test File

```bash
cd ~
nano testing_spark.txt
```

**Content:**
```
Hello Spark Wordcount ! Hello Hadoop :)
```

### Step 2: Upload File to HDFS

```bash
hdfs dfs -put testing_spark.txt /data
```

**Troubleshooting:** If you get "Connection refused" error:
```bash
./start-hadoop.sh
hdfs dfs -put testing_spark.txt /data
```

### Step 3: Verify Upload

```bash
hdfs dfs -ls /data/
```

**Output:**
```
Found 2 items
drwxr-xr-x   - root supergroup          0 2025-10-18 13:30 /data/bigdata_ilsi
-rw-r--r--   2 root supergroup         40 2025-11-08 12:11 /data/testing_spark.txt
```

### Step 4: Launch Spark Shell

```bash
spark-shell
```

**Output:**
```
Setting default log level to "WARN".
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
2025-11-08 12:20:43,605 WARN yarn.Client: Neither spark.yarn.jars nor spark.yarn.archive is set
Spark context Web UI available at http://hadoop-master:4040
Spark context available as 'sc' (master = yarn, app id = application_1762600272554_0002).
Spark session available as 'spark'.

Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 3.2.1
      /_/

Using Scala version 2.12.15 (OpenJDK 64-Bit Server VM, Java 1.8.0_312)
Type in expressions to have them evaluated.
Type :help for more information.

scala>
```

### Step 5: Execute WordCount Operations

```scala
val lines = sc.textFile("/data/testing_spark.txt")
val words = lines.flatMap(_.split("\\s+"))
val wc = words.map(w => (w, 1)).reduceByKey(_ + _)
wc.saveAsTextFile("testing_spark.wordcount")
```

**Output:**
```scala
scala> val lines = sc.textFile("/data/testing_spark.txt")
lines: org.apache.spark.rdd.RDD[String] = /data/testing_spark.txt MapPartitionsRDD[1] at textFile at <console>:23

scala> val words = lines.flatMap(_.split("\\s+"))
words: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[2] at flatMap at <console>:23

scala> val wc = words.map(w => (w, 1)).reduceByKey(_ + _)
wc: org.apache.spark.rdd.RDD[(String, Int)] = ShuffledRDD[4] at reduceByKey at <console>:23

scala> wc.saveAsTextFile("testing_spark.wordcount")
```

### Step 6: Exit Spark Shell

Press `Ctrl-C` to exit.

### Step 7: Retrieve Results from HDFS

```bash
hdfs dfs -get /user/root/testing_spark.wordcount
```

**Note:** If files already exist locally:
```bash
rm -rf testing_spark.wordcount
hdfs dfs -get /user/root/testing_spark.wordcount
```

### Step 8: View Results

```bash
cat testing_spark.wordcount/part-00000
```

**Output:**
```
(Hello,2)
(,1)
(Wordcount!,1)
```

```bash
cat testing_spark.wordcount/part-00001
```

**Output:**
```
(Spark,1)
(:),1)
(Hadoop,1)
```

### Understanding the Results

| Word | Count | Explanation |
|------|-------|-------------|
| Hello | 2 | Appears twice in the text |
| Spark | 1 | Appears once |
| Hadoop | 1 | Appears once |
| Wordcount! | 1 | Treated as one token with punctuation |
| :) | 1 | Emoticon treated as token |

---

## Part 3: Spark Batch Processing - Sensor Data Analytics

### Step 1: Create Project Structure

```bash
cd ~
mkdir -p sensor-analytics/{batch-processing/{data,src},stream-processing/src}
```

**Directory Structure Created:**
```
sensor-analytics/
├── batch-processing/
│   ├── data/
│   └── src/
└── stream-processing/
    └── src/
```

### Step 2: Create Requirements File

```bash
cd ~/sensor-analytics/batch-processing
nano requirements.txt
```

**Content:**
```
pyspark==3.5.0
```

**Note:** The lab environment uses Spark 3.2.1, so this is for reference. The installed version works fine.

### Step 3: Create Sensor Data

```bash
cd ~
nano sensor_data.json
```

**Content:**
```json
{"sensor_id": "A1", "temperature": 22.5, "humidity": 65, "timestamp": "2024-01-15T08:00:00"}
{"sensor_id": "A2", "temperature": 24.8, "humidity": 58, "timestamp": "2024-01-15T08:00:00"}
{"sensor_id": "A3", "temperature": 21.2, "humidity": 70, "timestamp": "2024-01-15T08:00:00"}
{"sensor_id": "A1", "temperature": 23.1, "humidity": 63, "timestamp": "2024-01-15T09:00:00"}
{"sensor_id": "A2", "temperature": 25.3, "humidity": 55, "timestamp": "2024-01-15T09:00:00"}
```

### Step 4: Upload Sensor Data to HDFS

```bash
hdfs dfs -put sensor_data.json /data
```

### Step 5: Create Batch Processing Script

```bash
cd ~/sensor-analytics/batch-processing/src
nano simple_batch.py
```

**Content:**
```python
from pyspark.sql import SparkSession

# Initialize Spark Session
spark = SparkSession.builder \
    .appName("batch_processing") \
    .master("local[*]") \
    .getOrCreate()

# Read sensor data from HDFS
data = spark.read.json("hdfs://hadoop-master:9000/data/sensor_data.json")

print(f"Total records: {data.count()}")

# Convert to RDD for processing
rdd = data.rdd

# Compute comfort_index = temperature - (humidity * 0.1)
transformed_rdd = rdd.map(lambda row: (
    row.sensor_id,
    round(row.temperature - (row.humidity * 0.1), 2)
))

# Show results
print("Comfort index per record:")
for record in transformed_rdd.collect():
    print(record)

# Stop Spark session
spark.stop()
```

### Step 6: Execute Batch Processing Job

```bash
cd ~/sensor-analytics/batch-processing/src
/usr/local/spark/bin/spark-submit simple_batch.py
```

### Step 7: Batch Processing Output

**Execution Log (Key Parts):**
```
2025-11-08 12:49:51,187 INFO spark.SparkContext: Running Spark version 3.2.1
2025-11-08 12:49:51,481 INFO spark.SparkContext: Submitted application: batch_processing
2025-11-08 12:49:52,799 INFO util.Utils: Successfully started service 'SparkUI' on port 4040.
2025-11-08 12:49:53,177 INFO storage.BlockManager: Initialized BlockManager
```

**Final Results:**
```
Total records: 5

Comfort index per record:
('A1', 16.0)
('A2', 19.0)
('A3', 14.2)
('A1', 16.8)
('A2', 19.8)
```

### Understanding Batch Results

| Sensor ID | Temperature | Humidity | Formula | Comfort Index |
|-----------|-------------|----------|---------|---------------|
| A1 | 22.5 | 65 | 22.5 - (65 * 0.1) | 16.0 |
| A2 | 24.8 | 58 | 24.8 - (58 * 0.1) | 19.0 |
| A3 | 21.2 | 70 | 21.2 - (70 * 0.1) | 14.2 |
| A1 | 23.1 | 63 | 23.1 - (63 * 0.1) | 16.8 |
| A2 | 25.3 | 55 | 25.3 - (55 * 0.1) | 19.8 |

**Comfort Index Interpretation:**
- **Higher values:** More comfortable (warmer, less humid)
- **Lower values:** Less comfortable (cooler or more humid)
- **Sensor A2:** Most comfortable (19.0-19.8)
- **Sensor A3:** Least comfortable (14.2)

---

## Part 4: Spark Structured Streaming

### Step 1: Create Stream Processing Script

```bash
cd ~/sensor-analytics/stream-processing/src
nano simple_stream.py
```

**Content:**
```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import split, col

# Initialize Spark Session
spark = SparkSession.builder \
    .appName("stream_processing") \
    .master("local[*]") \
    .getOrCreate()

# Read streaming data from socket
lines = spark.readStream \
    .format("socket") \
    .option("host", "localhost") \
    .option("port", 9999) \
    .load()

# Parse temperature data (format: "temp:20.5")
temperatures = lines.select(
    split(col("value"), ":").getItem(1).cast("double").alias("temperature")
)

# Write to console
query = temperatures.writeStream \
    .outputMode("append") \
    .format("console") \
    .start()

query.awaitTermination()
```

### Step 2: Start Netcat Server (Terminal 1)

Open a **PowerShell terminal** on your Windows host:

```powershell
# Note: nc might not be available on Windows by default
# You may need to use WSL or install netcat
nc -lk 9999
```

**Alternative for Windows (if nc not available):**
```powershell
# Use ncat from nmap package or run nc inside Docker container
docker exec -it hadoop-master bash
nc -lk 9999
```

### Step 3: Start Spark Streaming Application (Terminal 2)

Open another terminal and connect to the container:

```bash
docker exec -it hadoop-master bash
cd ~/sensor-analytics/stream-processing/src
$SPARK_HOME/bin/spark-submit simple_stream.py
```

### Step 4: Send Streaming Data (Terminal 1)

Type in the Netcat terminal:
```
temp:20.5
temp:23.0
temp:18.7
temp:25.2
```

### Step 5: Stream Processing Output (Terminal 2)

**Expected Output:**
```
-------------------------------------------
Batch: 0
-------------------------------------------
+-----------+
|temperature|
+-----------+
|       20.5|
+-----------+

-------------------------------------------
Batch: 1
-------------------------------------------
+-----------+
|temperature|
+-----------+
|       23.0|
+-----------+

-------------------------------------------
Batch: 2
-------------------------------------------
+-----------+
|temperature|
+-----------+
|       18.7|
+-----------+
```

### Execution Log (Key Parts):

```
2025-11-08 13:02:59,678 INFO spark.SparkContext: Running Spark version 3.2.1
2025-11-08 13:03:00,011 INFO spark.SparkContext: Submitted application: stream_processing
2025-11-08 13:03:01,088 INFO util.Utils: Successfully started service 'SparkUI' on port 4040
2025-11-08 13:03:06,603 INFO spark.SparkContext: Starting job
```

---

## Part 5: Performance Analysis

### Batch Processing Metrics

```
Application Name: batch_processing
Spark Context: Initialized successfully
Memory Allocated: 2004.6 MiB
Execution Time: ~2-3 seconds
Records Processed: 5
Processing Mode: Complete dataset analysis
```

### Stream Processing Metrics

```
Application Name: stream_processing
Memory Allocated: 2004.6 MiB
Execution Time: Continuous (until terminated)
Micro-batches: Processed as data arrives
Processing Mode: Real-time incremental
Latency: Sub-second per micro-batch
```

---

## Lab Questions and Answers

### Question 1: Key Differences Between Batch and Stream Processing

| Aspect | Batch Processing | Stream Processing |
|--------|------------------|-------------------|
| **Data Source** | Static files in HDFS | Real-time socket/Kafka streams |
| **Processing Model** | Complete dataset at once | Micro-batches as data arrives |
| **Latency** | Minutes to hours | Sub-second to seconds |
| **Use Case** | Historical analysis | Real-time monitoring |
| **APIs** | spark.read.json() | spark.readStream.format() |
| **Output** | Saved to file/database | Console/streaming sink |
| **Termination** | Completes after processing | Runs continuously |
| **State Management** | Not required | Window/watermark needed |

**Key Architectural Differences:**

1. **Batch Processing:**
   - Reads entire dataset into memory
   - Applies transformations on complete data
   - Produces final result in one execution
   - Example: Daily sales reports, monthly aggregations

2. **Stream Processing:**
   - Processes data in small increments (micro-batches)
   - Maintains state across batches if needed
   - Continuously produces results as data arrives
   - Example: Real-time fraud detection, live dashboards

### Question 2: Handling Real Sensor Data

**To extend this lab for real sensor data:**

#### Option 1: MQTT Broker Integration

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import from_json, col
from pyspark.sql.types import StructType, StructField, StringType, DoubleType

# Define schema
schema = StructType([
    StructField("sensor_id", StringType()),
    StructField("temperature", DoubleType()),
    StructField("humidity", DoubleType()),
    StructField("timestamp", StringType())
])

# Read from MQTT (requires spark-streaming-mqtt connector)
df = spark.readStream \
    .format("mqtt") \
    .option("host", "mqtt.broker.com") \
    .option("topic", "sensors/temperature") \
    .load()

# Parse JSON payload
parsed = df.select(from_json(col("value"), schema).alias("data")).select("data.*")
```

#### Option 2: Apache Kafka Integration

```python
# Read from Kafka topic
df = spark.readStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "sensor-data") \
    .load()

# Parse Kafka message
sensor_data = df.selectExpr("CAST(value AS STRING)") \
    .select(from_json(col("value"), schema).alias("data")) \
    .select("data.*")
```

#### Option 3: File-Based Streaming (CSV/JSON)

```python
# Read from directory (monitors for new files)
df = spark.readStream \
    .format("json") \
    .schema(schema) \
    .option("path", "/data/sensor-stream/") \
    .load()
```

#### Option 4: REST API Polling

```python
# Custom micro-batch receiver
from pyspark.sql.streaming import StreamingQuery
import requests

def fetch_sensor_data():
    response = requests.get("https://api.sensors.com/latest")
    return response.json()

# Use foreachBatch for custom processing
def process_batch(df, epoch_id):
    # Fetch from API
    data = fetch_sensor_data()
    # Process with Spark
    df.write.format("parquet").mode("append").save("/output/")
```

**Required Modifications:**

1. **Hardware Integration:**
   - Install sensor drivers/SDKs
   - Configure communication protocols (I2C, SPI, UART)
   - Set up edge gateway for data collection

2. **Data Pipeline:**
   - Message broker (Kafka/MQTT) for buffering
   - Schema registry for data validation
   - Error handling for sensor failures

3. **Processing Enhancements:**
   - Windowing for time-based aggregations
   - Watermarking for late-arriving data
   - Stateful operations for anomaly detection

4. **Storage:**
   - Time-series database (InfluxDB, TimescaleDB)
   - Data lake for historical analysis
   - Caching layer for real-time queries

---

## Spark Core Concepts

### RDD (Resilient Distributed Dataset)

**Definition:** Core abstraction in Spark representing a collection of elements partitioned across cluster nodes.

**Key Properties:**
- **Immutable:** Once created, cannot be changed
- **Distributed:** Partitioned across multiple nodes
- **Resilient:** Automatically recovered on failure
- **Lazily Evaluated:** Transformations not executed until action called

**Operations:**
- **Transformations:** map, flatMap, filter, reduceByKey (lazy)
- **Actions:** collect, count, saveAsTextFile (trigger execution)

### Example from Lab:

```scala
val lines = sc.textFile("/data/testing_spark.txt")  // Transformation
val words = lines.flatMap(_.split("\\s+"))           // Transformation
val wc = words.map(w => (w, 1)).reduceByKey(_ + _)  // Transformation
wc.saveAsTextFile("testing_spark.wordcount")        // Action (triggers execution)
```

---

## Web Interfaces

| Service | URL | Description |
|---------|-----|-------------|
| **Spark UI** | http://hadoop-master:4040 | Job monitoring and execution details |
| **NameNode UI** | http://hadoop-master:9870 | HDFS status and file browser |
| **ResourceManager UI** | http://hadoop-master:8088 | YARN application tracking |

---

## Troubleshooting

### Issue 1: HDFS Connection Refused

**Error:**
```
put: Call From hadoop-master/172.18.0.5 to hadoop-master:9000 failed
```

**Solution:**
```bash
./start-hadoop.sh
```

### Issue 2: File Already Exists

**Error:**
```
get: `testing_spark.wordcount/_SUCCESS': File exists
```

**Solution:**
```bash
rm -rf testing_spark.wordcount
hdfs dfs -get /user/root/testing_spark.wordcount
```

### Issue 3: Netcat Not Available on Windows

**Error:**
```
nc : The term 'nc' is not recognized
```

**Solutions:**
1. **Use Docker container:**
   ```bash
   docker exec -it hadoop-master bash
   nc -lk 9999
   ```

2. **Install ncat (part of nmap):**
   - Download from: https://nmap.org/download.html
   - Use: `ncat -lk 9999`

3. **Use WSL (Windows Subsystem for Linux):**
   ```bash
   wsl
   nc -lk 9999
   ```

### Issue 4: Spark Submit Cannot Find File

**Error:**
```
python: can't open file '/root/simple_batch.py': [Errno 2] No such file or directory
```

**Solution:**
```bash
cd ~/sensor-analytics/batch-processing/src
ls -la  # Verify file exists
/usr/local/spark/bin/spark-submit simple_batch.py
```

### Issue 5: Typo in Filename

**From lab output:**
```bash
# Created file with typo: simple_stram.py
mv simple_stram.py simple_stream.py
```

---

## HDFS Operations Summary

### Commands Used in This Lab

```bash
# Upload files
hdfs dfs -put testing_spark.txt /data
hdfs dfs -put sensor_data.json /data

# List directory contents
hdfs dfs -ls /data/

# Download results
hdfs dfs -get /user/root/testing_spark.wordcount

# View file contents
hdfs dfs -cat /data/testing_spark.txt
```

---

## Key Learnings

✅ **Spark Architecture**
- Driver program executes main function
- Executors run parallel operations
- RDD abstraction for distributed data

✅ **Batch Processing**
- Historical data analysis
- Complete dataset processing
- Transformations and actions

✅ **Stream Processing**
- Real-time data ingestion
- Micro-batch processing model
- Socket/Kafka source integration

✅ **PySpark Programming**
- SparkSession initialization
- DataFrame and RDD APIs
- Data transformations

✅ **HDFS Integration**
- File upload and retrieval
- Distributed storage
- Data persistence

---

## Comparison: Spark vs MapReduce

| Feature | Spark | MapReduce (Lab 3) |
|---------|-------|-------------------|
| **Speed** | 10-100x faster | Baseline |
| **Processing** | In-memory | Disk-based |
| **API** | High-level (DF, SQL, ML) | Low-level (map/reduce) |
| **Ease of Use** | More concise | More verbose |
| **Streaming** | Native support | Requires additional tools |
| **Iterative Algorithms** | Excellent | Poor performance |
| **Language Support** | Scala, Python, Java, R | Java, Streaming |

---

## Additional Resources

### Spark Documentation
- [Apache Spark Official Docs](https://spark.apache.org/docs/3.2.1/)
- [PySpark API Reference](https://spark.apache.org/docs/3.2.1/api/python/)
- [Structured Streaming Guide](https://spark.apache.org/docs/3.2.1/structured-streaming-programming-guide.html)

### Sensor Integration
- [MQTT for IoT](https://mqtt.org/)
- [Apache Kafka](https://kafka.apache.org/)
- [InfluxDB Time-Series DB](https://www.influxdata.com/)

---

## Conclusion

This lab successfully demonstrated:

1. **Spark Shell Operations:** Interactive Scala-based WordCount
2. **Batch Processing:** Historical sensor data analysis with PySpark
3. **Stream Processing:** Real-time temperature monitoring via socket streaming
4. **HDFS Integration:** Seamless data storage and retrieval
5. **Micro-Batch Concept:** Understanding continuous query execution model

The combination of batch and stream processing showcases Spark's versatility for both historical analysis and real-time monitoring scenarios.

---

**Lab Completed:** November 8, 2025  
**Status:** ✅ All Objectives Achieved  
**Student:** Zaid S.

---

## Appendix: Complete Code Files

### A. simple_batch.py (Complete)

```python
from pyspark.sql import SparkSession

# Initialize Spark Session
spark = SparkSession.builder \
    .appName("batch_processing") \
    .master("local[*]") \
    .getOrCreate()

# Read sensor data from HDFS
data = spark.read.json("hdfs://hadoop-master:9000/data/sensor_data.json")

print(f"Total records: {data.count()}")

# Convert DataFrame to RDD
rdd = data.rdd

# Compute comfort index: temperature - (humidity * 0.1)
transformed_rdd = rdd.map(lambda row: (
    row.sensor_id,
    round(row.temperature - (row.humidity * 0.1), 2)
))

# Display results
print("Comfort index per record:")
for record in transformed_rdd.collect():
    print(record)

# Stop Spark session
spark.stop()
```

### B. simple_stream.py (Complete)

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import split, col

# Initialize Spark Session
spark = SparkSession.builder \
    .appName("stream_processing") \
    .master("local[*]") \
    .getOrCreate()

# Read streaming data from socket
lines = spark.readStream \
    .format("socket") \
    .option("host", "localhost") \
    .option("port", 9999) \
    .load()

# Parse temperature data (format: "temp:20.5")
temperatures = lines.select(
    split(col("value"), ":").getItem(1).cast("double").alias("temperature")
)

# Write output to console
query = temperatures.writeStream \
    .outputMode("append") \
    .format("console") \
    .start()

# Wait for termination
query.awaitTermination()
```

### C. sensor_data.json (Complete)

```json
{"sensor_id": "A1", "temperature": 22.5, "humidity": 65, "timestamp": "2024-01-15T08:00:00"}
{"sensor_id": "A2", "temperature": 24.8, "humidity": 58, "timestamp": "2024-01-15T08:00:00"}
{"sensor_id": "A3", "temperature": 21.2, "humidity": 70, "timestamp": "2024-01-15T08:00:00"}
{"sensor_id": "A1", "temperature": 23.1, "humidity": 63, "timestamp": "2024-01-15T09:00:00"}
{"sensor_id": "A2", "temperature": 25.3, "humidity": 55, "timestamp": "2024-01-15T09:00:00"}
```

---

**End of Lab 4 Documentation**
