To Read:
  Replication :
    https://en.wikipedia.org/wiki/Replication_(computing)
  Replication Models :

# Replication Models: Architectural Blueprints

These models describe the high-level strategy for how data is distributed and managed across multiple machines.

## 🔹 Replication Models

| Model | How it Works | Best For | Trade-Offs |
|-------|--------------|----------|-------------|
| **Leader-Follower (Master-Slave)** | One leader node handles all writes → followers replicate and serve reads. | High read performance, simple setup, strong consistency (from leader). | Leader = single point of failure; write bottleneck. |
| **Multi-Leader (Master-Master)** | Multiple leaders accept writes and replicate changes to each other. | Geo-distributed apps, low write latency, high write availability. | Complex conflict resolution for concurrent writes. |
| **Leaderless Replication** | Any node can take writes; consistency via read/write quorums. | Fault-tolerant, evenly balanced writes, high availability. | Eventual consistency; requires complex conflict resolution. |
| **Chain Replication** | Nodes arranged in a chain; writes flow head → tail; reads often served from tail. | Strong consistency + fault tolerance. | Higher write latency (sequential propagation). |

---

## 🔹 Coordination Techniques: Operational Protocols

These are lower-level protocols for ensuring consistency and communication guarantees, often implemented within the broader architectural models above.

| Technique | What it Does | Consistency Model | Best For |
|-----------|--------------|-------------------|----------|
| **Transactional Replication** | Replicates individual DB transactions in order. | Transactional consistency (can be briefly inconsistent if async). | Databases, real-time migrations. |
| **State Machine Replication (SMR)** | Ensures replicas apply the same ordered sequence of commands. | Strong consistency (linearizability). | Consensus protocols (Raft, Paxos), fault-tolerant DBs. |
| **Virtual Synchrony** | Provides consistent group membership views and ordered message delivery. | Consistent message ordering within group views. | Reliable group communication (e.g., cluster coordination, data center mgmt). |

---

## 🔹 How They Combine in Practice

- **Leader-Follower + State Machine Replication** → Strongly consistent distributed databases (e.g., PostgreSQL HA setups, etcd).  
- **Multi-Leader + Conflict Resolution** → Geo-replicated databases (e.g., Active-Active Cassandra, MySQL multi-master).  
- **Leaderless + Quorums** → Dynamo-style eventually consistent key-value stores.  
- **Chain Replication + SMR** → High availability storage systems (e.g., Microsoft Azure Storage).  
