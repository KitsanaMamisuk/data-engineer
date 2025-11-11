ได้เลยครับ! เนื้อหาทั้งหมดนี้ถูกจัดทำในรูปแบบ Markdown อย่างสมบูรณ์แล้ว คุณสามารถคัดลอกและบันทึกเป็นไฟล์ .md เพื่อใช้งานได้ทันทีครับ 💾

📘 Python Data Engineer IoT Pipeline Handbook (Professional Learning Edition)
Author: Kitsana Mameesukh Edition: Professional Learning Edition Dataset: sensor_sample.csv, sensor_large.csv

📑 สารบัญ (Table of Contents)
Module 1: Python Fundamentals & Environment

Module 2: ETL: Data Loading & Cleansing

Module 3: ETL: Transformation & Optimization

Module 4: Cloud Integration & Storage

Module 5: Airflow: Orchestration Basics

Module 6: Airflow: Building the IoT Pipeline

Module 7: Data Quality & Testing

Module 8: Project Summary & Future Scope

<a id="module-1-python-fundamentals--environment"></a> Module 1: Python Fundamentals & Environment
### Data Structures (Pandas/NumPy)
ในงาน Data Engineering การจัดการข้อมูลขนาดใหญ่และเชิงตัวเลขมักจะใช้ไลบรารี Pandas สำหรับการจัดการตารางข้อมูล (DataFrame) และ NumPy สำหรับการคำนวณทางคณิตศาสตร์ที่รวดเร็ว โดยเฉพาะอย่างยิ่งข้อมูลจาก IoT Sensors ที่เป็น Time-Series Data ที่ต้องมีการคำนวณต่อเนื่อง

### Virtual Environment Setup
การตั้งค่า Virtual Environment (เช่น venv หรือ conda) เป็นสิ่งสำคัญอย่างยิ่งในการจัดการ Dependencies ของโครงการ ทำให้มั่นใจว่าไลบรารีที่ใช้จะไม่ไปกระทบกับโปรเจกต์อื่น

Bash

# ตัวอย่างการตั้งค่า Virtual Environment
python3 -m venv venv_iot
source venv_iot/bin/activate
pip install pandas numpy # ติดตั้งไลบรารีหลัก
### Time-Series Data Introduction
ข้อมูล IoT ส่วนใหญ่อยู่ในรูปแบบ Time-Series ซึ่งหมายถึงข้อมูลที่มีการบันทึกตามลำดับเวลาที่แน่นอน การจัดการข้อมูลประเภทนี้จำเป็นต้องมีการจัดเรียงตามเวลาที่ถูกต้อง และการใช้เทคนิคเฉพาะ เช่น Window Functions

Labs & Practical Exercises
Setup & Test: Verify pandas and numpy installation.

สร้าง venv และ Activate

ติดตั้ง pandas และ numpy

เขียนโค้ด Python ง่าย ๆ เพื่อ Import Pandas และแสดงเวอร์ชัน

Python

# [Jupyter Snippet]
import pandas as pd
import numpy as np
print(f"Pandas Version: {pd.__version__}")
print(f"NumPy Version: {np.__version__}")
<a id="module-2-etl-data-loading--cleansing"></a> Module 2: ETL: Data Loading & Cleansing
### Optimized Loading (dtype)
เมื่อต้องจัดการกับไฟล์ขนาดใหญ่ (เช่น sensor_large.csv) การระบุ Data Type (dtype) ที่เหมาะสมตั้งแต่ขั้นตอนการโหลดจะช่วยลดการใช้หน่วยความจำ (Memory Footprint) ของ DataFrame ได้อย่างมาก

Python

# [Python Snippet - Optimized Loading]
import pandas as pd
# กำหนด dtype เพื่อประหยัด Memory
data_types = {
    'sensor_id': 'int16', 
    'temperature': 'float32',
    'humidity': 'float32',
    'reading_time': 'object'
}
df = pd.read_csv('sensor_large.csv', dtype=data_types)
df['reading_time'] = pd.to_datetime(df['reading_time'])
print(f"Memory Usage (MB): {df.memory_usage(deep=True).sum() / (1024**2):.2f}")
### Handling Missing Values (Ffill/Mean)
การจัดการค่าว่าง (Missing Values) เป็นขั้นตอนสำคัญ Ffill (Forward Fill) เป็นเทคนิคที่เหมาะสมสำหรับ Time-Series Data โดยใช้ค่าล่าสุดที่วัดได้มาเติมเต็มช่องว่าง

Python

# [Python/Pandas Snippet - Ffill]
# ใช้ Ffill เติมค่าว่าง: ใช้ค่าก่อนหน้า
df['temperature'].fillna(method='ffill', inplace=True)

