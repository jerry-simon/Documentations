# Hive Local Setup – Silent INSERT Failure Fix (Mac + Homebrew + Hadoop)

## 1. Environment Overview

Local stack used:

MacOS
↓
Hadoop (HDFS)
↓
MapReduce Execution Engine
↓
HiveServer2
↓
Beeline Client

Hive stores table data in the HDFS warehouse directory:

```
/user/hive/warehouse
```

Metadata such as tables, schemas, and partitions are stored in the **Hive Metastore**, while actual data files live in **HDFS**.

---

# 2. Observed Problem

When executing:

```sql
INSERT INTO sales VALUES (1, 2000.00);
```

Hive returned:

```
Error running query (state=,code=0)
```

No detailed error message was shown.

However, other commands worked normally:

* `SHOW TABLES`
* `DESCRIBE sales`
* `SHOW DATABASES`

This indicates that **Hive Metastore and HiveServer2 were functioning correctly**, but the **query execution layer was failing**.

---

# 3. Root Cause

Hive requires a **query execution engine** to run operations such as:

* INSERT
* CTAS
* Data transformations
* Aggregations

Common execution engines include:

* MapReduce
* Tez
* Spark

In the local installation, **no execution engine was configured**, so Hive could parse the query but **could not execute the job**.

Therefore:

* Metadata queries succeeded.
* Data write queries failed silently.

---

# 4. Correct Solution

Configure Hive to use **MapReduce** as the execution engine.

### Step 1 — Temporary Fix (inside Beeline)

```sql
SET hive.execution.engine=mr;
```

After this change, the INSERT query worked successfully.

---

### Step 2 — Permanent Fix (Hive Configuration)

Edit Hive configuration:

```
$(brew --prefix hive)/libexec/conf/hive-site.xml
```

Add the following property inside `<configuration>`:

```xml
<property>
  <name>hive.execution.engine</name>
  <value>mr</value>
</property>
```

Restart HiveServer2 afterwards.

---

# 5. HDFS Warehouse Permissions

Hive writes table data into:

```
/user/hive/warehouse
```

For local development, full permissions were granted:

```bash
hdfs dfs -chmod -R 777 /user
hdfs dfs -chmod -R 777 /user/hive
hdfs dfs -chmod -R 777 /user/hive/warehouse
```

This ensures HiveServer2 can create table directories and write data files.

(For production clusters, permissions should be restricted.)

---

# 6. Final Working Test

After configuration, the following test confirmed the environment works correctly:

```sql
CREATE TABLE hive_test(id INT);

INSERT INTO hive_test VALUES (1),(2),(3);

SELECT * FROM hive_test;
```

Expected output:

```
1
2
3
```

This confirms:

* HiveServer2 is running
* Execution engine is working
* HDFS write operations succeed
* Hive Metastore is updating correctly

---

# 7. Key Lesson

When Hive shows:

```
Error running query (state=,code=0)
```

and metadata queries work, the likely issue is:

**Hive execution engine is not configured.**

Setting:

```
hive.execution.engine = mr
```

resolves the problem in local Hadoop installations.

---

# 8. Final Working Configuration Summary

Hive configuration (`hive-site.xml`):

```xml
<property>
  <name>hive.execution.engine</name>
  <value>mr</value>
</property>

<property>
  <name>hive.metastore.warehouse.dir</name>
  <value>/user/hive/warehouse</value>
</property>
```

HDFS permissions:

```
/user/hive/warehouse  → writable by Hive
```

Execution engine:

```
MapReduce
```

Hive client:

```
Beeline → HiveServer2 → MapReduce → HDFS
```

---

# 9. Verified Working Workflow

1. Start Hadoop
2. Start HiveServer2
3. Connect using Beeline
4. Create tables
5. Insert data
6. Query data

All operations now execute successfully.
