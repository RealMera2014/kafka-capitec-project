# Payment producer, topic/s and event/s

## Topic configs
kubectl exec -it kafka-0 -- kafka-topics \
  --bootstrap-server kafka-service:9092 \
  --create \
  --topic digital-payments.lifecycle \
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
  --topic digital-payments.lifecycle \
  --property parse.key=true \
  --property key.separator=: \
  --producer-property acks=all \
  --producer-property enable.idempotence=true \
  --producer-property max.in.flight.requests.per.connection=5 \
  --producer-property compression.type=snappy \
  --producer-property linger.ms=10 \
  --producer-property batch.size=16384 \
  --producer-property request.timeout.ms=10000 \
  --producer-property delivery.timeout.ms=30000

  # Successful payment flow:
PAY-1001:{"eventId":"EVT-001","paymentId":"PAY-1001","orderId":"ORD-1001","customerId":"CUST-123","eventType":"PaymentInitiated","eventTimestamp":"2026-09-25T10:15:00Z","amount":250.00,"currency":"USD","paymentMethod":"CARD","status":"INITIATED"}

PAY-1001:{"eventId":"EVT-002","paymentId":"PAY-1001","orderId":"ORD-1001","customerId":"CUST-123","eventType":"PaymentAuthorized","eventTimestamp":"2026-09-25T10:15:02Z","amount":250.00,"currency":"USD","status":"AUTHORIZED","fraudScore":15,"fraudStatus":"LOW"}

PAY-1001:{"eventId":"EVT-003","paymentId":"PAY-1001","orderId":"ORD-1001","customerId":"CUST-123","eventType":"PaymentValidated","eventTimestamp":"2026-09-25T10:15:03Z","amount":250.00,"currency":"USD","status":"VALIDATED"}

PAY-1001:{"eventId":"EVT-004","paymentId":"PAY-1001","orderId":"ORD-1001","customerId":"CUST-123","eventType":"PaymentCompleted","eventTimestamp":"2026-09-25T10:15:05Z","amount":250.00,"currency":"USD","status":"COMPLETED"}

# Failed payment (blocked by fraud):
PAY-1002:{"eventId":"EVT-005","paymentId":"PAY-1002","orderId":"ORD-1002","customerId":"CUST-124","eventType":"PaymentInitiated","eventTimestamp":"2026-09-25T10:16:00Z","amount":100.00,"currency":"USD","paymentMethod":"WALLET","status":"INITIATED"}

PAY-1002:{"eventId":"EVT-006","paymentId":"PAY-1002","orderId":"ORD-1002","customerId":"CUST-124","eventType":"PaymentAuthorized","eventTimestamp":"2026-09-25T10:16:02Z","amount":100.00,"currency":"USD","status":"AUTHORIZED","fraudScore":85,"fraudStatus":"HIGH"}

PAY-1002:{"eventId":"EVT-007","paymentId":"PAY-1002","orderId":"ORD-1002","customerId":"CUST-124","eventType":"PaymentFailed","eventTimestamp":"2026-09-25T10:16:03Z","amount":100.00,"currency":"USD","status":"FAILED","errorCode":"FRAUD_BLOCKED","fraudStatus":"HIGH"}
```

## Partition Key Strategy:
- **Key**: `paymentId` (ensures all events for one payment go to same partition)
- **Benefit**: fraud detector sees full payment lifecycle in order
- **Formula**: `partition = murmur2(paymentId) mod 6` 

  --property batch.size=<value>  \
  --property delivery.timeout.ms=<value>  \
  --property request.timeout.ms=<value> 
