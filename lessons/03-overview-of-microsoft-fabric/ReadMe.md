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

  ```python
  # ============================================================================
  # PIPELINE MEDALLION - ATHLÈTES PARIS 2024
  # Architecture : Bronze → Silver → Gold
  # ============================================================================
  
  # ===== CELLULE 1 : Vérification de l'environnement =====
  
  
  # ===== CELLULE 2 : BRONZE LAYER - Ingestion des données brutes =====
  print("📥 BRONZE LAYER - Chargement des données brutes...\n")
  
  # Adapter le chemin selon votre structure
  file_path = "/Files/bronze/SampleData/athletes.csv"  # Modifiez si nécessaire
  
  # Lire le CSV avec toutes les options
  df_bronze = spark.read.format("csv") \
      .option("header", "true") \
      .option("inferSchema", "true") \
      .option("encoding", "UTF-8") \
      .option("multiLine", "true") \
      .option("escape", '"') \
      .load(file_path)
  
  print(f"✅ Données chargées : {df_bronze.count()} athlètes")
  print(f"📊 Colonnes : {len(df_bronze.columns)}\n")
  
  # Aperçu des données
  print("📋 Aperçu des 5 premières lignes :")
  df_bronze.show(5, truncate=False)
  
  # Schéma des données
  print("\n🔍 Schéma des colonnes :")
  df_bronze.printSchema()
  
  # Statistiques de base
  print("\n📈 Statistiques des valeurs nulles par colonne :")
  null_counts = df_bronze.select([
      count(when(col(c).isNull(), c)).alias(c) for c in df_bronze.columns
  ])
  null_counts.show(vertical=True)
  
  # Sauvegarder en table Bronze
  df_bronze.write.mode("overwrite").saveAsTable("athletes_bronze")
  print("\n✅ Table 'athletes_bronze' créée avec succès !")
  
  print("\n" + "="*80 + "\n")
  
  
  # ===== CELLULE 3 : SILVER LAYER - Nettoyage et transformation =====
  print("🔄 SILVER LAYER - Nettoyage et transformation des données...\n")
  
  # Lire depuis Bronze
  df_silver = spark.read.table("athletes_bronze")
  
  # 1. Supprimer les doublons complets
  df_silver = df_silver.dropDuplicates()
  print(f"✅ Doublons supprimés")
  
  # 2. Nettoyer les colonnes clés (supprimer les lignes où ces colonnes sont nulles)
  colonnes_cles = ["code", "name", "country_code"]
  df_silver = df_silver.dropna(subset=colonnes_cles)
  print(f"✅ Lignes avec valeurs nulles clés supprimées")
  
  # 3. Nettoyer les espaces dans les colonnes texte
  colonnes_texte = ["name", "name_short", "name_tv", "country", "country_full", 
                    "nationality", "nationality_full", "birth_place", "birth_country",
                    "residence_place", "residence_country", "disciplines", "events"]
  
  for colonne in colonnes_texte:
      if colonne in df_silver.columns:
          df_silver = df_silver.withColumn(colonne, trim(col(colonne)))
  
  print(f"✅ Espaces superflus nettoyés")
  
  # 4. Standardiser les codes pays en majuscules
  df_silver = df_silver.withColumn("country_code", upper(col("country_code")))
  df_silver = df_silver.withColumn("nationality_code", upper(col("nationality_code")))
  
  # 5. Normaliser le genre
  df_silver = df_silver.withColumn("gender", upper(col("gender")))
  
  # 6. Nettoyer les valeurs numériques (height et weight)
  # Convertir en numérique et remplacer les valeurs aberrantes par NULL
  df_silver = df_silver.withColumn(
      "height", 
      when((col("height") > 0) & (col("height") < 300), col("height")).otherwise(None)
  )
  df_silver = df_silver.withColumn(
      "weight", 
      when((col("weight") > 0) & (col("weight") < 300), col("weight")).otherwise(None)
  )
  
  print(f"✅ Valeurs numériques nettoyées")
  
  # 7. Convertir la date de naissance en format date
  df_silver = df_silver.withColumn(
      "birth_date",
      when(col("birth_date").isNotNull(), col("birth_date").cast("date")).otherwise(None)
  )
  
  print(f"✅ Dates converties")
  
  print(f"\n📊 Résultat : {df_silver.count()} athlètes après nettoyage")
  
  # Aperçu des données nettoyées
  print("\n📋 Aperçu des données Silver :")
  df_silver.select("code", "name", "gender", "country_code", "disciplines", "height", "weight").show(10)
  
  # Sauvegarder en table Silver
  df_silver.write.mode("overwrite").saveAsTable("athletes_silver")
  print("\n✅ Table 'athletes_silver' créée avec succès !")
  
  print("\n" + "="*80 + "\n")
  
  
  # ===== CELLULE 4 : GOLD LAYER - Agrégations analytiques =====
  print("📊 GOLD LAYER - Création des agrégations analytiques...\n")
  
  # Lire depuis Silver
  df_gold = spark.read.table("athletes_silver")
  
  # ==== AGRÉGATION 1 : Athlètes par pays ====
  print("📍 Agrégation 1 : Athlètes par pays...")
  athletes_by_country = df_gold.groupBy("country_code", "country_full") \
      .agg(count("*").alias("total_athletes")) \
      .orderBy("total_athletes", ascending=False)
  
  athletes_by_country.write.mode("overwrite").saveAsTable("athletes_by_country_gold")
  print(f"✅ Table 'athletes_by_country_gold' créée ({athletes_by_country.count()} pays)")
  athletes_by_country.show(20)
  
  # ==== AGRÉGATION 2 : Athlètes par genre ====
  print("\n👥 Agrégation 2 : Répartition par genre...")
  athletes_by_gender = df_gold.groupBy("gender") \
      .agg(count("*").alias("total_athletes")) \
      .orderBy("total_athletes", ascending=False)
  
  athletes_by_gender.write.mode("overwrite").saveAsTable("athletes_by_gender_gold")
  print(f"✅ Table 'athletes_by_gender_gold' créée")
  athletes_by_gender.show()
  
  # ==== AGRÉGATION 3 : Athlètes par discipline ====
  print("\n🏃 Agrégation 3 : Athlètes par discipline...")
  # Note : disciplines peut contenir plusieurs valeurs séparées (à adapter selon le format)
  athletes_by_discipline = df_gold.groupBy("disciplines") \
      .agg(count("*").alias("total_athletes")) \
      .orderBy("total_athletes", ascending=False)
  
  athletes_by_discipline.write.mode("overwrite").saveAsTable("athletes_by_discipline_gold")
  print(f"✅ Table 'athletes_by_discipline_gold' créée ({athletes_by_discipline.count()} disciplines)")
  athletes_by_discipline.show(20)
  
  # ==== AGRÉGATION 4 : Statistiques physiques par pays ====
  print("\n📏 Agrégation 4 : Statistiques physiques par pays...")
  from pyspark.sql.functions import avg, min, max, stddev
  
  physical_stats = df_gold.groupBy("country_code", "country_full") \
      .agg(
          count("*").alias("total_athletes"),
          avg("height").alias("avg_height"),
          avg("weight").alias("avg_weight"),
          min("height").alias("min_height"),
          max("height").alias("max_height")
      ) \
      .orderBy("total_athletes", ascending=False)
  
  physical_stats.write.mode("overwrite").saveAsTable("physical_stats_by_country_gold")
  print(f"✅ Table 'physical_stats_by_country_gold' créée")
  physical_stats.show(20)
  
  # ==== AGRÉGATION 5 : Athlètes par pays de résidence vs nationalité ====
  print("\n🌍 Agrégation 5 : Pays de résidence vs nationalité...")
  residence_analysis = df_gold.filter(col("residence_country").isNotNull()) \
      .groupBy("residence_country", "nationality") \
      .agg(count("*").alias("total_athletes")) \
      .orderBy("total_athletes", ascending=False)
  
  residence_analysis.write.mode("overwrite").saveAsTable("residence_vs_nationality_gold")
  print(f"✅ Table 'residence_vs_nationality_gold' créée")
  residence_analysis.show(20)
  
  # ==== AGRÉGATION 6 : Top pays par genre ====
  print("\n🏅 Agrégation 6 : Top pays par genre...")
  country_gender = df_gold.groupBy("country_code", "country_full", "gender") \
      .agg(count("*").alias("total_athletes")) \
      .orderBy("country_code", "gender")
  
  country_gender.write.mode("overwrite").saveAsTable("country_gender_distribution_gold")
  print(f"✅ Table 'country_gender_distribution_gold' créée")
  country_gender.show(30)
  
  print("\n" + "="*80 + "\n")
  
  
  # ===== CELLULE 5 : Validation et résumé final =====
  print("✅ VALIDATION FINALE - Résumé du pipeline Medallion\n")
  
  # Lister toutes les tables créées
  print("📋 Tables créées dans le Lakehouse :")
  spark.sql("SHOW TABLES").show(truncate=False)
  
  # Statistiques de chaque couche
  print("\n📊 STATISTIQUES PAR COUCHE :")
  print(f"\n🟤 BRONZE : {spark.read.table('athletes_bronze').count()} lignes brutes")
  print(f"⚪ SILVER : {spark.read.table('athletes_silver').count()} lignes nettoyées")
  print(f"🟡 GOLD   : {spark.sql('SHOW TABLES').filter(col('tableName').like('%gold%')).count()} tables d'agrégation")
  
  # Qualité des données Silver
  print("\n📈 QUALITÉ DES DONNÉES SILVER :")
  df_check = spark.read.table("athletes_silver")
  
  print(f"  - Total athlètes : {df_check.count()}")
  print(f"  - Pays uniques : {df_check.select('country_code').distinct().count()}")
  print(f"  - Disciplines uniques : {df_check.select('disciplines').distinct().count()}")
  print(f"  - Genres : {df_check.select('gender').distinct().count()}")
  
  # Top 5 pays
  print("\n🏆 TOP 5 PAYS PAR NOMBRE D'ATHLÈTES :")
  spark.read.table("athletes_by_country_gold").show(5)
  
  print("\n" + "="*80)
  print("🎉 PIPELINE MEDALLION TERMINÉ AVEC SUCCÈS !")
  print("="*80)
  print("\n📌 PROCHAINES ÉTAPES :")
  print("  1. Ouvrir Power BI dans votre workspace")
  print("  2. Connecter aux tables Gold (*_gold)")
  print("  3. Créer des visualisations interactives")
  print("  4. Publier votre dashboard !")
  print("\n" + "="*80 + "\n")
  ```

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
