Data Engineer Learning Path Wiki

🎯 เป้าหมาย: เรียนรู้ Data Engineering best practices ผ่านการสร้าง production-ready ETL pipeline ด้วย Apache Airflow


📚 สารบัญ

Phase 1: Environment Setup
Phase 2: Dataset Selection
Phase 3: Data Pipeline Architecture
Phase 4: Task Implementation Deep Dive
Phase 5: Production Features
Phase 6: Testing & Deployment


Phase 1: Environment Setup
🎯 วัตถุประสงค์
ตั้งค่าสภาพแวดล้อมการพัฒนาบน local machine ด้วย Docker เพื่อให้สามารถรัน Airflow และ PostgreSQL ได้
⏱️ ระยะเวลา: 1-2 วัน
🛠️ เครื่องมือที่ต้องติดตั้ง

Docker Desktop (สำหรับ MacOS)

ดาวน์โหลดจาก: https://www.docker.com/products/docker-desktop
ตรวจสอบการติดตั้ง: docker --version และ docker-compose --version


Git (สำหรับ version control)

มักติดตั้งมาแล้วบน MacOS
ตรวจสอบ: git --version


Python 3.10+ (สำหรับทดสอบ code local)

ตรวจสอบ: python3 --version




📁 Project Structure
สร้างโครงสร้างโปรเจคดังนี้:
data-engineering-project/
├── docker-compose.yml          # ไฟล์สำหรับ orchestrate services
├── Dockerfile                  # Custom image (ถ้าจำเป็น)
├── requirements.txt            # Python dependencies
├── README.md                   # Documentation
├── .env.example               # Environment variables template
├── .gitignore                 # ไฟล์ที่ไม่ต้อง commit
│
├── dags/                      # Airflow DAG files
│   ├── marketing_etl_dag.py  # Main DAG
│   ├── config/
│   │   └── dag_config.yaml   # Configuration
│   └── tasks/                # Task functions (modular)
│       ├── __init__.py
│       ├── ingestion.py
│       ├── profiling.py
│       ├── cleaning.py
│       ├── validation.py
│       └── loading.py
│
├── plugins/                   # Airflow plugins
│   ├── __init__.py
│   ├── operators/            # Custom operators
│   └── hooks/                # Custom hooks
│
├── tests/                     # Unit tests
│   ├── __init__.py
│   ├── fixtures/
│   │   └── sample.csv
│   ├── test_dag_validation.py
│   ├── test_ingestion.py
│   └── test_cleaning.py
│
├── data/                      # Data directories
│   ├── raw/                  # Source CSV files
│   ├── processed/            # Cleaned Parquet files
│   ├── reports/              # Profiling reports
│   └── archive/              # Archived files
│
├── logs/                      # Airflow logs (auto-generated)
│
├── scripts/                   # Utility scripts
│   ├── setup_database.sql
│   └── create_tables.sql
│
└── docs/                      # Documentation
    ├── ARCHITECTURE.md
    └── TROUBLESHOOTING.md

🐳 Docker Compose Setup
สร้างไฟล์ docker-compose.yml:
yamlversion: '3.8'

x-airflow-common:
  &airflow-common
  image: apache/airflow:2.8.1
  environment:
    &airflow-common-env
    AIRFLOW__CORE__EXECUTOR: LocalExecutor
    AIRFLOW__DATABASE__SQL_ALCHEMY_CONN: postgresql+psycopg2://airflow:airflow@postgres/airflow
    AIRFLOW__CORE__FERNET_KEY: ''
    AIRFLOW__CORE__DAGS_ARE_PAUSED_AT_CREATION: 'true'
    AIRFLOW__CORE__LOAD_EXAMPLES: 'false'
    AIRFLOW__API__AUTH_BACKENDS: 'airflow.api.auth.backend.basic_auth'
  volumes:
    - ./dags:/opt/airflow/dags
    - ./logs:/opt/airflow/logs
    - ./plugins:/opt/airflow/plugins
    - ./data:/opt/airflow/data
  user: "${AIRFLOW_UID:-50000}:0"
  depends_on:
    postgres:
      condition: service_healthy

services:
  postgres:
    image: postgres:14
    environment:
      POSTGRES_USER: airflow
      POSTGRES_PASSWORD: airflow
      POSTGRES_DB: airflow
    volumes:
      - postgres-db-volume:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "airflow"]
      interval: 5s
      retries: 5
    ports:
      - "5432:5432"

  airflow-webserver:
    <<: *airflow-common
    command: webserver
    ports:
      - "8080:8080"
    healthcheck:
      test: ["CMD", "curl", "--fail", "http://localhost:8080/health"]
      interval: 10s
      timeout: 10s
      retries: 5

  airflow-scheduler:
    <<: *airflow-common
    command: scheduler
    healthcheck:
      test: ["CMD-SHELL", 'airflow jobs check --job-type SchedulerJob --hostname "$${HOSTNAME}"']
      interval: 10s
      timeout: 10s
      retries: 5

  airflow-init:
    <<: *airflow-common
    entrypoint: /bin/bash
    command:
      - -c
      - |
        mkdir -p /sources/logs /sources/dags /sources/plugins
        chown -R "${AIRFLOW_UID:-50000}:0" /sources/{logs,dags,plugins}
        exec /entrypoint airflow version

volumes:
  postgres-db-volume:

📝 Requirements.txt
สร้างไฟล์ requirements.txt:
txt# Core Airflow
apache-airflow==2.8.1

# Data Processing
pandas==2.1.4
numpy==1.24.3

# Data Profiling
ydata-profiling==4.18.0

# Database
sqlalchemy==2.0.23
psycopg2-binary==2.9.9

# File Formats
pyarrow==14.0.1

# Testing
pytest==7.4.3
pytest-cov==4.1.0

🚀 Setup Instructions
Step 1: Clone/Create Project
bashmkdir data-engineering-project
cd data-engineering-project
Step 2: สร้างไฟล์ทั้งหมดตาม structure ข้างต้น
Step 3: เริ่ม Services
bash# Set Airflow UID (MacOS)
echo -e "AIRFLOW_UID=$(id -u)" > .env

# Start services
docker-compose up -d
Step 4: Initialize Airflow Database
bashdocker-compose run airflow-init
Step 5: Create Admin User
bashdocker-compose exec airflow-webserver airflow users create \
    --username admin \
    --firstname Admin \
    --lastname User \
    --role Admin \
    --email admin@example.com \
    --password admin
Step 6: Verify Installation

Airflow UI: http://localhost:8080 (login: admin/admin)
PostgreSQL: localhost:5432 (user: airflow/airflow)


✅ Health Check
ตรวจสอบว่า services ทำงานปกติ:
bash# ดู status ของ containers
docker-compose ps

# ควรเห็น services เหล่านี้ running:
# - postgres
# - airflow-webserver
# - airflow-scheduler

# ดู logs
docker-compose logs -f airflow-webserver

🔧 Troubleshooting
ปัญหา: Permission denied
bash# Fix permissions
sudo chown -R $USER:$USER .
ปัญหา: Port 8080 ถูกใช้แล้ว
bash# หา process ที่ใช้ port
lsof -i :8080
# หรือเปลี่ยน port ใน docker-compose.yml
ปัญหา: Container ไม่ healthy
bash# Check logs
docker-compose logs postgres
docker-compose logs airflow-scheduler

Phase 2: Dataset Selection
🎯 วัตถุประสงค์
เลือก dataset จาก Kaggle ที่เหมาะสมสำหรับฝึก ETL pipeline
⏱️ ระยะเวลา: 2-3 ชั่วโมง

📊 Dataset แนะนำ
🥇 Option 1: E-commerce Customer Behavior
ชื่อ Dataset: Online Retail Dataset
Link: https://www.kaggle.com/datasets/lakshmi25npathi/online-retail-dataset
ขนาด: ~540,000 rows, 8 columns
Columns หลัก:

InvoiceNo: รหัสใบแจ้งหนี้
StockCode: รหัสสินค้า
Description: คำอธิบายสินค้า
Quantity: จำนวน
InvoiceDate: วันที่ซื้อ
UnitPrice: ราคาต่อหน่วย
CustomerID: รหัสลูกค้า
Country: ประเทศ

ปัญหาข้อมูลที่พบ:

✅ Missing values ใน CustomerID (~25%)
✅ Missing values ใน Description (~0.3%)
✅ Negative Quantity (returns)
✅ Zero UnitPrice
✅ Date format ต้อง parse
✅ Duplicate transactions

เหมาะสำหรับ:

ฝึก handle missing values
ฝึก business logic (returns, refunds)
ฝึก date manipulation
ฝึก aggregation (RFM analysis)


🥈 Option 2: Marketing Campaign Data
ชื่อ Dataset: Bank Marketing Dataset
Link: https://www.kaggle.com/datasets/henriqueyamahata/bank-marketing
ขนาด: ~45,000 rows, 17 columns
Columns หลัก:

age: อายุ
job: อาชีพ
marital: สถานะสมรส
education: การศึกษา
balance: ยอดเงินในบัญชี
campaign: จำนวนครั้งที่ติดต่อ
poutcome: ผลลัพธ์แคมเปญก่อนหน้า
y: สมัคร term deposit หรือไม่

ปัญหาข้อมูลที่พบ:

✅ Categorical data ต้อง encode
✅ Outliers ใน balance, campaign
✅ Imbalanced target (y)
✅ Missing values แฝงอยู่ใน "unknown"
✅ Duplicate records

เหมาะสำหรับ:

ฝึก categorical encoding
ฝึก outlier detection
ฝึก data validation rules
ฝึก feature engineering


🥉 Option 3: Sales Transaction Data
ชื่อ Dataset: Supermarket Sales
Link: https://www.kaggle.com/datasets/aungpyaeap/supermarket-sales
ขนาด: ~1,000 rows, 17 columns
⚠️ หมายเหตุ: Dataset นี้เล็กไป แต่ clean มาก เหมาะสำหรับเริ่มต้น

📥 วิธีดาวน์โหลด Dataset
Option 1: ดาวน์โหลดผ่าน Kaggle UI

ไปที่ link dataset
กด "Download" (ต้อง login Kaggle)
แตกไฟล์ zip
วางไฟล์ CSV ใน data/raw/

Option 2: ใช้ Kaggle API
bash# Install Kaggle CLI
pip install kaggle

# Setup API token (ไปเอาที่ Kaggle Account Settings)
mkdir -p ~/.kaggle
mv kaggle.json ~/.kaggle/
chmod 600 ~/.kaggle/kaggle.json

# Download dataset
kaggle datasets download -d lakshmi25npathi/online-retail-dataset
unzip online-retail-dataset.zip -d data/raw/

🔍 Data Exploration (เบื้องต้น)
สร้างไฟล์ notebooks/explore.ipynb (optional):
pythonimport pandas as pd

# Load data
df = pd.read_csv('data/raw/online_retail.csv', encoding='ISO-8859-1')

# Basic info
print(f"Shape: {df.shape}")
print(f"Memory: {df.memory_usage(deep=True).sum() / 1024**2:.2f} MB")

# Missing values
print("\nMissing Values:")
print(df.isnull().sum())

# Duplicates
print(f"\nDuplicates: {df.duplicated().sum()}")

# Data types
print("\nData Types:")
print(df.dtypes)

# Sample rows
df.head()
```

---

## Phase 3: Data Pipeline Architecture

### 🎯 วัตถุประสงค์
เข้าใจโครงสร้างและ flow ของ ETL pipeline ก่อนเริ่มเขียน code

### ⏱️ ระยะเวลา: 3-4 ชั่วโมง

---

### 📐 Pipeline Overview
```
┌─────────────┐
│  Raw CSV    │
│  (Source)   │
└──────┬──────┘
       │
       ▼
┌─────────────────────┐
│  Task 1: Ingestion  │  ← อ่านไฟล์, validate, save as Parquet
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Task 2: Profiling  │  ← วิเคราะห์คุณภาพข้อมูล, generate report
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Task 3: Cleaning   │  ← จัดการ missing, duplicates, outliers
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Task 4: Validation │  ← ตรวจสอบ business rules, data quality
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Task 5: Loading    │  ← Load เข้า PostgreSQL + save Parquet
└──────────┬──────────┘
           │
           ▼
    ┌──────────┐
    │PostgreSQL│  (Analytics Database)
    └──────────┘
    ┌──────────┐
    │ Parquet  │  (Data Lake)
    └──────────┘

🔄 Task Dependencies
Sequential Flow (แบบต่อเนื่อง):
pythoningestion >> profiling >> cleaning >> validation >> loading
เหตุผล:

Ingestion ก่อน: ต้องมีข้อมูลก่อนจะ process ได้
Profiling หลัง Ingestion: ต้องรู้ว่าข้อมูลมีปัญหาอะไร
Cleaning หลัง Profiling: รู้ปัญหาแล้วค่อยแก้
Validation หลัง Cleaning: ตรวจว่า clean ถูกต้องหรือไม่
Loading เป็นขั้นสุดท้าย: เก็บข้อมูลที่ clean แล้ว


🗂️ Data Flow & Storage
Intermediate Files:
StageFormatLocationPurposeRawCSVdata/raw/Original sourcePost-IngestionParquetdata/processed/ingested.parquetFaster I/OPost-ProfilingHTML + JSONdata/reports/profile.htmlAnalysis reportPost-CleaningParquetdata/processed/cleaned.parquetClean dataPost-ValidationJSONdata/reports/validation.jsonValidation resultsFinalPostgreSQLmarketing_data tableAnalytics readyFinalParquetdata/processed/final/Data lake

🎯 ทำไมใช้ Parquet ไม่ใช่ CSV?
FeatureCSVParquetRead Speed🐌 Slow⚡ Fast (10-100x)File Size📦 Large📦 Small (compression)Schema❌ No✅ Yes (typed columns)Partial Read❌ Read all✅ Read columns neededCompression❌ Limited✅ Built-in (snappy, gzip)
ตัวอย่าง:

CSV: 100 MB → โหลดทั้งไฟล์ 3 วินาที
Parquet: 20 MB → โหลดแค่ columns ที่ต้องการ 0.3 วินาที


📊 Data Schema Design
Target PostgreSQL Table:
sqlCREATE TABLE marketing_data (
    -- Primary Key
    id SERIAL PRIMARY KEY,
    
    -- Customer Information
    customer_id VARCHAR(50),
    age INTEGER,
    gender VARCHAR(10),
    
    -- Transaction
    invoice_no VARCHAR(50),
    invoice_date TIMESTAMP,
    product_code VARCHAR(50),
    product_name VARCHAR(255),
    quantity INTEGER,
    unit_price DECIMAL(10, 2),
    total_amount DECIMAL(10, 2),
    
    -- Location
    country VARCHAR(100),
    
    -- Data Quality Flags
    has_missing_customer BOOLEAN,
    is_return BOOLEAN,
    is_outlier BOOLEAN,
    
    -- Metadata
    loaded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    source_file VARCHAR(255)
);

-- Indexes สำหรับ query performance
CREATE INDEX idx_customer_id ON marketing_data(customer_id);
CREATE INDEX idx_invoice_date ON marketing_data(invoice_date);
CREATE INDEX idx_country ON marketing_data(country);

🔐 Configuration Management
ไฟล์ .env.example:
env# Airflow
AIRFLOW_UID=50000
AIRFLOW__CORE__EXECUTOR=LocalExecutor
AIRFLOW__CORE__LOAD_EXAMPLES=false

# PostgreSQL
POSTGRES_USER=airflow
POSTGRES_PASSWORD=airflow
POSTGRES_DB=airflow
POSTGRES_HOST=postgres
POSTGRES_PORT=5432

# Paths
DATA_RAW_PATH=/opt/airflow/data/raw
DATA_PROCESSED_PATH=/opt/airflow/data/processed
DATA_REPORTS_PATH=/opt/airflow/data/reports

# Processing
CHUNK_SIZE=10000
PROFILING_MODE=minimal
ENABLE_PROFILING=true
ไฟล์ dags/config/dag_config.yaml:
yaml# DAG Configuration
dag:
  schedule_interval: "0 2 * * *"  # Daily at 2 AM
  max_active_runs: 1
  catchup: false
  
# Data Sources
data:
  source_file: "online_retail.csv"
  encoding: "ISO-8859-1"
  date_columns:
    - InvoiceDate
  
# Data Quality Thresholds
quality:
  max_missing_percent: 30.0
  max_duplicate_percent: 5.0
  outlier_method: "iqr"
  outlier_threshold: 1.5

# Cleaning Rules
cleaning:
  fill_methods:
    CustomerID: "flag"  # Flag as missing
    Description: "Unknown"
  drop_columns:
    - unnecessary_col
  
# Validation Rules
validation:
  required_columns:
    - InvoiceNo
    - StockCode
    - Quantity
    - InvoiceDate
  ranges:
    Quantity:
      min: -10000
      max: 10000
    UnitPrice:
      min: 0
      max: 100000

📝 DAG File Structure (Preview)
python# dags/marketing_etl_dag.py

from airflow import DAG
from airflow.decorators import task
from datetime import datetime, timedelta
import yaml

# Load configuration
with open('/opt/airflow/dags/config/dag_config.yaml', 'r') as f:
    config = yaml.safe_load(f)

# Default arguments
default_args = {
    'owner': 'data-engineering',
    'retries': 3,
    'retry_delay': timedelta(minutes=5),
}

# DAG definition
with DAG(
    dag_id='marketing_etl_pipeline',
    default_args=default_args,
    description='Marketing Data ETL Pipeline',
    schedule_interval=config['dag']['schedule_interval'],
    start_date=datetime(2024, 1, 1),
    catchup=config['dag']['catchup'],
    tags=['marketing', 'etl', 'production'],
) as dag:
    
    @task
    def ingest_raw_data():
        # Implementation ใน Phase 4
        pass
    
    @task
    def profile_data():
        pass
    
    @task
    def clean_data():
        pass
    
    @task
    def validate_data():
        pass
    
    @task
    def load_data():
        pass
    
    # Dependencies
    ingestion = ingest_raw_data()
    profiling = profile_data()
    cleaning = clean_data()
    validation = validate_data()
    loading = load_data()
    
    ingestion >> profiling >> cleaning >> validation >> loading
```

---

## Phase 4: Task Implementation Deep Dive

> 🎯 **ส่วนนี้คือหัวใจของการเรียนรู้** - จะอธิบายทุก parameter, ทุก function, ทุกบรรทัด code แบบละเอียด

### ⏱️ ระยะเวลา: 10-12 วัน

---

## 📥 Task 1: Data Ingestion

### 🎯 จุดประสงค์หลัก
```
=============================================================================
TASK 1: DATA INGESTION - โหลดข้อมูลจาก CSV เข้าสู่ระบบ
=============================================================================
วัตถุประสงค์:

อ่านไฟล์ CSV จาก source directory
Validate ความถูกต้องของไฟล์ (exists, readable, format)
Load เข้า Pandas DataFrame พร้อมจัดการ encoding
บันทึกเป็น Parquet format (optimized สำหรับ processing)
Return metadata ผ่าน XCom สำหรับ tasks ถัดไป

📊 Flow การทำงาน:

Validate file path existence
Check file permissions
Read CSV with proper encoding
Validate basic schema (columns exist)
Save as Parquet (compressed)
Log statistics (rows, memory, file size)
Return metadata to XCom

⚠️ สิ่งที่ต้องระวังในงานจริง:

File encoding issues (UTF-8 vs ISO-8859-1 vs Windows-1252)
Large files → memory overflow
File path security (path traversal attacks)
Concurrent file access
Disk space limitations


📝 Complete Implementation
python@task(
    # task_id: ชื่อ unique identifier ของ task ใน DAG
    # - ต้องเป็น string ที่ไม่ซ้ำกันใน DAG
    # - Naming convention: snake_case, descriptive
    # - ห้ามมี special characters นอกจาก underscore
    # - ความยาวแนะนำ: 3-50 characters
    # - Best practice: ใช้ verb + noun (e.g., ingest_raw_data, load_customer_data)
    task_id='ingest_raw_data',
    
    # retries: จำนวนครั้งที่จะพยายามรัน task อีกครั้งถ้า fail
    # - Type: int
    # - Default: 0 (ไม่ retry)
    # - Range: 0-10 (แนะนำ)
    # - Use case: 
    #   * File I/O operations: 2-3 retries (transient errors)
    #   * API calls: 3-5 retries (network issues)
    #   * Database operations: 2-3 retries (connection issues)
    # - Best practice: retry สำหรับ transient errors เท่านั้น
    #   ถ้าเป็น logic error (เช่น bad data) ไม่ควร retry
    retries=3,
    
    # retry_delay: ระยะเวลารอก่อน retry ครั้งถัดไป
    # - Type: timedelta object
    # - Default: timedelta(seconds=300) = 5 minutes
    # - Best practice:
    #   * Short tasks: timedelta(seconds=30-60)
    #   * Medium tasks: timedelta(minutes=2-5)
    #   * Long tasks: timedelta(minutes=5-15)
    # - เหตุผล: ให้เวลาระบบ recover จาก transient issues
    # - Trade-off: delay นานเกินไป → pipeline slow, สั้นเกินไป → waste resources
    retry_delay=timedelta(minutes=2),
    
    # execution_timeout: เวลาสูงสุดที่ task สามารถรันได้
    # - Type: timedelta object
    # - Default: None (ไม่จำกัด)
    # - Use case: ป้องกัน hanging tasks
    # - การคำนวณ:
    #   * File size 100MB → ~30-60 seconds
    #   * File size 1GB → ~5-10 minutes
    #   * Add buffer 2-3x สำหรับ slow disk/network
    # - Best practice: ตั้งให้มากกว่า average runtime 3-5 เท่า
    # - ถ้า timeout → task fail ทันที (ไม่ retry automatic)
    execution_timeout=timedelta(minutes=10),
    
    # pool: resource pool ที่ task นี้ใช้
    # - Type: string
    # - Default: 'default_pool'
    # - Purpose: จำกัดจำนวน tasks ที่รันพร้อมกัน
    # - Use case:
    #   * 'cpu_intensive': สำหรับ tasks ที่ใช้ CPU สูง
    #   * 'io_intensive': สำหรับ file/network operations
    #   * 'memory_intensive': สำหรับ tasks ที่ใช้ RAM เยอะ
    # - Best practice: แยก pool ตาม resource type
    # - Configuration: ตั้งค่าใน Airflow UI → Admin → Pools
    pool='io_intensive',
    
    # pool_slots: จำนวน slots ที่ task นี้ใช้จาก pool
    # - Type: int
    # - Default: 1
    # - Range: 1-100
    # - ความหมาย: ถ้า pool มี 10 slots, task ใช้ 2 slots → รันได้ 5 tasks พร้อมกัน
    # - Use case:
    #   * Small file (< 100MB): 1 slot
    #   * Medium file (100MB-1GB): 2 slots
    #   * Large file (> 1GB): 3-5 slots
    # - Best practice: ประมาณจาก resource usage
    # - Trade-off: ใช้ slots เยอะ → tasks อื่นรอนาน
    pool_slots=2,
    
    # retry_exponential_backoff: เพิ่ม delay แบบ exponential สำหรับ retry
    # - Type: bool
    # - Default: False
    # - Algorithm: delay = retry_delay * (2 ** retry_number)
    # - Example: retry_delay=60s → 60s, 120s, 240s, 480s
    # - Use case: API rate limits, database connection pools
    # - Best practice: ใช้เมื่อต้องการลด load บน external systems
    retry_exponential_backoff=True,
# max_retry_delay: delay สูงสุดระหว่าง retries
# - Type: timedelta object
# - Default: None (ไม่จำกัด)
# - Use case: จำกัด exponential backoff ไม่ให้นานเกินไป
# - Example: retry_delay=1min, exponential=True
#   → 1min, 2min, 4min, 8min, 16min จะ cap ที่ max_retry_delay
max_retry_delay=timedelta(minutes=15),
)
def ingest_raw_data(
file_path: str = '/opt/airflow/data/raw/online_retail.csv',
# file_path: เส้นทางไปยังไฟล์ CSV ที่ต้องการอ่าน
# - Type: str (absolute path)
# - Format: '/path/to/file.csv' (Unix-style path)
# - Default: path ที่กำหนดไว้ใน configuration
# - Constraints:
#   * ต้องเป็น absolute path (เริ่มด้วย '/')
#   * Extension ต้องเป็น .csv
#   * File size < 10GB (memory limit)
# - Security: ต้อง validate ป้องกัน path traversal (../)
# - Best practice: เก็บ path ใน Airflow Variable หรือ config file
encoding: str = 'ISO-8859-1',  
# encoding: character encoding ของไฟล์ CSV
# - Type: str
# - Common options:
#   * 'utf-8': Standard encoding (most common)
#   * 'ISO-8859-1' (Latin-1): European characters
#   * 'Windows-1252': Windows default
#   * 'utf-8-sig': UTF-8 with BOM
# - Default: 'ISO-8859-1' (for this dataset)
# - ทดสอบ encoding:
#   ```python
#   import chardet
#   with open(file_path, 'rb') as f:
#       result = chardet.detect(f.read(100000))
#       print(result['encoding'])
#   ```
# - Error handling: ถ้า encoding ผิด → UnicodeDecodeError
# - Best practice: ระบุ encoding ชัดเจน อย่าใช้ default

chunk_size: int = None,  
# chunk_size: จำนวน rows ที่อ่านต่อครั้ง (สำหรับไฟล์ใหญ่)
# - Type: int or None
# - Default: None (อ่านทั้งไฟล์)
# - Range: 1000-100000 rows per chunk
# - Use case: ไฟล์ > 1GB → ใช้ chunking
# - Memory calculation:
#   * 10,000 rows × 20 columns × 8 bytes ≈ 1.6 MB
#   * 100,000 rows × 20 columns × 8 bytes ≈ 16 MB
# - Best practice:
#   * Available RAM / 4 = safe chunk size
#   * Example: 8GB RAM → 2GB chunks → ~250,000 rows
# - Trade-off: chunk เล็ก → I/O overhead, chunk ใหญ่ → memory spike

output_format: str = 'parquet',  
# output_format: รูปแบบไฟล์ที่จะบันทึก
# - Type: str
# - Options: 'parquet', 'csv', 'feather'
# - Default: 'parquet'
# - Comparison:
#   * Parquet: Columnar, compressed, schema-aware (recommended)
#   * CSV: Human-readable, no compression, no schema
#   * Feather: Fast, binary, but less compression
# - Performance:
#   * Write: CSV (slowest) < Parquet < Feather (fastest)
#   * Read: CSV (slowest) < Parquet < Feather (fastest)
#   * Size: CSV (largest) > Feather > Parquet (smallest)
# - Best practice: Parquet สำหรับ data pipelines
) -> dict:
# Return type: dictionary containing metadata
# - Structure:
#   {
#       'status': str,           # 'success' or 'failed'
#       'file_path': str,        # Input file path
#       'output_path': str,      # Output Parquet path
#       'rows': int,             # Number of rows
#       'columns': int,          # Number of columns
#       'memory_mb': float,      # Memory usage in MB
#       'file_size_mb': float,   # Original file size
#       'compression_ratio': float,  # Original/Compressed size
#       'duration_seconds': float,   # Processing time
#       'timestamp': str         # ISO format timestamp
#   }
# - Purpose: ส่งข้อมูลไปยัง tasks ถัดไปผ่าน XCom
# - XCom size limits:
#   * SQLite (dev): 2GB (แต่แนะนำ < 1MB)
#   * PostgreSQL: 1GB (แต่แนะนำ < 1MB)
#   * MySQL: 64KB-16MB (depending on config)
# - Best practice:
#   * Return เฉพาะ metadata (ไม่ใช่ data เอง)
#   * ถ้า metadata ใหญ่ → save เป็น file แทน
"""
อ่านไฟล์ CSV, validate, และบันทึกเป็น Parquet format

Args:
    file_path: เส้นทาง absolute ไปยังไฟล์ CSV
        Example: '/opt/airflow/data/raw/data.csv'
        
    encoding: character encoding ของไฟล์
        Example: 'utf-8', 'ISO-8859-1'
        
    chunk_size: จำนวน rows ต่อ chunk (None = อ่านทั้งไฟล์)
        Example: 10000 สำหรับไฟล์ขนาดใหญ่
        
    output_format: รูปแบบ output file
        Example: 'parquet' (recommended), 'csv'

Returns:
    Dict containing:
    - status: 'success' หรือ 'failed'
    - rows: จำนวน rows ที่ประมวลผล
    - columns: จำนวน columns
    - memory_mb: memory usage
    - output_path: path ของไฟล์ที่ save
    - compression_ratio: อัตราส่วนการบีบอัด
    - duration_seconds: เวลาที่ใช้
    
Raises:
    FileNotFoundError: ถ้าไฟล์ไม่พบ
    PermissionError: ถ้าไม่มีสิทธิ์อ่านไฟล์
    UnicodeDecodeError: ถ้า encoding ไม่ถูกต้อง
    pd.errors.ParserError: ถ้า CSV format ผิด
    MemoryError: ถ้า RAM ไม่พอ
"""

