# Hybrid Adaptive Tuple Partitioning for Large Distributed Database Systems

This repository presents a research project on **tuple-level partitioning in distributed database systems**, focusing on reducing distributed transactions, improving scalability, and enabling adaptive workload-aware data placement. The work combines an extensive survey of classical and modern partitioning techniques with the design of a novel hybrid partitioning framework that integrates graph-based partitioning, lookup-table routing, live migration, and lightweight machine-learning-assisted refinement.

The project investigates how modern distributed databases can efficiently partition data while adapting to changing workload patterns, minimizing cross-shard communication, and maintaining operational simplicity.

---

## Repository Contents

```text
.
├── README.md
├── Dds_part1.pdf
├── DDS_part2.pdf
├── Project_Presentation.pptx

```

## Interactive Simulator

To better illustrate the concepts discussed in this project, we also developed a simulator that demonstrates tuple partitioning, routing, workload drift, and adaptive migration behavior in a distributed database environment.

**Simulator Repository:**
https://github.com/YOUR_USERNAME/SIMULATOR_REPO

### Included Documents

- **Survey Paper** – Comprehensive review of tuple-partitioning algorithms for distributed databases.
- **Research Proposal / Progress Report** – Proposed hybrid adaptive partitioning framework.
- **Presentation Slides** – Project presentation summarizing the research and proposed system.
- **Architecture Diagrams** – System architecture, routing flow, and adaptive partitioning workflow.

---

# Motivation

Modern online transaction processing (OLTP) systems serve millions of users and process massive volumes of transactions. To scale beyond a single machine, databases are partitioned across multiple servers (shards).

A major challenge in distributed databases is the presence of **distributed transactions**, where a transaction accesses data stored on multiple shards. These transactions introduce:

- Network communication overhead
- Distributed locking costs
- Increased latency
- Reduced throughput
- Higher coordination complexity

The primary objective of data partitioning is therefore to place data that is frequently accessed together on the same shard, reducing cross-shard communication and improving performance.

---

# Problem Statement

Given:

- A large relational database
- A set of transactional workloads
- Multiple database shards

Determine a mapping:

```text
Tuple → Shard
```

such that:

- Distributed transactions are minimized
- Data remains balanced across shards
- Migration costs remain manageable
- The system can adapt to workload changes

The challenge becomes significantly harder when workloads evolve over time and access patterns change dynamically.

---

# Research Objectives

This project aims to:

- Study classical and modern tuple-partitioning techniques.
- Compare graph-based, lookup-table, migration-based, and machine-learning-based approaches.
- Analyze their strengths and limitations.
- Design a practical adaptive partitioning framework.
- Minimize distributed transaction rates.
- Support workload drift without expensive repartitioning.
- Maintain production-friendly operational complexity.

---

# Background: Tuple Partitioning

Tuple partitioning is a fine-grained database sharding strategy where individual tuples (records) are assigned to shards based on workload behavior.

Unlike traditional:

- Hash partitioning
- Range partitioning
- Round-robin partitioning

tuple partitioning considers how data is accessed together.

If two tuples frequently appear in the same transaction, they should ideally be stored on the same shard.

This reduces:

- Cross-shard joins
- Distributed commits
- Network communication

and significantly improves transaction processing performance.

---

# Survey of Existing Approaches

## 1. Schism and Graph-Based Partitioning

Schism introduced workload-driven partitioning using a graph representation.

### Core Idea

- Each tuple becomes a graph node.
- Co-accessed tuples are connected with weighted edges.
- Graph partitioning algorithms such as METIS minimize edge cuts.

### Advantages

- Excellent reduction of distributed transactions.
- Strong partition quality.

### Limitations

- Computationally expensive.
- Requires workload re-analysis when access patterns change.
- Primarily offline.

---

## 2. Hypergraph Partitioning

Hypergraph partitioning extends graph partitioning by modeling an entire transaction as a hyperedge.

### Advantages

- Better representation of multi-way joins.
- Captures transaction structure more accurately.

### Limitations

- Higher computational complexity.
- Difficult to maintain at very large scale.

---

## 3. Lookup Table Partitioning

Lookup-table approaches explicitly maintain mappings:

```text
Tuple ID → Shard ID
```

Routing becomes a simple lookup operation.

### Advantages

- Exact routing.
- Fine-grained control.
- Easy migration support.

### Limitations

- Metadata management overhead.
- Requires scalable routing infrastructure.

---

## 4. E-Store and Live Migration

E-Store focuses on elastic partitioning and hot-tuple migration.

### Key Features

- Detects workload skew.
- Moves hot tuples between shards.
- Supports online adaptation.

### Advantages

- Handles dynamic workloads.

### Limitations

- Migration introduces operational complexity.

---

## 5. Grep: Graph Neural Network Partitioning

Grep formulates partitioning as a graph-learning problem.

### Workflow

```text
Workload
   ↓
Graph Construction
   ↓
Graph Neural Network
   ↓
Cost Estimation
   ↓
Partition Recommendation
```

### Advantages

- Automates partition selection.
- Learns workload structure.

### Limitations

- Requires training data.
- Computationally expensive.

---

## 6. NeuroShard

NeuroShard applies pre-training and search strategies to discover efficient shard assignments.

### Advantages

- Multi-objective optimization.
- Adaptable to large-scale systems.

### Limitations

- Complex implementation.
- High computational cost.

---

## 7. Prescient / Hermes

Prescient and Hermes use future transaction knowledge to guide migration decisions.

### Advantages

- Proactive adaptation.
- Extremely low distributed transaction rates.

### Limitations

- Requires workload predictability.
- Strong assumptions about execution models.

---

# Proposed Research Contribution

After analyzing existing approaches, we identified a recurring trade-off:

