# Documentation: Open University Learning Analytics

---

## 1. Project Overview - 

- **Dataset Used:**  
  Dataset: Open University Learning Analytics dataset

  Domain:artists, albums, tracks, genres, playlist, media type, customer, employee, invoice, invoice line

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

### Raw Layer: Data Ingestion

- For the raw layer, tables from the Chinook sample database are ingested using `dlt`. This step brings in source tables for downstream processing.

- **Example Extraction Code:**  
  Extraction uses Python functions decorated with `@dlt.resource`. For example, with the naming convention update:

```python
import os
import dlt
import psycopg2
from psycopg2.extras import RealDictCursor

def get_connection():
    host     = os.environ["POSTGRES_HOST"]
    port     = int(os.environ["POSTGRES_PORT"])
    user     = os.environ["POSTGRES_USER"]
    password = os.environ["POSTGRES_PASSWORD"]
    dbname   = os.environ["POSTGRES_DB"]

    return psycopg2.connect(
        host=host,
        port=port,
        user=user,
        password=password,
        dbname=dbname
    )

@dlt.resource(write_disposition="append", name="artists")
def artists():
    """Extract all artists from the Chinook sample DB."""
    conn = get_connection()
    cur = conn.cursor(cursor_factory=RealDictCursor)
    cur.execute("SELECT * FROM artist;")
    for row in cur.fetchall():
        yield dict(row)
    conn.close()

def run():
    pipeline = dlt.pipeline(
        pipeline_name="chinook_pipeline",
        destination="clickhouse",
        dataset_name="chinook",
        dev_mode=False
    )
    print("Fetching and loading...")

    load_info = pipeline.run(artists())
    print("records loaded:", load_info)

if __name__ == "__main__":
    run()
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
- Using Metabase, we **queried and visualized** the data to answer the defined **business questions** (e.g., revenue by genre, sales trends).  

---

## 3. Modeling Process 

- **Star Schema Design:**  
![Schema](../assets/chinook_schema.png)
 

- **Challenges / Tradeoffs:**  
  - Repeating tables due to **append** 

---

## 4. Collaboration & Setup - Monette

- **Task Splitting:**  
  - One person is assigned for ingestion, cleaning, until pushing to mart
  - Each one took a business question to visualize and query using metabase
  - Each one collaborates in creating the documentation

- **Shared vs Local Work:**  
  - Server went down and since we were doing things the last minute, that has also affected our workflow greatly 

- **Best Practices Learned:**  
  - Learn how to collaborate 

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
Top Revenue by Genre per Country
  ```sql
  SELECT
    cu.country,
    ge.genre_name,
    SUM(il.line_amount) AS total_revenue
  FROM
    group6_fact_line_invoice il
  JOIN group6_dim_track t ON il.track_key = t.track_id
  JOIN group6_dim_genre ge ON t.genre_id = ge.genre_id
  JOIN group6_dim_customer cu ON il.customer_key = cu.customer_id
  GROUP BY
    cu.country,
    ge.genre_name
  ORDER BY
    cu.country,
    total_revenue DESC;
  

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
