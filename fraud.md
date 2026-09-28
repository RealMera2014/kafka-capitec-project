# Fraud producer, consumer, topic/s and event/s
## Topic configs (Fraud Events Output)
kubectl exec -it kafka-0 -- kafka-topics \
  --bootstrap-server kafka-service:9092 \
  --create \
  --topic digital-payments.fraud-events \
  --partitions 6 \
  --replication-factor 3 \
  --config retention.ms=220752600000 \
  --config cleanup.policy=delete \
  --config min.insync.replicas=2 \
  --config compression.type=snappy \
  --config max.message.bytes=1048576

  ### Consumer configs

kubectl exec -it kafka-0 -- kafka-console-consumer \
  --bootstrap-server kafka-service:9092 \
  --topic digital-payments.lifecycle \
  --property parse.key=true \
  --property key.deserializer=org.apache.kafka.common.serialization.StringDeserializer \
  --property value.deserializer=org.apache.kafka.common.serialization.StringDeserializer \
  --group fraud-detection-service \
  --property max.poll.records=500 \
  --property session.timeout.ms=30000 \
  --property heartbeat.interval.ms=10000 \
  --property auto.offset.reset=earliest \
  --property enable.auto.commit=false \
  --property auto.commit.interval.ms=5000 \
  --property max.poll.interval.ms=300000 \
  --property fetch.min.bytes=1024 \
  --property fetch.max.wait.ms=500 \
  --property max.partition.fetch.bytes=1048576 \
  --property partition.assignment.strategy=cooperative-sticky


### Producer configs

kubectl exec -it kafka-0 -- kafka-console-producer \
  --bootstrap-server kafka-service:9092 \
  --topic digital-payments.fraud-events \
  --property parse.key=true \
  --property key.separator=: \
  --producer-property acks=all \
  --producer-property enable.idempotence=true \
  --producer-property max.in.flight.requests.per.connection=5 \
  --producer-property compression.type=snappy \
  --producer-property linger.ms=5 \
  --producer-property batch.size=16384 \
  --producer-property request.timeout.ms=10000 \
  --producer-property delivery.timeout.ms=20000

  ## Fraud Score Ranges:
- **LOW (0-30)**: normal payment pattern, approve automatically
- **MEDIUM (30-70)**: elevated risk, may require additional authentication (3D Secure, etc.)
- **HIGH (70-100)**: block payment, notify customer to verify 
  --property batch.size=<value>  \
  --property delivery.timeout.ms=<value>  \
  --property request.timeout.ms=<value> 