import pandas as pd
import logging
import os
from pathlib import Path
from datetime import datetime
import time

# ============================================================
# STEP 1: SETUP LOGGING
# ============================================================
# logging module: Python's built-in logging framework
# - Levels: DEBUG < INFO < WARNING < ERROR < CRITICAL
# - Airflow automatically captures logs from Python's logging
# - Logs are stored in /opt/airflow/logs/dag_id/task_id/execution_date/
# - Best practice: ใช้ logger แทน print()
logger = logging.getLogger(__name__)
# __name__: module name (automatic naming)
# Alternative: logger = logging.getLogger('marketing_etl.ingestion')

logger.info("🚀 [STEP 1/7] Starting data ingestion task")
# Emoji ใน logs: ช่วยให้หา logs ได้ง่าย (แต่ใช้อย่างระมัดระวัง)
# Pattern: [STEP X/Y] เพื่อ track progress

start_time = time.time()
# time.time(): Unix timestamp (seconds since 1970-01-01)
# ใช้สำหรับคำนวณ duration

# ============================================================
# STEP 2: VALIDATE FILE PATH
# ============================================================
logger.info(f"🔍 [STEP 2/7] Validating file path: {file_path}")

# Path validation - Security check
# ป้องกัน path traversal attacks (e.g., '../../../etc/passwd')
file_path_obj = Path(file_path)
# Path object from pathlib: modern way to handle file paths
# Benefits: 
# - OS-independent (works on Windows/Linux/Mac)
# - Rich methods (.exists(), .is_file(), .stat())
# - Safer than string manipulation

# Security check 1: Resolve to absolute path
try:
    resolved_path = file_path_obj.resolve(strict=True)
    # resolve(): convert to absolute path
    # strict=True: raise error ถ้าไฟล์ไม่มีจริง
    # ทำไมต้อง resolve:
    # - '../data.csv' → '/opt/airflow/data/data.csv'
    # - Prevent path traversal: '../../etc/passwd'
except FileNotFoundError:
    error_msg = f"❌ File not found: {file_path}"
    logger.error(error_msg)
    # logger.error(): log level สำหรับ errors ที่ task fail
    # Airflow จะ capture error และแสดงใน UI
    raise FileNotFoundError(error_msg)
    # raise: throw exception เพื่อให้ task fail
    # Airflow จะ retry task ตาม retries parameter

# Security check 2: Verify file is within allowed directory
allowed_base = Path('/opt/airflow/data/raw').resolve()
# allowed_base: directory ที่อนุญาตให้อ่านไฟล์ได้
# Best practice: hardcode allowed paths หรือเก็บใน config

if not str(resolved_path).startswith(str(allowed_base)):
    error_msg = f"❌ File path outside allowed directory: {resolved_path}"
    logger.error(error_msg)
    raise PermissionError(error_msg)
    # PermissionError: exception สำหรับ security violations

# Security check 3: Verify it's a file (not directory)
if not resolved_path.is_file():
    # .is_file(): returns True ถ้าเป็น file (not directory, symlink)
    error_msg = f"❌ Path is not a file: {resolved_path}"
    logger.error(error_msg)
    raise ValueError(error_msg)

# Check file extension
if resolved_path.suffix.lower() != '.csv':
    # .suffix: file extension (e.g., '.csv', '.txt')
    # .lower(): case-insensitive comparison
    error_msg = f"❌ Invalid file extension: {resolved_path.suffix}"
    logger.error(error_msg)
    raise ValueError(error_msg)

# Check file size (prevent loading huge files)
file_size_bytes = resolved_path.stat().st_size
# .stat(): file metadata (size, timestamps, permissions)
# .st_size: file size in bytes

file_size_mb = file_size_bytes / (1024 * 1024)
# Convert bytes to MB: bytes / 1024 / 1024
# Alternative: file_size_mb = file_size_bytes / 1048576

max_file_size_mb = 10240  # 10 GB limit
# Best practice: ตั้ง limit ตาม available RAM
# Rule of thumb: file size < RAM / 4

if file_size_mb > max_file_size_mb:
    error_msg = f"❌ File too large: {file_size_mb:.2f}MB (max: {max_file_size_mb}MB)"
    logger.error(error_msg)
    raise ValueError(error_msg)

logger.info(f"✅ File validation passed: {file_size_mb:.2f}MB")

# ============================================================
# STEP 3: READ CSV FILE
# ============================================================
logger.info(f"📖 [STEP 3/7] Reading CSV file...")

try:
    # pd.read_csv(): Pandas function to read CSV files
    # มี parameters มากกว่า 50 ตัว - เราจะใช้ที่สำคัญ
    
    df = pd.read_csv(
        filepath_or_buffer=str(resolved_path),
        # filepath_or_buffer: path to file, URL, or file-like object
        # - Type: str, Path object, or file handle
        # - ต้อง convert Path object เป็น str
        # - รองรับ: local files, URLs, S3 (with s3fs), GCS (with gcsfs)
        
        encoding=encoding,
        # encoding: character encoding
        # - ต้องระบุชัดเจน ไม่ควรใช้ default
        # - ถ้าไม่แน่ใจ: ใช้ chardet library detect ก่อน
        
        encoding_errors='strict',
        # encoding_errors: วิธีจัดการ encoding errors
        # - Options:
        #   * 'strict': raise UnicodeDecodeError (recommended)
        #   * 'ignore': skip invalid characters (data loss!)
        #   * 'replace': replace with � (visible data loss)
        # - Best practice: 'strict' เพื่อตรวจจับปัญหาทันที
        
        low_memory=False,
        # low_memory: ใช้ memory น้อยกว่า แต่อ่านช้ากว่า
        # - Default: True
        # - ถ้า True: Pandas จะ infer dtype โดยดูแค่ chunk แรก
        #   → อาจได้ dtype ผิด (mixed types)
        # - ถ้า False: อ่านทั้งไฟล์เพื่อ infer dtype ที่ถูกต้อง
        # - Best practice: 
        #   * File < 1GB: low_memory=False
        #   * File > 1GB: low_memory=True + specify dtype manually
        
        na_values=['', 'NA', 'N/A', 'null', 'NULL', 'None', 'nan', 'NaN'],
        # na_values: ค่าที่ถือว่าเป็น missing/null
        # - Type: list, dict, or scalar
        # - Default: ['', '#N/A', '#N/A N/A', '#NA', '-1.#IND', ...]
        # - Best practice: เพิ่ม domain-specific null values
        #   * Example: '999', 'Unknown', 'N.A.', '--'
        # - Dict format: {'column': ['specific', 'null', 'values']}
        
        keep_default_na=True,
        # keep_default_na: ใช้ default na_values ของ Pandas หรือไม่
        # - Default: True
        # - ถ้า False: ใช้เฉพาะค่าใน na_values parameter
        # - Use case: ถ้า 'NA' เป็นค่าจริง (e.g., state code for Nebraska)
        
        skipinitialspace=True,
        # skipinitialspace: ตัด leading whitespace หลัง delimiter
        # - Default: False
        # - Example: 'a, b, c' → ['a', 'b', 'c'] (not ['a', ' b', ' c'])
        # - Best practice: True เพื่อ normalize data
        
        skip_blank_lines=True,
        # skip_blank_lines: ข้าม empty rows
        # - Default: True
        # - ถ้า False: empty rows จะกลายเป็น rows ของ NaN
        # - Best practice: True เพื่อความสะอาดของ data
        
        parse_dates=False,
        # parse_dates: แปลง columns เป็น datetime
        # - Type: bool, list of columns, or dict
        # - Examples:
        #   * parse_dates=True: ลอง parse ทุก column (slow!)
        #   * parse_dates=['date_col1', 'date_col2']
        #   * parse_dates={'datetime': ['date', 'time']} (combine columns)
        # - Best practice: 
        #   * ไม่ใช้ใน ingestion (เพราะ format อาจไม่ consistent)
        #   * Parse ใน cleaning step แทน (มี control มากกว่า)
        
        infer_datetime_format=False,
        # infer_datetime_format: ให้ Pandas ลอง detect format
        # - Default: False (deprecated in Pandas 2.0)
        # - แนะนำ: ระบุ format ชัดเจนใน pd.to_datetime() แทน
        
        dayfirst=False,
        # dayfirst: ถ้าวันมาก่อนเดือนใน ambiguous dates
        # - Default: False (MM/DD/YYYY - US format)
        # - True: DD/MM/YYYY (European format)
        # - Example: '01/02/2024'
        #   * dayfirst=False → Jan 2, 2024
        #   * dayfirst=True → Feb 1, 2024
        # - Best practice: ระบุ format ชัดเจนใน pd.to_datetime() แทน
        
        dtype=None,
        # dtype: กำหนด data type ของแต่ละ column
        # - Type: dict or None
        # - Example: {'col1': 'int64', 'col2': 'float64', 'col3': 'string'}
        # - Benefits:
        #   * Faster reading (no type inference)
        #   * Prevent type errors (e.g., mixed types)
        #   * Save memory (e.g., int32 vs int64)
        # - Best practice:
        #   * None สำหรับ ingestion (ยืดหยุ่น)
        #   * Specify dtype ใน production (consistency)
        
        chunksize=chunk_size,
        # chunksize: จำนวน rows ต่อ chunk
        # - ถ้า None: อ่านทั้งไฟล์
        # - ถ้าระบุ: return iterator of DataFrames
        # - Use case: ไฟล์ใหญ่ที่ไม่พอเก็บใน RAM
        # - Example usage:
        #   ```python
        #   for chunk in pd.read_csv(file, chunksize=10000):
        #       process(chunk)
        #       save(chunk)
        #   ```
        
        on_bad_lines='error',
        # on_bad_lines: วิธีจัดการ rows ที่ malformed
        # - Options:
        #   * 'error': raise error (recommended for quality)
        #   * 'warn': skip row และ log warning
        #   * 'skip': skip row แบบเงียบๆ (อันตราย!)
        # - Best practice: 'error' เพื่อตรวจจับปัญหา
        #   * ถ้าเกิด error → ไปตรวจ source data
        
        memory_map=True,
        # memory_map: ใช้ memory-mapped file I/O
        # - Default: False
        # - Benefits: faster สำหรับไฟล์ใหญ่
        # - ทำงาน: map file เข้า RAM แบบ lazy (load ตอนใช้)
        # - Trade-off: ใช้ได้ดีกับ local files, ไม่เหมาะกับ network files
        
        verbose=False,
        # verbose: แสดง progress ขณะอ่าน
        # - Default: False
        # - ถ้า True: print diagnostic info
        # - Best practice: False (ใช้ logger แทน)
    )
    
    logger.info(f"✅ CSV loaded successfully: {len(df):,} rows × {len(df.columns)} columns")
    # f-string formatting:
    # - {len(df):,}: format number with comma separator
    # - Example: 125483 → 125,483
    
except UnicodeDecodeError as e:
    # UnicodeDecodeError: encoding ไม่ถูกต้อง
    # - Common causes:
    #   * Wrong encoding specified
    #   * File corrupted
    #   * Mixed encodings in file
    error_msg = f"❌ Encoding error: {str(e)}\nTry different encoding (utf-8, ISO-8859-1, Windows-1252)"
    logger.error(error_msg)
    raise UnicodeDecodeError(e.encoding, e.object, e.start, e.end, error_msg)
    # Re-raise with additional context
    
except pd.errors.ParserError as e:
    # ParserError: CSV format ผิด
    # - Common causes:
    #   * Mixed delimiters (comma vs tab)
    #   * Unquoted delimiters inside fields
    #   * Inconsistent number of columns
    #   * Malformed rows
    error_msg = f"❌ CSV parsing error: {str(e)}\nCheck file format and delimiters"
    logger.error(error_msg)
    raise pd.errors.ParserError(error_msg)
    
except MemoryError as e:
    # MemoryError: RAM ไม่พอ
    # - Solutions:
    #   * Use chunksize parameter
    #   * Use Dask instead of Pandas
    #   * Increase Docker memory limit
    #   * Use sampling for development
    error_msg = f"❌ Out of memory: {str(e)}\nTry using chunksize parameter or increase RAM"
    logger.error(error_msg)
    raise MemoryError(error_msg)
    
except Exception as e:
    # Catch-all for unexpected errors
    # - Always log unexpected errors
    # - Re-raise to fail task
    error_msg = f"❌ Unexpected error during CSV read: {str(e)}"
    logger.error(error_msg)
    raise Exception(error_msg) from e
    # raise ... from e: chain exceptions (preserve stack trace)

# ============================================================
# STEP 4: BASIC SCHEMA VALIDATION
# ============================================================
logger.info(f"🔍 [STEP 4/7] Validating basic schema...")

# Check ว่า DataFrame ไม่ empty
if df.empty:
    # df.empty: True ถ้าไม่มี rows
    error_msg = "❌ DataFrame is empty (0 rows)"
    logger.error(error_msg)
    raise ValueError(error_msg)

# Check ว่ามี columns
if len(df.columns) == 0:
    error_msg = "❌ DataFrame has no columns"
    logger.error(error_msg)
    raise ValueError(error_msg)

# Log column names and types
logger.info("📋 Columns detected:")
for col in df.columns:
    # df.dtypes: Series of data types
    # - dtype: NumPy/Pandas data type
    # - Common types:
    #   * int64: 64-bit integer
    #   * float64: 64-bit float
    #   * object: string or mixed types
    #   * datetime64[ns]: datetime with nanosecond precision
    #   * bool: boolean
    #   * category: categorical (efficient for repeated values)
    logger.info(f"  - {col}: {df[col].dtype}")

logger.info("✅ Schema validation passed")

# ============================================================
# STEP 5: COLLECT METADATA
# ============================================================
logger.info(f"📊 [STEP 5/7] Collecting metadata...")

# Memory usage
memory_usage_bytes = df.memory_usage(deep=True).sum()
# df.memory_usage(): memory ที่ใช้ต่อ column
# - deep=True: รวม object overhead (strings, etc.)
# - deep=False: เฉพาะ pointer size (underestimate)
# - .sum(): total memory
# - Best practice: always use deep=True สำหรับ accurate measurement

memory_usage_mb = memory_usage_bytes / (1024 * 1024)

# Row and column count
num_rows, num_cols = df.shape
# df.shape: tuple (rows, columns)
# - Alternative: len(df), len(df.columns)

logger.info(f"  - Rows: {num_rows:,}")
logger.info(f"  - Columns: {num_cols}")
logger.info(f"  - Memory: {memory_usage_mb:.2f} MB")
logger.info(f"  - File size: {file_size_mb:.2f} MB")

# ============================================================
# STEP 6: SAVE AS PARQUET
# ============================================================
logger.info(f"💾 [STEP 6/7] Saving as Parquet...")

# Create output directory if not exists
output_dir = Path('/opt/airflow/data/processed')
output_dir.mkdir(parents=True, exist_ok=True)
# mkdir(): create directory
# - parents=True: create parent directories ถ้ายังไม่มี
# - exist_ok=True: ไม่ raise error ถ้า directory มีอยู่แล้ว
# - Best practice: always create directories before writing

# Generate output filename with timestamp
timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
# strftime(): format datetime as string
# - %Y: year (4 digits)
# - %m: month (01-12)
# - %d: day (01-31)
# - %H: hour (00-23)
# - %M: minute (00-59)
# - %S: second (00-59)
# - Result: '20241224_143022'

output_filename = f'ingested_{timestamp}.parquet'
output_path = output_dir / output_filename
# Path operator /: join paths
# - Same as: output_path = output_dir.joinpath(output_filename)

try:
    df.to_parquet(
        path=str(output_path),
        # path: output file path
        # - ต้อง convert Path object เป็น str
        
        engine='pyarrow',
        # engine: library สำหรับ write Parquet
        # - Options: 'auto', 'pyarrow', 'fastparquet'
        # - pyarrow: เร็วกว่า, stable, recommended
        # - fastparquet: alternative, บาง features ไม่มี
        # - Best practice: always specify 'pyarrow'
        
        compression='snappy',
        # compression: algorithm สำหรับบีบอัด
        # - Options:
        #   * 'snappy': fast, moderate compression (recommended)
        #   * 'gzip': slower, better compression
        #   * 'lz4': fastest, light compression
        #   * 'zstd': balanced, best compression ratio
        #   * 'brotli': slowest, highest compression
        #   * None: no compression
        # - Comparison (100MB file):
        #   * snappy: 30MB, 2s write, 0.5s read
        #   * gzip: 25MB, 5s write, 1s read
        #   * lz4: 35MB, 1s write, 0.3s read
        #   * zstd: 28MB, 3s write, 0.7s read
        # - Best practice:
        #   * Default: snappy (balanced)
        #   * Storage-critical: zstd or gzip
        #   * Speed-critical: lz4
        
        index=False,
        # index: เก็บ DataFrame index หรือไม่
        # - Default: None (auto-detect)
        # - False: ไม่เก็บ index (recommended)
        # - True: เก็บ index as column
        # - Best practice: False (เพราะ index มักไม่มีความหมาย)
        
        partition_cols=None,
        # partition_cols: แบ่ง files ตาม column values
        # - Type: list of column names or None
        # - Example: partition_cols=['year', 'month']
        #   → Creates directory structure:
        #     year=2024/month=01/data.parquet
        #     year=2024/month=02/data.parquet
        # - Benefits:
        #   * Query efficiency (read เฉพาะ partitions ที่ต้องการ)
        #   * Parallel processing
        # - Best practice:
        #   * Partition by columns ที่ query บ่อย (date, region)
        #   * ไม่ควร partition ถ้า data น้อย (< 1GB)
        #   * Avoid high cardinality columns (e.g., user_id)
        # - ใน ingestion step: ไม่ partition (partition ใน loading step)
        
        row_group_size=None,
        # row_group_size: จำนวน rows ต่อ row group
        # - Default: None (auto-calculate ~64MB per group)
        # - Row group: unit of parallelism ใน Parquet
        # - Small row groups: 
        #   * Better parallelism
        #   * More file overhead
        # - Large row groups:
        #   * Lessoverhead
#   * Less parallelism
# - Best practice: ใช้ default (ให้ PyArrow คำนวณ)
)
    # Get output file size
    output_size_bytes = output_path.stat().st_size
    output_size_mb = output_size_bytes / (1024 * 1024)
    
    # Calculate compression ratio
    compression_ratio = file_size_mb / output_size_mb if output_size_mb > 0 else 0
    # Compression ratio: original size / compressed size
    # - Example: 100MB → 30MB = 3.33x compression
    # - Typical Parquet compression: 2-5x สำหรับ CSV
    
    logger.info(f"✅ Parquet saved: {output_size_mb:.2f} MB ({compression_ratio:.2f}x compression)")
    
except Exception as e:
    error_msg = f"❌ Error saving Parquet: {str(e)}"
    logger.error(error_msg)
    raise Exception(error_msg) from e

# ============================================================
# STEP 7: RETURN METADATA (via XCom)
# ============================================================
logger.info(f"🎯 [STEP 7/7] Task completed successfully")

# Calculate duration
end_time = time.time()
duration_seconds = end_time - start_time

# Create metadata dictionary
# XCom (Cross-Communication): Airflow's mechanism สำหรับแชร์ data ระหว่าง tasks
# - How it works:
#   * Task returns value → Airflow stores in XCom table
#   * Next task pulls value from XCom
# - Storage:
#   * SQLite (dev): xcom table in airflow.db
#   * PostgreSQL/MySQL (prod): xcom table
# - Size limits:
#   * SQLite: 2GB (but not recommended > 1MB)
#   * PostgreSQL: 1GB (but not recommended > 1MB)
#   * MySQL: 64KB default, max 16MB
# - Best practices:
#   * Return เฉพาะ metadata (paths, counts, flags)
#   * ห้าม return DataFrame, large objects
#   * ถ้า data ใหญ่ → save file และ return path
# - Alternative approaches:
#   * File-based: save metadata.json, return path
#   * Database: save to metadata table, return ID
#   * Object storage: S3/GCS, return URI

metadata = {
    'status': 'success',
    # status: task execution status
    # - Values: 'success', 'failed', 'partial'
    # - ใช้สำหรับ conditional branching ใน DAG
    
    'file_path': str(resolved_path),
    # file_path: input file path
    # - เก็บเพื่อ audit trail
    # - ใช้ใน error reporting
    
    'output_path': str(output_path),
    # output_path: Parquet file path
    # - Tasks ถัดไปจะใช้ path นี้อ่านข้อมูล
    # - Important: ต้อง return absolute path
    
    'rows': int(num_rows),
    # rows: จำนวน rows
    # - Type: int (convert from numpy.int64)
    # - ใช้สำหรับ validation, monitoring
    
    'columns': int(num_cols),
    # columns: จำนวน columns
    # - ใช้ตรวจสอบ schema consistency
    
    'memory_mb': round(memory_usage_mb, 2),
    # memory_mb: memory usage
    # - round(x, 2): round to 2 decimal places
    # - ใช้สำหรับ monitoring, capacity planning
    
    'file_size_mb': round(file_size_mb, 2),
    # file_size_mb: original file size
    # - ใช้คำนวณ compression ratio
    
    'output_size_mb': round(output_size_mb, 2),
    # output_size_mb: Parquet file size
    # - ใช้ track storage growth
    
    'compression_ratio': round(compression_ratio, 2),
    # compression_ratio: ประสิทธิภาพการบีบอัด
    # - Monitor compression performance
    # - Detect data quality issues (low compression = dirty data)
    
    'duration_seconds': round(duration_seconds, 2),
    # duration_seconds: processing time
    # - ใช้สำหรับ performance monitoring
    # - Detect performance degradation
    
    'timestamp': datetime.now().isoformat(),
    # timestamp: ISO 8601 format timestamp
    # - Format: '2024-12-24T14:30:22.123456'
    # - ISO 8601: international standard, machine-readable
    # - Alternative: .strftime('%Y-%m-%d %H:%M:%S')
    
    'columns_list': df.columns.tolist(),
    # columns_list: actual column names
    # - ใช้สำหรับ schema validation ใน tasks ถัดไป
    # - .tolist(): convert Index to Python list
    
    'dtypes': {col: str(dtype) for col, dtype in df.dtypes.items()},
    # dtypes: column data types
    # - dict comprehension: {key: value for ...}
    # - str(dtype): convert numpy dtype to string
    # - ใช้ตรวจสอบ type consistency
}

