# Hands-on with Azure Databricks

Let's explore how to deploy, manage, and process data on an **Apache Spark cluster** using **Azure Databricks**.  
This exercise covers setting up a Databricks workspace, ingesting data, running a Spark job in a notebook, and monitoring the cluster.

---

## Objective

Deploy an **Apache Spark environment** using **Azure Databricks**, process a sample dataset related to the **Paris 2024 Olympic Games**, and manage the cluster efficiently using Databricks tools.

---

## Step-by-step guide

---

### **Step 1: Create an Azure Databricks Workspace**

**Objective:** Set up your Databricks environment on Azure.

1. **Go to the Azure Portal:**  
   - Navigate to [Azure Portal](https://portal.azure.com).

2. **Create a Databricks Workspace:**  
   - Click **Create a resource** → Search for **Azure Databricks**.  
   - Click **Create**.

3. **Configure the Workspace:**  
   - **Subscription:** Choose your active (free trial) subscription.  
   - **Resource Group:** Create a new one or select an existing one.  
   - **Workspace Name:** e.g., `databricks-paris2024`.  
   - **Region:** Choose a nearby region (e.g., *West Europe*).  
   - **Pricing Tier:** Select **Trial (Premium)** if available — this uses your free credits.  
   - Click **Review + Create** → then **Create**.

4. **Launch Databricks:**  
   - Once deployment is complete, go to **Resource** → click **Launch Workspace**.  
   - You’ll be redirected to the **Databricks web interface**.

**Expected result:**  
A running **Azure Databricks workspace**, ready to create Spark clusters and notebooks.

---

### **Step 2: Create and Configure a Cluster**

**Objective:** Launch a small Spark cluster for testing.

1. In Databricks, go to **Compute → Create Cluster**.  
2. Set up your cluster:
   - **Cluster Name:** `spark-cluster-demo`
   - **Cluster Mode:** *Single Node* (to reduce cost)
   - **Databricks Runtime:** Choose a recent **Runtime with Apache Spark 3.x** (e.g., 13.x)
   - **Termination:** Enable **Auto Termination** (e.g., after 15 minutes)
3. Click **Create Cluster**.

**Expected result:**  
A lightweight **Spark cluster** running and ready for PySpark processing.

---

### **Step 3: Upload the Paris 2024 Dataset**

**Objective:** Upload a CSV dataset to Databricks FileStore for analysis.

- **Example Dataset:** [Paris 2024 Event Schedules]([https://data.paris2024.org/explore/?sort=modified](https://github.com/rythamsaini/Paris-Olympic-Games-2024))

#### Upload Data:
1. In Databricks, go to **Data → Add Data → Upload File**  
2. Select your CSV file (e.g., `Paris2024_Events.csv`)  
3. Upload it to **DBFS (Databricks File System)** under `/FileStore/tables/`

#### Verify Upload:
```python
df = spark.read.csv("/FileStore/tables/Paris2024_Events.csv", header=True, inferSchema=True)
df.show(5)
```

**Expected result:**  
Your dataset is now uploaded and accessible within Databricks for Spark processing.

---

### **Step 4: Run a Spark Job in a Notebook**

**Objective:** Process the Paris 2024 dataset using PySpark.

#### Create a Notebook:
1. In Databricks, go to **Workspace → Create → Notebook**  
2. **Name:** `Paris2024_SparkJob`  
3. **Language:** Python (PySpark)  
4. **Cluster:** Attach your cluster (`spark-cluster-demo`)

#### Example PySpark Job:
```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col

spark = SparkSession.builder.appName("Paris2024").getOrCreate()

# Load data from DBFS
df = spark.read.csv("/FileStore/tables/Paris2024_Events.csv", header=True, inferSchema=True)

# Display dataset structure
df.printSchema()

# Filter events with more than 100 participants
df_filtered = df.filter(col("participants") > 100)

# Show results
df_filtered.show(10)
```

#### Visualize Data:
- In the output table view, click **+** → **Visualization**  
- Create a **bar chart** or **pie chart** based on columns like *sport*, *venue*, or *participants*.

**Expected result:**  
The Spark job runs successfully and produces filtered, visualized data about Paris 2024 events.

---

### **Step 5: Manage and Monitor Your Cluster**

**Objective:** Optimize resources and monitor performance.

1. **Monitor Cluster Health:**  
   - Go to **Compute → spark-cluster-demo → Metrics**  
   - Observe CPU usage, memory, and active tasks.

2. **Enable Auto Termination:**  
   - Ensures the cluster stops automatically after inactivity.

3. **Stop the Cluster Manually:**  
   - Go to **Compute → Stop** to avoid using credits unnecessarily.

4. **Security:**  
   - Databricks uses Azure AD authentication.  
   - For shared projects, assign roles or separate notebooks per user.

**Expected result:**  
You efficiently manage your cluster resources and maintain a secure environment.

---

### **Summary of the Activity**

In this exercise, you have learned how to:
1. Deploy an **Apache Spark environment** with **Azure Databricks**.  
2. Upload and manage **Paris 2024 data** in **DBFS**.  
3. Submit and run a **PySpark job** to process and analyze the dataset.  
4. Visualize results and **monitor your cluster’s performance**.  
5. Optimize resources through **auto-termination and proper management**.

This hands-on lab demonstrates how to manage and process **Paris 2024 Olympic Games datasets** on a scalable, cloud-based platform using **Azure Databricks** — ideal for data analysis, transformation, and visualization.

---

✅ **Tip:**  
If you finish early, try experimenting with:
- Grouping events by sport (`df.groupBy("sport").count().show()`)
- Plotting participation per country or venue  
- Using Databricks SQL for interactive queries
