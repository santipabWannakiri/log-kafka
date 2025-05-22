# Kafka Partition Sizing Example

When designing Kafka topics and partitions, consider the expected message throughput and desired consumer parallelism.

---

## Given:

- **Expected messages per second:** 50,000 msg/sec  
- **Throughput per partition:** 5,000 msg/sec  
- **Number of consumers to run in parallel:** 12

---

## Calculate minimum required partitions:

50,000 / 5,000 = 10 -> max(10,12) = 12

---

## Conclusion:

You should create **at least 12 partitions** for your topic to handle the expected load and to allow 12 consumers to process data in parallel efficiently.

---

This ensures:

- Each partition handles the message throughput without bottlenecking.
- Consumer parallelism matches the number of partitions for maximum efficiency.