logger.info(f"✅ Metadata prepared for XCom")
logger.info(f"⏱️  Total duration: {duration_seconds:.2f}s")

# Return metadata → Airflow stores in XCom automatically
# Access in next task:
# ```python
# @task
# def next_task(metadata):  # Airflow passes XCom value as argument
#     file_path = metadata['output_path']
#     df = pd.read_parquet(file_path)
# ```
return metadata

---

### 🎓 Key Learnings from Task 1

**1. File I/O Best Practices**:
- Always validate paths (security)
- Handle encoding explicitly
- Check file sizes before loading
- Use Parquet for intermediate storage

**2. Error Handling Strategy**:
- Specific exceptions first (FileNotFoundError, UnicodeDecodeError)
- Generic Exception as fallback
- Always log errors with context
- Re-raise to fail task (enable retry)

**3. Memory Management**:
- Monitor memory usage with `df.memory_usage(deep=True)`
- Use `low_memory=False` for accurate dtypes
- Consider chunking for large files
- Parquet reduces memory footprint 2-5x

**4. XCom Guidelines**:
- Return metadata only (paths, counts, flags)
- Never return DataFrames or large objects
- Keep XCom payloads < 1MB
- Use ISO 8601 for timestamps

**5. Logging Best Practices**:
- Use structured logging with [STEP X/Y]
- Include metrics (rows, size, duration)
- Log at appropriate levels (INFO, WARNING, ERROR)
- Emoji for visual scanning (use sparingly)

---

### ✅ Testing Task 1
```python
# tests/test_ingestion.py
import pytest
from dags.tasks.ingestion import ingest_raw_data

def test_ingest_valid_file():
    """Test ingestion with valid CSV file"""
    metadata = ingest_raw_data(
        file_path='/opt/airflow/tests/fixtures/sample.csv'
    )
    
    assert metadata['status'] == 'success'
    assert metadata['rows'] > 0
    assert Path(metadata['output_path']).exists()

def test_ingest_missing_file():
    """Test ingestion with missing file"""
    with pytest.raises(FileNotFoundError):
        ingest_raw_data(file_path='/nonexistent/file.csv')

def test_ingest_invalid_encoding():
    """Test ingestion with wrong encoding"""
    with pytest.raises(UnicodeDecodeError):
        ingest_raw_data(
            file_path='/opt/airflow/tests/fixtures/sample.csv',
            encoding='ascii'  # Wrong encoding
        )
```

---
📊 Task 2: Data Profiling
🎯 จุดประสงค์หลัก
=============================================================================
TASK 2: DATA PROFILING - วิเคราะห์คุณภาพและลักษณะของข้อมูล
=============================================================================
วัตถุประสงค์:

โหลด Parquet file จาก Task 1
สร้าง comprehensive data quality report ด้วย ydata-profiling
วิเคราะห์ missing values, duplicates, distributions
สร้าง HTML report สำหรับ manual review
Extract key metrics เป็น JSON สำหรับ automated decisions
Flag potential data quality issues

📊 Flow การทำงาน:

Read Parquet from Task 1 output
Generate ProfileReport with ydata-profiling
Save HTML report for stakeholders
Extract JSON metrics for pipeline logic
Analyze key indicators (missing %, duplicates, correlations)
Flag issues that need attention
Return summary metrics to XCom

⚠️ สิ่งที่ต้องระวังในงานจริง:

Profiling ใช้เวลานานสำหรับ large datasets (อาจถึง 10-30 นาที)
Memory intensive (ใช้ RAM มากกว่า data size 2-3 เท่า)
HTML report size ใหญ่ (10-100 MB) → storage management
Correlation calculations expensive (O(n²) columns)
ควร skip profiling ใน CI/CD pipelines (ช้าเกินไป)


📝 Complete Implementation
python@task(
    # task_id: ชื่อ unique identifier ของ task
    # - Naming: snake_case, verb + noun
    # - Example: profile_data, generate_quality_report
    task_id='profile_data',
    
    # retries: จำนวน retry
    # - Profiling ควร retry น้อยกว่า ingestion
    # - เหตุผล: ถ้า profiling fail มักเป็น resource issue (memory, timeout)
    # - Resource issues มักไม่ recover ด้วย retry
    # - Use case:
    #   * Transient errors: 1-2 retries (e.g., temporary file lock)
    #   * Memory errors: 0-1 retries (retry ไม่ช่วย)
    # - Best practice: 1 retry พอ, ถ้ายัง fail → investigate
    retries=1,
    
    # retry_delay: เวลารอก่อน retry
    # - Profiling ควรรอนานกว่า ingestion
    # - เหตุผล: ให้เวลา OS reclaim memory
    # - Best practice: 5-10 minutes สำหรับ memory-intensive tasks
    retry_delay=timedelta(minutes=5),
    
    # execution_timeout: เวลาสูงสุดที่ task รันได้
    # - Profiling ใช้เวลานานมาก
    # - การคำนวณ:
    #   * Small dataset (< 100K rows): 5-10 minutes
    #   * Medium dataset (100K-1M rows): 10-30 minutes
    #   * Large dataset (> 1M rows): 30-60 minutes
    # - Best practice: ตั้งให้มากพอ + monitor actual runtime
    # - ถ้า timeout บ่อย → ใช้ minimal profiling mode
    execution_timeout=timedelta(minutes=30),
    
    # pool: resource pool สำหรับ task
    # - Profiling เป็น memory-intensive task
    # - ควรใช้ dedicated pool เพื่อไม่ให้กิน resources ของ tasks อื่น
    # - Configuration:
    #   * Pool name: 'memory_intensive'
    #   * Slots: 2-3 (จำกัดจำนวน profiling tasks พร้อมกัน)
    pool='memory_intensive',
    
    # pool_slots: จำนวน slots ที่ใช้
    # - Profiling ใช้ resources เยอะ → ควรใช้หลาย slots
    # - Small dataset: 1 slot
    # - Medium dataset: 2 slots
    # - Large dataset: 3 slots
    # - Trade-off: slots เยอะ → tasks อื่นรอนาน
    pool_slots=2,
    
    # retry_exponential_backoff: เพิ่ม delay แบบ exponential
    # - สำหรับ profiling: ไม่ค่อยจำเป็น
    # - เหตุผล: profiling fail มักเป็น deterministic (memory, timeout)
    # - แต่ถ้ามี transient errors: ใช้ได้
    retry_exponential_backoff=False,
)
def profile_data(
    ingestion_metadata: dict,
    # ingestion_metadata: metadata จาก Task 1 (ผ่าน XCom)
    # - Type: dict
    # - Airflow TaskFlow API จะ pass XCom value อัตโนมัติ
    # - Structure: {
    #     'output_path': '/path/to/parquet',
    #     'rows': 125483,
    #     'columns': 8,
    #     ...
    #   }
    # - การใช้งาน: ดึง output_path เพื่ออ่านไฟล์
    # - Alternative approach (explicit pull):
    #   ```python
    #   from airflow.operators.python import get_current_context
    #   context = get_current_context()
    #   metadata = context['ti'].xcom_pull(task_ids='ingest_raw_data')
    #   ```
    
    profiling_mode: str = 'minimal',
    # profiling_mode: ระดับความละเอียดของ profiling
    # - Type: str
    # - Options:
    #   * 'minimal': basic statistics only (fastest)
    #     - Missing values, types, basic stats
    #     - No correlations, no interactions
    #     - Time: ~1-5 minutes สำหรับ 1M rows
    #   
    #   * 'explorative': moderate analysis (balanced)
    #     - Everything in minimal
    #     - + Correlations (Pearson, Spearman, etc.)
    #     - + Missing value patterns
    #     - Time: ~5-15 minutes สำหรับ 1M rows
    #   
    #   * 'full': complete analysis (slowest)
    #     - Everything in explorative
    #     - + Interactions between variables
    #     - + Detailed duplicate analysis
    #     - Time: ~15-60 minutes สำหรับ 1M rows
    # 
    # - Best practice:
    #   * Development: 'minimal' (ประหยัดเวลา)
    #   * Production (daily): 'minimal' (performance)
    #   * Ad-hoc analysis: 'explorative' or 'full'
    #   * New dataset: 'full' (ครั้งแรกเพื่อเข้าใจ data)
    # 
    # - Trade-off:
    #   * More detail = more time + more memory
    #   * Minimal อาจพลาด important patterns
    
    enable_correlations: bool = False,
    # enable_correlations: คำนวณ correlation matrix หรือไม่
    # - Type: bool
    # - Default: False (เพราะ expensive)
    # - Correlation types calculated:
    #   * Pearson: linear correlation (-1 to 1)
    #   * Spearman: rank correlation (non-linear)
    #   * Kendall: rank correlation (alternative to Spearman)
    #   * Cramér's V: categorical correlation
    # - Computational cost:
    #   * O(n²) where n = number of columns
    #   * 10 columns: 45 correlations (fast)
    #   * 50 columns: 1,225 correlations (slow)
    #   * 100 columns: 4,950 correlations (very slow)
    # - Memory cost:
    #   * Stores full correlation matrices
    #   * 100 columns × 100 columns × 8 bytes = 80KB per matrix
    #   * × 4 types = 320KB (acceptable)
    # - Best practice:
    #   * False สำหรับ routine profiling
    #   * True สำหรับ feature selection, EDA
    # - Use case: ต้องการหา multicollinearity ก่อน modeling
    
    sample_size: int = None,
    # sample_size: จำนวน rows ที่จะ sample สำหรับ profiling
    # - Type: int or None
    # - Default: None (ใช้ data ทั้งหมด)
    # - Use case: dataset ใหญ่มาก (> 5M rows)
    # - Sampling strategy:
    #   * Random sampling: df.sample(n=sample_size)
    #   * Representative: stratified sampling by key columns
    # - Best practice:
    #   * < 1M rows: None (ใช้ทั้งหมด)
    #   * 1-5M rows: 500K-1M (representative)
    #   * > 5M rows: 1M (statistical validity)
    # - Trade-off:
    #   * Smaller sample = faster profiling
    #   * But: อาจพลาด outliers, rare patterns
    # - Statistical note:
    #   * n=1000: margin of error ±3%
    #   * n=10000: margin of error ±1%
    #   * n=100000: margin of error ±0.3%
    
    output_format: str = 'html',
    # output_format: format ของ profiling report
    # - Type: str
    # - Options:
    #   * 'html': interactive HTML report (recommended)
    #   * 'json': machine-readable JSON
    #   * 'both': สร้างทั้ง HTML และ JSON
    # - HTML benefits:
    #   * Interactive charts (hover, zoom)
    #   * Easy to share with non-technical stakeholders
    #   * Visual data exploration
    # - JSON benefits:
    #   * Programmatic access to metrics
    #   * Version control friendly (ถ้า pretty-printed)
    #   * Can be loaded into dashboards
    # - Best practice: 'both' สำหรับ production
    
) -> dict:
    # Return type: dictionary containing profiling results
    # - Structure:
    #   {
    #       'status': str,                    # 'success' or 'failed'
    #       'html_report_path': str,          # Path to HTML report
    #       'json_report_path': str,          # Path to JSON report
    #       'total_rows': int,                # Number of rows profiled
    #       'total_columns': int,             # Number of columns
    #       'missing_cells_pct': float,       # % of missing values
    #       'duplicate_rows': int,            # Number of duplicates
    #       'duplicate_rows_pct': float,      # % duplicates
    #       'columns_with_missing': list,     # Columns with nulls
    #       'high_missing_columns': list,     # Columns with >30% missing
    #       'high_cardinality_columns': list, # Columns with many unique values
    #       'skewed_columns': list,           # Highly skewed distributions
    #       'constant_columns': list,         # Columns with 1 unique value
    #       'flags': dict,                    # Quality flags/warnings
    #       'duration_seconds': float,        # Processing time
    #       'timestamp': str                  # ISO timestamp
    #   }
    
    """
    สร้าง data profiling report เพื่อวิเคราะห์คุณภาพข้อมูล
    
    Args:
        ingestion_metadata: metadata จาก ingestion task
            Required keys: 'output_path' (path to Parquet file)
            
        profiling_mode: ระดับความละเอียดของ profiling
            Options: 'minimal', 'explorative', 'full'
            
        enable_correlations: คำนวณ correlation หรือไม่
            True = ช้ากว่า แต่ได้ข้อมูล relationship ระหว่าง columns
            
        sample_size: จำนวน rows ที่จะ sample (None = ทั้งหมด)
            Use for large datasets (> 1M rows)
            
        output_format: format ของ report
            Options: 'html', 'json', 'both'
    
    Returns:
        Dict containing:
        - status: success/failed
        - report paths: HTML and/or JSON
        - summary metrics: missing %, duplicates, etc.
        - quality flags: issues ที่ต้อง attention
        - processing metadata: duration, timestamp
        
    Raises:
        FileNotFoundError: ถ้าไฟล์ Parquet ไม่พบ
        MemoryError: ถ้า RAM ไม่พอสำหรับ profiling
        ValueError: ถ้า profiling_mode ไม่ถูกต้อง
    """
    
    import pandas as pd
    import logging
    from ydata_profiling import ProfileReport
    from pathlib import Path
    from datetime import datetime
    import time
    import json
    
    # ============================================================
    # STEP 1: SETUP AND VALIDATION
    # ============================================================
    logger = logging.getLogger(__name__)
    logger.info("🚀 [STEP 1/8] Starting data profiling task")
    
    start_time = time.time()
    
    # Validate ingestion metadata
    if not ingestion_metadata:
        error_msg = "❌ No metadata received from ingestion task"
        logger.error(error_msg)
        raise ValueError(error_msg)
    
    if 'output_path' not in ingestion_metadata:
        error_msg = "❌ Missing 'output_path' in ingestion metadata"
        logger.error(error_msg)
        raise KeyError(error_msg)
    
    # Validate profiling_mode
    valid_modes = ['minimal', 'explorative', 'full']
    if profiling_mode not in valid_modes:
        error_msg = f"❌ Invalid profiling_mode: {profiling_mode}. Must be one of {valid_modes}"
        logger.error(error_msg)
        raise ValueError(error_msg)
    
    logger.info(f"✅ Validation passed")
    logger.info(f"  - Profiling mode: {profiling_mode}")
    logger.info(f"  - Enable correlations: {enable_correlations}")
    logger.info(f"  - Sample size: {sample_size or 'Full dataset'}")
    
    # ============================================================
    # STEP 2: READ PARQUET FILE
    # ============================================================
    logger.info(f"📖 [STEP 2/8] Reading Parquet file...")
    
    parquet_path = Path(ingestion_metadata['output_path'])
    
    # Validate file exists
    if not parquet_path.exists():
        error_msg = f"❌ Parquet file not found: {parquet_path}"
        logger.error(error_msg)
        raise FileNotFoundError(error_msg)
    
    try:
        # pd.read_parquet(): read Parquet file into DataFrame
        df = pd.read_parquet(
            path=str(parquet_path),
            # path: file path or URL
            
            engine='pyarrow',
            # engine: backend library สำหรับอ่าน Parquet
            # - Options: 'auto', 'pyarrow', 'fastparquet'
            # - Best practice: always use 'pyarrow' (faster, more features)
            
            columns=None,
            # columns: list of columns to read (None = ทั้งหมด)
            # - Use case: อ่านเฉพาะ columns ที่ต้องการ (save memory)
            # - Example: columns=['col1', 'col2', 'col3']
            # - Best practice: 
            #   * Profiling: อ่านทั้งหมด (ต้องการ complete picture)
            #   * Processing: อ่านเฉพาะที่จำเป็น (optimize memory)
            
            use_nullable_dtypes=False,
            # use_nullable_dtypes: ใช้ Pandas nullable dtypes หรือไม่
            # - Default: False
            # - Pandas nullable dtypes (Int64, string, boolean):
            #   * Support NA values properly
            #   * Traditional types: int64 ไม่รองรับ NA (convert to float64)
            # - Example:
            #   * False: int column with NA → float64
            #   * True: int column with NA → Int64 (capital I)
            # - Best practice:
            #   * False สำหรับ backward compatibility
            #   * True สำหรับ new projects (cleaner type handling)
        )
        
        logger.info(f"✅ Parquet loaded: {len(df):,} rows × {len(df.columns)} columns")
        
    except FileNotFoundError as e:
        error_msg = f"❌ File not found: {str(e)}"
        logger.error(error_msg)
        raise
        
    except Exception as e:
        error_msg = f"❌ Error reading Parquet: {str(e)}"
        logger.error(error_msg)
        raise Exception(error_msg) from e
    
    # ============================================================
    # STEP 3: SAMPLE DATA (if needed)
    # ============================================================
    original_rows = len(df)
    
    if sample_size and original_rows > sample_size:
        logger.info(f"🎲 [STEP 3/8] Sampling {sample_size:,} rows from {original_rows:,}...")
        
        # df.sample(): random sampling
        df = df.sample(
            n=sample_size,
            # n: number of rows to sample
            # - Alternative: frac=0.5 (sample 50%)
            
            random_state=42,
            # random_state: seed สำหรับ reproducibility
            # - Same seed → same sample every time
            # - Best practice: always set random_state
            # - Convention: 42 (Hitchhiker's Guide reference)
            
            replace=False,
            # replace: sampling with replacement
            # - False: แต่ละ row ถูกเลือกได้ครั้งเดียว (default)
            # - True: แต่ละ row อาจถูกเลือกหลายครั้ง
            # - Best practice: False สำหรับ profiling
        )
        
        logger.info(f"✅ Sampled to {len(df):,} rows")
    else:
        logger.info(f"ℹ️  [STEP 3/8] Using full dataset ({original_rows:,} rows)")
    
    # ============================================================
    # STEP 4: CONFIGURE PROFILING
    # ============================================================
    logger.info(f"⚙️  [STEP 4/8] Configuring ProfileReport...")
    
    # ProfileReport configuration
    # ydata-profiling มี configuration ที่ซับซ้อนมาก
    # เราจะสร้าง config based on profiling_mode
    
    # Base configuration (ใช้ทุก mode)
    config = {
        # ==============================================
        # DATASET CONFIGURATION
        # ==============================================
        'title': f'Data Quality Report - {datetime.now().strftime("%Y-%m-%d")}',
        # title: ชื่อ report
        # - แสดงที่ header ของ HTML report
        
        'dataset': {
            'description': f'Marketing data profiling ({profiling_mode} mode)',
            # description: คำอธิบาย dataset
            
            'url': str(parquet_path),
            # url: path หรือ URL ของ data source
        },
        
        # ==============================================
        # VARIABLES (COLUMNS) CONFIGURATION
        # ==============================================
        'variables': {
            'descriptions': {},
            # descriptions: คำอธิบายแต่ละ column
            # - Format: {'column_name': 'description'}
            # - Best practice: ดึงจาก data dictionary/metadata
        },
        
        # ==============================================
        # CORRELATIONS CONFIGURATION
        # ==============================================
        'correlations': {
            'auto': {
                'calculate': enable_correlations,
                # calculate: คำนวณ correlation อัตโนมัติหรือไม่
                # - False: skip (save time)
                # - True: calculate all correlation types
                
                'warn_high_correlations': True,
                # warn_high_correlations: แจ้งเตือนถ้ามี high correlation
                # - Threshold: > 0.9 (default)
                # - Use case: detect multicollinearity
            },
            
            'pearson': {
                'calculate': enable_correlations,
                # Pearson correlation: linear relationship
                # - Range: -1 (perfect negative) to +1 (perfect positive)
                # - 0: no linear correlation
                # - Use case: continuous variables
                
                'warn_high_correlations': True,
                'threshold': 0.9,
                # threshold: ค่าที่ถือว่า "high correlation"
                # - 0.9: very strong correlation
                # - 0.7-0.9: strong correlation
                # - Best practice: 0.9 สำหรับ multicollinearity detection
            },
            
            'spearman': {
                'calculate': enable_correlations,
                # Spearman correlation: monotonic relationship
                # - Rank-based (not sensitive to outliers)
                # - Can detect non-linear relationships
                # - Use case: ordinal data, non-linear relationships
                
                'warn_high_correlations': False,
                # Usually less critical than Pearson
            },
            
            'kendall': {
                'calculate': False,
                # Kendall's tau: alternative rank correlation
                # - More robust than Spearman for small samples
                # - Slower to compute
                # - Best practice: skip ใน production (redundant with Spearman)
                
                'warn_high_correlations': False,
            },
            
            'cramers': {
                'calculate': enable_correlations,
                # Cramér's V: association between categorical variables
                # - Range: 0 (no association) to 1 (perfect association)
                # - Based on chi-square test
                # - Use case: categorical columns only
                
                'warn_high_correlations': False,
            },
        },
        
        # ==============================================
        # MISSING VALUES CONFIGURATION
        # ==============================================
        'missing_diagrams': {
            'bar': True,
            # bar chart: missing values per column
            # - Visual: easy to spot columns with many nulls
            
            'matrix': True if profiling_mode != 'minimal' else False,
            # matrix: pattern of missing values across rows
            # - Shows: missing data patterns (MCAR, MAR, MNAR)
            # - Expensive: O(n*m) where n=rows, m=columns
            # - Best practice: skip in minimal mode
            
            'heatmap': True if profiling_mode == 'full' else False,
            # heatmap: correlation of missingness
            # - Shows: columns that tend to be missing together
            # - Very expensive: correlation matrix of null indicators
            # - Use case: understand missing data mechanisms
        },
        
        # ==============================================
        # DUPLICATES CONFIGURATION
        # ==============================================
        'duplicates': {
            'head': 10,
            # head: จำนวน duplicate rows ที่แสดงใน report
            # - Default: 10
            # - Show examples of duplicates สำหรับ investigation
            
            'key': None,
            # key: columns to use for duplicate detection
            # - None: ใช้ทุก column
            # - List: ['col1', 'col2'] = duplicates ตาม columns นี้เท่านั้น
            # - Best practice:
            #   * None สำหรับ profiling (comprehensive view)
            #   * Specific columns สำหรับ business logic
        },
        
        # ==============================================
        # SAMPLES CONFIGURATION
        # ==============================================
        'samples': {
            'head': 10,
            # head: จำนวน rows แรกที่แสดง
            
            'tail': 10,
            # tail: จำนวน rows สุดท้ายที่แสดง
            # - Use case: ตรวจสอบว่า data loading ถูกต้องไหม
        },
        
        # ==============================================
        # INTERACTIONS CONFIGURATION
        # ==============================================
        'interactions': {
            'continuous': profiling_mode == 'full',
            # continuous: scatter plots between numeric variables
            # - Very expensive: n choose 2 plots
            # - Example: 10 columns → 45 plots
            # - Best practice: เฉพาะ full mode
            
            'targets': [],
            # targets: specific columns to analyze interactions with
            # - Empty: analyze all pairs
            # - ['target']: analyze only with target variable
        },
        
        # ==============================================
        # PLOTS CONFIGURATION
        # ==============================================
        'plot': {
            'histogram': {
                'bins': 50,
                # bins: จำนวน bins ใน histogram
                # - More bins = more detail, but noisier
                # - Best practice: 50 for most cases
                
                'max_bins': 250,
                # max_bins: maximum bins for adaptive binning
            },
            
            'scatter_threshold': 1000,
            # scatter_threshold: max points in scatter plot
            # - ถ้ามากกว่า → sample or use density plot
            # - Best practice: 1000-5000 (readable without overplotting)
        },
        
        # ==============================================
        # HTML REPORT CONFIGURATION
        # ==============================================
        'html': {
            'style': {
                'theme': 'flatly',
                # theme: Bootstrap theme
                # - Options: 'flatly', 'united', 'simplex', etc.
                # - Best practice: 'flatly' (clean, professional)
                
                'logo': None,
                # logo: path to custom logo image
                # - Use case: company branding
                
                'primary_color': '#337ab7',
                # primary_color: main color สำหรับ charts
            },
            
            'minify_html': True,
            # minify_html: compress HTML file
            # - True: smaller file size
            # - False: readable HTML source (for debugging)
            # - Best practice: True สำหรับ production
        },
        
        # ==============================================
        # PERFORMANCE CONFIGURATION
        # ==============================================
        'pool_size': 4,
        # pool_size: จำนวน worker processes
        # - Default: 0 (single-threaded)
        # - > 0: parallel processing
        # - Best practice: 4 (balance between speed and memory)
        # - Trade-off: more workers = faster but more memory
        
        'progress_bar': False,
        # progress_bar: แสดง progress bar ใน console
        # - False: cleaner logs (recommended สำหรับ Airflow)
        # - True: useful for interactive sessions
    }
    
    # Mode-specific configuration
    if profiling_mode == 'minimal':
        # Minimal mode: fastest, basic stats only
        config.update({
            'correlations': {
                'auto': {'calculate': False},
                'pearson': {'calculate': False},
                'spearman': {'calculate': False},
                'kendall': {'calculate': False},
                'cramers': {'calculate': False},
            },
            'missing_diagrams': {
                'bar': True,
                'matrix': False,
                'heatmap': False,
            },
            'interactions': {
                'continuous': False,
            },
        })
        logger.info("  - Mode: MINIMAL (fast, basic stats only)")
        
    elif profiling_mode == 'explorative':
        # Explorative mode: balanced
        # config already set appropriately above
        logger.info("  - Mode: EXPLORATIVE (balanced, includes correlations)")
        
    elif profiling_mode == 'full':
        # Full mode: comprehensive analysis
        config.update({
            'missing_diagrams': {
                'bar': True,
                'matrix': True,
                'heatmap': True,
            },
            'interactions': {
                'continuous': True,
            },
        })
        logger.info("  - Mode: FULL (comprehensive, slow)")
    
    logger.info(f"✅ Configuration prepared")
    
    # ============================================================
    # STEP 5: GENERATE PROFILE REPORT
    # ============================================================
    logger.info(f"🔬 [STEP 5/8] Generating ProfileReport (this may take several minutes)...")
    
    profiling_start = time.time()
    
    try:
        # ProfileReport: main class from ydata-profiling
        profile = ProfileReport(
            df=df,
            # df: Pandas DataFrame to profile
            
            minimal=(profiling_mode == 'minimal'),
            # minimal: shortcut for minimal mode
            # - True: override config to minimal settings
            # - False: use custom config
            # - Best practice: use config parameter สำหรับ full control
            
            explorative=(profiling_mode == 'explorative'),
            # explorative: shortcut for explorative mode
            # - Similar to minimal parameter
            
            dark_mode=False,
            # dark_mode: ใช้ dark theme ใน HTML report
            # - True: dark background (easier on eyes)
            # - False: light background (better for printing)
            # - Best practice: False สำหรับ professional reports
            
            orange_mode=False,
            # orange_mode: special theme (deprecated)
            # - Best practice: ใช้ config['html']['style']['theme'] แทน
            
            config=config,
            # config: custom configuration dict
            # - Override default settings
            # - See config structure above
            
            lazy=False,
            # lazy: defer computation until needed
            # - False: compute everything now (default)
            # - True: compute on-demand (for interactive use)
            # - Best practice: False สำหรับ pipelines
            
            type_schema=None,
            # type_schema: override automatic type detection
            # - None: auto-detect types
            # - Dict: {'column': 'Categorical', 'column2': 'Numeric'}
            # - Types: 'Categorical', 'Numeric', 'DateTime', 'Boolean', 'URL', 'Path', 'File', 'Image'
            # - Use case: fix incorrect type detection
# - Example:
#   * ZIP codes detected as Numeric → should be Categorical
#   * IDs detected as Numeric → should be Categorical
        tsmode=False,
        # tsmode: time series mode
        # - True: additional time series analyses
        # - False: standard profiling (default)
        # - Use case: time series data with datetime index
        
        sortby=None,
        # sortby: column to sort DataFrame by
        # - None: no sorting (keep original order)
        # - String: column name
        # - Use case: sort by timestamp for time series
        
        sample=None,
        # sample: internal sampling parameter
        # - None: use full DataFrame (we already sampled in Step 3)
        # - Dict: sampling configuration
        # - Best practice: do sampling before ProfileReport
    )
    
    profiling_duration = time.time() - profiling_start
    logger.info(f"✅ ProfileReport generated in {profiling_duration:.2f}s")
    