# จัดการค่าว่างที่เหลือ (ถ้ามี) ด้วยค่าเฉลี่ย
df['temperature'].fillna(df['temperature'].mean(), inplace=True)
### Data Integrity Checks
การตรวจสอบความสมบูรณ์ของข้อมูล เช่น การลบแถวที่ซ้ำซ้อน (Duplicate Rows) หรือการจัดการกับค่าที่อยู่นอกขอบเขต (Outliers)

Labs & Practical Exercises
Lab 2.1: Load sensor_large.csv using type optimization and calculate memory saving.

โหลดไฟล์ sensor_large.csv โดยไม่กำหนด dtype และบันทึก Memory Usage

โหลดไฟล์เดิมโดยกำหนด dtype และบันทึก Memory Usage

แสดงผลลัพธ์การจัดการค่าว่างในคอลัมน์ temperature ด้วย method='ffill'

<a id="module-3-etl-transformation--optimization"></a> Module 3: ETL: Transformation & Optimization
### Feature Engineering (Rolling Average)
Feature Engineering คือการสร้างคุณลักษณะใหม่จากข้อมูลดิบเพื่อใช้ในการวิเคราะห์หรือโมเดล Rolling Average (ค่าเฉลี่ยเคลื่อนที่) ช่วยให้ข้อมูล Time-Series มีความราบรื่นและลด Noise

Python

# [Python/Pandas Snippet - Rolling Average]
# คำนวณค่าเฉลี่ยอุณหภูมิย้อนหลัง 5 นาที
df['temp_rolling_avg_5m'] = (
    df['temperature']
    .rolling(window='5min', closed='left')
    .mean()
)
### Performance Benchmarking (Pandas vs. NumPy)
เพื่อยืนยันว่าโค้ด ETL ของเรามีประสิทธิภาพสูงสุด เราต้องทำ Benchmarking เพื่อเปรียบเทียบความเร็วระหว่างวิธีการเขียนโค้ดที่แตกต่างกัน

### Chunking
เทคนิคในการอ่านไฟล์ขนาดใหญ่เป็นส่วน ๆ (Chunks) เพื่อป้องกันปัญหา Memory Overflow

Labs & Practical Exercises
Lab 3.1: Implement Rolling Average (5min window).

ตั้งค่าคอลัมน์ reading_time เป็น Index

คำนวณ temperature.rolling(window='5min').mean()

Lab 3.2: Benchmark NumPy subtraction vs. Pandas series subtraction.

วัดเวลาการคำนวณโดยใช้ Pandas: df['diff_pd'] = df['temperature'] - df['humidity']

วัดเวลาการคำนวณโดยใช้ NumPy Array

<a id="module-4-cloud-integration--storage"></a> Module 4: Cloud Integration & Storage
### Cloud Storage Concepts (Data Lake)
Data Lake คือที่เก็บข้อมูลดิบขนาดใหญ่ในรูปแบบดั้งเดิม โดยทั่วไปใช้บริการ Object Storage เช่น Amazon S3 หรือ Google Cloud Storage (GCS)

### Using Cloud SDKs (Conceptual)
การเชื่อมต่อและโต้ตอบกับบริการ Cloud Storage ต้องใช้ SDK (Software Development Kits) ที่จัดทำโดยผู้ให้บริการ Cloud นั้น ๆ (เช่น Boto3 สำหรับ AWS).

### Data Partitioning Strategy
Partitioning คือการจัดโครงสร้างโฟลเดอร์ข้อมูลตามค่าของคอลัมน์ (เช่น ปี, เดือน, วัน) ซึ่งช่วยให้การสืบค้นข้อมูลในภายหลังทำได้เร็วขึ้นมาก

Python

# [Python Snippet - Data Partitioning Concept]
base_path = "s3://my-iot-lake/processed_data/"

# การบันทึกไฟล์ด้วย Partitioning (ใช้ pyarrow/pandas to_parquet)
# df.to_parquet(
#     base_path,
#     partition_cols=['year', 'month', 'day'],
#     index=False
# )
Labs & Practical Exercises
Lab 4.1: Define file path and naming convention for Time-Series Partitioning (year/month/day).

กำหนดรูปแบบ Path สำหรับการบันทึกไฟล์ Parquet ที่ประมวลผลแล้ว

จำลองการสร้างคอลัมน์ Partitioning (year, month, day)

<a id="module-5-airflow-orchestration-basics"></a> Module 5: Airflow: Orchestration Basics
### DAGs, Tasks, Operators
Apache Airflow คือเครื่องมือสำหรับ Workflow Orchestration หัวใจของมันคือ DAG (Directed Acyclic Graph) ซึ่งเป็นชุดของ Tasks ที่จัดเรียงตามลำดับความสัมพันธ์ (Dependencies) โดยใช้ Operators

