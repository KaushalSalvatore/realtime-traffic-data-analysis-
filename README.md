#### realtime-traffic-data-analysis-
- Real Time Traffic Data Analysis With Kafka Spark And Delta Lake

- This project demonstrates a real-time traffic data analysis system built using modern big data tools including Apache Kafka, Apache Spark, Hive, and Power BI.

- The system simulates live traffic data, processes it through a scalable pipeline, and visualizes insights in an interactive dashboard.

### 🏗️ Architecture
Producer → Kafka → Spark Streaming → Bronze Layer → Silver Layer → Gold Layer → Hive → Power BI

### 🔄 Workflow

#### 1. Data Producer

- Simulates real-time traffic data (vehicle count, speed, congestion level, timestamps, etc.)
- Sends streaming data to Kafka topics

#### 2. Apache Kafka

- Acts as a distributed message broker
- Buffers and streams incoming traffic data

#### 3. Apache Spark (Structured Streaming)

- Consumes data from Kafka
- Processes data in real-time
- Stores data in layered architecture:
- Bronze Layer → Raw data ingestion
- Silver Layer → Cleaned and transformed data
- Gold Layer → Aggregated and analytics-ready data

#### 4. Hive Data Warehouse

- Gold layer data is stored in Hive tables
- Enables SQL-based querying and schema management

#### 5. Power BI Dashboard

- Connects to Hive schema
- Provides interactive visualizations and real-time insights

### 🧱 Data Lake Layers

#### 🥉 Bronze Layer (Raw Data)
- Stores unprocessed streaming data from Kafka
- Maintains original schema and format
- Used for replay and auditing

#### 🥈 Silver Layer (Cleaned Data)
- Data cleansing (remove nulls, duplicates)
- Schema enforcement
- Data transformation (formatting timestamps, standardization)

#### 🥇 Gold Layer (Business-Level Data)
- Aggregated metrics (traffic density, avg speed, congestion trends)
- Optimized for analytics and reporting
- Stored in Hive tables

# Create Kafka Topic
docker exec -it kafka /opt/kafka/bin/kafka-topics.sh --create --topic traffic-topic --bootstrap-server kafka:9092 --partitions 3 --replication-factor 1

### 🚀 Setup & Execution

#### Install tools for Producer

```bash
install Kafka , spark and hive images 
RUN : docker compose up -d
```

```bash
pip install kafka-python faker pytz

kafka-python → work with streaming data (Kafka)
Faker → generate dummy/test data
pytz → handle timezones correctly

RUN : Python traffic_data_producer.py
```

```bash
Create Kafka Topic

docker exec -it kafka /opt/kafka/bin/kafka-topics.sh --create --topic traffic-topic --bootstrap-server kafka:9092 --partitions 3 --replication-factor 1

docker exec -it kafka :-> Runs a command inside a running Docker container
/opt/kafka/bin/kafka-topics.sh :-> This is Kafka’s topic management script (create topics,list topics,delete topics)
--create :-> Tells Kafka: create a new topic
--topic traffic-topic :-> Name of the topic = traffic-topic
--bootstrap-server kafka:9092 :-> This is how the CLI knows which Kafka server to talk to
--partitions 3 :-> Creates 3 partitions for the topic (3 consumers can read simultaneously from 3 partitions)
--replication-factor 1 :-> Number of copies of data = 1
```

```bash
Run Bronze Transformation

docker exec -it spark-worker /opt/spark/bin/spark-submit --conf spark.jars.ivy=/tmp/.ivy --packages io.delta:delta-spark_2.12:3.2.0,org.apache.spark:spark-sql-kafka-0-10_2.12:3.5.1 /opt/spark-apps/traffic_bronze.py

docker exec -it spark-worker :-> Run this Spark job inside my Spark worker container

/opt/spark/bin/spark-submit :-> This is the main Spark job launcher

--conf spark.jars.ivy=/tmp/.ivy :-> Sets Spark config for dependency storage location , 
/tmp/.ivy → where downloaded packages (JARs) will be cached

--packages ... :-> io.delta:delta-spark_2.12:3.2.0 
(Used for building bronze/silver/gold data pipelines)
ACID transactions
schema evolution
reliable data lakes

org.apache.spark:spark-sql-kafka-0-10_2.12:3.5.1
(This is how Spark connects to your traffic-topic)

/opt/spark-apps/traffic_bronze.py :-> actual Spark application script , This is the job being executed
```