except MemoryError as e:
    error_msg = f"❌ Out of memory during profiling: {str(e)}"
    logger.error(error_msg)
    logger.error("💡 Try: reduce sample_size or use profiling_mode='minimal'")
    raise MemoryError(error_msg)
    
except Exception as e:
    error_msg = f"❌ Error generating profile: {str(e)}"
    logger.error(error_msg)
    raise Exception(error_msg) from e

# ============================================================
# STEP 6: SAVE REPORTS
# ============================================================
logger.info(f"💾 [STEP 6/8] Saving reports...")

# Create reports directory
reports_dir = Path('/opt/airflow/data/reports')
reports_dir.mkdir(parents=True, exist_ok=True)

timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')

html_report_path = None
json_report_path = None

# Save HTML report
if output_format in ['html', 'both']:
    html_filename = f'profile_report_{timestamp}.html'
    html_report_path = reports_dir / html_filename
    
    try:
        # profile.to_file(): save report as file
        profile.to_file(
            output_file=str(html_report_path),
            # output_file: path to save HTML
            
            silent=True,
            # silent: suppress progress messages
            # - True: no console output (cleaner logs)
            # - False: show progress
        )
        
        html_size_mb = html_report_path.stat().st_size / (1024 * 1024)
        logger.info(f"✅ HTML report saved: {html_size_mb:.2f} MB")
        logger.info(f"   Path: {html_report_path}")
        
    except Exception as e:
        error_msg = f"❌ Error saving HTML report: {str(e)}"
        logger.error(error_msg)
        # Don't raise - continue with JSON

# Save JSON report
if output_format in ['json', 'both']:
    json_filename = f'profile_report_{timestamp}.json'
    json_report_path = reports_dir / json_filename
    
    try:
        # profile.to_json(): export as JSON string
        json_data = profile.to_json()
        # Returns: JSON string containing all profiling results
        # Structure: nested dict with all statistics
        
        # Save to file
        with open(json_report_path, 'w', encoding='utf-8') as f:
            # Write JSON with pretty printing
            json.dump(
                json.loads(json_data),  # Parse string to dict
                # json.loads(): convert JSON string to Python dict
                
                f,
                indent=2,
                # indent: spaces per indentation level
                # - Makes JSON human-readable
                # - Trade-off: larger file size
                # - Best practice: 2 or 4 spaces
                
                ensure_ascii=False,
                # ensure_ascii: allow non-ASCII characters
                # - False: support UTF-8 (Thai, Chinese, etc.)
                # - True: escape non-ASCII (safe but ugly)
                
                sort_keys=False,
                # sort_keys: alphabetically sort dict keys
                # - False: preserve original order (Python 3.7+)
                # - True: consistent ordering (useful for diffs)
            )
        
        json_size_mb = json_report_path.stat().st_size / (1024 * 1024)
        logger.info(f"✅ JSON report saved: {json_size_mb:.2f} MB")
        logger.info(f"   Path: {json_report_path}")
        
    except Exception as e:
        error_msg = f"❌ Error saving JSON report: {str(e)}"
        logger.error(error_msg)
        # Don't raise - we have HTML at least

# ============================================================
# STEP 7: EXTRACT KEY METRICS
# ============================================================
logger.info(f"📈 [STEP 7/8] Extracting key metrics...")

# profile.get_description(): get all statistics as dict
description = profile.get_description()
# Returns: comprehensive dict with all profiling results
# Structure:
# {
#     'analysis': {...},
#     'table': {...},         # Dataset-level stats
#     'variables': {...},     # Column-level stats
#     'correlations': {...},
#     'missing': {...},
#     'alerts': [...],
#     'package': {...}
# }

# Extract dataset-level metrics
table_stats = description.get('table', {})

total_rows = table_stats.get('n', len(df))
# n: number of observations (rows)

total_columns = table_stats.get('n_var', len(df.columns))
# n_var: number of variables (columns)

total_cells = total_rows * total_columns

missing_cells = table_stats.get('n_cells_missing', 0)
# n_cells_missing: total missing values across dataset

missing_cells_pct = (missing_cells / total_cells * 100) if total_cells > 0 else 0

duplicate_rows = table_stats.get('n_duplicates', 0)
# n_duplicates: number of duplicate rows

duplicate_rows_pct = (duplicate_rows / total_rows * 100) if total_rows > 0 else 0

memory_size_bytes = table_stats.get('memory_size', 0)
memory_size_mb = memory_size_bytes / (1024 * 1024)

logger.info(f"  - Total rows: {total_rows:,}")
logger.info(f"  - Total columns: {total_columns}")
logger.info(f"  - Missing cells: {missing_cells:,} ({missing_cells_pct:.2f}%)")
logger.info(f"  - Duplicate rows: {duplicate_rows:,} ({duplicate_rows_pct:.2f}%)")
logger.info(f"  - Memory size: {memory_size_mb:.2f} MB")

# Extract column-level metrics
variables = description.get('variables', {})

columns_with_missing = []
high_missing_columns = []  # > 30% missing
high_cardinality_columns = []  # > 50% unique values
constant_columns = []  # only 1 unique value
skewed_columns = []  # |skewness| > 2

for col_name, col_stats in variables.items():
    # Missing values analysis
    n_missing = col_stats.get('n_missing', 0)
    missing_pct = (n_missing / total_rows * 100) if total_rows > 0 else 0
    
    if n_missing > 0:
        columns_with_missing.append({
            'column': col_name,
            'missing_count': n_missing,
            'missing_pct': round(missing_pct, 2)
        })
    
    if missing_pct > 30:
        high_missing_columns.append(col_name)
    
    # Cardinality analysis
    n_distinct = col_stats.get('n_distinct', 0)
    distinct_pct = (n_distinct / total_rows * 100) if total_rows > 0 else 0
    
    if distinct_pct > 50 and n_distinct > 100:
        # High cardinality: >50% unique AND >100 unique values
        # Note: small datasets naturally have high %
        high_cardinality_columns.append({
            'column': col_name,
            'unique_count': n_distinct,
            'unique_pct': round(distinct_pct, 2)
        })
    
    # Constant columns
    if n_distinct == 1:
        constant_columns.append(col_name)
    
    # Skewness analysis (numeric columns only)
    if col_stats.get('type') in ['Numeric', 'Continuous']:
        skewness = col_stats.get('skewness', 0)
        # Skewness interpretation:
        # - |skewness| < 0.5: fairly symmetric
        # - 0.5 < |skewness| < 1: moderately skewed
        # - |skewness| > 1: highly skewed
        # - |skewness| > 2: extremely skewed
        
        if abs(skewness) > 2:
            skewed_columns.append({
                'column': col_name,
                'skewness': round(skewness, 2)
            })

logger.info(f"  - Columns with missing: {len(columns_with_missing)}")
logger.info(f"  - High missing columns (>30%): {len(high_missing_columns)}")
logger.info(f"  - High cardinality columns: {len(high_cardinality_columns)}")
logger.info(f"  - Constant columns: {len(constant_columns)}")
logger.info(f"  - Skewed columns: {len(skewed_columns)}")

# ============================================================
# STEP 8: GENERATE QUALITY FLAGS
# ============================================================
logger.info(f"🚩 [STEP 8/8] Generating quality flags...")

flags = {
    'high_missing_data': missing_cells_pct > 10,
    # Flag if > 10% of cells are missing
    # - Threshold: business-dependent
    # - Action: investigate missing data mechanism
    
    'high_duplicate_rate': duplicate_rows_pct > 5,
    # Flag if > 5% rows are duplicates
    # - May indicate data quality issue or legitimate repeats
    # - Action: verify business logic
    
    'constant_columns_exist': len(constant_columns) > 0,
    # Flag if any columns have only 1 unique value
    # - These columns provide no information
    # - Action: consider dropping
    
    'high_cardinality_issues': len(high_cardinality_columns) > total_columns * 0.2,
    # Flag if > 20% columns have high cardinality
    # - May indicate IDs, timestamps, or data quality issues
    # - Action: review column types
    
    'severely_skewed_columns': len(skewed_columns) > 0,
    # Flag if any columns are extremely skewed
    # - May need transformation (log, sqrt, Box-Cox)
    # - Action: consider transformation in cleaning step
    
    'memory_intensive': memory_size_mb > 1000,
    # Flag if dataset > 1GB
    # - May need optimization (sampling, chunking, Dask)
    # - Action: consider memory optimization strategies
}

# Count total flags
flags_raised = sum(flags.values())

if flags_raised == 0:
    logger.info(f"✅ No quality flags raised - data looks good!")
else:
    logger.warning(f"⚠️  {flags_raised} quality flag(s) raised:")
    for flag_name, flag_value in flags.items():
        if flag_value:
            logger.warning(f"     - {flag_name}")

# ============================================================
# RETURN METADATA
# ============================================================
end_time = time.time()
total_duration = end_time - start_time

metadata = {
    'status': 'success',
    
    # Report paths
    'html_report_path': str(html_report_path) if html_report_path else None,
    'json_report_path': str(json_report_path) if json_report_path else None,
    
    # Dataset metrics
    'total_rows': total_rows,
    'total_columns': total_columns,
    'memory_mb': round(memory_size_mb, 2),
    
    # Missing data metrics
    'missing_cells': missing_cells,
    'missing_cells_pct': round(missing_cells_pct, 2),
    'columns_with_missing': columns_with_missing,
    'high_missing_columns': high_missing_columns,
    
    # Duplicate metrics
    'duplicate_rows': duplicate_rows,
    'duplicate_rows_pct': round(duplicate_rows_pct, 2),
    
    # Column analysis
    'high_cardinality_columns': high_cardinality_columns,
    'constant_columns': constant_columns,
    'skewed_columns': skewed_columns,
    
    # Quality flags
    'flags': flags,
    'flags_raised_count': flags_raised,
    
    # Processing metadata
    'profiling_mode': profiling_mode,
    'profiling_duration_seconds': round(profiling_duration, 2),
    'total_duration_seconds': round(total_duration, 2),
    'sampled': sample_size is not None and original_rows > sample_size,
    'sample_size': len(df) if sample_size else None,
    'original_rows': original_rows,
    'timestamp': datetime.now().isoformat(),
}

logger.info(f"✅ Profiling completed successfully")
logger.info(f"⏱️  Total duration: {total_duration:.2f}s")

return metadata

---

### 🎓 Key Learnings from Task 2

**1. ydata-profiling Configuration**:
- **Modes**: minimal (fast) vs explorative (balanced) vs full (comprehensive)
- **Correlations**: Expensive O(n²) - enable only when needed
- **Sampling**: Use for large datasets (> 1M rows)
- **Trade-off**: Detail vs Performance

**2. Profiling Metrics to Watch**:
- **Missing data %**: > 10% = investigate pattern
- **Duplicate rate**: > 5% = verify business logic
- **Constant columns**: No information = consider dropping
- **High cardinality**: May be IDs or dirty data
- **Skewness**: |skewness| > 2 = consider transformation

**3. Performance Optimization**:
- Minimal mode: 5-10x faster than full
- Disable correlations: 3-5x faster
- Sampling: linear speedup with sample ratio
- Parallel processing: pool_size=4 recommended

**4. Memory Management**:
- Profiling uses 2-3x data size in RAM
- Large datasets (> 1GB): use sampling or minimal mode
- Monitor peak memory usage
- Consider chunking for very large data

**5. Output Management**:
- **HTML**: For stakeholders (visual, interactive)
- **JSON**: For automation (machine-readable)
- **Both**: Best practice for production
- Manage report retention (can be 10-100 MB each)

---

### ✅ Testing Task 2
```python
# tests/test_profiling.py
import pytest
from dags.tasks.profiling import profile_data

def test_profile_minimal_mode():
    """Test profiling in minimal mode"""
    ingestion_metadata = {
        'output_path': '/opt/airflow/tests/fixtures/sample.parquet',
        'rows': 1000,
        'columns': 10
    }
    
    metadata = profile_data(
        ingestion_metadata=ingestion_metadata,
        profiling_mode='minimal'
    )
    
    assert metadata['status'] == 'success'
    assert metadata['total_rows'] > 0
    assert metadata['html_report_path'] is not None

def test_profile_with_correlations():
    """Test profiling with correlation analysis"""
    ingestion_metadata = {
        'output_path': '/opt/airflow/tests/fixtures/sample.parquet'
    }
    
    metadata = profile_data(
        ingestion_metadata=ingestion_metadata,
        profiling_mode='explorative',
        enable_correlations=True
    )
    
    assert metadata['status'] == 'success'
    # Verify correlations were calculated (check JSON output)

def test_profile_with_sampling():
    """Test profiling with sampling"""
    ingestion_metadata = {
        'output_path': '/opt/airflow/tests/fixtures/large_sample.parquet'
    }
    
    metadata = profile_data(
        ingestion_metadata=ingestion_metadata,
        sample_size=10000
    )
    
    assert metadata['sampled'] == True
    assert metadata['sample_size'] == 10000
```

---

Data Cleaning
🎯 จุดประสงค์หลัก
=============================================================================
TASK 3: DATA CLEANING - ทำความสะอาดและปรับปรุงคุณภาพข้อมูล
=============================================================================
วัตถุประสงค์:

โหลดข้อมูลจาก Task 1 (Parquet file)
ใช้ insights จาก Task 2 (Profiling) เพื่อตัดสินใจ cleaning strategies
Handle missing values (imputation, flagging, removal)
Remove/flag duplicates ตาม business logic
Fix data types (dates, numerics, categories)
Standardize formats (dates, strings, phone numbers)
Handle outliers (detect และ treat)
Create data quality flags
Log cleaning summary พร้อม before/after comparison

📊 Flow การทำงาน:

Load data และ profiling metadata
Analyze missing values → decide strategy per column
Handle duplicates based on business rules
Fix data types (date parsing, numeric conversion)
Standardize string formats
Detect และ treat outliers
Create quality flag columns
Validate cleaning results
Save cleaned data
Return cleaning summary

⚠️ สิ่งที่ต้องระวังในงานจริง:

Idempotency: รัน cleaning หลายรอบต้องได้ผลเหมือนเดิม
Data loss tracking: บันทึกทุกครั้งที่ลบหรือเปลี่ยนแปลงข้อมูล
Business logic: cleaning decisions ต้องสอดคล้องกับ business requirements
Reversibility: เก็บ backup หรือ flag แทนการลบ (ถ้าเป็นไปได้)
Documentation: บันทึก WHY not just WHAT สำหรับทุก cleaning decision


