# Hands-on with Azure HDInsight

Let's explore how to deploy, manage, and process data on an **Apache Spark cluster** in **Azure HDInsight**.<br />
This exercise covers setting up the cluster, ingesting data, running a Spark job, and managing the cluster for big data processing.

## Objective

Deploy an **Apache Spark cluster** on **Azure HDInsight** and manage it to process a sample big data workload, followed by real-time scaling and monitoring.

### Step-by-step guide

---

### **Step 1: Create an Azure HDInsight cluster**

**Objective**: Set up an Apache Spark cluster on **Azure HDInsight**.

- **Navigate to the Azure Portal**:
  - Go to the [Azure Portal](https://portal.azure.com).
- **Create a new HDInsight Cluster**:
  - In the Azure portal, select **Create a resource** and search for **HDInsight**.
  - Click on **Apache Spark Cluster**, then click **Create**.
- **Configure the Cluster**:
  - **Basics**:
    - Choose your **Subscription** and **Resource Group**.
    - Enter a **Cluster name** (e.g., `spark-cluster-demo`).
    - Select **Cluster type**: Choose **Spark**.
    - Select **Version**: Choose a supported Spark version (e.g., **Spark 3.0**).
  - **Cluster Size**:
    - Define the number of **worker nodes** and the **VM size** for each node.
    - For testing purposes, you can use a smaller cluster with fewer nodes.
  - **Storage**:
    - Choose a **storage account**. You can either create a new one or use an existing **Azure Blob Storage** account.
  - **Security + Networking**:
    - Configure **network settings** and **cluster access permissions**.
  - **Review + Create**:
    - Review the configuration and click **Create** to deploy the cluster. It may take several minutes for the cluster to be provisioned.

**Expected result**: A fully provisioned Apache Spark cluster ready for data ingestion and processing.

---

### **Step 2: Upload Paris 2024 data to Azure Storage**

**Objective**: Upload a sample dataset (e.g., a CSV file) related to the **Paris 2024 Olympic Games** to **Azure Blob Storage** to use with the Apache Spark cluster.

- **Example Dataset**: [Paris 2024 Event Schedules](https://data.paris2024.org/explore/?sort=modified)
  - You can upload a dataset that contains **event schedules**, **venue information**, or **participant data**.

- **Navigate to Azure Storage account**:
  - Go to the **Azure portal**, and open the **storage account** associated with your HDInsight cluster.
- **Upload Data**:
  - In the storage account, go to **Containers** and select a container (or create a new one).
  - Click **Upload** to add your data files (e.g., a CSV file from the Paris 2024 dataset).

**Expected result**: Your data (e.g., a CSV file from the Paris 2024 dataset) is now uploaded to **Azure Blob Storage** and ready for processing by the Spark cluster.

---

### **Step 3: Run a Spark job on the cluster**

**Objective**: Submit and run a **PySpark job** to process the uploaded Paris 2024 data on the Spark cluster.

- **Access the cluster**:
  - In the **Azure portal**, navigate to your HDInsight cluster and select **Cluster Dashboard**.
- **Submit a Spark Job**:
  - Use either **Jupyter Notebook** or **SSH** into the cluster to submit a **Spark job**.
  - Example job using **PySpark**:
    ```python
    from pyspark.sql import SparkSession
    spark = SparkSession.builder.appName("example").getOrCreate()
    
    # Load data from Azure Blob Storage
    df = spark.read.csv("wasbs://<container>@<storage_account>.blob.core.windows.net/<file.csv>")
    
    # Perform simple transformation
    df_filtered = df.filter(df['_c0'] > 100)  # Example filtering
    df_filtered.show()
    ```
- **Monitor the job**:
  - Monitor job progress via the **Apache Ambari** dashboard, accessible through the cluster's dashboard link in the Azure portal.

**Expected result**: The Spark job processes the Paris 2024 data, performing transformations such as filtering or aggregating the event or participant data.

---

### **Step 4: Manage the cluster**

**Objective**: Monitor and manage your Spark cluster to ensure performance, scalability, and security.

- **Scale the Cluster**:
  - In the **Azure portal**, navigate to your HDInsight cluster and select **Scale Cluster**.
  - Adjust the number of **worker nodes** to scale the cluster up or down based on workload.
- **Monitor Cluster Health**:
  - Use **Azure Monitor** and **Apache Ambari** to keep track of cluster health, performance metrics, and logs.
- **Secure the Cluster**:
  - Implement security practices such as:
    - **Azure Active Directory (AAD)** integration for access management.
    - **Role-based access control (RBAC)** to restrict permissions based on user roles.

**Expected result**: The cluster is securely managed and can scale efficiently based on workload demands.

---

### **Summary of the activity**

In this exercise, you have learned how to:
1. Deploy an **Apache Spark cluster** on **Azure HDInsight**.
2. Upload **Paris 2024 data** to **Azure Blob Storage** for use with the cluster.
3. Submit and run a **Spark job** using **PySpark** to process and transform the Paris 2024 data.
4. Monitor, scale, and secure the cluster to ensure performance and efficiency.

This hands-on session demonstrates how to manage and process **Paris 2024 Olympic Games** datasets on a scalable, secure platform using **Azure HDInsight**.