| Approach | Strong At | Weak At |
|-----------|-----------|-----------|
| Schism | Partition Quality | Adaptability |
| Lookup Tables | Routing Simplicity | Workload Awareness |
| E-Store | Live Migration | Complexity |
| Grep | Intelligent Decisions | Training Cost |
| Hermes | Future Awareness | Operational Complexity |

No single approach simultaneously provides:

- High partition quality
- Continuous adaptation
- Low migration overhead
- Practical deployment

---

# Proposed Hybrid Adaptive Partitioning Framework

To address these limitations, we propose a hybrid architecture that combines the strengths of multiple approaches.

## Core Components

### Offline Partitioner

Initial partitioning is generated using:

- Schism-style workload graphs
- METIS-based partitioning

This creates a high-quality baseline partition.

---

### Lookup Router

A lightweight routing layer stores metadata:

```text
key → [shard_id, version, flags]
```

Flags indicate:

- Migration state
- Replication status
- Hot-key information

The router caches metadata for fast tuple resolution.

---

### Monitoring Engine

The monitoring layer continuously collects:

- Transaction traces
- Access frequencies
- Hot-key statistics
- Co-access relationships

These metrics are aggregated into workload summaries.

---

### Drift Detection Module

The system computes a workload drift metric:

```text
Δ = Change in co-access behavior
```

When drift exceeds a threshold, repartitioning candidates are generated.

---

### Migration Engine

Migration follows a safe copy-then-switchover protocol:

1. Mark tuple as migrating.
2. Copy data in the background.
3. Forward writes during migration.
4. Switch metadata atomically.
5. Remove old copy after validation.

This minimizes service disruption.

---

### Learned Refinement Layer

A lightweight learning model evaluates candidate partition updates.

Possible implementations include:

- Graph Neural Networks
- Gradient Boosting Models
- Cost Prediction Models

The learned component is intentionally lightweight to reduce operational burden.

---

# Adaptive Partitioning Algorithm

```text
1. Compute initial partition using Schism/METIS.
2. Build lookup metadata table.
3. Monitor workload continuously.
4. Update co-access matrix.
5. Detect workload drift.
6. Generate candidate updates.
7. Evaluate partition cost.
8. Accept beneficial updates.
9. Migrate affected tuples.
10. Update metadata.
11. Continue serving requests.
```

This creates a closed-loop adaptive partitioning system.

---

# System Architecture

The proposed architecture consists of:

```text
Clients
   ↓
Router / Metadata Service
   ↓
Monitoring Engine
   ↓
Partition Controller
   ↓
Migration Engine
   ↓
Database Shards
```

The architecture separates routing, monitoring, and migration responsibilities, improving maintainability and scalability.

---

# Evaluation Plan

The proposed system will be evaluated using:

### Metrics

- Distributed Transaction Rate
- Throughput (TPS)
- Tail Latency (P95 / P99)
- Migration Overhead
- Partition Stability
- Load Balance

### Workloads

- TPC-C Style Workloads
- Synthetic OLTP Traces
- Hotspot Shift Workloads
- Dynamic Access Pattern Traces

### Baselines

- Hash Partitioning
- Schism
- Lookup Table Routing
- Prescient/Hermes
- Proposed Hybrid Framework

---

# Expected Contributions

This project contributes:

- A comprehensive survey of tuple-partitioning research.
- Comparative analysis of classical and modern approaches.
- A practical hybrid adaptive partitioning architecture.
- A workload-drift-aware control framework.
- Safe migration mechanisms.
- A production-oriented design suitable for future implementation.

The proposed framework bridges the gap between highly optimized academic partitioning algorithms and deployable industrial database systems.

---

# References

### Schism
Curino, C., Jones, E. P. C., Zhang, Y., & Madden, S.

*Schism: A Workload-Driven Approach to Database Replication and Partitioning* (VLDB 2010)

https://cs.brown.edu/courses/cs227/archives/2012/papers/partitioning/schism-vldb2010.pdf

### Lookup Tables
Tatarowicz, A., Pavlo, A., Elmore, A. J., & Bailis, P.

*Lookup Tables: Fine-Grained Partitioning for Distributed Databases* (2012)

https://dspace.mit.edu/bitstream/handle/1721.1/137764/lookup_tables.pdf

### E-Store
Taft, R., Wu, M., Pavlo, Y., Madden, S., & DeWitt, D.

*Fine-Grained Elastic Partitioning for Distributed Transactions* (VLDB 2014)

https://hstore.cs.brown.edu/papers/hstore-elastic.pdf

### Prescient / Hermes
Pavlo, A., et al.

*Prescient Data Partitioning and Migration for Deterministic Database Systems* (SIGMOD 2021)

https://dl.acm.org/doi/10.1145/3448016.3452827

### Grep
Li, Z., Roe, J., & Lee, H.

*Grep: A Graph-Learning Based Database Partitioning System* (SIGMOD 2023)

https://dbgroup.cs.tsinghua.edu.cn/ligl/papers/grep.pdf

### NeuroShard
Zha, D., Wang, X., et al.

*NeuroShard: Pre-train and Search for Efficient Sharding* (MLSys 2023)

https://proceedings.mlsys.org/paper_files/paper/2023/file/2f982f26692c7fd45831598085f02588-Paper-mlsys2023.pdf

---

# Authors

**Viswanadhapalli Sujay**  
**Aiswarya**  
**Suresh**  
**Nithin Manoj**

Department of Computer Science and Engineering  
Indian Institute of Technology Jodhpur

---

# Acknowledgement

We sincerely thank **Prof. Romi Banerjee** for her guidance, valuable feedback, and continuous support throughout this project. We also acknowledge the contributions of the researchers whose work inspired and informed this study.