📝 Complete Implementation
python@task(
    # task_id: ชื่อ unique identifier
    task_id='clean_data',
    
    # retries: cleaning ควร retry หรือไม่?
    # - Cleaning operations are deterministic
    # - ถ้า fail → มักเป็น logic error, data quality issue
    # - Retry ไม่ช่วย แต่ให้ 1-2 retries สำหรับ transient I/O errors
    retries=2,
    
    # retry_delay: delay before retry
    # - Cleaning ไม่ต้องรอนาน (not resource-intensive like profiling)
    retry_delay=timedelta(minutes=2),
    
    # execution_timeout: cleaning อาจใช้เวลานาน
    # - Depends on: data size, number of transformations
    # - Estimation:
    #   * Small (< 100K rows): 2-5 minutes
    #   * Medium (100K-1M rows): 5-15 minutes
    #   * Large (> 1M rows): 15-30 minutes
    execution_timeout=timedelta(minutes=20),
    
    # pool: cleaning เป็น CPU-intensive task
    # - ใช้ CPU pool แทน memory pool
    pool='cpu_intensive',
    
    # pool_slots: cleaning ไม่ค่อย intensive
    pool_slots=1,
)
def clean_data(
    ingestion_metadata: dict,
    # ingestion_metadata: metadata จาก Task 1
    # - Required: output_path to Parquet file
    
    profiling_metadata: dict,
    # profiling_metadata: metadata จาก Task 2
    # - ใช้ insights เพื่อตัดสินใจ cleaning strategies
    # - Example: high_missing_columns, skewed_columns
    
    missing_threshold: float = 50.0,
    # missing_threshold: % missing ที่ยอมรับได้ก่อนจะ drop column
    # - Type: float (0-100)
    # - Default: 50.0 (drop columns with > 50% missing)
    # - Best practice:
    #   * Conservative: 30-40% (high quality)
    #   * Moderate: 50-60% (balanced)
    #   * Permissive: 70-80% (keep more data)
    # - Business consideration:
    #   * Important columns: keep even with high missing
    #   * Redundant columns: drop even with low missing
    
    duplicate_strategy: str = 'keep_first',
    # duplicate_strategy: วิธีจัดการ duplicates
    # - Type: str
    # - Options:
    #   * 'keep_first': เก็บ occurrence แรก (default)
    #   * 'keep_last': เก็บ occurrence สุดท้าย
    #   * 'keep_all': ไม่ลบ, flag only
    #   * 'remove_all': ลบทุก occurrences (รวมตัวแรก)
    # - Best practice:
    #   * Temporal data: 'keep_first' or 'keep_last' (ตาม business logic)
    #   * Transactions: investigate before removing
    #   * 'keep_all' + flag: safest approach
    
    date_columns: list = None,
    # date_columns: columns ที่เป็น dates
    # - Type: list of column names or None
    # - None: auto-detect (อาจผิดพลาด)
    # - Best practice: specify explicitly
    # - Example: ['InvoiceDate', 'OrderDate', 'ShipDate']
    
    date_format: str = None,
    # date_format: format string สำหรับ parse dates
    # - Type: str or None
    # - None: infer format (slower, less reliable)
    # - Format codes:
    #   * %Y: 4-digit year (2024)
    #   * %y: 2-digit year (24)
    #   * %m: month (01-12)
    #   * %d: day (01-31)
    #   * %H: hour (00-23)
    #   * %M: minute (00-59)
    #   * %S: second (00-59)
    # - Examples:
    #   * '%Y-%m-%d': 2024-12-24
    #   * '%d/%m/%Y': 24/12/2024
    #   * '%Y-%m-%d %H:%M:%S': 2024-12-24 14:30:00
    # - Best practice: specify format for consistency and speed
    
    outlier_method: str = 'iqr',
    # outlier_method: วิธีตรวจจับ outliers
    # - Type: str
    # - Options:
    #   * 'iqr': Interquartile Range (Q1-1.5*IQR, Q3+1.5*IQR)
    #   * 'zscore': Z-score method (|z| > 3)
    #   * 'isolation_forest': ML-based (sklearn)
    #   * 'none': skip outlier detection
    # - Comparison:
    #   * IQR: simple, robust, good for skewed data
    #   * Z-score: assumes normal distribution, sensitive to extremes
    #   * Isolation Forest: sophisticated, handles multiple dimensions
    # - Best practice: 'iqr' for most cases
    
    outlier_action: str = 'flag',
    # outlier_action: วิธีจัดการ outliers
    # - Type: str
    # - Options:
    #   * 'flag': เพิ่ม boolean column (safest)
    #   * 'cap': cap at min/max thresholds (winsorizing)
    #   * 'remove': ลบ rows (data loss!)
    #   * 'transform': log/sqrt transformation
    # - Best practice:
    #   * Default: 'flag' (reversible, transparent)
    #   * Modeling: 'cap' or 'transform'
    #   * Never: 'remove' without investigation
    
    categorical_threshold: int = 50,
    # categorical_threshold: จำนวน unique values สูงสุดสำหรับ categorical
    # - Type: int
    # - ถ้า unique values ≤ threshold → convert to category dtype
    # - Benefits of category dtype:
    #   * Memory savings (significant for large data)
    #   * Faster operations (group by, merge)
    #   * Explicit value set (validation)
    # - Example:
    #   * 10K rows, 'Country' column (50 countries)
    #   * object: 10K strings = ~200KB
    #   * category: 50 strings + 10K codes = ~30KB
    #   * Savings: ~85%
    # - Best practice:
    #   * Low cardinality (< 50): always use category
    #   * Medium (50-1000): consider case by case
    #   * High (> 1000): usually keep as object
    
) -> dict:
    # Return type: cleaning summary metadata
    # Structure:
    # {
    #     'status': str,
    #     'output_path': str,
    #     'rows_before': int,
    #     'rows_after': int,
    #     'rows_removed': int,
    #     'columns_before': int,
    #     'columns_after': int,
    #     'columns_dropped': list,
    #     'missing_values_filled': dict,
    #     'duplicates_removed': int,
    #     'outliers_detected': dict,
    #     'dtypes_changed': dict,
    #     'cleaning_summary': dict,
    #     'duration_seconds': float,
    #     'timestamp': str
    # }
    
    """
    ทำความสะอาดข้อมูลตาม best practices และ business rules
    
    Args:
        ingestion_metadata: metadata จาก ingestion task
        profiling_metadata: metadata จาก profiling task
        missing_threshold: % missing ที่ drop column (0-100)
        duplicate_strategy: 'keep_first', 'keep_last', 'keep_all', 'remove_all'
        date_columns: list ของ column names ที่เป็น dates
        date_format: format string สำหรับ parse dates
        outlier_method: 'iqr', 'zscore', 'isolation_forest', 'none'
        outlier_action: 'flag', 'cap', 'remove', 'transform'
        categorical_threshold: max unique values สำหรับ category dtype
    
    Returns:
        Dict containing:
        - status: success/failed
        - output_path: path to cleaned Parquet
        - cleaning statistics: rows/columns before/after
        - transformation details: what changed
        - duration: processing time
        
    Raises:
        FileNotFoundError: ถ้า input Parquet ไม่พบ
        ValueError: ถ้า parameters ไม่ถูกต้อง
        KeyError: ถ้า specified columns ไม่มีใน data
    """
    
    import pandas as pd
    import numpy as np
    import logging
    from pathlib import Path
    from datetime import datetime
    import time
    from scipy import stats
    
    # ============================================================
    # STEP 1: SETUP AND VALIDATION
    # ============================================================
    logger = logging.getLogger(__name__)
    logger.info("🚀 [STEP 1/10] Starting data cleaning task")
    
    start_time = time.time()
    
    # Validate metadata
    if not ingestion_metadata or 'output_path' not in ingestion_metadata:
        error_msg = "❌ Invalid ingestion metadata"
        logger.error(error_msg)
        raise ValueError(error_msg)
    
    # Validate parameters
    if not 0 <= missing_threshold <= 100:
        error_msg = f"❌ missing_threshold must be 0-100, got {missing_threshold}"
        logger.error(error_msg)
        raise ValueError(error_msg)
    
    valid_dup_strategies = ['keep_first', 'keep_last', 'keep_all', 'remove_all']
    if duplicate_strategy not in valid_dup_strategies:
        error_msg = f"❌ Invalid duplicate_strategy: {duplicate_strategy}"
        logger.error(error_msg)
        raise ValueError(error_msg)
    
    valid_outlier_methods = ['iqr', 'zscore', 'isolation_forest', 'none']
    if outlier_method not in valid_outlier_methods:
        error_msg = f"❌ Invalid outlier_method: {outlier_method}"
        logger.error(error_msg)
        raise ValueError(error_msg)
    
    valid_outlier_actions = ['flag', 'cap', 'remove', 'transform']
    if outlier_action not in valid_outlier_actions:
        error_msg = f"❌ Invalid outlier_action: {outlier_action}"
        logger.error(error_msg)
        raise ValueError(error_msg)
    
    logger.info(f"✅ Configuration validated")
    
    # ============================================================
    # STEP 2: LOAD DATA
    # ============================================================
    logger.info(f"📖 [STEP 2/10] Loading data from Parquet...")
    
    parquet_path = Path(ingestion_metadata['output_path'])
    
    if not parquet_path.exists():
        error_msg = f"❌ Parquet file not found: {parquet_path}"
        logger.error(error_msg)
        raise FileNotFoundError(error_msg)
    
    try:
        df = pd.read_parquet(parquet_path, engine='pyarrow')
        logger.info(f"✅ Data loaded: {len(df):,} rows × {len(df.columns)} columns")
        
    except Exception as e:
        error_msg = f"❌ Error loading Parquet: {str(e)}"
        logger.error(error_msg)
        raise
    
    # Store original counts for comparison
    rows_before = len(df)
    columns_before = len(df.columns)
    original_columns = df.columns.tolist()
    
    # Initialize tracking dictionaries
    cleaning_summary = {
        'missing_values_handled': {},
        'duplicates_info': {},
        'outliers_info': {},
        'dtypes_changed': {},
        'columns_dropped': [],
        'transformations': []
    }
    
    # ============================================================
    # STEP 3: HANDLE MISSING VALUES
    # ============================================================
    logger.info(f"🔍 [STEP 3/10] Handling missing values...")
    
    # Calculate missing percentage per column
    missing_stats = df.isnull().sum()
    # df.isnull(): return DataFrame of boolean (True = missing)
    # .sum(): count True values per column
    # Result: Series with column names and missing counts
    
    missing_pct = (missing_stats / len(df) * 100).round(2)
    # Calculate percentage and round to 2 decimals
    
    # Log columns with missing values
    cols_with_missing = missing_stats[missing_stats > 0]
    if len(cols_with_missing) > 0:
        logger.info(f"  - Found {len(cols_with_missing)} columns with missing values:")
        for col in cols_with_missing.index[:10]:  # Show first 10
            logger.info(f"    * {col}: {missing_stats[col]:,} ({missing_pct[col]:.2f}%)")
        if len(cols_with_missing) > 10:
            logger.info(f"    ... and {len(cols_with_missing) - 10} more")
    else:
        logger.info(f"  - No missing values found!")
    
    # Strategy 1: Drop columns with excessive missing values
    # --------------------------------------------------------
    cols_to_drop = missing_pct[missing_pct > missing_threshold].index.tolist()
    # missing_pct > threshold: boolean Series
    # .index: get column names
    # .tolist(): convert to list
    
    if cols_to_drop:
        logger.info(f"  - Dropping {len(cols_to_drop)} columns with >{missing_threshold}% missing:")
        for col in cols_to_drop:
            logger.info(f"    * {col}: {missing_pct[col]:.2f}% missing")
            cleaning_summary['columns_dropped'].append({
                'column': col,
                'reason': f'>{missing_threshold}% missing',
                'missing_pct': missing_pct[col]
            })
        
        df = df.drop(columns=cols_to_drop)
        # df.drop(): remove columns
        # - columns: list of column names to drop
        # - axis: 0 (rows) or 1 (columns) - use columns= instead
        # - inplace: False (return new DataFrame) or True (modify in place)
        # - Best practice: inplace=False (functional style, safer)
        
        logger.info(f"  - Columns after drop: {len(df.columns)}")
    
    # Strategy 2: Fill numeric columns with median
    # --------------------------------------------------------
    numeric_cols = df.select_dtypes(include=[np.number]).columns
    # df.select_dtypes(): select columns by data type
    # - include: list of types to include
    #   * np.number: all numeric types (int, float)
    #   * 'object': strings
    #   * 'datetime64': datetimes
    #   * 'category': categorical
    # - exclude: list of types to exclude
    # - Returns: DataFrame with selected columns
    # - .columns: get column names only
    
    numeric_cols_with_missing = [col for col in numeric_cols if df[col].isnull().any()]
    # List comprehension: filter numeric columns that have missing values
    # df[col].isnull().any(): True if column has any missing values
    
    if numeric_cols_with_missing:
        logger.info(f"  - Filling {len(numeric_cols_with_missing)} numeric columns with median:")
        
        for col in numeric_cols_with_missing:
            missing_count = df[col].isnull().sum()
            
            # Calculate median (ignoring NaN)
            median_value = df[col].median()
            # df[col].median(): calculate median
            # - Automatically ignores NaN values
            # - Returns: scalar value
            # - Alternative methods:
            #   * .mean(): arithmetic mean (sensitive to outliers)
            #   * .mode(): most frequent value
            #   * Forward fill: .ffill() (time series)
            #   * Backward fill: .bfill() (time series)
            
            # Why median vs mean?
            # - Median: robust to outliers (recommended)
            # - Mean: affected by outliers, can be misleading
            # - Example: [1, 2, 3, 4, 1000]
            #   * Mean: 202 (skewed by 1000)
            #   * Median: 3 (representative)
            
            # Fill missing values
            df[col] = df[col].fillna(median_value)
            # df[col].fillna(): fill missing values
            # - value: scalar, dict, Series, or DataFrame
            # - method: 'ffill' (forward fill), 'bfill' (backward fill)
            # - inplace: False (default) or True
            # - limit: maximum number of consecutive NaNs to fill
            # - Best practice: assign back to column (inplace=False)
            
            logger.info(f"    * {col}: filled {missing_count:,} values with median={median_value:.2f}")
            
            cleaning_summary['missing_values_handled'][col] = {
                'method': 'median',
                'count': int(missing_count),
                'fill_value': float(median_value)
            }
    
    # Strategy 3: Fill categorical columns with mode or 'Unknown'
    # --------------------------------------------------------
    categorical_cols = df.select_dtypes(include=['object', 'category']).columns
    categorical_cols_with_missing = [col for col in categorical_cols if df[col].isnull().any()]
    
    if categorical_cols_with_missing:
        logger.info(f"  - Filling {len(categorical_cols_with_missing)} categorical columns:")
        
        for col in categorical_cols_with_missing:
            missing_count = df[col].isnull().sum()
            
            # Calculate mode (most frequent value)
            mode_values = df[col].mode()
            # df[col].mode(): return Series of most frequent value(s)
            # - Can return multiple values if there's a tie
            # - Returns: Series (even if single value)
            
            if len(mode_values) > 0:
                # Use mode if exists
                fill_value = mode_values[0]
                method = 'mode'
            else:
                # Use 'Unknown' if no mode (shouldn't happen, but safe)
                fill_value = 'Unknown'
                method = 'constant'
            
            df[col] = df[col].fillna(fill_value)
            
            logger.info(f"    * {col}: filled {missing_count:,} values with {method}='{fill_value}'")
            
            cleaning_summary['missing_values_handled'][col] = {
                'method': method,
                'count': int(missing_count),
                'fill_value': str(fill_value)
            }
    
    # Strategy 4: Create missing value flags
    # --------------------------------------------------------
    # สำหรับ columns สำคัญ: flag แทนการ fill
    # เพื่อให้ model รู้ว่าค่านี้เป็น imputed
    
    if profiling_metadata and 'high_missing_columns' in profiling_metadata:
        high_missing_cols = profiling_metadata['high_missing_columns']
        # Get columns ที่มี missing สูงจาก profiling
        
        for col in high_missing_cols:
            if col in df.columns:
                # Create flag column
                flag_col_name = f'{col}_was_missing'
                df[flag_col_name] = df[col].isnull().astype(int)
                # .isnull(): return boolean Series
                # .astype(int): convert True→1, False→0
                # Reason: boolean flags ใช้ memory น้อยกว่า int
                # แต่ int ดีกว่าสำหรับ CSV export และ compatibility
                
                logger.info(f"    * Created flag: {flag_col_name}")
                
                cleaning_summary['transformations'].append({
                    'type': 'create_flag',
                    'column': flag_col_name,
                    'source': col
                })
    
    logger.info(f"✅ Missing values handled")
    
    # ============================================================
    # STEP 4: HANDLE DUPLICATES
    # ============================================================
    logger.info(f"🔄 [STEP 4/10] Handling duplicate rows...")
    
    # Count duplicates
    duplicate_mask = df.duplicated(keep=False)
    # df.duplicated(): identify duplicate rows
    # - subset: columns to consider (None = all columns)
    # - keep: which duplicates to mark
    #   * 'first': mark all except first occurrence (default)
    #   * 'last': mark all except last occurrence
    #   * False: mark all duplicates (including first)
    # - Returns: boolean Series (True = duplicate)
    
    n_duplicates = duplicate_mask.sum()
    duplicate_pct = (n_duplicates / len(df) * 100) if len(df) > 0 else 0
    
    logger.info(f"  - Found {n_duplicates:,} duplicate rows ({duplicate_pct:.2f}%)")
    
    if n_duplicates > 0:
        # Show examples of duplicates
        if n_duplicates <= 10:
            logger.info(f"  - Duplicate examples:")
            duplicate_examples = df[duplicate_mask].head(5)
            for idx, row in duplicate_examples.iterrows():
                logger.info(f"    * Index {idx}: {row.to_dict()}")
        
        # Apply duplicate strategy
        if duplicate_strategy == 'keep_first':
            # Keep first occurrence, remove rest
            df = df.drop_duplicates(keep='first')
            # df.drop_duplicates(): remove duplicate rows
            # - Returns: DataFrame without duplicates
            # - subset: columns to consider
            # - keep: 'first', 'last', or False
            # - inplace: False or True
            
            rows_removed = rows_before - len(df)
            logger.info(f"  - Removed {rows_removed:,} duplicate rows (kept first)")
            
        elif duplicate_strategy == 'keep_last':
            df = df.drop_duplicates(keep='last')
            rows_removed = rows_before - len(df)
            logger.info(f"  - Removed {rows_removed:,} duplicate rows (kept last)")
            
        elif duplicate_strategy == 'keep_all':
            # Don't remove, just flag
            df['is_duplicate'] = duplicate_mask.astype(int)
            rows_removed = 0
            logger.info(f"  - Flagged {n_duplicates:,} duplicates (kept all)")
            
            cleaning_summary['transformations'].append({
                'type': 'create_flag',
                'column': 'is_duplicate',
                'count': int(n_duplicates)
            })
            
        elif duplicate_strategy == 'remove_all':
            # Remove all occurrences (including first)
            df = df[~duplicate_mask]
            # ~duplicate_mask: invert boolean (True→False, False→True)
            # df[boolean_series]: filter rows where True
            
            rows_removed = n_duplicates
            logger.info(f"  - Removed {rows_removed:,} rows (all duplicates)")
        
        cleaning_summary['duplicates_info'] = {
            'found': int(n_duplicates),
            'strategy': duplicate_strategy,
            'removed': int(rows_removed) if duplicate_strategy != 'keep_all' else 0
        }
    else:
        logger.info(f"  - No duplicates found")
        cleaning_summary['duplicates_info'] = {
            'found': 0,
            'strategy': duplicate_strategy,
            'removed': 0
        }
    
    logger.info(f"✅ Duplicates handled")
    
    # ============================================================
    # STEP 5: FIX DATA TYPES - DATES
    # ============================================================
    logger.info(f"📅 [STEP 5/10] Converting date columns...")
    
    if date_columns is None:
        # Auto-detect date columns
        # Look for columns with 'date', 'time', 'timestamp' in name
        potential_date_cols = [col for col in df.columns 
                              if any(keyword in col.lower() 
                                    for keyword in ['date', 'time', 'timestamp'])]
        date_columns = potential_date_cols
        
        if date_columns:
            logger.info(f"  - Auto-detected {len(date_columns)} potential date columns:")
            for col in date_columns:
                logger.info(f"    * {col}")
    
    if date_columns:
        for col in date_columns:
            if col not in df.columns:
                logger.warning(f"  ⚠️  Column '{col}' not found in data, skipping")
                continue
            
            # Skip if already datetime
            if pd.api.types.is_datetime64_any_dtype(df[col]):
                # pd.api.types.is_datetime64_any_dtype(): check if datetime
                # - Returns: True if column is datetime type
                # - Alternative: df[col].dtype == 'datetime64[ns]'
                logger.info(f"  - {col}: already datetime, skipping")
                continue
            
            logger.info(f"  - Converting {col} to datetime...")
            
            original_dtype = str(df[col].dtype)
            
            try:
                # pd.to_datetime(): convert to datetime
                df[col] = pd.to_datetime(
                    df[col],
                    # arg: scalar, list, Series, DataFrame
                    
                    format=date_format,
                    # format: strftime format string
                    # - None: infer format (slower)
                    # - String: exact format (faster, more reliable)
                    # - Examples:
                    #   * '%Y-%m-%d': 2024-12-24
                    #   * '%d/%m/%Y %H:%M': 24/12/2024 14:30
                    # - Best practice: specify format if known
                    
                    errors='coerce',
                    # errors: how to handle parsing errors
                    # - 'raise': raise exception on error
                    # - 'coerce': invalid dates → NaT (Not a Time)
                    # - 'ignore': return original data on error
                    # - Best practice: 'coerce' (convert what you can, flag rest)
                    
                    dayfirst=False,
                    # dayfirst: interpret first value as day?
                    # - False: MM/DD/YYYY (US format)
                    # - True: DD/MM/YYYY (European format)
                    # - Only affects ambiguous dates (e.g., 01/02/2024)
                    # - Best practice: use format parameter instead
                    
                    infer_datetime_format=True,
                    # infer_datetime_format: auto-detect format
                    # - True: try to infer format (faster if consistent)
                    # - False: parse each value individually
                    # - Deprecated in Pandas 2.0
                    # - Best practice: specify format explicitly
                    
                    utc=False,
                    # utc: convert to UTC timezone
                    # - False: keep timezone-naive (default)
                    # - True: convert to UTC
                    # - Use case: when working with multiple timezones
                )
                
                # Count invalid dates (converted to NaT)
                nat_count = df[col].isnull().sum()
                if nat_count > 0:
                    logger.warning(f"    ⚠️  {nat_count:,} invalid dates converted to NaT")
                
                logger.info(f"    ✅ Converted: {original_dtype} → datetime64[ns]")
                
                cleaning_summary['dtypes_changed'][col] = {
                    'from': original_dtype,
                    'to': 'datetime64[ns]',
                    'invalid_count': int(nat_count)
                }
                
            except Exception as e:
                logger.error(f"    ❌ Error converting {col}: {str(e)}")
                logger.error(f"       Sample values: {df[col].head(3).tolist()}")
                # Don't raise - continue with other columns
    else:
        logger.info(f"  - No date columns specified")
    
    logger.info(f"✅ Date conversion completed")
    
    # ============================================================
    # STEP 6: FIX DATA TYPES - NUMERIC COLUMNS
    # ============================================================
    logger.info(f"🔢 [STEP 6/10] Converting numeric columns...")
    
    # Find object columns that might be numeric
    object_cols = df.select_dtypes(include=['object']).columns
for col in object_cols:
    # Try to convert to numeric
    # Skip if column name suggests it's categorical
    if any(keyword in col.lower() for keyword in ['id', 'code', 'category', 'type', 'name']):
        continue
    
    # Sample a few values to check if numeric
    sample = df[col].dropna().head(100)
    if len(sample) == 0:
        continue
    
    # Try conversion on sample
    try:
        pd.to_numeric(sample, errors='raise')
        # If successful, convert entire column
        
        logger.info(f"  - Converting {col} to numeric...")
        original_dtype = str(df[col].dtype)
        
        df[col] = pd.to_numeric(
            df[col],
            # arg: scalar, list, Series
            
            errors='coerce',
            # errors: how to handle non-numeric values
            # - 'raise': raise exception
            # - 'coerce': non-numeric → NaN
            # - 'ignore': return original if error
            # - Best practice: 'coerce' (convert what you can)
            
            downcast=None,
            # downcast: convert to smaller dtype
            # - None: no downcasting (default)
            # - 'integer': try int8, int16, int32, int64
            # - 'signed': try signed integers
            # - 'unsigned': try unsigned integers
            # - 'float': try float32, float64
            # - Benefits: memory savings
            # - Trade-off: potential overflow
            # - Best practice: None (safety over memory)
        )
        
        # Count non-numeric values
        nan_count = df[col].isnull().sum()
        if nan_count > 0:
            logger.warning(f"    ⚠️  {nan_count:,} non-numeric values converted to NaN")
        
        logger.info(f"    ✅ Converted: {original_dtype} → {df[col].dtype}")
        
        cleaning_summary['dtypes_changed'][col] = {
            'from': original_dtype,
            'to': str(df[col].dtype),
            'invalid_count': int(nan_count)
        }
        
    except:
        # Not numeric, skip
        pass

logger.info(f"✅ Numeric conversion completed")

# ============================================================
# STEP 7: OPTIMIZE CATEGORICAL COLUMNS
# ============================================================
logger.info(f"🏷️  [STEP 7/10] Optimizing categorical columns...")

# Find low-cardinality object columns
object_cols = df.select_dtypes(include=['object']).columns

categorical_converted = 0

for col in object_cols:
    n_unique = df[col].nunique()
    # df[col].nunique(): count unique values
    # - dropna: True (default) - exclude NaN from count
    # - Returns: int
    
    if n_unique <= categorical_threshold and n_unique > 1:
        # Convert to category if:
        # - Unique values ≤ threshold
        # - More than 1 unique value (not constant)
        
        memory_before = df[col].memory_usage(deep=True)
        # memory_usage(deep=True): actual memory including strings
        # - deep=False: only pointer size
        # - deep=True: full object size (accurate)
        
        df[col] = df[col].astype('category')
        # .astype('category'): convert to categorical dtype
        # - Pandas stores: categories + codes
        # - categories: unique values
        # - codes: integer mapping to categories
        # - Example:
        #   * Values: ['A', 'B', 'A', 'A', 'B']
        #   * Categories: ['A', 'B']
        #   * Codes: [0, 1, 0, 0, 1]
        
        memory_after = df[col].memory_usage(deep=True)
        memory_saved = memory_before - memory_after
        memory_saved_pct = (memory_saved / memory_before * 100) if memory_before > 0 else 0
        
        logger.info(f"  - {col}: {n_unique} unique values")
        logger.info(f"    Memory: {memory_before/1024:.1f}KB → {memory_after/1024:.1f}KB ({memory_saved_pct:.1f}% saved)")
        
        categorical_converted += 1
        
        cleaning_summary['dtypes_changed'][col] = {
            'from': 'object',
            'to': 'category',
            'unique_values': int(n_unique),
            'memory_saved_kb': round(memory_saved / 1024, 2)
        }

logger.info(f"✅ Converted {categorical_converted} columns to category dtype")

# ============================================================
# STEP 8: STANDARDIZE STRING FORMATS
# ============================================================
logger.info(f"✏️  [STEP 8/10] Standardizing string formats...")

# Get remaining object columns (not converted to category)
string_cols = df.select_dtypes(include=['object']).columns

for col in string_cols:
    if df[col].dtype == 'object':  # Skip if already processed
        logger.info(f"  - Standardizing {col}...")
        
        # Strip whitespace
        # df[col].str: string accessor for vectorized string operations
        # - Only works on object/string columns
        # - Methods: .lower(), .upper(), .strip(), .replace(), etc.
        df[col] = df[col].str.strip()
        # .strip(): remove leading/trailing whitespace
        # - Equivalent to: df[col].apply(lambda x: x.strip() if isinstance(x, str) else x)
        # - Much faster than .apply()
        
        # Convert to lowercase (optional, based on use case)
        # df[col] = df[col].str.lower()
        
        cleaning_summary['transformations'].append({
            'type': 'standardize_strings',
            'column': col,
            'operations': ['strip_whitespace']
        })

logger.info(f"✅ String standardization completed")

# ============================================================
# STEP 9: HANDLE OUTLIERS
# ============================================================
logger.info(f"📊 [STEP 9/10] Detecting and handling outliers...")

if outlier_method != 'none':
    numeric_cols = df.select_dtypes(include=[np.number]).columns
    
    for col in numeric_cols:
        # Skip columns that are likely IDs or counts
        if any(keyword in col.lower() for keyword in ['id', 'count', 'quantity']):
            continue
        
        logger.info(f"  - Analyzing {col}...")
        
        if outlier_method == 'iqr':
            # IQR Method (Interquartile Range)
            # --------------------------------
            Q1 = df[col].quantile(0.25)
            # quantile(0.25): 25th percentile (Q1)
            # - 25% of data ≤ Q1
            
            Q3 = df[col].quantile(0.75)
            # quantile(0.75): 75th percentile (Q3)
            # - 75% of data ≤ Q3
            
            IQR = Q3 - Q1
            # IQR: Interquartile Range
            # - Middle 50% of data
            # - Robust to outliers (unlike std)
            
            # Define outlier boundaries
            lower_bound = Q1 - 1.5 * IQR
            upper_bound = Q3 + 1.5 * IQR
            # 1.5 * IQR: standard multiplier
            # - Tukey's fences for outlier detection
            # - 1.5: moderate outliers
            # - 3.0: extreme outliers
            # - Values outside [lower, upper] = outliers
            
            outlier_mask = (df[col] < lower_bound) | (df[col] > upper_bound)
            # |: logical OR
            # &: logical AND
            # ~: logical NOT
            
        elif outlier_method == 'zscore':
            # Z-score Method
            # --------------
            # z = (x - mean) / std
            # |z| > 3: outlier (99.7% within ±3σ in normal distribution)
            
            z_scores = np.abs(stats.zscore(df[col], nan_policy='omit'))
            # stats.zscore(): calculate z-scores
            # - nan_policy: 'propagate', 'raise', or 'omit'
            # - 'omit': ignore NaN values
            # - np.abs(): absolute value (direction doesn't matter)
            
            outlier_mask = z_scores > 3
            # Threshold: 3 standard deviations
            # - 3σ: 99.7% confidence
            # - 2.5σ: 99.0% confidence (more sensitive)
            
        else:
            # Other methods (isolation_forest) can be added here
            continue
        
        n_outliers = outlier_mask.sum()
        outlier_pct = (n_outliers / len(df) * 100) if len(df) > 0 else 0
        
        if n_outliers > 0:
            logger.info(f"    Found {n_outliers:,} outliers ({outlier_pct:.2f}%)")
            
            if outlier_action == 'flag':
                # Create flag column
                flag_col = f'{col}_is_outlier'
                df[flag_col] = outlier_mask.astype(int)
                logger.info(f"    Created flag: {flag_col}")
                
            elif outlier_action == 'cap':
                # Cap outliers at boundaries (winsorizing)
                if outlier_method == 'iqr':
                    df[col] = df[col].clip(lower=lower_bound, upper=upper_bound)
                    # .clip(): limit values to range
                    # - lower: minimum value
                    # - upper: maximum value
                    # - Values < lower → lower
                    # - Values > upper → upper
                    logger.info(f"    Capped values to [{lower_bound:.2f}, {upper_bound:.2f}]")
                
            elif outlier_action == 'remove':
                # Remove outlier rows (data loss!)
                df = df[~outlier_mask]
                logger.info(f"    Removed {n_outliers:,} rows with outliers")
                
            elif outlier_action == 'transform':
                # Log transformation
                # Only if all values > 0
                if (df[col] > 0).all():
                    df[col] = np.log1p(df[col])
                    # np.log1p(x): log(1 + x)
                    # - Handles x=0 gracefully (log(1)=0)
                    # - Use np.expm1() to inverse: expm1(log1p(x)) = x
                    logger.info(f"    Applied log transformation")
                else:
                    logger.warning(f"    ⚠️  Cannot transform (has negative/zero values)")
            
            cleaning_summary['outliers_info'][col] = {
                'method': outlier_method,
                'action': outlier_action,
                'count': int(n_outliers),
                'percentage': round(outlier_pct, 2)
            }
