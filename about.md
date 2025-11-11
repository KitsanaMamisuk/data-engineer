You are an expert Data Engineer specializing in IoT data pipelines and workflow orchestration using Python, Pandas, NumPy, Cloud SDKs, and Apache Airflow.

Your task is to generate the complete Markdown content for a professional learning handbook titled: **"📘 Python Data Engineer IoT Pipeline Handbook (Professional Learning Edition)"**. The output MUST be a single, structured Markdown file suitable for a Wiki or documentation platform.

**Structure Requirements:**

1.  **Title and Metadata:** Start with the main title, author (Kitsana Mameesukh), and edition (Learning Edition).
2.  **Table of Contents (TOC):** Create a clear, hyperlinked TOC based on the 8 Modules provided below. Each link must jump to the corresponding Module section.
3.  **8 Modules (Chapters):** Each module must contain:
    * A **Main Heading (##)** for the Module Title.
    * A **Sub-Heading (###)** for key concepts.
    * Clear, concise **Explanatory Text** in Thai, maintaining English technical terms (e.g., ETL, DAG, Sensor).
    * A concluding **"Labs & Practical Exercises"** section.

**Detailed Module Outline (8 Modules):**

| Module (No. & Title) | Key Focus Areas (Sub-Headings) | Labs & Practical Exercises |
| :--- | :--- | :--- |
| **1: Python Fundamentals & Environment** | Data Structures (Pandas/NumPy), Virtual Environment Setup, Time-Series Data Introduction | Setup & Test: Verify `pandas` and `numpy` installation. |
| **2: ETL: Data Loading & Cleansing** | Optimized Loading (`dtype`), Handling Missing Values (Ffill/Mean), Data Integrity Checks | Lab 2.1: Load `sensor_large.csv` using type optimization and calculate memory saving. |
| **3: ETL: Transformation & Optimization** | Feature Engineering (Rolling Average), Performance Benchmarking (Pandas vs. NumPy), Chunking | Lab 3.1: Implement Rolling Average (5min window). Lab 3.2: Benchmark NumPy subtraction vs. Pandas series subtraction. |
| **4: Cloud Integration & Storage** | Cloud Storage Concepts (Data Lake), Using Cloud SDKs (Conceptual), Data Partitioning Strategy | Lab 4.1: Define file path and naming convention for Time-Series Partitioning (`year/month/day`). |
| **5: Airflow: Orchestration Basics** | DAGs, Tasks, Operators, Dependencies, Airflow UI Navigation | Lab 5.1: Install Airflow (via Docker or local). Lab 5.2: Create a simple "Hello World" DAG. |
| **6: Airflow: Building the IoT Pipeline** | FileSensor Implementation, PythonOperator Integration, Cloud Operator Use (Conceptual) | Lab 6.1: Define a DAG with T1 (FileSensor) >> T2 (Python ETL). |
| **7: Data Quality & Testing** | Data Validation (Range/Null Checks), Pipeline Idempotency, Unit Testing ETL Functions | Lab 7.1: Write a Python function to check if processed data has any NULLs in the 'temperature' column. |
| **8: Project Summary & Future Scope** | Executive Summary, Key Skills Mapping, Real-Time/MLOps Vision | Exercise: Map your project steps to the Data Engineering lifecycle (Ingest, Transform, Serve). |

**Constraint:**
* Use Markdown syntax exclusively.
* Ensure all links in the TOC work correctly within the Markdown file.
* The overall tone must be formal and instructional.
