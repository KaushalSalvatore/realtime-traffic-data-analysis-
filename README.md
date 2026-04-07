# realtime-traffic-data-analysis-
real time traffic data analysis with kafka spark and delta lake


# Create Kafka Topic
docker exec -it kafka /opt/kafka/bin/kafka-topics.sh --create --topic traffic-topic --bootstrap-server kafka:9092 --partitions 3 --replication-factor 1


# Install tools for Producer
pip install kafka-python faker pytz