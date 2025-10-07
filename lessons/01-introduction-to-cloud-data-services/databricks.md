# Hands-on with Databricks (Paris 2024 TP)

Let's explore how to deploy, manage, and process data on an **Apache Spark cluster** in **Databricks Community Edition**.<br />
This exercise covers setting up the cluster, ingesting data, running a Spark job, and managing the environment for big data processing.

## Objective

Deploy an **Apache Spark cluster** on **Databricks Community Edition** and manage it to process a sample big data workload, followed by real-time monitoring and visualization.

### Step-by-step guide

---

### **Step 1: Create a Databricks Spark cluster**

**Objective**: Set up an Apache Spark cluster on **Databricks Community Edition**.

- **Create a Databricks account**:
  - Go to [Databricks Community Edition](https://databricks.com/try-databricks) and sign up.
- **Log in to your workspace**.
- **Create a cluster**:
  - Navigate to **Clusters → Create Cluster**
  - **Cluster Name**: `spark-cluster-demo`
  - **Cluster Mode**: `Standard`
  - **Databricks Runtime Version**: Spark 3.3+ (or the latest available)
  - **Worker Type**: Small (Databricks CE automatically sets number of workers)
  - **Auto Termination**: 30 minutes
  - Click **Create Cluster**.

**Expected result**: A fully functional Spark cluster ready to process data.

---

### **Step 2: Upload Paris 2024 data**

**Objective**: Upload a sample dataset related to the **Paris 2024 Olympic Games** to **Databricks DBFS** for use with the Spark cluster.

- **Example Dataset**: [Paris 2024 Event Schedules](https://data.paris2024.org/explore/?sort=modified)
  - The dataset can contain **event schedules**, **venue information**, or **participant data**.

- **Upload Data**:
  - In Databricks, go to **Data → Add Data → Upload File**
  - Select your CSV file (e.g., `Paris2024_Events.csv`) and upload it to DBFS
- **Verify Upload**:
```python
df = spark.read.csv("/FileStore/tables/Paris2024_Events.csv", header=True, inferSchema=True)
df.show(5)
```
**Expected result**: Your dataset is uploaded and accessible in Databricks for Spark processing.

---

### **Step 3: Run a Spark job on the cluster**

**Objective**: Process the Paris 2024 dataset using **PySpark**.

- **Create a Notebook**:
  - Go to **Workspace → Create → Notebook**
  - Select **Language**: Python (PySpark)
  - Attach it to your cluster (`spark-cluster-demo`)

- **Example PySpark job**:
```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col

spark = SparkSession.builder.appName("Paris2024").getOrCreate()

# Load data from DBFS
df = spark.read.csv("/FileStore/tables/Paris2024_Events.csv", header=True, inferSchema=True)

# Filter events with more than 100 participants
df_filtered = df.filter(col("participants") > 100)

# Show results
df_filtered.show()
```

- **Visualize Data**:
  - Use Databricks visualization tools (bar charts, pie charts, etc.) within the notebook.

**Expected result**: The Spark job processes and filters the Paris 2024 data.

---

### **Step 4: Manage and monitor the cluster**

**Objective**: Monitor your Spark cluster and manage resources efficiently.

- **Cluster Monitoring**:
  - Navigate to **Clusters → spark-cluster-demo → Metrics**
  - Observe CPU, memory, and job execution details.

- **Auto Termination**:
  - Ensure **Auto Termination** is enabled to save resources when the cluster is idle.

- **Stop Cluster**:
  - Manually stop the cluster after finishing the TP to avoid unnecessary usage.

- **Security**:
  - Databricks CE provides user-based login access.
  - Optionally, create separate notebooks for different user roles to simulate RBAC.

**Expected result**: The cluster is monitored, resource usage is optimized, and data is processed securely.

---

### **Summary of the activity**

In this exercise, you have learned how to:

1. Deploy an **Apache Spark cluster** on Databricks Community Edition.
2. Upload **Paris 2024 data** to Databricks DBFS for processing.
3. Submit and run a **Spark job** using PySpark.
4. Visualize results and monitor cluster performance.

This hands-on session demonstrates how to manage and process **Paris 2024 Olympic Games** datasets on a scalable, secure platform using **Databricks Community Edition**.
