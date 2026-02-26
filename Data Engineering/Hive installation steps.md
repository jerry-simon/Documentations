# 🐝 HIVE INSTALLATION & EXECUTION GUIDE (macOS – Apple Silicon)

## ✅ PART 1 — Full Hive Installation & Configuration

### 1️⃣ Prerequisites

Ensure the following are installed:

```
java -version
hadoop version
brew --version
```

You should already have:

```
JDK 21
Hadoop (via Homebrew)
HDFS running successfully
```

### 2️⃣ Install Hive via Homebrew

```
brew install hive
```

Verify installation:

```
hive --version
```

Expected output example:

```
Hive 4.2.0
```

### 3️⃣ Start HDFS (Very Important)

Before running Hive, HDFS must be running.

Check:

```
jps
```

You should see:

```
NameNode
DataNode
```

If not, start it:

```start-dfs.sh```

### 4️⃣ Create Hive Warehouse Directory in HDFS

```hdfs dfs -mkdir -p /user/hive/warehouse```

Verify:

```hdfs dfs -ls /user```

### 5️⃣ Initialize Hive Metastore Schema

Since Hive 4 uses Derby by default:

```schematool -dbType derby -initSchema```

Expected output:

```Initialization script completed```

### 6️⃣ Configure hive-site.xml (Important for Local Setup)

Open:

```nano $(brew --prefix hive)/libexec/conf/hive-site.xml```

Add:

```

<configuration>



  <property>

    <name>hive.server2.enable.doAs</name>

    <value>false</value>

  </property>



</configuration>

```

Save and exit.

```Ctrl + X, Y, Enter```

This prevents impersonation errors.



### 7️⃣ Start HiveServer2

In Terminal 1:

```hiveserver2```

Leave this terminal running.

8️⃣ Connect Using Beeline

Open Terminal 2:

```beeline -u jdbc:hive2://localhost:10000```

If successful, you’ll see:

```0: jdbc:hive2://localhost:10000>```

You are now connected.

## ✅ PART 2 — How To Run Hive After Opening a Fresh Terminal

Every time you restart your Mac or open a new session:

Step 1 — Start HDFS

```start-dfs.sh```

Verify:

```jps```

You must see:

```NameNode

DataNode```

Step 2 — Start HiveServer2

In Terminal 1:

```hiveserver2```

Wait 20–30 seconds.

Step 3 — Connect to Hive

In Terminal 2:

```beeline -u jdbc:hive2://localhost:10000```

You should see:

```0: jdbc:hive2://localhost:10000>```

## 🧠 Optional: Stop Everything Cleanly

To stop:

Stop HiveServer2:

```Ctrl + C```

Stop HDFS:

```stop-dfs.sh```

## 🎯 Quick Mental Model



Component	Role



HDFS	Storage layer



Hive Metastore	Metadata



HiveServer2	Query server



Beeline	Client



## Architecture:



Beeline → HiveServer2 → Metastore → HDFS



## 🚀 You Now Have



A production-style Big Data stack running locally:



Java ✔



Hadoop ✔



HDFS ✔



Hive ✔



JDBC access ✔

