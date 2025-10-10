# 🛒 InstaCart Online Grocery Basket dataset

This repository contains an end-to-end data pipeline built around the **InstaCart Online Grocery Basket dataset**, a real-world dataset that captures customer purchasing behavior across thousands of grocery orders.

---

## 1. Project Overview

- **Dataset Used:**  
  - InstaCart Online Grocery Basket Analysis Dataset

- **Goal of the Exercise:**  
  - Develop a business question, Apply Normalization → Dimensional Modeling → Data Quality Dashboard using the Instacart Market Basket dataset.

- **Team Setup:**  
  - The team collaborated by dividing responsibilities across data modeling, dashboard development, data quality testing, and documentation. Workloads were adjusted according to each member’s availability. 

- **Environment Setup:**  
  - Docker containers ran on a shared VM and local laptops, ensuring consistent and collaborative development.

---

## 2. Architecture & Workflow

- **Pipeline Flow:**
  ```mermaid
    graph LR
        A[dlt ingestion] --> B[ClickHouse - raw schema]
        B --> C[dbt - clean schema]
        C --> F[Data quality testing]
        F --> D[dbt - mart schema]
        D --> E[Metabase dashboards]
  ```

- **Tools Used:**  
  - Ingestion: `dlt`  
  - Modeling: `dbt`
  - Data Quality: 
  - Visualization: `Metabase`  

- **Medallion Architecture Application:**  
  - **Bronze (Raw):** Initial ingestion of source data  
  - **Silver (Clean):** Cleaning, type casting, handling missing values  
  - **Gold (Mart):** Business-ready star schema for BI  
 

---

## 3. Modeling Process

- **Source Structure (Normalized):**  
  *(Describe how the original tables were structured — 3NF, relationships, etc.)*  

- **Star Schema Design:**  
  - Fact Tables: *(e.g., FactSales, FactAssessment, FactRatings)*  
  - Dimension Tables: *(e.g., Customer, Date, Genre, Student, Demographics, Title, Person)*  

- **Challenges / Tradeoffs:**  
  *(E.g., handling missing data, many-to-many joins, exploding arrays, performance considerations.)*  

---

## 4. Collaboration & Setup

- **Task Splitting:**  
  *(How the team divided ingestion, modeling, BI dashboards, documentation.)*  

- **Shared vs Local Work:**  
  *(Issues faced with sync conflicts, version control, DB connections, etc.)*  

- **Best Practices Learned:**  
  *(E.g., using Git for dbt projects, naming conventions, documenting assumptions, group debugging sessions.)*  

---

## 5. Business Questions & Insights

- **Business Questions Explored:**  
  1. *(Example: Who are the top customers by revenue?)*  
  2. *(Example: What factors contribute to student dropout?)*  
  3. *(Example: Which genres/actors perform best in ratings?)*  

- **Dashboards / Queries:**  
  *(Add screenshots, SQL snippets, or summaries of dashboards created in Metabase.)*  

- **Key Insights:**  
  - *(Highlight 1–2 interesting findings. Example: “Rock was the top genre in North America, while Latin genres dominated in South America.”)*  

---

## 6. Key Learnings

- **Technical Learnings:**  
  *(E.g., SQL joins, window functions, dbt builds/tests, schema design.)*  

- **Team Learnings:**  
  *(E.g., collaboration in shared environments, version control, importance of documentation.)*  

- **Real-World Connection:**  
  *(How this exercise relates to actual data engineering workflows in industry.)*  

---

## 7. Future Improvements

- **Next Steps with More Time:**  
  *(E.g., add orchestration with Airflow/Prefect, implement testing, optimize queries, handle larger datasets.)*  

- **Generalization:**  
  *(How this workflow could be applied to other datasets or business domains.)*  
