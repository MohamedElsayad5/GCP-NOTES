<div align="center">

# 🗄️ GCP Databases - Complete Quick Revision Guide

**A concise, production-ready cheat sheet for Google Cloud database services**

[[Cloud SQL](https://img.shields.io/badge/Cloud%20SQL-Managed%20RDBMS-4285F4?style=for-the-badge&logo=googlecloud&logoColor=white)](https://cloud.google.com/sql)
[[Spanner](https://img.shields.io/badge/Cloud%20Spanner-Global%20SQL-34A853?style=for-the-badge)](#)
[[BigQuery](https://img.shields.io/badge/BigQuery-OLAP%20Warehouse-F9AB00?style=for-the-badge)](#)
[[Interview Ready](https://img.shields.io/badge/Status-Interview%20Ready-00C853?style=for-the-badge)](#)

[🔵 Relational](#2-cloud-sql-managed-rdbms) • [🟢 NoSQL](#5-firestore-serverless-nosql-documents) • [🟡 Analytics](#7-bigquery-serverless-olap-data-warehouse) • [🎯 Decision Flow](#9-ultimate-database-selection-flowchart)

</div>

---

## 📖 Table of Contents

1. [Core Database Classifications](#1-core-database-classifications)
2. [Cloud SQL (Managed RDBMS)](#2-cloud-sql-managed-rdbms)
3. [AlloyDB (High-Performance PostgreSQL)](#3-alloydb-high-performance-postgresql)
4. [Cloud Spanner (Globally Distributed Relational)](#4-cloud-spanner-globally-distributed-relational)
5. [Firestore (Serverless NoSQL Documents)](#5-firestore-serverless-nosql-documents)
6. [Cloud Bigtable (Massive Scale Wide-Column NoSQL)](#6-cloud-bigtable-massive-scale-wide-column-nosql)
7. [BigQuery (Serverless OLAP Data Warehouse)](#7-bigquery-serverless-olap-data-warehouse)
8. [Side-by-Side Architectural Comparisons](#8-side-by-side-architectural-comparisons)
9. [Ultimate Database Selection Flowchart](#9-ultimate-database-selection-flowchart)
10. [One-Line Memory Tricks & Interview Cheat Sheet](#10-one-line-memory-tricks--interview-cheat-sheet)

---

## 1. Core Database Classifications

Google Cloud segments its database portfolio into three major pillars based on operational workloads:

* **Relational (OLTP):** Structured schemas, strong relationships, strict ACID compliance. *Cloud SQL, AlloyDB, Cloud Spanner*
* **NoSQL (Non-Relational):** Highly dynamic schemas, horizontal scaling, ultra-low latency sub-millisecond lookups. *Firestore, Cloud Bigtable*
* **Analytics (OLAP / Data Warehouse):** Columnar storage engines built for complex multi-terabyte queries, reporting, and BI. *BigQuery*

---

## 2. Cloud SQL (Managed RDBMS)

A fully managed relational database engine supporting **MySQL**, **PostgreSQL**, and **SQL Server**. Automates provisioning, replication, backups, and patching.

### 🔌 Connectivity Models
* **Public IP:** Accessible over the internet. Secured via Authorized Networks or firewall rules.
* **Private IP (Production Standard):** Uses VPC Network Peering. Traffic stays on Google's private global fiber network.
* **Cloud SQL Auth Proxy:** 🔥 **Interview Favorite.** Runs locally to provide secure, encrypted IAM-authenticated tunnels into instance without managing SSL certs or whitelisting IPs.

```bash
# Run Auth Proxy
./cloud-sql-proxy PROJECT:REGION:INSTANCE
# Connect via localhost
psql -h 127.0.0.1 -U postgres
```

### 🏗️ High Availability & Scaling
* **High Availability (HA):** Synchronous replication to **Standby replica** in different Availability Zone. Automatic failover if Primary zone fails.
* **Read Replicas:** Asynchronous replication to offload heavy read traffic from primary.
* **Scaling Vectors:** **Vertical Scaling Only** - Upgrade CPU and RAM. No horizontal sharding built-in.

---

## 3. AlloyDB (High-Performance PostgreSQL)

Google's enterprise-grade, next-generation relational database designed to be fully compatible with PostgreSQL. Think **PostgreSQL++**.

```text
  ┌────────────────────────────────────────────────────────┐
  │                   AlloyDB Engine                       │
  ├───────────────────────────┬────────────────────────────┤
  │       Compute Node        │        Compute Node        │
  │     (Primary / Read)      │      (Primary / Read)      │
  └───────────────────────────┴────────────────────────────┘
                                │
        Separated Log & Storage Processing Layer
                                │
  ┌────────────────────────────────────────────────────────┐
  │          Distributed Shared Storage Elastic Pool        │
  └────────────────────────────────────────────────────────┘
```

* **Decoupled Architecture:** Separates compute from shared storage pool. Delivers **up to 4x faster** than standard PostgreSQL for transactional workloads.
* **Intelligent Indexing:** Built-in ML for automatic indexing, memory optimization, and analytical caching.
* **Use Case:** When you need PostgreSQL but hit performance limits on Cloud SQL.

---

## 4. Cloud Spanner (Globally Distributed Relational)

Enterprise-grade, globally scalable SQL database designed to break the compromise between Relational features and NoSQL scale.

* **The Core Hybrid:** Standard relational SQL schemas + absolute **ACID transactions** + NoSQL **Horizontal Scaling**.
* **Automatic Sharding:** Dynamically partitions tables across nodes globally based on load. No manual sharding.
* **TrueTime API:** 🔥 **Interview Keyword.** Google's proprietary hardware using synchronized atomic clocks + GPS receivers across datacenters to guarantee **external consistency** globally. Solves distributed transaction problem.

**Use Case:** Global apps needing SQL + strong consistency + horizontal scale. e.g. Banking, inventory, gaming leaderboards.

---

## 5. Firestore (Serverless NoSQL Documents)

Flexible, fully serverless NoSQL document database built for mobile/web apps and real-time sync.

```text
[Collection: users] ──► [Document: mohamed] ──► { name: "Mohamed", age: 24, city: "Cairo" }
                      └──► [Subcollection: orders]
```

* **Hierarchical Data:** Collections contain Documents containing fields + Subcollections.
* **Key Strengths:**
    * **Real-Time Synchronization:** Pushes data updates to connected clients via web sockets.
    * **Offline Support:** Local caching on iOS/Android. Auto-syncs when online.
    * **Serverless:** No servers to manage. Pay per read/write/storage.
* **Modes:** Native Mode - Real-time. Datastore Mode - Strong consistency, legacy.

---

## 6. Cloud Bigtable (Massive Scale Wide-Column NoSQL)

High-throughput, compressed, wide-column NoSQL storage for **multi-petabyte datasets** requiring **single-digit millisecond latency**.

* **Data Model:** Sparse table. Row Key + Column Families + Columns + Timestamped cells.
* **Ideal Streams:** IoT sensor data, telemetry, financial time-series, user clickstreams, ad tech.
* **Design Rules:** Optimized for **key-value lookups via Row Key only**. **No joins. No multi-row ACID transactions.** Design Row Key carefully for query patterns.

**Scale:** Handles millions of QPS. Used by Google Search, Maps, Gmail.

---

## 7. BigQuery (Serverless OLAP Data Warehouse)

Enterprise-grade, serverless data warehouse for running analytical SQL on **multi-terabyte/petabyte datasets in seconds**.

### 📐 Structural Optimization
* **Partitioning:** Slices table into physical files by Date/Integer. Queries scan only relevant partitions.
* **Clustering:** Sorts data within partitions by columns. Maximizes caching and reduces scan.

### 💰 The Cost-Saving Rule
BigQuery charges based on **Data Scanned**, not compute time.

**Anti-pattern:** `SELECT * FROM huge_table` - Scans all columns, expensive.

**Best Practice:** 
```sql
-- Good: Scans only needed columns + partition
SELECT user_id, event_time 
FROM `project.dataset.events`
WHERE _PARTITIONDATE = "2026-06-08"
```

---

## 8. Side-by-Side Architectural Comparisons

### Relational Database Options (OLTP)

| Feature | Cloud SQL | AlloyDB | Cloud Spanner |
| :--- | :--- | :--- | :--- |
| **Scaling Mechanism** | Vertical - Scale Up CPU/RAM | Hybrid - Decoupled Compute | Horizontal - Scale Out Nodes |
| **Target Engine** | MySQL, Postgres, SQL Server | Optimized PostgreSQL | Proprietary Spanner SQL |
| **Geographic Reach** | Single Region + HA | Single Region + HA | Multi-Region / Global |
| **Sharding Layout** | Manual Sharding Required | Manual Sharding Required | Fully Automated Sharding |
| **Max Size** | 64 TB | 128 TB | Petabytes |
| **Consistency** | Strong - Single region | Strong - Single region | Strong - Global via TrueTime |

### NoSQL Options (Application vs. Big Data)

| Feature | Firestore | Cloud Bigtable |
| :--- | :--- | :--- |
| **Data Format** | Hierarchical Documents JSON-like | Sparse Wide-Column Table |
| **Primary Workload** | Mobile App State & Sync | IoT, Telemetry, Time-Series |
| **Query Mechanism** | Rich Compound Filters / SDK | Single Index Row-Key Lookups Only |
| **Real-Time Push** | Native Support | Not Supported Natively |
| **Transactions** | Multi-document ACID | Single-row atomic |
| **Scale** | Automatic, up to TB | Petabytes, millions QPS |

### Operational Transactional vs. Analytics

| Feature | Cloud SQL (OLTP) | BigQuery (OLAP) |
| :--- | :--- | :--- |
| **Primary Intent** | Transaction processing - Web Backend | Deep Business Analytics & BI |
| **Storage Layout** | Row-oriented storage | Columnar compression |
| **Scaling Architecture** | Resource Capacity Limits | Fully Dynamic Serverless Slots |
| **Query Type** | Small, frequent, point queries | Large, complex, aggregations |
| **Latency** | Milliseconds | Seconds to minutes |

---

## 9. Ultimate Database Selection Flowchart

```text
Is your data model Relational (SQL)?
 ├── YES: Do you require worldwide horizontal scaling + global consistency?
 │     ├── YES ──► Cloud Spanner
 │     └── NO:  Do you require ultra-high PostgreSQL performance?
 │           ├── YES ──► AlloyDB
 │           └── NO  ──► Cloud SQL - MySQL / Postgres / SQL Server
 │
 └── NO (NoSQL/Analytics): Is primary objective Analytical BI/Data Warehousing?
       ├── YES ──► BigQuery
       └── NO:  Is source mobile/web app needing real-time sync?
             ├── YES ──► Firestore - Native or Datastore Mode
             └── NO  ──► Cloud Bigtable - IoT / Time-Series / Telemetry
```

---

## 10. One-Line Memory Tricks & Interview Cheat Sheet

### 🎯 One-Line Core Identifiers
* **Cloud SQL:** Managed traditional single-region SQL - MySQL/PostgreSQL/SQL Server. Vertical scale.
* **AlloyDB:** High-end, enterprise decoupled storage PostgreSQL. 4x faster than standard PG.
* **Cloud Spanner:** Global relational SQL scale without sacrificing ACID. TrueTime API.
* **Firestore:** Lightweight, serverless real-time NoSQL document DB for mobile/web apps.
* **Cloud Bigtable:** Industrial-scale, high-throughput NoSQL for IoT and time-series. Row-key only.
* **BigQuery:** Serverless analytical data warehouse. Pay per data scanned.

### 🧠 High-Frequency Interview Keywords

| Keyword | Mention When Asked About |
| :--- | :--- |
| **TrueTime API** | How **Cloud Spanner** guarantees globally consistent transactions without lock contention |
| **Data Scanned Cost Model** | Why you must use **BigQuery Partitioning & Clustering** to reduce query costs |
| **Cloud SQL Auth Proxy** | Security best practice for connecting apps/K8s to **Cloud SQL** without public IP |
| **Decoupled Storage** | How **AlloyDB** achieves 4x PostgreSQL performance |
| **Row Key Design** | Critical for **Bigtable** performance - determines data distribution |
| **VPC Peering** | How **Cloud SQL Private IP** keeps traffic on Google network |
| **Horizontal vs Vertical** | Spanner = Horizontal. Cloud SQL = Vertical. AlloyDB = Hybrid |

### ⚡ Quick Decision Matrix

| If You Need... | Choose... |
| :--- | :--- |
| Standard MySQL/Postgres for web app | **Cloud SQL** |
| PostgreSQL but need 4x performance | **AlloyDB** |
| Global SQL with strong consistency | **Cloud Spanner** |
| Mobile app real-time sync | **Firestore Native Mode** |
| IoT sensors, millions writes/sec | **Cloud Bigtable** |
| BI dashboards on TB+ data | **BigQuery** |
| Lift & shift existing SQL Server | **Cloud SQL for SQL Server** |

---

## 🔐 Security & Operations Best Practices

| Service | Security Best Practice |
| :--- | :--- |
| **Cloud SQL** | Use Private IP + Cloud SQL Auth Proxy. Enable automated backups + PITR |
| **AlloyDB** | Same as Cloud SQL. Enable automated backups |
| **Spanner** | Use IAM for fine-grained access. Enable CMEK for encryption |
| **Firestore** | Use Security Rules to control client access |
| **Bigtable** | Use IAM. Design Row Keys to avoid hotspots |
| **BigQuery** | Use Authorized Views. Partition + Cluster tables |

---

<div align="center">

**⭐ Save this for your GCP Data Engineer / Cloud Architect interview!**

Made with ❤️ for Cloud Databases

[⬆️ Back to Top](#️-gcp-databases---complete-quick-revision-guide)

</div>