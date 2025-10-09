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

- **Example Dataset:** [Paris 2024 Event Schedules](https://data.paris2024.org/explore/?sort=modified)

#### Upload Data:
1. In Databricks, go to **Data → Add Data → Upload File**  
2. Select your CSV file (e.g., `Paris2024_Events.csv`)  
3. Upload it to **DBFS (Databricks File System)** under `/FileStore/tables/`

#### Verify Upload:
```python
df = spark.read.csv("/FileStore/tables/Paris2024_Events.csv", header=True, inferSchema=True)
df.show(5)