### Dependencies
การกำหนดลำดับการรันของ Tasks โดยใช้สัญลักษณ์ >> (รันก่อน) หรือ << (รันหลัง)

### Airflow UI Navigation
ทำความคุ้นเคยกับ User Interface (UI) ของ Airflow สำหรับการดูสถานะ DAGs, Logs, และ Graphs

Labs & Practical Exercises
Lab 5.1: Install Airflow (via Docker or local).

ติดตั้ง Airflow และเริ่มต้น Webserver / Scheduler

Lab 5.2: Create a simple "Hello World" DAG.

เขียน Python File เพื่อสร้าง DAG ง่าย ๆ ที่มีเพียง BashOperator เพื่อ echo "Hello Airflow"

<a id="module-6-airflow-building-the-iot-pipeline"></a> Module 6: Airflow: Building the IoT Pipeline
### FileSensor Implementation
Sensor Operator ใน Airflow ใช้สำหรับ รอ ให้เงื่อนไขภายนอกเป็นจริงก่อน Tasks ถัดไปจะเริ่มทำงาน ในกรณีนี้คือ FileSensor เพื่อรอไฟล์ข้อมูลเข้า

### PythonOperator Integration
การใช้ PythonOperator เพื่อเรียกใช้ฟังก์ชัน Python ที่เราเขียนไว้ใน Module 2 & 3 (ฟังก์ชัน ETL หลัก)

### Cloud Operator Use (Conceptual)
การเชื่อมต่อกับ Cloud Services โดยตรงจาก Airflow โดยใช้ Operator เฉพาะ (เช่น S3Hook, S3CopyObjectOperator)

Labs & Practical Exercises
Lab 6.1: Define a DAG with T1 (FileSensor) >> T2 (Python ETL).

เขียนโค้ด Airflow DAG ที่มี Task T1 (FileSensor) และ Task T2 (PythonOperator)

กำหนด Dependency: T1 >> T2 และทดสอบรัน DAG

Python

# [Airflow DAG Snippet]
from airflow.sensors.filesystem import FileSensor
from airflow.operators.python import PythonOperator
# ... (imports)

with DAG(
    dag_id="iot_pipeline_with_sensor",
    schedule="@daily",
    # ...
) as dag:
    wait_for_data = FileSensor(task_id="wait_for_sensor_data_file", ...)
    etl_process = PythonOperator(task_id="run_etl_data_transformation", ...)
    # upload_to_cloud = ...
    
    wait_for_data >> etl_process # >> upload_to_cloud
<a id="module-7-data-quality--testing"></a> Module 7: Data Quality & Testing
### Data Validation (Range/Null Checks)
การทำ Data Quality (DQ) Check เพื่อตรวจสอบว่าข้อมูลที่ผ่านการประมวลผลแล้วยังคงมีคุณภาพตามมาตรฐาน เช่น ข้อมูลไม่อยู่ในขอบเขตที่กำหนด หรือมีค่าว่างในคอลัมน์ที่จำเป็น

### Pipeline Idempotency
Idempotency คือคุณสมบัติของ Pipeline ที่ช่วยให้การรันซ้ำหลายครั้งมีผลลัพธ์เหมือนกับการรันเพียงครั้งเดียว

### Unit Testing ETL Functions
การเขียน Unit Test สำหรับฟังก์ชัน ETL โดยใช้ไลบรารีอย่าง pytest

Labs & Practical Exercises
Lab 7.1: Write a Python function to check if processed data has any NULLs in the 'temperature' column.

สร้างฟังก์ชัน perform_data_validation(df)

เพิ่ม Logic ที่ใช้ df['temperature'].isnull().any() และ raise ValueError หากพบค่าว่าง

Python

# [Python DQ Check Snippet]
def perform_data_validation(df):
    if df['temperature'].isnull().any():
        raise ValueError("DQ Check Failed: Missing Values found in 'temperature'")
    # เพิ่ม check อื่นๆ เช่น Range Check
    return True
<a id="module-8-project-summary--future-scope"></a> Module 8: Project Summary & Future Scope
### Executive Summary
การสรุปภาพรวมของโครงการสำหรับผู้บริหารหรือผู้สัมภาษณ์ เน้นที่ปัญหาที่แก้ไข, แนวทางการนำไปใช้, และผลลัพธ์ที่สำคัญ

### Key Skills Mapping
การจัดทำตารางสรุปทักษะและเครื่องมือทั้งหมดที่ใช้ในโครงการ เพื่อให้สอดคล้องกับตำแหน่ง Data Engineer Expert

### Real-Time/MLOps Vision
การแสดงวิสัยทัศน์ในอนาคตของ Pipeline เช่น การต่อยอดสู่ระบบ Real-Time Streaming (Kafka/Spark) หรือการเชื่อมโยงกับ MLOps