else:
    logger.info(f"  - Outlier detection skipped")

logger.info(f"✅ Outlier handling completed")

# ============================================================
# STEP 10: SAVE CLEANED DATA
# ============================================================
logger.info(f"💾 [STEP 10/10] Saving cleaned data...")

# Create output directory
output_dir = Path('/opt/airflow/data/processed')
output_dir.mkdir(parents=True, exist_ok=True)

timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
output_filename = f'cleaned_{timestamp}.parquet'
output_path = output_dir / output_filename

try:
    df.to_parquet(
        path=str(output_path),
        engine='pyarrow',
        compression='snappy',
        index=False
    )
    
    output_size_mb = output_path.stat().st_size / (1024 * 1024)
    logger.info(f"✅ Cleaned data saved: {output_size_mb:.2f} MB")
    logger.info(f"   Path: {output_path}")
    
except Exception as e:
    error_msg = f"❌ Error saving cleaned data: {str(e)}"
    logger.error(error_msg)
    raise

# ============================================================
# FINAL SUMMARY
# ============================================================
end_time = time.time()
duration = end_time - start_time

rows_after = len(df)
columns_after = len(df.columns)

logger.info(f"")
logger.info(f"{'='*60}")
logger.info(f"CLEANING SUMMARY")
logger.info(f"{'='*60}")
logger.info(f"Rows:    {rows_before:,} → {rows_after:,} ({rows_before - rows_after:,} removed)")
logger.info(f"Columns: {columns_before} → {columns_after} ({columns_before - columns_after} removed)")
logger.info(f"Duration: {duration:.2f}s")
logger.info(f"{'='*60}")

# Create metadata
metadata = {
    'status': 'success',
    'output_path': str(output_path),
    
    # Before/After comparison
    'rows_before': rows_before,
    'rows_after': rows_after,
    'rows_removed': rows_before - rows_after,
    'columns_before': columns_before,
    'columns_after': columns_after,
    'columns_removed': columns_before - columns_after,
    
    # Detailed cleaning summary
    'cleaning_summary': cleaning_summary,
    
    # Processing metadata
    'duration_seconds': round(duration, 2),
    'timestamp': datetime.now().isoformat(),
    
    # Final columns list
    'final_columns': df.columns.tolist(),
}

logger.info(f"✅ Data cleaning completed successfully!")

return metadata

---

### 🎓 Key Learnings from Task 3

**1. Missing Values Strategy**:
- **Drop columns**: > threshold % missing
- **Numeric columns**: fill with median (robust to outliers)
- **Categorical columns**: fill with mode or 'Unknown'
- **Important columns**: flag instead of fill

**2. Duplicate Handling**:
- **keep_first/keep_last**: temporal logic
- **keep_all + flag**: safest approach
- **remove_all**: only after investigation

**3. Data Type Conversion**:
- **Dates**: use pd.to_datetime() with explicit format
- **Numerics**: pd.to_numeric() with errors='coerce'
- **Categories**: convert low-cardinality columns (memory savings)

**4. Outlier Detection**:
- **IQR method**: robust, good for skewed data
- **Z-score**: assumes normality
- **Action**: flag > cap > transform > remove

**5. Best Practices**:
- **Idempotency**: cleaning should be reproducible
- **Transparency**: log all transformations
- **Reversibility**: flag instead of remove when possible
- **Documentation**: track WHY not just WHAT

---

Task 4: Data Validation
🎯 จุดประสงค์หลัก
=============================================================================
TASK 4: DATA VALIDATION - ตรวจสอบคุณภาพและความถูกต้องของข้อมูลหลัง Cleaning
=============================================================================
วัตถุประสงค์:

โหลดข้อมูลที่ cleaned แล้วจาก Task 3
Validate schema (columns, data types)
Check data ranges (min/max values)
Validate business rules (cross-column logic)
Check referential integrity (foreign keys)
Statistical validation (distributions, anomalies)
Completeness checks (required fields)
Generate validation report พร้อม severity levels
Decide pass/fail criteria
Return validation results

📊 Flow การทำงาน:

Load cleaned data
Schema validation (columns, types)
Range validation (min/max, allowed values)
Business rules validation (domain logic)
Statistical validation (distributions)
Completeness validation (required fields)
Generate detailed report
Aggregate by severity (CRITICAL, WARNING, INFO)
Decide overall pass/fail
Return validation metadata

⚠️ สิ่งที่ต้องระวังในงานจริง:

Validation vs Cleaning: Validation ตรวจสอบ, ไม่แก้ไข
Fail Fast: CRITICAL errors ควร fail pipeline ทันที
Alert Fatigue: มี warnings มากเกินไป → ถูกละเลย
False Positives: validation rules เข้มงวดเกินไป → reject good data
Performance: validation ต้องเร็ว (ไม่ควรช้ากว่า cleaning)


📝 Complete Implementation
python@task(
    # task_id: ชื่อ task
    task_id='validate_data',
    
    # retries: validation ควร retry น้อย
    # - Validation เป็น deterministic operation
    # - ถ้า fail → data มีปัญหาจริงๆ (ไม่ใช่ transient error)
    # - Retry เพื่อ handle I/O errors เท่านั้น
    retries=1,
    
    # retry_delay: delay สั้น
    retry_delay=timedelta(minutes=1),
    
    # execution_timeout: validation ควรเร็ว
    # - เร็วกว่า cleaning (ไม่มี transformation)
    # - Estimation:
    #   * Small (< 100K rows): 1-2 minutes
    #   * Medium (100K-1M rows): 2-5 minutes
    #   * Large (> 1M rows): 5-10 minutes
    execution_timeout=timedelta(minutes=15),
    
    # pool: validation เป็น CPU-bound
    pool='cpu_intensive',
    
    pool_slots=1,
    
    # trigger_rule: ควร trigger อย่างไร
    # - Default: 'all_success' (all upstream tasks must succeed)
    # - Options:
    #   * 'all_success': รันถ้า upstream tasks ทั้งหมด success
    #   * 'all_failed': รันถ้า upstream tasks ทั้งหมด failed
    #   * 'all_done': รันไม่ว่า upstream จะ success/failed
    #   * 'one_success': รันถ้ามี upstream อย่างน้อย 1 success
    #   * 'one_failed': รันถ้ามี upstream อย่างน้อย 1 failed
    #   * 'none_failed': รันถ้าไม่มี upstream failed (allow skipped)
    # - Best practice: 'all_success' (default)
    # - Use case: validation ควรรันเฉพาะเมื่อ cleaning สำเร็จ
    trigger_rule='all_success',
)
def validate_data(
    cleaning_metadata: dict,
    # cleaning_metadata: metadata จาก Task 3
    # - Required: output_path to cleaned Parquet
    
    expected_columns: list = None,
    # expected_columns: list ของ column names ที่คาดหวัง
    # - Type: list of strings or None
    # - None: ไม่ validate schema (flexible)
    # - List: ต้องมี columns ตามที่ระบุ (strict)
    # - Best practice: 
    #   * Development: None (flexible)
    #   * Production: specify explicitly (prevent schema drift)
    # - Example: ['customer_id', 'order_date', 'amount', 'country']
    
    required_columns: list = None,
    # required_columns: columns ที่ห้าม missing ทั้งหมด
    # - Type: list of strings or None
    # - Subset of expected_columns
    # - Business-critical columns
    # - Example: ['customer_id', 'order_date', 'amount']
    # - Validation: every row must have non-null value
    
    column_types: dict = None,
    # column_types: expected data types
    # - Type: dict or None
    # - Format: {'column_name': 'expected_type'}
    # - Type strings:
    #   * 'int64', 'float64', 'object', 'category'
    #   * 'datetime64[ns]', 'bool'
    # - Example: {'customer_id': 'object', 'amount': 'float64'}
    # - Best practice: specify critical columns only
    
    numeric_ranges: dict = None,
    # numeric_ranges: min/max values สำหรับ numeric columns
    # - Type: dict or None
    # - Format: {'column_name': {'min': value, 'max': value}}
    # - Example: {
    #     'age': {'min': 18, 'max': 120},
    #     'amount': {'min': 0, 'max': 1000000}
    #   }
    # - Validation: all values must be within range
    # - Use case: prevent unrealistic values
    
    date_ranges: dict = None,
    # date_ranges: min/max dates
    # - Type: dict or None
    # - Format: {'column_name': {'min': 'YYYY-MM-DD', 'max': 'YYYY-MM-DD'}}
    # - Example: {
    #     'order_date': {'min': '2020-01-01', 'max': '2025-12-31'}
    #   }
    # - Validation: dates must be within range
    
    categorical_values: dict = None,
    # categorical_values: allowed values สำหรับ categorical columns
    # - Type: dict or None
    # - Format: {'column_name': ['value1', 'value2', ...]}
    # - Example: {
    #     'country': ['US', 'UK', 'FR', 'DE', 'JP'],
    #     'status': ['pending', 'completed', 'cancelled']
    #   }
    # - Validation: values must be in allowed list
    
    business_rules: list = None,
    # business_rules: custom validation rules
    # - Type: list of dicts or None
    # - Format: [
    #     {
    #       'name': 'rule_name',
    #       'description': 'what this rule checks',
    #       'expression': 'pandas boolean expression',
    #       'severity': 'CRITICAL' | 'WARNING' | 'INFO'
    #     }
    #   ]
    # - Example: [
    #     {
    #       'name': 'positive_quantity',
    #       'description': 'Quantity must be positive',
    #       'expression': 'df["Quantity"] > 0',
    #       'severity': 'CRITICAL'
    #     }
    #   ]
    # - Best practice: define in config file, not hardcode
    
    max_missing_pct: float = 5.0,
    # max_missing_pct: maximum % missing values allowed per column
    # - Type: float (0-100)
    # - Default: 5.0 (after cleaning ควรมี missing น้อย)
    # - Severity: WARNING if exceeded
    
    fail_on_critical: bool = True,
    # fail_on_critical: fail task ถ้ามี CRITICAL errors
    # - Type: bool
    # - True: raise exception on critical errors
    # - False: log errors but continue
    # - Best practice: True (prevent bad data from loading)
    
) -> dict:
    # Return type: validation results
    # Structure:
    # {
    #     'status': 'PASS' | 'PASS_WITH_WARNINGS' | 'FAILED',
    #     'overall_result': bool,
    #     'validation_report': {
    #         'schema': [...],
    #         'ranges': [...],
    #         'business_rules': [...],
    #         'statistical': [...],
    #         'completeness': [...]
    #     },
    #     'summary': {
    #         'total_checks': int,
    #         'passed': int,
    #         'warnings': int,
    #         'critical_errors': int
    #     },
    #     'failed_checks': [...],
    #     'timestamp': str
    # }
    
    """
    ตรวจสอบคุณภาพและความถูกต้องของข้อมูลหลัง cleaning
    
    Args:
        cleaning_metadata: metadata จาก cleaning task
        expected_columns: columns ที่คาดหวัง
        required_columns: columns ที่ห้ามมี missing
        column_types: data types ที่คาดหวัง
        numeric_ranges: min/max สำหรับ numeric columns
        date_ranges: min/max สำหรับ date columns
        categorical_values: allowed values สำหรับ categorical
        business_rules: custom validation rules
        max_missing_pct: maximum % missing allowed
        fail_on_critical: fail task on critical errors
    
    Returns:
        Dict containing:
        - status: PASS/PASS_WITH_WARNINGS/FAILED
        - validation_report: detailed results by category
        - summary: counts of passed/failed checks
        - failed_checks: list of failures
        
    Raises:
        FileNotFoundError: ถ้าไฟล์ cleaned data ไม่พบ
        ValidationError: ถ้ามี critical errors และ fail_on_critical=True
    """
    
    import pandas as pd
    import numpy as np
    import logging
    from pathlib import Path
    from datetime import datetime
    import time
    
    # ============================================================
    # STEP 1: SETUP
    # ============================================================
    logger = logging.getLogger(__name__)
    logger.info("🚀 [STEP 1/10] Starting data validation task")
    
    start_time = time.time()
    
    # Validate metadata
    if not cleaning_metadata or 'output_path' not in cleaning_metadata:
        error_msg = "❌ Invalid cleaning metadata"
        logger.error(error_msg)
        raise ValueError(error_msg)
    
    # Initialize validation report structure
    validation_report = {
        'schema': [],
        'ranges': [],
        'business_rules': [],
        'statistical': [],
        'completeness': []
    }
    
    # Counters
    total_checks = 0
    passed_checks = 0
    warnings = 0
    critical_errors = 0
    
    logger.info(f"✅ Setup completed")
    
    # ============================================================
    # STEP 2: LOAD CLEANED DATA
    # ============================================================
    logger.info(f"📖 [STEP 2/10] Loading cleaned data...")
    
    parquet_path = Path(cleaning_metadata['output_path'])
    
    if not parquet_path.exists():
        error_msg = f"❌ Cleaned data not found: {parquet_path}"
        logger.error(error_msg)
        raise FileNotFoundError(error_msg)
    
    try:
        df = pd.read_parquet(parquet_path, engine='pyarrow')
        logger.info(f"✅ Data loaded: {len(df):,} rows × {len(df.columns)} columns")
        
    except Exception as e:
        error_msg = f"❌ Error loading data: {str(e)}"
        logger.error(error_msg)
        raise
    
    # ============================================================
    # STEP 3: SCHEMA VALIDATION
    # ============================================================
    logger.info(f"📋 [STEP 3/10] Validating schema...")
    
    # Validation 3.1: Expected columns present
    # -----------------------------------------
    if expected_columns:
        total_checks += 1
        
        actual_columns = set(df.columns)
        expected_set = set(expected_columns)
        
        missing_columns = expected_set - actual_columns
        # Set difference: elements in expected but not in actual
        
        extra_columns = actual_columns - expected_set
        # Columns in data but not expected
        
        if missing_columns:
            critical_errors += 1
            validation_report['schema'].append({
                'check': 'expected_columns_present',
                'status': 'FAILED',
                'severity': 'CRITICAL',
                'message': f'Missing required columns: {list(missing_columns)}',
                'details': {
                    'missing': list(missing_columns),
                    'expected': expected_columns,
                    'actual': list(actual_columns)
                }
            })
            logger.error(f"  ❌ Missing columns: {missing_columns}")
        elif extra_columns:
            warnings += 1
            validation_report['schema'].append({
                'check': 'expected_columns_present',
                'status': 'WARNING',
                'severity': 'WARNING',
                'message': f'Extra columns found: {list(extra_columns)}',
                'details': {
                    'extra': list(extra_columns)
                }
            })
            logger.warning(f"  ⚠️  Extra columns: {extra_columns}")
        else:
            passed_checks += 1
            validation_report['schema'].append({
                'check': 'expected_columns_present',
                'status': 'PASSED',
                'severity': 'INFO',
                'message': 'All expected columns present'
            })
            logger.info(f"  ✅ All expected columns present")
    
    # Validation 3.2: Column data types
    # ---------------------------------
    if column_types:
        for col, expected_type in column_types.items():
            total_checks += 1
            
            if col not in df.columns:
                critical_errors += 1
                validation_report['schema'].append({
                    'check': f'column_type_{col}',
                    'status': 'FAILED',
                    'severity': 'CRITICAL',
                    'message': f'Column {col} not found in data'
                })
                continue
            
            actual_type = str(df[col].dtype)
            
            # Type matching with flexibility
            # Allow: int64 == int, float64 == float, etc.
            type_match = (
                actual_type == expected_type or
                expected_type in actual_type or
                (expected_type == 'int' and 'int' in actual_type) or
                (expected_type == 'float' and 'float' in actual_type) or
                (expected_type == 'datetime' and 'datetime' in actual_type)
            )
            
            if not type_match:
                critical_errors += 1
                validation_report['schema'].append({
                    'check': f'column_type_{col}',
                    'status': 'FAILED',
                    'severity': 'CRITICAL',
                    'message': f'Type mismatch for {col}: expected {expected_type}, got {actual_type}',
                    'details': {
                        'column': col,
                        'expected': expected_type,
                        'actual': actual_type
                    }
                })
                logger.error(f"  ❌ {col}: type mismatch (expected {expected_type}, got {actual_type})")
            else:
                passed_checks += 1
                validation_report['schema'].append({
                    'check': f'column_type_{col}',
                    'status': 'PASSED',
                    'severity': 'INFO',
                    'message': f'{col}: type correct ({actual_type})'
                })
                logger.info(f"  ✅ {col}: type correct")
    
    logger.info(f"✅ Schema validation completed")
    
    # ============================================================
    # STEP 4: RANGE VALIDATION
    # ============================================================
    logger.info(f"📏 [STEP 4/10] Validating numeric ranges...")
    
    if numeric_ranges:
        for col, ranges in numeric_ranges.items():
            if col not in df.columns:
                logger.warning(f"  ⚠️  Column {col} not found, skipping range check")
                continue
            
            min_val = ranges.get('min')
            max_val = ranges.get('max')
            
            # Check minimum
            if min_val is not None:
                total_checks += 1
                
                violations = df[col] < min_val
                # Boolean Series: True where value < min
                
                n_violations = violations.sum()
                
                if n_violations > 0:
                    violation_pct = (n_violations / len(df) * 100)
                    
                    # Sample violating values
                    sample_violations = df.loc[violations, col].head(5).tolist()
                    # df.loc[boolean_series, column]: filter rows
                    
                    warnings += 1
                    validation_report['ranges'].append({
                        'check': f'{col}_min_value',
                        'status': 'WARNING',
                        'severity': 'WARNING',
                        'message': f'{col}: {n_violations:,} values below minimum {min_val}',
                        'details': {
                            'column': col,
                            'threshold': min_val,
                            'violations': int(n_violations),
                            'percentage': round(violation_pct, 2),
                            'sample_values': sample_violations
                        }
                    })
                    logger.warning(f"  ⚠️  {col}: {n_violations:,} values < {min_val} ({violation_pct:.2f}%)")
                else:
                    passed_checks += 1
                    validation_report['ranges'].append({
                        'check': f'{col}_min_value',
                        'status': 'PASSED',
                        'severity': 'INFO',
                        'message': f'{col}: all values >= {min_val}'
                    })
            
            # Check maximum
            if max_val is not None:
                total_checks += 1
                
                violations = df[col] > max_val
                n_violations = violations.sum()
                
                if n_violations > 0:
                    violation_pct = (n_violations / len(df) * 100)
                    sample_violations = df.loc[violations, col].head(5).tolist()
                    
                    warnings += 1
                    validation_report['ranges'].append({
                        'check': f'{col}_max_value',
                        'status': 'WARNING',
                        'severity': 'WARNING',
                        'message': f'{col}: {n_violations:,} values above maximum {max_val}',
                        'details': {
                            'column': col,
                            'threshold': max_val,
                            'violations': int(n_violations),
                            'percentage': round(violation_pct, 2),
                            'sample_values': sample_violations
                        }
                    })
                    logger.warning(f"  ⚠️  {col}: {n_violations:,} values > {max_val} ({violation_pct:.2f}%)")
                else:
                    passed_checks += 1
                    validation_report['ranges'].append({
                        'check': f'{col}_max_value',
                        'status': 'PASSED',
                        'severity': 'INFO',
                        'message': f'{col}: all values <= {max_val}'
                    })
    
    logger.info(f"✅ Range validation completed")
    
    # ============================================================
    # STEP 5: DATE RANGE VALIDATION
    # ============================================================
    logger.info(f"📅 [STEP 5/10] Validating date ranges...")
    
    if date_ranges:
        for col, ranges in date_ranges.items():
            if col not in df.columns:
                logger.warning(f"  ⚠️  Column {col} not found, skipping")
                continue
            
            # Convert to datetime if not already
            if not pd.api.types.is_datetime64_any_dtype(df[col]):
                logger.warning(f"  ⚠️  {col} is not datetime type, skipping")
                continue
            
            min_date = pd.to_datetime(ranges.get('min')) if ranges.get('min') else None
            max_date = pd.to_datetime(ranges.get('max')) if ranges.get('max') else None
            
            if min_date is not None:
                total_checks += 1
                violations = df[col] < min_date
                n_violations = violations.sum()
                
                if n_violations > 0:
                    warnings += 1
                    validation_report['ranges'].append({
                        'check': f'{col}_min_date',
                        'status': 'WARNING',
                        'severity': 'WARNING',
                        'message': f'{col}: {n_violations:,} dates before {min_date.date()}',
                        'details': {
                            'column': col,
                            'threshold': str(min_date.date()),
                            'violations': int(n_violations)
                        }
                    })
                    logger.warning(f"  ⚠️  {col}: {n_violations:,} dates < {min_date.date()}")
                else:
                    passed_checks += 1
            
            if max_date is not None:
                total_checks += 1
                violations = df[col] > max_date
                n_violations = violations.sum()
                
                if n_violations > 0:
                    warnings += 1
                    validation_report['ranges'].append({
                        'check': f'{col}_max_date',
                        'status': 'WARNING',
                        'severity': 'WARNING',
                        'message': f'{col}: {n_violations:,} dates after {max_date.date()}',
                        'details': {
                            'column': col,
                            'threshold': str(max_date.date()),
                            'violations': int(n_violations)
                        }
                    })
                    logger.warning(f"  ⚠️  {col}: {n_violations:,} dates > {max_date.date()}")
                else:
                    passed_checks += 1
    
    logger.info(f"✅ Date range validation completed")
    
    # ============================================================
    # STEP 6: CATEGORICAL VALUES VALIDATION
    # ============================================================
    logger.info(f"🏷️  [STEP 6/10] Validating categorical values...")
    
    if categorical_values:
        for col, allowed_values in categorical_values.items():
            total_checks += 1
            
            if col not in df.columns:
                logger.warning(f"  ⚠️  Column {col} not found, skipping")
                continue
            
            # Get unique values in column
            actual_values = set(df[col].dropna().unique())
            # .dropna(): exclude NaN values
            # .unique(): get unique values
            # set(): convert to set for comparison
            
            allowed_set = set(allowed_values)
            
            # Find invalid values (not in allowed list)
            invalid_values = actual_values - allowed_set
            
            if invalid_values:
                # Count rows with invalid values
                n_violations = df[col].isin(invalid_values).sum()
                # .isin(list): check if values are in list
                # Returns: boolean Series
                
                violation_pct = (n_violations / len(df) * 100)
                
                warnings += 1
                validation_report['ranges'].append({
                    'check': f'{col}_allowed_values',
                    'status': 'WARNING',
                    'severity': 'WARNING',
                    'message': f'{col}: found {len(invalid_values)} invalid values',
                    'details': {
                        'column': col,
                        'invalid_values': list(invalid_values),
                        'allowed_values': allowed_values,
                        'violations': int(n_violations),
                        'percentage': round(violation_pct, 2)
                    }
                })
                logger.warning(f"  ⚠️  {col}: invalid values found: {invalid_values}")
            else:
                passed_checks += 1
                validation_report['ranges'].append({
                    'check': f'{col}_allowed_values',
                    'status': 'PASSED',
                    'severity': 'INFO',
                    'message': f'{col}: all values are valid'
                })
                logger.info(f"  ✅ {col}: all values valid")
    
    logger.info(f"✅ Categorical validation completed")
    
    # ============================================================
    # STEP 7: BUSINESS RULES VALIDATION
    # ============================================================
    logger.info(f"📐 [STEP 7/10] Validating business rules...")
    
    if business_rules:
        for rule in business_rules:
            total_checks += 1
            
            rule_name = rule.get('name', 'unnamed_rule')
            description = rule.get('description', 'No description')
            expression = rule.get('expression')
            severity = rule.get('severity', 'WARNING')
            
            if not expression:
                logger.warning(f"  ⚠️  Rule {rule_name}: no expression provided")
                continue
            
            try:
                # Evaluate expression
                # expression is string like: 'df["Quantity"] > 0'
                result = eval(expression)
                # eval(): evaluate Python expression
                # - Security risk: NEVER use with user input
                # - Safe here: expressions from config file (controlled)
                # - Alternative: use query() method (safer but less flexible)
                
                # result should be boolean Series
                if isinstance(result, pd.Series):
                    # Count violations (where condition is False)
                    violations = ~result
                    # ~ : logical NOT (invert boolean)
                    
                    n_violations = violations.sum()
                    violation_pct = (n_violations / len(df) * 100) if len(df) > 0 else 0
                    
                    if n_violations > 0:
                        # Get sample of violating rows
                        sample_rows = df[violations].head(3).to_dict('records')
                        # .to_dict('records'): convert to list of dicts
                        # Format: [{'col1': val1, 'col2': val2}, ...]
                        
                        if severity == 'CRITICAL':
                            critical_errors += 1
                        else:
                            warnings += 1
                        
                        validation_report['business_rules'].append({
                            'check': rule_name,
                            'status': 'FAILED',
                            'severity': severity,
                            'message': f'{description}: {n_violations:,} violations',
                            'details': {
                                'rule': expression,
                                'violations': int(n_violations),
                                'percentage': round(violation_pct, 2),
                                'sample_violations': sample_rows
                            }
                        })
                        
                        log_func = logger.error if severity == 'CRITICAL' else logger.warning
                        log_symbol = '❌' if severity == 'CRITICAL' else '⚠️'
                        log_func(f"  {log_symbol} {rule_name}: {n_violations:,} violations ({violation_pct:.2f}%)")
                    else:
                        passed_checks += 1
                        validation_report['business_rules'].append({
                            'check': rule_name,
                            'status': 'PASSED',
                            'severity': 'INFO',
                            'message': f'{description}: passed'
                        })
                        logger.info(f"  ✅ {rule_name}: passed")
                else:
                    logger.warning(f"  ⚠️  {rule_name}: expression did not return boolean Series")
                    
            except Exception as e:
                logger.error(f"  ❌ {rule_name}: error evaluating rule: {str(e)}")
                critical_errors += 1
                validation_report['business_rules'].append({
                    'check': rule_name,
                    'status': 'ERROR',
                    'severity': 'CRITICAL',
                    'message': f'Error evaluating rule: {str(e)}',
                    'details': {
                        'expression': expression,
                        'error': str(e)
                    }
                })
    else:
        logger.info(f"  - No business rules specified")
    
    logger.info(f"✅ Business rules validation completed")
    
    # ============================================================
    # STEP 8: STATISTICAL VALIDATION
    # ============================================================
    logger.info(f"📊 [STEP 8/10] Statistical validation...")
    
    # Validation 8.1: Check for constant columns (should not exist after cleaning)
    # ---------------------------------------------------------------------------
    total_checks += 1
    
    constant_cols = []
    for col in df.columns:
        if df[col].nunique() == 1:
            # Column has only 1 unique value
            constant_cols.append(col)
    
    if constant_cols:
        warnings += 1
        validation_report['statistical'].append({
            'check': 'constant_columns',
            'status': 'WARNING',
            'severity': 'WARNING',
            'message': f'Found {len(constant_cols)} constant columns',
            'details': {
                'columns': constant_cols
            }
        })
        logger.warning(f"  ⚠️  Constant columns found: {constant_cols}")
    else:
        passed_checks += 1
        validation_report['statistical'].append({
            'check': 'constant_columns',
            'status': 'PASSED',
            'severity': 'INFO',
            'message': 'No constant columns'
        })
        logger.info(f"  ✅ No constant columns")
    
    # Validation 8.2: Check for high cardinality (potential data quality issue)
    # --------------------------------------------------------------------------
    total_checks += 1
    
    high_cardinality_threshold = 0.95  # > 95% unique = high cardinality
    high_cardinality_cols = []
    
    for col in df.select_dtypes(include=['object', 'category']).columns:
        unique_pct = (df[col].nunique() / len(df) * 100) if len(df) > 0 else 0
        
        if unique_pct > high_cardinality_threshold * 100:
            high_cardinality_cols.append({
                'column': col,
                'unique_pct': round(unique_pct, 2),
                'unique_count': df[col].nunique()
        })

