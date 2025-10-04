# Documentation: Open University Learning Analytics

---

## 1. Project Overview - 

- **Dataset Used:**  
  Dataset: Open University Learning Analytics dataset

  Domain: VLE, Student VLE, Student Registration, Student Info, Student Assessment, Courses, Assessments

- **Goal of the Exercise:**  
  1. Ingest CSVs into the `raw` schema.
  2. Standardize types, handle missing values in the `clean` schema.
  3. Design a star schema around **student performance & engagement**:
   * **Facts:** `FactAssessments`, `FactVLEInteractions`.
   * **Dimensions:** module, moduleVLE, presentation, registered module, student, date
  4. Implement in dbt (`mart` schema).
  5. Build dashboards in Metabase to analyze **cohorts, dropout risk, and engagement**.

- **Team Setup:**  
  1. We createad a monitoring sheet to monitor each one trying the task. (Link: https://docs.google.com/spreadsheets/d/1VOidsWkT1EnYQ2q9agZKiv-X-Abc_UGZ/edit?gid=504770803#gid=504770803)
  2. Each one tries to ingest and clean the data.
  3. We huddled to combine what we have done alone and then created the fact and dimension tables using one source of truth
  4. Each one input a code in Metabase to visualize answers to business questions.

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

### Raw Layer: Data Ingestion

- For the raw layer, tables from OULAD e are ingested using `dlt`. This step brings in source tables for downstream processing.

- **Example Extraction Code:**  
  Extraction uses Python functions decorated with `@dlt.resource`. For example, with the naming convention update:

```python
"# dlt/pipeline.py
import dlt, pandas as pd
import os
from glob import glob

# Approach 1: Single resource yielding multiple DataFrames
@dlt.resource(name=""monette_oulad"")
def oulad_all_files():
    ROOT_DIR = os.path.dirname(__file__)
    STAGING_DIR = os.path.join(ROOT_DIR, ""staging"", ""oulad"")
    
    # List all your CSV files
    csv_files = [
        ""assessments.csv"",
        ""courses.csv"", 
        ""studentAssessment.csv"",
        ""studentInfo.csv"",
        ""studentRegistration.csv"",
        ""vle.csv""
    ]
    
    for csv_file in csv_files:
        file_path = os.path.join(STAGING_DIR, csv_file)
        if os.path.exists(file_path):
            df = pd.read_csv(file_path)
            # Add a column to identify the source file
            df['_source_file'] = csv_file.replace('.csv', '')
            yield df
        else:
            print(f""Warning: File {csv_file} not found"")

# Approach 2: Separate resources for each file
@dlt.resource(name=""assessments"")
def assessments():
    ROOT_DIR = os.path.dirname(__file__)
    STAGING_DIR = os.path.join(ROOT_DIR, ""staging"", ""oulad"")
    FILE_PATH = os.path.join(STAGING_DIR, ""assessments.csv"")
    yield pd.read_csv(FILE_PATH)

@dlt.resource(name=""courses"")
def courses():
    ROOT_DIR = os.path.dirname(__file__)
    STAGING_DIR = os.path.join(ROOT_DIR, ""staging"", ""oulad"")
    FILE_PATH = os.path.join(STAGING_DIR, ""courses.csv"")
    yield pd.read_csv(FILE_PATH)

@dlt.resource(name=""student_assessment"")
def student_assessment():
    ROOT_DIR = os.path.dirname(__file__)
    STAGING_DIR = os.path.join(ROOT_DIR, ""staging"", ""oulad"")
    FILE_PATH = os.path.join(STAGING_DIR, ""studentAssessment.csv"")
    yield pd.read_csv(FILE_PATH)

@dlt.resource(name=""student_info"")
def student_info():
    ROOT_DIR = os.path.dirname(__file__)
    STAGING_DIR = os.path.join(ROOT_DIR, ""staging"", ""oulad"")
    FILE_PATH = os.path.join(STAGING_DIR, ""studentInfo.csv"")
    yield pd.read_csv(FILE_PATH)

@dlt.resource(name=""student_registration"")
def student_registration():
    ROOT_DIR = os.path.dirname(__file__)
    STAGING_DIR = os.path.join(ROOT_DIR, ""staging"", ""oulad"")
    FILE_PATH = os.path.join(STAGING_DIR, ""studentRegistration.csv"")
    yield pd.read_csv(FILE_PATH)
    
@dlt.resource(name=""vle"")
def vle():
    ROOT_DIR = os.path.dirname(__file__)
    STAGING_DIR = os.path.join(ROOT_DIR, ""staging"", ""oulad"")
    FILE_PATH = os.path.join(STAGING_DIR, ""vle.csv"")
    yield pd.read_csv(FILE_PATH)

# Approach 3: Dynamic file discovery
@dlt.resource(name=""oulad_dynamic"")
def oulad_dynamic():
    ROOT_DIR = os.path.dirname(__file__)
    STAGING_DIR = os.path.join(ROOT_DIR, ""staging"", ""oulad"")
    
    # Find all CSV files in the directory
    csv_pattern = os.path.join(STAGING_DIR, ""*.csv"")
    csv_files = glob(csv_pattern)
    
    for file_path in csv_files:
        df = pd.read_csv(file_path)
        # Add source file information
        filename = os.path.basename(file_path).replace('.csv', '')
        df['_source_file'] = filename
        yield df

def run_approach_1():
    """"""Single resource, all files combined""""""
    p = dlt.pipeline(
        pipeline_name=""oulad-pipeline"",
        destination=""clickhouse"",
        dataset_name=""monette_oulad"",
        dev_mode=True
    )
    print(""Fetching and loading all files as one resource..."")
    info = p.run(oulad_all_files())
    print(""Records loaded:"", info)

def run_approach_2():
    """"""Separate resources for each file""""""
    p = dlt.pipeline(
        pipeline_name=""oulad-pipeline"",
        destination=""clickhouse"",
        dataset_name=""monette_oulad"",
    )
    print(""Fetching and loading each file as separate resource..."")
    info = p.run([
        assessments(),
        courses(),
        student_assessment(),
        student_info(),
        student_registration(),
        vle()
    ])
    print(""Records loaded:"", info)

def run_approach_3():
    """"""Dynamic file discovery""""""
    p = dlt.pipeline(
        pipeline_name=""oulad-pipeline"",
        destination=""clickhouse"",
        dataset_name=""monette_oulad"",
    )
    print(""Fetching and loading dynamically discovered files..."")
    info = p.run(oulad_dynamic())
    print(""Records loaded:"", info)

if __name__ == ""__main__"":
    # Choose which approach to use:
    
    # Option 1: All files as one resource (creates one table)
    # run_approach_1()
    
    # Option 2: Each file as separate resource (creates separate tables)
    run_approach_2()
    
    # Option 3: Dynamic discovery (creates one table with source file column)
    # run_approach_3()"
```



- **How Ingestion Was Run:**  
The pipeline was executed in the terminal:

```bash
docker compose --profile jobs run --rm dlt python extract-loads/01-dlt-mpg-pipeline.py
```


**Tables Ingested:**  
  - **genre**
  - **media_type**
  - **track**
  - **playlist**
  - **playlist_track**
  - **invoice_line**
  - **invoice**
  - **customer**
  - **employee**


### Clean Layer: Data Transformation

- After ingestion into the raw layer, each table is **cleaned and standardized** using `dbt`.  
- Cleaning involves **casting types, renaming columns, and ensuring nullable fields are properly handled**.  
- Models are materialized as **tables** in the `clean` schema (❗ originally these were materialized as `views`, but we switched them to `tables` for better performance and stability).  

- **Example dbt Model (Albums):**

```sql
{{ config(materialized="table", schema="clean", tags=["staging","chinook"]) }}

-- Keep album grain; standardize names/types.
select
  cast(album_id  as Nullable(Int64))      as album_id,
  cast(title     as Nullable(String))     as album_title,
  cast(artist_id as Nullable(Int64))      as artist_id
from {{ source('raw', 'chinook___albums_group6') }}
```

**Note:**  
  Each raw table has its own corresponding clean model (e.g., `artists`, `tracks`, `genres`, etc.), all following the same pattern:  
  - Define `config()`  
  - Cast columns to consistent types  
  - Use `source()` to reference the raw schema


### Mart Layer: Dimensional Modeling

- After the **clean layer**, we created **dimension and fact tables** and stored them in the **mart**.  
- This step converts the staging/cleaned data into a **star schema** ready for BI tools like Metabase.  

- **Example Fact Table (Invoice Line):**

```sql
{{ config(materialized="table", schema="mart", tags=["mart","chinook"]) }}

-- CTEs for base staging tables
with invoice_lines as (
  select
    invoice_line_id,
    invoice_id,
    track_id,
    unit_price,
    quantity
  from {{ ref('stg_group6_chinook___invoice_line') }}
),
invoices as (
  select
    invoice_id,
    customer_id,
    invoice_date,
    billing_address,
    billing_city,
    billing_state,
    billing_country,
    billing_postal_code,
    total
  from {{ ref('stg_group6_chinook___invoice') }}
),
dim_customers as (
  select
    customer_id,
    first_name,
    last_name,
    country
  from {{ ref('group6_dim_customer') }}
),
dim_tracks as (
  select
    track_id,
    track_name,
    album_id,
    genre_id,
    media_type_id,
    composer,
    milliseconds,
    bytes
  from {{ ref('group6_dim_track') }}
)

-- Final fact_invoice_line model
select
    il.invoice_line_id          as invoice_line_key,
    c.customer_id               as customer_key,  -- still natural key
    t.track_id                  as track_key,     -- fix here
    i.invoice_date              as invoice_date,
    il.quantity                 as quantity,
    il.unit_price               as unit_price,
    il.unit_price * il.quantity as line_amount
from {{ ref('stg_group6_chinook___invoice_line') }} il
join {{ ref('stg_group6_chinook___invoice') }} i on il.invoice_id = i.invoice_id
join {{ ref('group6_dim_customer') }} c on i.customer_id = c.customer_id
join {{ ref('group6_dim_track') }} t on il.track_id = t.track_id
```

- **Example Dimension Table (Customer):**

```sql
{{ config(materialized="table", schema="mart", tags=["mart","chinook"]) }}

-- Base staging customers
with stg_customers as (
  select
    customer_id,
    first_name,
    last_name,
    company,
    address,
    city,
    state,
    country,
    postal_code,
    phone,
    email,
    support_rep_id
  from {{ ref('stg_angel_chinook___customer') }}
)

-- Dimension table
select
  customer_id             as customer_key,    -- still natural key, surrogate possible later
  first_name,
  last_name,
  country,
  city,
  state,
  email
from stg_customers
```

- Transformation was executed via Docker with the following command:

```bash
ddocker compose --profile jobs run --rm \
  -w /workdir/transforms/02_chinook \
  dbt build --profiles-dir . --target remote
```

### Visualization Layer: Metabase

- After building the **mart layer** (facts and dims), we connected the database to **Metabase**.  
- Using Metabase, we **queried and visualized** the data to answer the defined **business questions** (e.g., gender engagement and VLE).  

---

## 3. Modeling Process 

- **Star Schema Design:**  
![Schema](../assets/chinook_schema.png)
 

- **Challenges / Tradeoffs:**  
- Identifying which data should be placed in the fact and dimension tables (e.g., two possible data sources: student VLE and assessment).
- Same number of rows but different file sizes (raw vs. sandbox).
- Created a table in sandbox, but it shows up empty.
- During cleaning, the same SQL script produced different results: one member ran it successfully, while another encountered an error.

---

## 4. Collaboration & Setup 

- **Task Splitting:**  
- Each member individually performed the data ingestion and cleaning tasks to ensure consistency and validate the process across different environments.
- The team met to compare data outputs and SQL queries across the cleaned tables to ensure consistency.
- Collaborated to create a single source of truth through the fact and dimension tables.
  
- **Shared vs Local Work:**  
  - Server went down and since we were doing things the last minute, that has also affected our workflow greatly 

- **Best Practices Learned:**  
  - Comparing queries and data
  - Unit Tests for Pipelines
  - Maintain a single source of truth to avoid duplication and inconsistency.
    
---

## 5. Business Questions & Insights

- **Business Questions Explored:**  
  1. Which music genres generate the most revenue in each country?
  2. How many customers fall into each tier - high, medium, low? 
  3. How has revenue trended month-by-month over the last 2 years?
  4. What are the top 20 tracks by total units sold, and which albums/artists do they belong to?
  5. Do average unit prices differ across countries or regions?

- **Dashboards / Queries:**  
  *(Add screenshots, SQL snippets, or summaries of dashboards created in Metabase.)*
1. Sample dbt script:
```sql
assessment -
{{ config(materialized="table", schema="clean", tags=["staging","oulad"]) }}

-- Standardize column names/types per table; no business logic.
select
    CAST(code_module AS Nullable(Varchar(45)))            AS code_module,
    CAST(code_presentation AS Nullable(Varchar(45)))      AS code_presentation,
    CAST(id_assessment AS Nullable(Int64))                  AS id_assessment,
    CAST(assessment_type  AS Nullable(Varchar(45)))      AS assessment_type,
    CAST(date AS Nullable(Int64))      AS date,
    CAST(weight AS Nullable(Float64)) AS weight

from {{ source('raw', 'monette_oulad___assessments') }}
```

2. Sample slq script- fact_assessment
   
4. 
  

- **Key Insights:**  
  - Rock is the most profitable genre in the US with heavy margins compared to the others
  - The customers fall in mid tier only

---

## 6. Key Learnings

- **Technical Learnings:**  
  - Creating Fact and Dim table using SQL

- **Team Learnings:**  
  - Collaborating using remote setup
  - Proper assignment of tasks to prevent backlogs

- **Real-World Connection:**  
  - The exercise showed how it is possible for data engineers to connect and collaborate with each other.

---

## 7. Future Improvements

- **Next Steps with More Time:**
  - Each one of use would try it out and we would present with each other how we did the exercise.

- **Generalization:**  
  *(How this workflow could be applied to other datasets or business domains.)*  
