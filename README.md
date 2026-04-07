# realtime-traffic-data-analysis-
real time traffic data analysis with kafka spark and delta lake


# Create Kafka Topic
docker exec -it kafka /opt/kafka/bin/kafka-topics.sh --create --topic traffic-topic --bootstrap-server kafka:9092 --partitions 3 --replication-factor 1


# Install tools for Producer
pip install kafka-python faker pytz


# Run Bronze Transformation (after creating taffice_bronze.py file run this command)
docker exec -it spark-worker /opt/spark/bin/spark-submit --conf spark.jars.ivy=/tmp/.ivy --packages io.delta:delta-spark_2.12:3.2.0,org.apache.spark:spark-sql-kafka-0-10_2.12:3.5.1 /opt/spark-apps/traffic_bronze.py
