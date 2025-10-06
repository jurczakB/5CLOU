# Hands-on with Microsoft Fabric Services

Let's explore how to ingest, transform, analyze, and visualize **Paris 2024 Olympic Games** data using **Microsoft Fabric**.<br />
We’ll focus on using data such as venue locations, event schedules, or ticket sales, and apply it across **Data Factory**, **Synapse Analytics**, and **Power BI**.

## Objective

Set up a data pipeline using **Microsoft Fabric** to ingest Paris 2024 data, process it, and visualize key metrics such as event locations, athlete participation, or ticket sales trends.

### step-by-step huide

---

### **Step 1: Ingest Paris 2024 data with Data Factory**

**Objective**: Use **Data Factory** to ingest data from the **Paris 2024 dataset** (e.g., event schedules or venues) into **OneLake**.

- **Navigate to Data Factory**:
  - Go to the **Microsoft Fabric** portal and access **Data Factory**.
- **Create a new Data Pipeline**:
  - In the Data Factory workspace, click on **Create pipeline**.
  - **Source**: Configure the data source as a **Paris 2024 dataset** (e.g., **event schedule CSV**).
  - **Destination**: Set the destination as **OneLake** for storage.
  - **Run the pipeline**: Execute the pipeline to ingest the data from the Paris 2024 dataset repository into OneLake.

**Example dataset**: [Paris 2024 Event Schedules](https://data.paris2024.org/explore/?sort=modified)

**Expected result**: The event schedule data should be available in **OneLake**, ready for transformation and analysis.

---

### **Step 2: Transform data in Data Factory**

**Objective**: Clean and transform the ingested Paris 2024 data (e.g., filter by venue or event type) using **Data Factory**.

- **Add Data Transformation**:
  - In the same pipeline, add a **Data Flow** activity.
  - **Transformation**: Apply transformations such as filtering for a specific venue (e.g., **Stade de France**) or event type (e.g., **Track and Field**).
  - **Destination**: Output the cleaned and transformed data to a new **OneLake** location for further analysis.
  - **Run the transformation**: Execute the pipeline and verify that the transformed data is correctly stored in OneLake.

**Expected result**: The data should now be filtered and structured, allowing for further analysis (e.g., filtering by venue or event).

---

### **Step 3: Analyze data with Synapse Analytics**

**Objective**: Use **Synapse Analytics** to perform real-time analytics on the transformed Paris 2024 data.

- **Navigate to Synapse Studio**:
  - Go to **Synapse Analytics** in the Microsoft Fabric portal.
- **Create a new workspace**:
  - Set up a **SQL pool** or **Spark pool** to run queries on the transformed data.
- **Connect to Data**:
  - In **Synapse Studio**, connect to the data stored in **OneLake** (e.g., event data by venue).
- **Run Analytics Queries**:
  - Write a query to analyze the data (e.g., count the number of events per venue).
    ```sql
    SELECT venue, COUNT(event) AS total_events
    FROM paris2024_event_data
    GROUP BY venue;
    ```
  - **Monitor progress**: Use **Synapse Studio** to monitor the job and visualize results.

**Expected result**: You will get real-time insights such as the number of events hosted by each venue during the Paris 2024 Olympics.

---

### **Step 4: Visualize data with Power BI**

**Objective**: Create a real-time dashboard in **Power BI** that visualizes Paris 2024 event data.

- **Navigate to Power BI**:
  - Go to **Power BI** in the Microsoft Fabric portal.
- **Connect to OneLake Data**:
  - Import the processed event or venue data from **OneLake**.
- **Create a dashboard**:
  - Build an interactive dashboard showing key metrics such as **total events by venue**, **athlete participation**, or **ticket sales trends**.
  - Use visualizations such as **bar charts**, **maps** for venue locations, and **KPIs** for tracking event schedules.
- **Publish the dashboard**:
  - Publish the report and share it with stakeholders or embed it in a website.

**Expected result**: A fully interactive Power BI dashboard showing real-time data from Paris 2024, such as event locations, ticket sales, or venue participation.

---

### **Step 5: Manage and monitor in Microsoft Fabric**

**Objective**: Use Microsoft Fabric’s management tools to monitor and optimize the data pipeline.

- **Monitor pipeline health in Data Factory**:
  - Check the status and performance of the data ingestion and transformation pipeline using Data Factory’s monitoring tools.
- **Scale synapse Analytics pools**:
  - Adjust the size of your **SQL pool** or **Spark pool** in **Synapse Analytics** based on workload.
- **Secure your Data**:
  - Implement security best practices, such as **Azure Active Directory (AAD)** for access control and **role-based access control (RBAC)** to manage permissions.

**Expected result**: A secure, scalable, and well-managed environment for processing Paris 2024 data and generating insights.

---

### **Summary of the Activity**

In this exercise, you have learned how to:
1. Ingest Paris 2024 data into **OneLake** using **Data Factory**.
2. Perform data transformations to filter or clean event-related data.
3. Use **Synapse Analytics** to analyze real-time event metrics.
4. Visualize insights from the Paris 2024 dataset in **Power BI**.
5. Manage the entire workflow, ensuring secure and scalable data handling.

This hands-on session demonstrates how to leverage **Microsoft Fabric** to manage and analyze real-world datasets from the **Paris 2024 Olympic Games** repository.