```bash
Run Silver Transformation

docker exec -it spark-worker /opt/spark/bin/spark-submit --conf spark.jars.ivy=/tmp/.ivy --packages io.delta:delta-spark_2.12:3.2.0,org.apache.spark:spark-sql-kafka-0-10_2.12:3.5.1 /opt/spark-apps/traffic_silver.py
```

```bash
Run Gold Transformation

docker exec -it spark-worker /opt/spark/bin/spark-submit --conf spark.jars.ivy=/tmp/.ivy --packages io.delta:delta-spark_2.12:3.2.0,org.apache.spark:spark-sql-kafka-0-10_2.12:3.5.1 /opt/spark-apps/traffic_gold.py
```

```bash
Spark SQL Connect to metastore

docker exec -it spark-worker bash

docker exec :-> Used to run a command inside a running Docker container
-it :-> gives you a terminal
spark-worker :-> Name of the running container
bash :-> Starts a Bash shell inside the container
```

```bash
mkdir -p /tmp/spark-warehouse

Create a safe directory where Spark can store its table data.

chmod -R 777 /tmp/spark-warehouse

Gives full permissions to everyone on that directory and everything inside it.
```

```bash
/opt/spark/bin/spark-sql \
--packages io.delta:delta-spark_2.12:3.2.0 \
--conf spark.jars.ivy=/tmp/.ivy \
--conf spark.sql.extensions=io.delta.sql.DeltaSparkSessionExtension \
--conf spark.sql.catalog.spark_catalog=org.apache.spark.sql.delta.catalog.DeltaCatalog \
--conf spark.sql.catalogImplementation=hive \
--conf spark.hadoop.hive.metastore.uris=thrift://hive-metastore:9083 \
--conf spark.sql.warehouse.dir=/tmp/spark-warehouse

/opt/spark/bin/spark-sql :-> Starts the Spark SQL CLI
--packages io.delta:delta-spark_2.12:3.2.0 :-> Without this, Spark can’t read/write Delta tables
--conf spark.jars.ivy=/tmp/.ivy :-> Where Spark stores downloaded dependencies (JARs)
--conf spark.sql.extensions=io.delta.sql.DeltaSparkSessionExtension :-> Adds Delta features to Spark SQL engine
--conf spark.sql.catalog.spark_catalog=org.apache.spark.sql.delta.catalog.DeltaCatalog :->
Makes Spark treat tables as Delta tables by default
--conf spark.sql.catalogImplementation=hive :-> Allows Spark to use Hive-style tables
--conf spark.hadoop.hive.metastore.uris=thrift://hive-metastore:9083 :->
Connects to external Hive Metastore service
--conf spark.sql.warehouse.dir=/tmp/spark-warehouse :-> Location where actual table data is stored

Spark SQL Shell
      ↓
Delta Lake Enabled
      ↓
Hive Metastore Connected
      ↓
Warehouse Directory Ready
```

#### Connect Hive table to PowerBI
```bash
cd /opt/spark/jars
Go into the folder where Spark keeps all its library files (JARs).

wget https://repo1.maven.org/maven2/io/delta/delta-spark_2.12/3.2.0/delta-spark_2.12-3.2.0.jar
wget https://repo1.maven.org/maven2/io/delta/delta-storage/3.2.0/delta-storage-3.2.0.jar

Download Delta Lake libraries and place them directly into Spark’s JAR folder so Spark can use them without downloading at runtime.
```

```bash
/opt/spark/sbin/start-thriftserver.sh \
--master spark://spark-master:7077 \
--conf spark.sql.extensions=io.delta.sql.DeltaSparkSessionExtension \
--conf spark.sql.catalog.spark_catalog=org.apache.spark.sql.delta.catalog.DeltaCatalog \
--conf spark.sql.catalogImplementation=hive \
--conf spark.hadoop.hive.metastore.uris=thrift://hive-metastore:9083 \
--conf spark.sql.warehouse.dir=/opt/spark/warehouse

/opt/spark/sbin/start-thriftserver.sh :-> Starts the Spark Thrift Server (Power BI can connect to it)
--master spark://spark-master:7077 :-> Run queries on the Spark cluster, not just locally
--conf spark.sql.extensions=io.delta.sql.DeltaSparkSessionExtension :-> Adds Delta features to Spark
--conf spark.sql.catalog.spark_catalog=org.apache.spark.sql.delta.catalog.DeltaCatalog :->
Makes Spark treat tables as Delta tables
--conf spark.sql.catalogImplementation=hive :-> Enables Hive-style table management
--conf spark.hadoop.hive.metastore.uris=thrift://hive-metastore:9083 :->
table schemas ,database info ,metadata
```

#### Dashboard 
![Image 1](/dashboard/dashboard.png)