if high_cardinality_cols:
    warnings += 1
    validation_report['statistical'].append({
        'check': 'high_cardinality',
        'status': 'WARNING',
        'severity': 'WARNING',
        'message': f'{len(high_cardinality_cols)} columns with very high cardinality',
        'details': {
            'columns': high_cardinality_cols
        }
    })
    logger.warning(f"  ⚠️  High cardinality columns: {[c['column'] for c in high_cardinality_cols]}")
else:
    passed_checks += 1
    validation_report['statistical'].append({
        'check': 'high_cardinality',
        'status': 'PASSED',
        'severity': 'INFO',
        'message': 'No high cardinality issues'
    })

logger.info(f"✅ Statistical validation completed")

# ============================================================
# STEP 9: COMPLETENESS VALIDATION
# ============================================================
logger.info(f"📝 [STEP 9/10] Validating completeness...")

# Validation 9.1: Required columns must have no missing values
# ------------------------------------------------------------
if required_columns:
    for col in required_columns:
        total_checks += 1
        
        if col not in df.columns:
            critical_errors += 1
            validation_report['completeness'].append({
                'check': f'{col}_required',
                'status': 'FAILED',
                'severity': 'CRITICAL',
                'message': f'Required column {col} not found'
            })
            logger.error(f"  ❌ Required column {col} not found")
            continue
        
        missing_count = df[col].isnull().sum()
        
        if missing_count > 0:
            critical_errors += 1
            validation_report['completeness'].append({
                'check': f'{col}_required',
                'status': 'FAILED',
                'severity': 'CRITICAL',
                'message': f'Required column {col} has {missing_count:,} missing values',
                'details': {
                    'column': col,
                    'missing': int(missing_count),
                    'percentage': round((missing_count / len(df) * 100), 2)
                }
            })
            logger.error(f"  ❌ {col}: {missing_count:,} missing values in required column")
        else:
            passed_checks += 1
            validation_report['completeness'].append({
                'check': f'{col}_required',
                'status': 'PASSED',
                'severity': 'INFO',
                'message': f'{col}: no missing values'
            })
            logger.info(f"  ✅ {col}: complete")

# Validation 9.2: Check missing percentage per column
# ---------------------------------------------------
total_checks += 1

high_missing_cols = []
for col in df.columns:
    missing_pct = (df[col].isnull().sum() / len(df) * 100) if len(df) > 0 else 0
    
    if missing_pct > max_missing_pct:
        high_missing_cols.append({
            'column': col,
            'missing_pct': round(missing_pct, 2),
            'missing_count': df[col].isnull().sum()
        })

if high_missing_cols:
    warnings += 1
    validation_report['completeness'].append({
        'check': 'missing_threshold',
        'status': 'WARNING',
        'severity': 'WARNING',
        'message': f'{len(high_missing_cols)} columns exceed {max_missing_pct}% missing threshold',
        'details': {
            'threshold': max_missing_pct,
            'columns': high_missing_cols
        }
    })
    logger.warning(f"  ⚠️  {len(high_missing_cols)} columns exceed missing threshold")
else:
    passed_checks += 1
    validation_report['completeness'].append({
        'check': 'missing_threshold',
        'status': 'PASSED',
        'severity': 'INFO',
        'message': f'All columns below {max_missing_pct}% missing threshold'
    })
    logger.info(f"  ✅ All columns below missing threshold")

logger.info(f"✅ Completeness validation completed")

# ============================================================
# STEP 10: GENERATE SUMMARY AND FINAL STATUS
# ============================================================
logger.info(f"📋 [STEP 10/10] Generating validation summary...")

end_time = time.time()
duration = end_time - start_time

# Determine overall status
if critical_errors > 0:
    overall_status = 'FAILED'
    overall_result = False
elif warnings > 0:
    overall_status = 'PASS_WITH_WARNINGS'
    overall_result = True
else:
    overall_status = 'PASS'
    overall_result = True

# Collect all failed/warning checks
failed_checks = []
for category, checks in validation_report.items():
    for check in checks:
        if check['status'] in ['FAILED', 'WARNING']:
            failed_checks.append({
                'category': category,
                **check
            })

# Create summary
summary = {
    'total_checks': total_checks,
    'passed': passed_checks,
    'warnings': warnings,
    'critical_errors': critical_errors,
    'pass_rate': round((passed_checks / total_checks * 100), 2) if total_checks > 0 else 0
}

# Log summary
logger.info(f"")
logger.info(f"{'='*60}")
logger.info(f"VALIDATION SUMMARY")
logger.info(f"{'='*60}")
logger.info(f"Status:          {overall_status}")
logger.info(f"Total checks:    {total_checks}")
logger.info(f"Passed:          {passed_checks} ✅")
logger.info(f"Warnings:        {warnings} ⚠️")
logger.info(f"Critical errors: {critical_errors} ❌")
logger.info(f"Pass rate:       {summary['pass_rate']:.2f}%")
logger.info(f"Duration:        {duration:.2f}s")
logger.info(f"{'='*60}")

# Save validation report to file
reports_dir = Path('/opt/airflow/data/reports')
reports_dir.mkdir(parents=True, exist_ok=True)

timestamp_str = datetime.now().strftime('%Y%m%d_%H%M%S')
report_filename = f'validation_report_{timestamp_str}.json'
report_path = reports_dir / report_filename

import json
try:
    with open(report_path, 'w', encoding='utf-8') as f:
        json.dump({
            'status': overall_status,
            'summary': summary,
            'validation_report': validation_report,
            'failed_checks': failed_checks,
            'metadata': {
                'total_rows': len(df),
                'total_columns': len(df.columns),
                'duration_seconds': round(duration, 2),
                'timestamp': datetime.now().isoformat()
            }
        }, f, indent=2, ensure_ascii=False)
    
    logger.info(f"✅ Validation report saved: {report_path}")
except Exception as e:
    logger.warning(f"⚠️  Could not save validation report: {str(e)}")

# Create metadata for XCom
metadata = {
    'status': overall_status,
    'overall_result': overall_result,
    'report_path': str(report_path),
    'summary': summary,
    'validation_report': validation_report,
    'failed_checks': failed_checks[:10],  # Limit to first 10 for XCom size
    'duration_seconds': round(duration, 2),
    'timestamp': datetime.now().isoformat()
}

# Fail task if critical errors and fail_on_critical=True
if critical_errors > 0 and fail_on_critical:
    error_msg = f"❌ Validation failed with {critical_errors} critical error(s)"
    logger.error(error_msg)
    logger.error(f"   See report for details: {report_path}")
    raise ValueError(error_msg)

logger.info(f"✅ Data validation completed: {overall_status}")

return metadata

---

### 🎓 Key Learnings from Task 4

**1. Validation vs Cleaning**:
- **Cleaning**: แก้ไขข้อมูล
- **Validation**: ตรวจสอบเท่านั้น ไม่แก้ไข
- Validation หลัง cleaning เพื่อ verify quality

**2. Severity Levels**:
- **CRITICAL**: Pipeline ต้อง fail (bad data ห้ามผ่าน)
- **WARNING**: Log แต่ continue (investigate later)
- **INFO**: Informational only

**3. Validation Categories**:
- **Schema**: columns, types (CRITICAL)
- **Ranges**: min/max values (WARNING)
- **Business Rules**: domain logic (varies)
- **Statistical**: distributions, anomalies (WARNING)
- **Completeness**: missing values (varies)

**4. Best Practices**:
- **Fail Fast**: Stop on critical errors
- **Detailed Reporting**: Include samples, percentages
- **Actionable Messages**: Tell what's wrong AND how to fix
- **Version Control**: Save validation reports

**5. Common Pitfalls**:
- **Too Strict**: Reject good data (false positives)
- **Too Lenient**: Accept bad data (false negatives)
- **Alert Fatigue**: Too many warnings → ignored
- **Poor Messages**: "Validation failed" without details

---

### ✅ Testing Task 4
```python
# tests/test_validation.py
import pytest
from dags.tasks.validation import validate_data

def test_validation_pass():
    """Test validation with clean data"""
    cleaning_metadata = {
        'output_path': '/opt/airflow/tests/fixtures/clean_data.parquet'
    }
    
    metadata = validate_data(
        cleaning_metadata=cleaning_metadata,
        expected_columns=['col1', 'col2', 'col3'],
        fail_on_critical=False
    )
    
    assert metadata['overall_result'] == True
    assert metadata['summary']['critical_errors'] == 0

def test_validation_fail_missing_column():
    """Test validation with missing required column"""
    cleaning_metadata = {
        'output_path': '/opt/airflow/tests/fixtures/clean_data.parquet'
    }
    
    with pytest.raises(ValueError):
        validate_data(
            cleaning_metadata=cleaning_metadata,
            expected_columns=['col1', 'col2', 'missing_col'],
            fail_on_critical=True
        )

def test_validation_warnings():
    """Test validation with warnings"""
    cleaning_metadata = {
        'output_path': '/opt/airflow/tests/fixtures/data_with_outliers.parquet'
    }
    
    metadata = validate_data(
        cleaning_metadata=cleaning_metadata,
        numeric_ranges={'age': {'min': 0, 'max': 120}},
        fail_on_critical=False
    )
    
    assert metadata['status'] == 'PASS_WITH_WARNINGS'
    assert metadata['summary']['warnings'] > 0
```

---

Task 5: Data Loading
🎯 จุดประสงค์หลัก
=============================================================================
TASK 5: DATA LOADING - โหลดข้อมูลที่ผ่าน validation เข้า Database และ Data Lake
=============================================================================
วัตถุประสงค์:

โหลดข้อมูลที่ validated แล้วจาก Task 4
เชื่อมต่อ PostgreSQL database
สร้าง/update table schema
Load data เข้า PostgreSQL (Analytics Database)
สร้าง indexes ที่เหมาะสม
Save เป็น Parquet ใน Data Lake (partitioned)
Update metadata table (audit trail)
Archive source files
Verify data integrity post-load
Return loading summary

📊 Flow การทำงาน:

Load validated data
Create database connection
Check if table exists → create/update schema
Choose loading strategy (full/incremental/upsert)
Load data to PostgreSQL
Create indexes for query performance
Verify row counts and data integrity
Save to Data Lake (partitioned Parquet)
Update metadata table
Archive source files
Return loading summary

⚠️ สิ่งที่ต้องระวังในงานจริง:

Transaction Management: ใช้ transactions เพื่อ atomicity
Connection Pooling: จัดการ connections อย่างมีประสิทธิภาพ
Batch Size: chunk data เพื่อไม่ให้ memory overflow
Schema Evolution: handle column add/remove/rename
Rollback Strategy: มีแผนสำรอง ถ้า load fail
Performance: optimize batch size, parallel loading


