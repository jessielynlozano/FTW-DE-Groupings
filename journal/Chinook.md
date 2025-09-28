# 🎵Documentation: Chinook Music Store

---

## 1. Project Overview - 

- **Dataset Used:**  
  **Dataset:**Chinook music store
  **Domain:**artists, albums, tracks, genres, playlist, media type, customer, employee, invoice, invoice line

- **Goal of the Exercise:**  
  Convert the Chinook dataset into a dimensional model to answer business questions. After creating the schema, do the RCM pipeline.

- **Team Setup:**  
  *(State if you worked individually, as a group, or both. Mention collaboration style.)*  

---

## 2. Architecture & Workflow
- **Pipeline Flow:**  
  **raw → clean → mart → Metabase**  

- **Tools Used:**  
  - Ingestion: `dlt`  
  - Modeling: `dbt`  
  - Visualization: `Metabase`  

- **Medallion Architecture Application:**  
  - **Bronze (Raw):** Initial ingestion of source data  
  - **Silver (Clean):** Cleaning, type casting, handling missing values  
  - **Gold (Mart):** Business-ready star schema for BI  

---

## 3. Modeling Process 

- **Star Schema Design:**  
![Schema](..assets/chinook_schema.png)
 

- **Challenges / Tradeoffs:**  
  - Repeating tables due to **append** 

---

## 4. Collaboration & Setup - Monette

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

---

## 📢 Presentation Tips

- Keep it **5–10 minutes**, like a project walkthrough.  
- Use **diagrams, screenshots, and SQL snippets**.  
- Focus on both **technical process** and **business insights**.  
- End with your **key learnings and future improvements**.  
- For other documentation tips. Read [this](TECHNICAL-DOCS.md).

---

✅ By filling this template, your group will produce a professional-style project guide **just like real data engineers** — clear, structured, and insight-driven.
