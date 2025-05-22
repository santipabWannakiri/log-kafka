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

# Kafka Topics, Partitions & Sharding Example for Uber

This example shows how to organize Kafka topics by country and shard driver events using partitions.

---

## Topics and Partitions

| Topic Name   | Number of Partitions | Description                    |
|--------------|----------------------|-------------------------------|
| rides-usa    | 12                   | Uber rides data for USA        |
| rides-canada | 12                   | Uber rides data for Canada     |
| rides-uk     | 12                   | Uber rides data for UK         |

---

## Driver ID Sharding (Partition Assignment)


This ensures messages related to the same driver go to the same partition for ordered processing.

---

## Example Driver ID to Partition Mapping

| Driver ID | Partition (Shard) Calculation | Assigned Partition |
| --------- | ----------------------------- | ------------------ |
| 12345     | hash(12345) % 12 = 5          | 5                  |
| 67890     | hash(67890) % 12 = 3          | 3                  |
| 54321     | hash(54321) % 12 = 7          | 7                  |

---

## Summary

- Using hashing for driver IDs balances load across partitions.
- Keeps all messages for a driver in the same partition for ordered consumption.
- Number of partitions should be planned based on throughput and parallel consumers.


