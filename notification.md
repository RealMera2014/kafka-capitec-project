# Notification producer, consumer, topic/s and event/s

## Topic configs (Notification Events Output)
kubectl exec -it kafka-0 -- kafka-topics \
  --bootstrap-server kafka-service:9092 \
  --create \
  --topic digital-payments.notifications \
  --partitions 6 \
  --replication-factor 3 \
  --config retention.ms=86400000 \
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
  --group notification-service \
  --property max.poll.records=1000 \
  --property session.timeout.ms=45000 \
  --property heartbeat.interval.ms=15000 \
  --property auto.offset.reset=earliest \
  --property enable.auto.commit=true \
  --property auto.commit.interval.ms=3000 \
  --property max.poll.interval.ms=60000 \
  --property fetch.min.bytes=512 \
  --property fetch.max.wait.ms=1000 \
  --property max.partition.fetch.bytes=2097152 \
  --property partition.assignment.strategy=cooperative-sticky


### Producer configs (Notification Events Producer)
### Publishes to: digital-payments.notifications topic
kubectl exec -it kafka-0 -- kafka-console-producer \
  --bootstrap-server kafka-service:9092 \
  --topic digital-payments.notifications \
  --property parse.key=true \
  --property key.separator=: \
  --producer-property acks=1 \
  --producer-property enable.idempotence=true \
  --producer-property max.in.flight.requests.per.connection=10 \
  --producer-property compression.type=snappy \
  --producer-property linger.ms=5 \
  --producer-property batch.size=32768 \
  --producer-property request.timeout.ms=5000 \
  --producer-property delivery.timeout.ms=10000


## Notification Channels:
- **SMS**: immediate, 100% delivery guarantee via carrier
- **Email**: standard, may take a few seconds
- **Push**: app-native notifications (not shown in examples)
- **In-App**: browser notifications
### Producer configs

