# Kafka Broker Cluster and Data Access

- **Cluster Purpose:**  
  Brokers replicate data across the cluster to provide fault tolerance and data durability.

- **Leader Role:**  
  Each partition has a single leader broker that handles all producer and consumer read/write requests.

- **Follower Role:**  
  Followers replicate data from the leader but do **not** serve read or write requests during normal operation.

- **Failover:**  
  If the leader fails, a follower is elected as the new leader to maintain availability and continuity.


- Consumers **read only from the leader broker** of each partition.
- Followers replicate data but do **not serve read requests** unless promoted.
- If a leader broker goes down, one of the followers is **promoted to leader**.
- After promotion, consumers start reading from the **new leader broker**.
---

This architecture ensures high availability and data safety, while



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

# Kafka: Partition and Consumer Group Mapping
## Key Concepts

- Each **partition** can be read by **only one consumer** in a **consumer group** at a time.
- **Multiple consumer groups** can consume messages from the same topic independently.
- This enables different microservices (e.g., billing, notifications) to process the same stream of events for different business logic.

## Example Configuration

- **Topic**: `ride-events`
- **Partitions**: 3
- **Consumer Groups**:
  - `group-billing`
  - `group-notification`

## Partition to Consumer Mapping

| Partition | `group-billing` (Billing Service) | `group-notification` (Notification Service) |
|-----------|-----------------------------------|---------------------------------------------|
| P0        | consumer-a                        | consumer-x                                  |
| P1        | consumer-b                        | consumer-y                                  |
| P2        | consumer-c                        | consumer-z                                  |

## Processing Flow

Each consumer group independently processes the same message stream:

- **group-billing**
  - Calculates fare
  - Applies discounts
  - Processes payments

- **group-notification**
  - Sends SMS
  - Sends push/email notifications
  - Logs user communication events

## Key Notes

- Kafka guarantees **at-least-once delivery per consumer group**.
- Consumers in **different groups** do not affect each other.
- Kafka **scales** horizontally by increasing the number of **partitions**.
- To maximize parallelism, ensure number of **consumers ≤ partitions** per group.


## Kafka Ordering Guarantee

Kafka guarantees ordering **only within a partition**. When designing producers:

- Use **consistent keys** (e.g., user ID, transaction ID) to ensure related messages go to the same partition.
- **Do not expect total ordering** across multiple partitions.

| Scope               | Guarantee |
|---------------------|-----------|
| Within Partition     | ✅ Yes    |
| Across Partitions    | ❌ No     |



