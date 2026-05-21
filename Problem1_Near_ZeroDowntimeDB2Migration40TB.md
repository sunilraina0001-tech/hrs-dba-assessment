# Problem 2: DBSTAT Data Warehouse Migration Strategy

# Enterprise Data Warehouse Migration – 40TB DBSTAT Warehouse

| Document Information | Details |
|---|---|
| Document Version | Draft v1.0 |
| Prepared By | Sunil Raina |
| Date | 21 May 2026 |
| Assessment | HRS DBA Technical Assessment |
| Source Environment | IBM Db2 on AIX |
| Target Environment | AWS RDS for Db2 |
| Workload Type | Analytical / Reporting Data Warehouse |
| Database Size | 40TB |
| Downtime Tolerance | 48 Hours |

---

# Business Requirements

| Requirement | Target |
|---|---|
| Migration Completion | Within 48 Hours |
| Query Impact | Minimal impact during migration |
| Post Migration Replication | Production → Warehouse replication |
| Availability Requirement | Medium |
| Recovery Objective | Restore analytical services quickly |

---

# Existing Architecture Assumption

## Current Environment
- Separate DBSTAT reporting/data warehouse environment
- Analytical workloads
- Large reporting queries
- ETL/Data ingestion from Production OLTP systems
- Lower transaction rate compared to OLTP production database

---

# Proposed AWS Target Architecture

## Architecture Objective
Migrate the 40TB DBSTAT analytical warehouse to AWS RDS for Db2 and establish continuous replication from the Production AWS RDS Db2 OLTP environment.

---

# AWS Target Architecture Diagram

```mermaid
flowchart TB

    subgraph PROD["AWS Production Region"]
        direction TB

        PRODDB["AWS RDS for Db2\n8TB Production OLTP"]
    end

    subgraph DWH["AWS Analytics Environment"]
        direction TB

        WH["AWS RDS for Db2\n40TB DBSTAT Warehouse"]
    end

    USERS["Analytics / Reporting Users"]

    PRODDB -->|CDC / Q Replication / ETL| WH

    USERS --> WH

    style PRODDB fill:#d4f4dd
    style WH fill:#e1ecf4
```

---

# Recommended Migration Strategy

Unlike the OLTP production database:
- 48-hour downtime is acceptable.
- Continuous active-active replication is not required during migration.
- Focus is on:
  - Migration speed
  - Bulk data movement
  - Minimal reporting disruption

---

# Recommended Enterprise Migration Approach

## Preferred Strategy
Use:
- Native Backup Restore
OR
- Parallel Export/Import
OR
- AWS Snowball (if network bandwidth insufficient)

followed by:
- CDC/Q Replication from Production OLTP database after migration completion.

---

# Migration Execution Plan

# Phase 1 – Environment Assessment

## Capture
- Warehouse size and growth
- Large tables and partitions
- ETL schedules
- Reporting windows
- Peak analytical workloads
- Compression usage
- Storage consumption

## Purpose
Determine:
- AWS storage sizing
- Query workload requirements
- Migration duration estimation

---

# Phase 2 – AWS Environment Build

## Configure
- AWS RDS for Db2
- Storage autoscaling
- Optimized storage throughput
- Backup strategy
- Security Groups
- Monitoring and alerts

---

# Phase 3 – Initial Bulk Migration

## Recommended Methods

| Method | Use Case |
|---|---|
| Native Backup Restore | Fastest for Db2-to-Db2 |
| Parallel Export/Import | Table-level migration |
| AWS Snowball | Limited network bandwidth |
| rsync/offline copy | Large offline datasets |

## Enterprise Recommendation
For 40TB warehouse:
- Native compressed backup/restore is preferred.
- Parallel restore streams recommended.

---

# Phase 4 – Validation

## Perform
- Row count validation
- Object validation
- ETL validation
- Reporting query validation
- Aggregation consistency checks
- Performance benchmarking

---

# Phase 5 – Enable Ongoing Replication

## Post Migration Requirement
After migration:
```text
Production AWS RDS Db2 --> DBSTAT Warehouse AWS RDS Db2
```

## Recommended Technologies
- IBM Q Replication
- IBM CDC
- Kafka-based ingestion
- ETL tools

---

# Recommended Replication Design

## Production to Warehouse Replication

```mermaid
flowchart LR

    PROD["AWS RDS Db2\nProduction OLTP"]

    CDC["CDC / Q Replication"]

    DWH["AWS RDS Db2\nDBSTAT Warehouse"]

    PROD --> CDC --> DWH
```

---

# Why Separate Warehouse Replication?

## Benefits
- Offload reporting from OLTP production database
- Improve production application performance
- Enable large analytical workloads
- Support downstream BI/reporting tools
- Enable near real-time analytics

---

# Minimal Query Impact Strategy

## During Migration
- Perform migration during reporting off-hours if possible.
- Use backup snapshots to reduce production impact.
- Limit long-running extraction queries.
- Use parallelized data movement.

## Goal
Prevent heavy warehouse migration workload from affecting production OLTP operations.

---

# AWS Sizing Considerations

| Area | Purpose |
|---|---|
| Storage Throughput | Large analytical scans |
| Memory Sizing | Query performance |
| CPU Sizing | Aggregation workloads |
| Autoscaling | Future warehouse growth |
| IO Optimization | Reporting performance |

---

# Enterprise Best Practices

| Area | Best Practice |
|---|---|
| Migration Method | Native backup/restore for very large datasets |
| Replication | CDC/Q Replication post migration |
| Reporting Isolation | Separate warehouse instance |
| Validation | Aggregation and ETL validation |
| Storage | Autoscaling enabled |
| Performance | Optimize for analytical workload |
| Monitoring | CloudWatch and Enhanced Monitoring |

---

# Key Industry Recommendations

## Avoid
- Running heavy migration jobs during peak reporting hours
- Direct analytical workloads on OLTP production database
- Single-threaded data movement for 40TB datasets

## Recommended
- Parallel backup/restore
- Compression-enabled backups
- Dedicated warehouse sizing
- Separate replication pipelines
- Post-migration performance tuning

---

# Conclusion

The proposed solution provides a scalable and enterprise-grade migration strategy for the 40TB DBSTAT analytical warehouse to AWS RDS for Db2.

The design:
- Meets the 48-hour migration window
- Minimizes reporting disruption
- Enables ongoing Production-to-Warehouse replication
- Separates OLTP and analytical workloads
- Provides scalable cloud-native reporting architecture
