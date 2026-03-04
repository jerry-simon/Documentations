# Essential `SET` Commands for Apache Hive Sessions (Local Development)

## 1. Purpose

This document lists **commonly used Hive `SET` configuration commands** that should be applied when starting a Hive session. These settings help enable dynamic partitions, improve query flexibility, and avoid common execution issues during data engineering workflows.

These commands are particularly useful for:

* Data loading
* Partitioned tables
* ETL workflows
* Local development environments

Note: The execution engine (`hive.execution.engine = mr`) is already configured permanently in `hive-site.xml`, so it is **not included here**.

---

# 2. Mandatory Session Configuration

These settings are recommended to run at the start of every Hive session.

```sql
SET hive.exec.dynamic.partition = true;
SET hive.exec.dynamic.partition.mode = nonstrict;
SET hive.exec.max.dynamic.partitions = 1000;
SET hive.exec.max.dynamic.partitions.pernode = 100;
SET hive.exec.max.created.files = 100000;
SET hive.exec.parallel = true;
SET hive.mapred.mode = nonstrict;
```

## Explanation

### Enable Dynamic Partitioning

```sql
SET hive.exec.dynamic.partition = true;
```

Allows Hive to automatically create partition values during insert operations.

Example use case:

```sql
INSERT INTO table PARTITION (year)
SELECT col1, col2, year FROM source_table;
```

---

### Allow Fully Dynamic Partition Inserts

```sql
SET hive.exec.dynamic.partition.mode = nonstrict;
```

Default Hive mode is **strict**, which requires at least one static partition column.

Setting `nonstrict` allows Hive to dynamically generate all partitions.

---

### Limit Total Dynamic Partitions

```sql
SET hive.exec.max.dynamic.partitions = 1000;
```

Controls the maximum number of partitions created in a single query.

Prevents runaway partition creation.

---

### Limit Dynamic Partitions Per Node

```sql
SET hive.exec.max.dynamic.partitions.pernode = 100;
```

Limits the number of partitions created by each mapper/reducer node.

Helps prevent excessive resource consumption.

---

### Limit File Creation

```sql
SET hive.exec.max.created.files = 100000;
```

Prevents a job from generating too many output files.

This is important in large ETL operations.

---

### Enable Parallel Execution

```sql
SET hive.exec.parallel = true;
```

Allows Hive to execute independent stages of a query in parallel.

This improves performance for complex queries.

---

### Disable Strict Query Mode

```sql
SET hive.mapred.mode = nonstrict;
```

Allows operations such as:

* Full table scans
* Cartesian joins
* Unpartitioned queries

Useful for development environments.

---

# 3. Optional Performance Settings

These settings improve performance in certain scenarios.

### Enable Local Mode for Small Jobs

```sql
SET hive.exec.mode.local.auto = true;
```

Allows Hive to run small jobs locally instead of launching a MapReduce job.

---

### Enable Automatic Join Optimization

```sql
SET hive.auto.convert.join = true;
```

Hive will convert large joins into **map-side joins** when possible.

---

### Control Join Size Threshold

```sql
SET hive.mapjoin.smalltable.filesize = 25000000;
```

Defines the size threshold for tables to be considered "small" during map joins.

---

# 4. Example Session Initialization

When starting a new Hive session, the following setup block can be executed:

```sql
SET hive.exec.dynamic.partition = true;
SET hive.exec.dynamic.partition.mode = nonstrict;
SET hive.exec.max.dynamic.partitions = 1000;
SET hive.exec.max.dynamic.partitions.pernode = 100;
SET hive.exec.max.created.files = 100000;
SET hive.exec.parallel = true;
SET hive.mapred.mode = nonstrict;
SET hive.exec.mode.local.auto = true;
SET hive.auto.convert.join = true;
SET hive.mapjoin.smalltable.filesize = 25000000;
```

---

# 5. Recommended Workflow

Typical Hive workflow during development:

1. Start HiveServer2
2. Connect using Beeline
3. Run session configuration (`SET` commands)
4. Create or load tables
5. Execute queries
6. Validate results

---

# 6. Key Takeaway

For most Hive data engineering workflows, the two **most important settings** are:

```sql
SET hive.exec.dynamic.partition = true;
SET hive.exec.dynamic.partition.mode = nonstrict;
```

These enable flexible partition-based data ingestion and are widely used in production ETL pipelines.