📝 Complete Implementation
python@task(
    # task_id: ชื่อ task
    task_id='load_data',
    
    # retries: loading ควร retry
    # - Database operations อาจมี transient failures
    # - Network issues, connection timeouts
    # - Best practice: 2-3 retries
    retries=3,
    
    # retry_delay: delay ก่อน retry
    # - Database operations: รอให้ connection pool recover
    # - Best practice: 2-5 minutes
    retry_delay=timedelta(minutes=3),
    
    # execution_timeout: loading อาจใช้เวลานาน
    # - Depends on: data size, network speed, database performance
    # - Estimation:
    #   * Small (< 100K rows): 2-5 minutes
    #   * Medium (100K-1M rows): 5-15 minutes
    #   * Large (> 1M rows): 15-30 minutes
    execution_timeout=timedelta(minutes=30),
    
    # pool: loading เป็น I/O intensive
    pool='io_intensive',
    
    # pool_slots: database loading ควรใช้หลาย slots
    # - เพราะ hold connection นาน
    pool_slots=2,
    
    # retry_exponential_backoff: ใช้ exponential backoff
    # - Database recovery may take time
    retry_exponential_backoff=True,
    
    max_retry_delay=timedelta(minutes=15),
)
def load_data(
    validation_metadata: dict,
    # validation_metadata: metadata จาก Task 4
    # - Includes: validation status, data path
    
    db_connection_id: str = 'postgres_default',
    # db_connection_id: Airflow connection ID
    # - Type: str
    # - Airflow Connections store: host, port, user, password, database
    # - Configure in: Airflow UI → Admin → Connections
    # - Example: 'postgres_default', 'postgres_analytics', 'postgres_prod'
    # - Best practice: different connections for dev/staging/prod
    
    target_table: str = 'marketing_data',
    # target_table: ชื่อ table ใน PostgreSQL
    # - Type: str
    # - Naming convention: snake_case, descriptive
    # - Best practice: prefix by domain (e.g., 'marketing_', 'sales_')
    
    target_schema: str = 'public',
    # target_schema: database schema
    # - Type: str
    # - Default: 'public' (PostgreSQL default schema)
    # - Best practice: organize by domain
    #   * 'raw': raw data
    #   * 'staging': intermediate tables
    #   * 'analytics': production tables
    # - Example: 'analytics.marketing_data'
    
    load_strategy: str = 'replace',
    # load_strategy: วิธีการ load data
    # - Type: str
    # - Options:
    #   * 'replace': drop table และสร้างใหม่ (full refresh)
    #   * 'append': เพิ่ม rows ใหม่ (incremental)
    #   * 'upsert': update existing, insert new (merge)
    # - Comparison:
    #   * replace: simple, ใช้เมื่อ data size เล็ก, no history
    #   * append: fast, ใช้สำหรับ append-only data (logs, events)
    #   * upsert: complex, ใช้เมื่อต้อง update existing records
    # - Best practice:
    #   * Development: 'replace' (simple)
    #   * Production: 'upsert' or 'append' (preserve history)
    
    upsert_keys: list = None,
    # upsert_keys: columns สำหรับ identify unique rows
    # - Type: list of column names or None
    # - Required for: load_strategy='upsert'
    # - Example: ['customer_id', 'order_date']
    # - Use case: composite primary key
    # - PostgreSQL: ON CONFLICT (keys) DO UPDATE
    
    batch_size: int = 10000,
    # batch_size: จำนวน rows ต่อ batch insert
    # - Type: int
    # - Range: 1,000 - 50,000
    # - Trade-off:
    #   * Small batch: many transactions, slower, less memory
    #   * Large batch: few transactions, faster, more memory
    # - Best practice:
    #   * < 100K total rows: 10,000 (default)
    #   * 100K - 1M rows: 20,000
    #   * > 1M rows: 50,000
    # - Network consideration: larger batches if network is bottleneck
    
    create_indexes: bool = True,
    # create_indexes: สร้าง indexes หลัง load หรือไม่
    # - Type: bool
    # - True: create indexes automatically (recommended)
    # - False: skip (manual index creation later)
    # - Trade-off:
    #   * Create indexes: slower load, faster queries
    #   * No indexes: faster load, slower queries
    # - Best practice: True (query performance is critical)
    
    index_columns: list = None,
    # index_columns: columns ที่จะสร้าง index
    # - Type: list of column names or None
    # - None: auto-detect (date columns, foreign keys)
    # - Example: ['customer_id', 'order_date', 'country']
    # - Guidelines:
    #   * Columns ใน WHERE clause บ่อย
    #   * Foreign keys
    #   * Date/timestamp columns
    #   * High selectivity columns
    # - Cost: indexes ใช้ disk space, slow down inserts
    
    partition_by: list = None,
    # partition_by: columns สำหรับ partition Parquet files
    # - Type: list of column names or None
    # - None: no partitioning (single file or auto-split)
    # - Example: ['year', 'month'] or ['country', 'year']
    # - Partitioning benefits:
    #   * Query efficiency (read เฉพาะ partitions ที่ต้องการ)
    #   * Parallel processing
    #   * Data management (easy to delete old partitions)
    # - Best practice:
    #   * Partition by time (year/month/day)
    #   * Low cardinality (< 1000 partitions)
    #   * Columns ที่ filter บ่อย
    
    archive_source: bool = True,
    # archive_source: archive raw files หลัง load สำเร็จ
    # - Type: bool
    # - True: move to archive directory (recommended)
    # - False: leave in place
    # - Best practice: True (clean up processed files)
    
) -> dict:
    # Return type: loading summary
    # Structure:
    # {
    #     'status': 'success' | 'failed',
    #     'table_name': str,
    #     'rows_loaded': int,
    #     'load_strategy': str,
    #     'database_size_mb': float,
    #     'parquet_path': str,
    #     'parquet_size_mb': float,
    #     'indexes_created': list,
    #     'duration_seconds': float,
    #     'timestamp': str
    # }
    
    """
    Load validated data เข้า PostgreSQL และ Data Lake
    
    Args:
        validation_metadata: metadata จาก validation task
        db_connection_id: Airflow connection ID
        target_table: table name ใน PostgreSQL
        target_schema: schema name
        load_strategy: 'replace', 'append', 'upsert'
        upsert_keys: columns สำหรับ upsert (required if strategy='upsert')
        batch_size: rows ต่อ batch insert
        create_indexes: สร้าง indexes หรือไม่
        index_columns: columns ที่จะ index
        partition_by: columns สำหรับ partition Parquet
        archive_source: archive source files หรือไม่
    
    Returns:
        Dict containing:
        - status: success/failed
        - loading statistics: rows, size, duration
        - database info: table name, indexes
        - data lake info: parquet path, partitions
        
    Raises:
        ValueError: ถ้า validation failed
        ConnectionError: ถ้าเชื่อมต่อ database ไม่ได้
        DatabaseError: ถ้า load operation failed
    """
    
    import pandas as pd
    import numpy as np
    import logging
    from pathlib import Path
    from datetime import datetime
    import time
    from sqlalchemy import create_engine, inspect, Table, Column, Integer, String, Float, DateTime, Boolean, MetaData, Index, text
    from sqlalchemy.exc import SQLAlchemyError
    from sqlalchemy.dialects.postgresql import insert
    import shutil
    
    # ============================================================
    # STEP 1: SETUP AND VALIDATION
    # ============================================================
    logger = logging.getLogger(__name__)
    logger.info("🚀 [STEP 1/11] Starting data loading task")
    
    start_time = time.time()
    
    # Check validation status
    if not validation_metadata:
        error_msg = "❌ No validation metadata received"
        logger.error(error_msg)
        raise ValueError(error_msg)
    
    validation_status = validation_metadata.get('status')
    if validation_status == 'FAILED':
        error_msg = f"❌ Cannot load data: validation failed"
        logger.error(error_msg)
        logger.error(f"   Critical errors: {validation_metadata.get('summary', {}).get('critical_errors', 0)}")
        raise ValueError(error_msg)
    
    if validation_status == 'PASS_WITH_WARNINGS':
        logger.warning(f"⚠️  Loading data with {validation_metadata.get('summary', {}).get('warnings', 0)} warnings")
    
    # Validate upsert configuration
    if load_strategy == 'upsert' and not upsert_keys:
        error_msg = "❌ upsert_keys required when load_strategy='upsert'"
        logger.error(error_msg)
        raise ValueError(error_msg)
    
    logger.info(f"✅ Configuration validated")
    logger.info(f"  - Load strategy: {load_strategy}")
    logger.info(f"  - Target: {target_schema}.{target_table}")
    logger.info(f"  - Batch size: {batch_size:,}")
    
    # ============================================================
    # STEP 2: LOAD VALIDATED DATA
    # ============================================================
    logger.info(f"📖 [STEP 2/11] Loading validated data...")
    
    # Get data path from cleaning metadata (passed through validation)
    # In practice, we'd get this from validation_metadata or XCom
    # For this example, we'll assume it's in validation_metadata
    
    # Find the cleaned parquet file
    processed_dir = Path('/opt/airflow/data/processed')
    parquet_files = sorted(processed_dir.glob('cleaned_*.parquet'))
    
    if not parquet_files:
        error_msg = "❌ No cleaned data files found"
        logger.error(error_msg)
        raise FileNotFoundError(error_msg)
    
    # Use most recent file
    data_path = parquet_files[-1]
    logger.info(f"  - Loading from: {data_path}")
    
    try:
        df = pd.read_parquet(data_path, engine='pyarrow')
        logger.info(f"✅ Data loaded: {len(df):,} rows × {len(df.columns)} columns")
        
    except Exception as e:
        error_msg = f"❌ Error loading data: {str(e)}"
        logger.error(error_msg)
        raise
    
    rows_to_load = len(df)
    
    # ============================================================
    # STEP 3: CREATE DATABASE CONNECTION
    # ============================================================
    logger.info(f"🔌 [STEP 3/11] Creating database connection...")
    
    try:
        # Get connection from Airflow
        from airflow.hooks.base import BaseHook
        
        conn = BaseHook.get_connection(db_connection_id)
        # BaseHook.get_connection(): retrieve Airflow Connection
        # - Returns: Connection object
        # - Properties: host, port, schema, login, password
        # - Raises: AirflowException if connection not found
        
        # Build connection string
        # Format: postgresql://user:password@host:port/database
        connection_string = (
            f"postgresql://{conn.login}:{conn.password}"
            f"@{conn.host}:{conn.port}/{conn.schema}"
        )
        # Security note: ห้าม log connection string (contains password)
        
        # Create SQLAlchemy engine
        engine = create_engine(
            connection_string,
            # URL: database connection string
            
            pool_size=5,
            # pool_size: จำนวน connections ใน pool
            # - Default: 5
            # - Range: 2-10 สำหรับ most applications
            # - Benefits: reuse connections (faster than creating new)
            # - Trade-off: more connections = more database load
            
            max_overflow=10,
            # max_overflow: connections เพิ่มเติมถ้า pool full
            # - Default: 10
            # - Total connections = pool_size + max_overflow
            # - Example: pool_size=5, max_overflow=10 → max 15 connections
            # - ถ้าเกิน → block until connection available
            
            pool_pre_ping=True,
            # pool_pre_ping: test connection ก่อนใช้
            # - True: execute "SELECT 1" ก่อนทุกครั้ง
            # - Benefits: detect stale connections, auto-reconnect
            # - Trade-off: slight overhead (1-2ms per query)
            # - Best practice: True (reliability > performance)
            
            pool_recycle=3600,
            # pool_recycle: recycle connections after N seconds
            # - Default: -1 (no recycling)
            # - Recommended: 3600 (1 hour)
            # - Reason: prevent stale connections, database timeouts
            # - Database timeout ทั่วไป: 8 hours (PostgreSQL default)
            
            echo=False,
            # echo: log all SQL statements
            # - True: log to stdout (debugging)
            # - False: no logging (production)
            # - Best practice: False (use SQLAlchemy logger instead)
        )
        
        # Test connection
        with engine.connect() as connection:
            # engine.connect(): get connection from pool
            # - Context manager: auto-close connection
            # - Returns: Connection object
            
            result = connection.execute(text("SELECT version()"))
            # text(): create SQL text clause
            # - Required in SQLAlchemy 2.0+
            # - Returns: TextClause object
            
            version = result.fetchone()[0]
            # fetchone(): get first row
            # - Returns: Row object or None
            # [0]: get first column
            
            logger.info(f"✅ Connected to PostgreSQL")
            logger.info(f"   Version: {version[:50]}...")  # Truncate long version string
        
    except Exception as e:
        error_msg = f"❌ Database connection failed: {str(e)}"
        logger.error(error_msg)
        raise ConnectionError(error_msg) from e
    
    # ============================================================
    # STEP 4: PREPARE TABLE SCHEMA
    # ============================================================
    logger.info(f"📐 [STEP 4/11] Preparing table schema...")
    
    # Create SQLAlchemy metadata
    metadata = MetaData(schema=target_schema)
    # MetaData: container for database schema
    # - Holds: Table definitions, relationships
    # - schema: database schema name
    
    # Inspect existing database
    inspector = inspect(engine)
    # inspect(): create Inspector object
    # - Used to: query database metadata
    # - Methods: get_table_names(), get_columns(), get_indexes()
    
    table_exists = inspector.has_table(target_table, schema=target_schema)
    # has_table(): check if table exists
    # - Returns: bool
    # - Alternative: target_table in inspector.get_table_names(schema)
    
    if table_exists:
        logger.info(f"  - Table {target_schema}.{target_table} exists")
        
        if load_strategy == 'replace':
            logger.info(f"  - Will drop and recreate table")
        elif load_strategy == 'append':
            logger.info(f"  - Will append to existing table")
        elif load_strategy == 'upsert':
            logger.info(f"  - Will upsert based on keys: {upsert_keys}")
    else:
        logger.info(f"  - Table {target_schema}.{target_table} does not exist")
        logger.info(f"  - Will create new table")
    
    # Map Pandas dtypes to SQL types
    # --------------------------------------------------------
    def pandas_dtype_to_sql_type(dtype):
        """
        Convert Pandas dtype to SQLAlchemy SQL type
        
        Mapping rules:
        - int64 → Integer
        - float64 → Float
        - object (strings) → String(255) or Text
        - datetime64 → DateTime
        - bool → Boolean
        - category → String(100)
        """
        dtype_str = str(dtype)
        
        if 'int' in dtype_str:
            return Integer
        elif 'float' in dtype_str:
            return Float
        elif 'datetime' in dtype_str:
            return DateTime
        elif dtype_str == 'bool':
            return Boolean
        elif dtype_str == 'category':
            return String(100)  # Categories usually short
        else:  # object (strings)
            # Estimate string length from sample
            # Check max length in first 1000 rows
            return String(255)  # Default string length
    
    # Create table definition
    # --------------------------------------------------------
    columns = []
    
    for col_name in df.columns:
        col_type = pandas_dtype_to_sql_type(df[col_name].dtype)
        
        # Create Column object
        col = Column(
            col_name,
            # name: column name
            
            col_type,
            # type_: SQLAlchemy type
            
            nullable=True,
            # nullable: allow NULL values
            # - True: column can be NULL
            # - False: NOT NULL constraint
            # - Best practice: True (flexible), use validation instead
        )
        
        columns.append(col)
    
    # Define table
    table = Table(
        target_table,
        # name: table name
        
        metadata,
        # metadata: MetaData container
        
        *columns,
        # *columns: unpack list of Column objects
        
        schema=target_schema,
        # schema: database schema
    )
    
    logger.info(f"✅ Schema prepared with {len(columns)} columns")
    
    # ============================================================
    # STEP 5: CREATE/UPDATE TABLE
    # ============================================================
    logger.info(f"🔨 [STEP 5/11] Creating/updating table...")
    
    try:
        if load_strategy == 'replace' and table_exists:
            # Drop existing table
            logger.info(f"  - Dropping existing table...")
            
            with engine.begin() as connection:
                # engine.begin(): start transaction
                # - Context manager: auto-commit on success, auto-rollback on error
                # - Best practice: use transactions for schema changes
                
                connection.execute(text(f'DROP TABLE IF EXISTS {target_schema}.{target_table} CASCADE'))
                # CASCADE: drop dependent objects (views, foreign keys)
                # - Required: ถ้ามี dependencies
                # - Alternative: RESTRICT (fail if dependencies exist)
            
            logger.info(f"  - Table dropped")
            table_exists = False
        
        if not table_exists:
            # Create new table
            logger.info(f"  - Creating table...")
            
            metadata.create_all(engine, tables=[table])
            # create_all(): create tables in database
            # - tables: list of specific tables to create
            # - None: create all tables in metadata
            # - Idempotent: won't error if table exists
            
            logger.info(f"  ✅ Table created: {target_schema}.{target_table}")
        else:
            logger.info(f"  - Using existing table")
            
            # TODO: Handle schema evolution (column add/remove)
            # In production, compare existing schema with new schema
            # and alter table if needed
        
    except SQLAlchemyError as e:
        error_msg = f"❌ Error creating table: {str(e)}"
        logger.error(error_msg)
        raise
    
    # ============================================================
    # STEP 6: LOAD DATA TO POSTGRESQL
    # ============================================================
    logger.info(f"📊 [STEP 6/11] Loading data to PostgreSQL...")
    logger.info(f"  - Strategy: {load_strategy}")
    logger.info(f"  - Batch size: {batch_size:,}")
    
    load_start_time = time.time()
    rows_loaded = 0
    
    try:
        if load_strategy in ['replace', 'append']:
            # Use pandas to_sql for simple insert
            # -------------------------------------
            
            # Determine if_exists parameter
            if_exists_param = 'replace' if load_strategy == 'replace' else 'append'
            
            df.to_sql(
                name=target_table,
                # name: table name
                
                con=engine,
                # con: SQLAlchemy engine or connection
                
                schema=target_schema,
                # schema: database schema
                
                if_exists=if_exists_param,
                # if_exists: what to do if table exists
                # - 'fail': raise error (default)
                # - 'replace': drop table and recreate
                # - 'append': insert into existing table
                
                index=False,
                # index: write DataFrame index as column
                # - False: don't write index (recommended)
                # - True: write index (rarely needed)
                
                method='multi',
                # method: insert method
                # - None: individual INSERT statements (slowest)
                # - 'multi': multi-row INSERT (faster)
                # - callable: custom insert function
                # - 'multi' format: INSERT INTO table VALUES (row1), (row2), ...
                # - Performance: 'multi' is 5-10x faster than None
                
                chunksize=batch_size,
                # chunksize: rows per INSERT statement
                # - None: insert all at once (memory intensive)
                # - int: insert in batches (recommended)
                # - Trade-off: smaller chunks = more transactions
            )
            
            rows_loaded = len(df)
            logger.info(f"  ✅ Loaded {rows_loaded:,} rows")
            
        elif load_strategy == 'upsert':
            # Use PostgreSQL ON CONFLICT for upsert
            # --------------------------------------
            
            logger.info(f"  - Upsert keys: {upsert_keys}")
            
            # Process in batches
            num_batches = (len(df) + batch_size - 1) // batch_size
            # Ceiling division: (n + d - 1) // d
            # Alternative: math.ceil(len(df) / batch_size)
            
            logger.info(f"  - Processing {num_batches} batches...")
            
            for batch_num in range(num_batches):
                start_idx = batch_num * batch_size
                end_idx = min((batch_num + 1) * batch_size, len(df))
                
                batch_df = df.iloc[start_idx:end_idx]
                # .iloc[start:end]: slice rows by integer position
                
                # Convert to records (list of dicts)
                records = batch_df.to_dict('records')
                # .to_dict('records'): list of row dicts
                # Format: [{'col1': val1, 'col2': val2}, ...]
                
                with engine.begin() as connection:
                    # Create INSERT statement
                    stmt = insert(table).values(records)
                    # insert(): PostgreSQL INSERT statement
                    # - dialect-specific (from sqlalchemy.dialects.postgresql)
                    # - values(): list of dicts to insert
                    
                    # Add ON CONFLICT clause
                    # UPDATE all columns except keys
                    update_cols = {
                        col: stmt.excluded[col]
                        for col in batch_df.columns
                        if col not in upsert_keys
                    }
                    # stmt.excluded: reference to proposed values
                    # - Used in ON CONFLICT DO UPDATE
                    # - Format: {col: stmt.excluded.col}
                    
                    stmt = stmt.on_conflict_do_update(
                        index_elements=upsert_keys,
                        # index_elements: columns ที่ define conflict
                        # - Must have UNIQUE constraint or index
                        # - Composite key: list of columns
                        
                        set_=update_cols,
                        # set_: columns to update on conflict
                        # - dict: {col: value}
                        # - value: usually stmt.excluded.col
                    )
                    # ON CONFLICT (keys) DO UPDATE SET col1=excluded.col1, ...
                    
                    connection.execute(stmt)
                
                rows_loaded += len(batch_df)
                
                if (batch_num + 1) % 10 == 0:
                    logger.info(f"    Processed {batch_num + 1}/{num_batches} batches ({rows_loaded:,} rows)")
            
            logger.info(f"  ✅ Upserted {rows_loaded:,} rows")
        
    except SQLAlchemyError as e:
        error_msg = f"❌ Error loading data: {str(e)}"
        logger.error(error_msg)
        raise
    
    load_duration = time.time() - load_start_time
    rows_per_second = rows_loaded / load_duration if load_duration > 0 else 0
    
    logger.info(f"  - Load duration: {load_duration:.2f}s")
    logger.info(f"  - Throughput: {rows_per_second:,.0f} rows/sec")
    
    # ============================================================
    # STEP 7: VERIFY DATA INTEGRITY
    # ============================================================
    logger.info(f"🔍 [STEP 7/11] Verifying data integrity...")
    
    try:
        with engine.connect() as connection:
            # Count rows in table
            result = connection.execute(text(f'SELECT COUNT(*) FROM {target_schema}.{target_table}'))
            db_row_count = result.fetchone()[0]
            
            logger.info(f"  - Rows in database: {db_row_count:,}")
            
            if load_strategy == 'replace':
                # Row counts should match exactly
                if db_row_count != rows_to_load:
                    logger.warning(f"  ⚠️  Row count mismatch: loaded {rows_to_load:,}, found {db_row_count:,}")
                else:
                    logger.info(f"  ✅ Row count matches")
            elif load_strategy == 'append':
                # Database should have at least the loaded rows
                if db_row_count < rows_to_load:
                    logger.error(f"  ❌ Database has fewer rows than loaded!")
                else:
                    logger.info(f"  ✅ Rows appended successfully")
            else:  # upsert
                logger.info(f"  ✅ Upsert completed (total rows: {db_row_count:,})")
            
            # Get table size
            size_query = f"""
                SELECT pg_total_relation_size('{target_schema}.{target_table}')
            """
            # pg_total_relation_size(): total size including indexes, TOAST
            # - Returns: size in bytes
            # - Alternative: pg_relation_size() (table only, no indexes)
            
            result = connection.execute(text(size_query))
            table_size_bytes = result.fetchone()[0]
            table_size_mb = table_size_bytes / (1024 * 1024)
            
            logger.info(f"  - Table size: {table_size_mb:.2f} MB")
        
    except SQLAlchemyError as e:
        logger.warning(f"  ⚠️  Could not verify integrity: {str(e)}")
        table_size_mb = 0
        db_row_count = rows_loaded
    
    # ============================================================
    # STEP 8: CREATE INDEXES
    # ============================================================
    logger.info(f"🔑 [STEP 8/11] Creating indexes...")
    
    indexes_created = []
    
    if create_indexes:
        # Auto-detect index columns if not specified
        if index_columns is None:
            index_columns = []
            
            # Add datetime columns
            for col in df.select_dtypes(include=['datetime64']).columns:
                index_columns.append(col)
            
            # Add columns with '_id' in name (likely foreign keys)
            for col in df.columns:
                if '_id' in col.lower() and col not in index_columns:
                    index_columns.append(col)
            
            logger.info(f"  - Auto index columns: {index_columns}")
    if index_columns:
        try:
            with engine.begin() as connection:
                for col in index_columns:
                    if col not in df.columns:
                        logger.warning(f"    ⚠️  Column {col} not found, skipping index")
                        continue
                    
                    # Create index name
                    # Convention: idx_{table}_{column}
                    index_name = f'idx_{target_table}_{col}'
                    
                    # Check if index already exists
                    existing_indexes = inspector.get_indexes(target_table, schema=target_schema)
                    # get_indexes(): list of index definitions
                    # Returns: [{'name': 'idx_name', 'column_names': ['col'], ...}, ...]
                    
                    if any(idx['name'] == index_name for idx in existing_indexes):
                        logger.info(f"    - Index {index_name} already exists")
                        continue
                    
                    # Create index
                    create_index_sql = f'''
                        CREATE INDEX {index_name}
                        ON {target_schema}.{target_table} ({col})
                    '''
                    # CREATE INDEX syntax:
                    # CREATE INDEX name ON table (column)
                    # Options:
                    # - UNIQUE: unique index
                    # - CONCURRENTLY: don't block writes (PostgreSQL)
                    # - USING method: btree (default), hash, gist, gin
                    
                    connection.execute(text(create_index_sql))
                    
                    indexes_created.append(index_name)
                    logger.info(f"    ✅ Created index: {index_name}")
            
            logger.info(f"  ✅ Created {len(indexes_created)} indexes")
            
        except SQLAlchemyError as e:
            logger.warning(f"  ⚠️  Error creating indexes: {str(e)}")
            # Don't fail task - indexes are optimization, not critical
    else:
        logger.info(f"  - No index columns specified")
else:
    logger.info(f"  - Index creation skipped")

# ============================================================
# STEP 9: SAVE TO DATA LAKE (PARQUET)
# ============================================================
logger.info(f"💾 [STEP 9/11] Saving to Data Lake...")

# Create Data Lake directory structure
datalake_dir = Path('/opt/airflow/data/datalake')
datalake_dir.mkdir(parents=True, exist_ok=True)

# Add timestamp columns for partitioning if not present
if partition_by:
    for col in partition_by:
        if col not in df.columns:
            # Try to extract from date column
            if 'date' in df.columns or 'timestamp' in df.columns:
                date_col = 'date' if 'date' in df.columns else 'timestamp'
                
                if pd.api.types.is_datetime64_any_dtype(df[date_col]):
                    if col == 'year':
                        df['year'] = df[date_col].dt.year
                        # .dt.year: extract year from datetime
                        # - Returns: int64 Series
                        # - .dt accessor: datetime-specific methods
                        
                    elif col == 'month':
                        df['month'] = df[date_col].dt.month
                        # .dt.month: extract month (1-12)
                        
                    elif col == 'day':
                        df['day'] = df[date_col].dt.day
                        # .dt.day: extract day (1-31)
                    
                    logger.info(f"    Created partition column: {col}")

try:
    if partition_by:
        # Save with partitioning
        parquet_path = datalake_dir / target_table
        
        df.to_parquet(
            path=str(parquet_path),
            engine='pyarrow',
            compression='snappy',
            index=False,
            partition_cols=partition_by,
            # partition_cols: columns to partition by
            # - Creates directory structure: col1=val1/col2=val2/
            # - Example: year=2024/month=12/data.parquet
            
            # Advanced options:
            # partition_filename_cb: custom filename function
            # - Default: auto-generated UUID
            # - Example: lambda x: f"data_{x}.parquet"
        )
        
        logger.info(f"  ✅ Saved partitioned Parquet: {parquet_path}")
        logger.info(f"    Partitioned by: {partition_by}")
        
        # Calculate total size of partitioned files
        parquet_size_bytes = sum(f.stat().st_size for f in parquet_path.rglob('*.parquet'))
        # .rglob('*.parquet'): recursive glob (all subdirectories)
        
    else:
        # Save as single file or auto-partitioned
        timestamp_str = datetime.now().strftime('%Y%m%d_%H%M%S')
        parquet_filename = f'{target_table}_{timestamp_str}.parquet'
        parquet_path = datalake_dir / parquet_filename
        
        df.to_parquet(
            path=str(parquet_path),
            engine='pyarrow',
            compression='snappy',
            index=False
        )
        
        logger.info(f"  ✅ Saved Parquet: {parquet_path}")
        
        parquet_size_bytes = parquet_path.stat().st_size
    
    parquet_size_mb = parquet_size_bytes / (1024 * 1024)
    logger.info(f"    Size: {parquet_size_mb:.2f} MB")
    
except Exception as e:
    logger.warning(f"  ⚠️  Error saving to Data Lake: {str(e)}")
    parquet_path = None
    parquet_size_mb = 0

# ============================================================
# STEP 10: UPDATE METADATA TABLE
# ============================================================
logger.info(f"📝 [STEP 10/11] Updating metadata...")

try:
    metadata_table = 'etl_metadata'
    
    # Create metadata table if not exists
    metadata_create_sql = f'''
        CREATE TABLE IF NOT EXISTS {target_schema}.{metadata_table} (
            id SERIAL PRIMARY KEY,
            table_name VARCHAR(255),
            load_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
            load_strategy VARCHAR(50),
            rows_loaded INTEGER,
            source_file VARCHAR(500),
            status VARCHAR(50),
            duration_seconds FLOAT,
            table_size_mb FLOAT,
            error_message TEXT
        )
    '''
    # SERIAL: auto-increment integer
    # - Equivalent: INTEGER AUTO_INCREMENT (MySQL)
    # - Generates: sequence for ID generation
    
    with engine.begin() as connection:
        connection.execute(text(metadata_create_sql))
        
        # Insert metadata record
        metadata_insert_sql = f'''
            INSERT INTO {target_schema}.{metadata_table}
            (table_name, load_strategy, rows_loaded, source_file, status, duration_seconds, table_size_mb)
            VALUES (:table, :strategy, :rows, :source, :status, :duration, :size)
        '''
        # Parameterized query: use :param_name for parameters
        # - Prevents SQL injection
        # - Automatic type conversion
        
        connection.execute(
            text(metadata_insert_sql),
            {
                'table': target_table,
                'strategy': load_strategy,
                'rows': rows_loaded,
                'source': str(data_path),
                'status': 'success',
                'duration': round(load_duration, 2),
                'size': round(table_size_mb, 2)
            }
        )
    
    logger.info(f"  ✅ Metadata updated")
    
except SQLAlchemyError as e:
    logger.warning(f"  ⚠️  Could not update metadata: {str(e)}")

# ============================================================
# STEP 11: ARCHIVE SOURCE FILES
# ============================================================
logger.info(f"📦 [STEP 11/11] Archiving source files...")

if archive_source:
    try:
        archive_dir = Path('/opt/airflow/data/archive')
        archive_dir.mkdir(parents=True, exist_ok=True)
        
        # Create timestamped archive subdirectory
        archive_subdir = archive_dir / datetime.now().strftime('%Y%m%d')
        archive_subdir.mkdir(exist_ok=True)
        
        # Find original raw file
        raw_dir = Path('/opt/airflow/data/raw')
        raw_files = list(raw_dir.glob('*.csv'))
        
        for raw_file in raw_files:
            archive_dest = archive_subdir / raw_file.name
            
            # Move file
            shutil.move(str(raw_file), str(archive_dest))
            # shutil.move(): move file
            # - Similar to: os.rename() but works across filesystems
            # - Alternative: shutil.copy2() + os.remove() (preserve metadata)
            
            logger.info(f"  - Archived: {raw_file.name}")
        
        logger.info(f"  ✅ Files archived to {archive_subdir}")
        
    except Exception as e:
        logger.warning(f"  ⚠️  Error archiving files: {str(e)}")
else:
    logger.info(f"  - Archive skipped")

# ============================================================
# FINAL SUMMARY
# ============================================================
end_time = time.time()
total_duration = end_time - start_time

logger.info(f"")
logger.info(f"{'='*60}")
logger.info(f"LOADING SUMMARY")
logger.info(f"{'='*60}")
logger.info(f"Table:           {target_schema}.{target_table}")
logger.info(f"Strategy:        {load_strategy}")
logger.info(f"Rows loaded:     {rows_loaded:,}")
logger.info(f"Database size:   {table_size_mb:.2f} MB")
logger.info(f"Parquet size:    {parquet_size_mb:.2f} MB")
logger.info(f"Indexes created: {len(indexes_created)}")
logger.info(f"Total duration:  {total_duration:.2f}s")
logger.info(f"{'='*60}")

# Create metadata
metadata = {
    'status': 'success',
    'table_name': f'{target_schema}.{target_table}',
    'load_strategy': load_strategy,
    'rows_loaded': rows_loaded,
    'database_row_count': db_row_count,
    'database_size_mb': round(table_size_mb, 2),
    'parquet_path': str(parquet_path) if parquet_path else None,
    'parquet_size_mb': round(parquet_size_mb, 2),
    'indexes_created': indexes_created,
    'partitioned_by': partition_by,
    'load_duration_seconds': round(load_duration, 2),
    'total_duration_seconds': round(total_duration, 2),
    'rows_per_second': round(rows_per_second, 0),
    'timestamp': datetime.now().isoformat()
}

logger.info(f"✅ Data loading completed successfully!")

return metadata

---

### 🎓 Key Learnings from Task 5

**1. Loading Strategies**:
- **Replace**: Full refresh, simple, no history
- **Append**: Incremental, fast, preserves history
- **Upsert**: Complex, handles updates, uses ON CONFLICT

**2. Database Best Practices**:
- **Connection Pooling**: Reuse connections (pool_size, max_overflow)
- **Transactions**: Use engine.begin() for atomicity
- **Batch Processing**: Insert in chunks (10K-50K rows)
- **Parameterized Queries**: Prevent SQL injection

**3. Performance Optimization**:
- **Batch Size**: Balance memory vs speed
- **method='multi'**: 5-10x faster than individual inserts
- **Indexes**: Create after bulk load (faster)
- **COPY command**: Fastest (alternative to to_sql)

**4. Data Lake Strategy**:
- **Partitioning**: By time (year/month) or dimension (country)
- **Compression**: Snappy (balanced) vs Gzip (smaller)
- **Organization**: Consistent directory structure

**5. Production Considerations**:
- **Rollback Plan**: Transaction-based, backup before replace
- **Monitoring**: Track throughput, duration, size
- **Metadata**: Audit trail of all loads
- **Archive**: Move processed files, retention policy

---

### ✅ Testing Task 5
```python
# tests/test_loading.py
import pytest
from dags.tasks.loading import load_data
from sqlalchemy import create_engine, text

def test_load_replace_strategy():
    """Test full replace load"""
    validation_metadata = {
        'status': 'PASS',
        'summary': {'critical_errors': 0}
    }
    
    metadata = load_data(
        validation_metadata=validation_metadata,
        target_table='test_table',
        load_strategy='replace'
    )
    
    assert metadata['status'] == 'success'
    assert metadata['rows_loaded'] > 0

def test_load_with_indexes():
    """Test index creation"""
    validation_metadata = {
        'status': 'PASS',
        'summary': {'critical_errors': 0}
    }
    
    metadata = load_data(
        validation_metadata=validation_metadata,
        target_table='test_table',
        create_indexes=True,
        index_columns=['customer_id', 'order_date']
    )
    
    assert len(metadata['indexes_created']) >= 2

def test_load_partitioned_parquet():
    """Test partitioned Data Lake save"""
    validation_metadata = {
        'status': 'PASS',
        'summary': {'critical_errors': 0}
    }
    
    metadata = load_data(
        validation_metadata=validation_metadata,
        partition_by=['year', 'month']
    )
    
    assert metadata['partitioned_by'] == ['year', 'month']
    assert metadata['parquet_path'] is not None
```

---

## 🎉 ขั้นตอนถัดไป

คุณได้เรียนรู้ครบทั้ง 5 Tasks หลักของ ETL Pipeline แล้ว! 

**สิ่งที่เหลืออยู่คือ Phase 5: Production Features**:
1. Error Handling & Retry Logic
2. Monitoring & Logging
3. Configuration Management
4. Testing
5. Documentation

ต้องการให้ฉันอธิบาย Phase 5 ต่อไหม? หรือมีคำถามเกี่ยวกับ 5 Tasks ที่ผ่านมา? 🚀