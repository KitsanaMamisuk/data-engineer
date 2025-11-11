# Module 5: Airflow - Orchestration Basics

**🎯 วัตถุประสงค์การเรียนรู้:**
- เข้าใจแนวคิด Workflow Orchestration และความสำคัญ
- รู้จัก Apache Airflow และสถาปัตยกรรม
- เรียนรู้ DAG (Directed Acyclic Graph) และการออกแบบ
- ใช้งาน Tasks, Operators, และ Dependencies
- จัดการ Scheduling ด้วย Cron Expressions
- นำทางและใช้งาน Airflow UI

**⏱️ เวลาที่ใช้:** 5 ชั่วโมง (ทฤษฎี 3 ชม. + Lab 2 ชม.)

---

## 📚 สารบัญ

1. [Workflow Orchestration คืออะไร?](#1-workflow-orchestration-คืออะไร)
2. [Apache Airflow Introduction](#2-apache-airflow-introduction)
3. [DAG: Directed Acyclic Graph](#3-dag-directed-acyclic-graph)
4. [Tasks and Operators](#4-tasks-and-operators)
5. [Dependencies and Scheduling](#5-dependencies-and-scheduling)
6. [Airflow UI Navigation](#6-airflow-ui-navigation)
7. [Installation Methods](#7-installation-methods)
8. [Labs & Practical Exercises](./labs/lab-module-5.md)

---

## 1. Workflow Orchestration คืออะไร?

### 1.1 ปัญหาของ Data Pipeline แบบเดิม

ก่อนมี Orchestration Tools, Data Engineers มักประสบปัญหา:

❌ **Manual Execution** - ต้องรัน Scripts ด้วยมือทีละ Step
❌ **No Dependency Management** - ไม่รู้ว่า Task ไหนต้องรันก่อน-หลัง
❌ **No Monitoring** - ไม่มี Dashboard ดูสถานะ Pipeline
❌ **Hard to Retry** - เมื่อ Fail ต้องรันใหม่ทั้งหมด
❌ **No Scheduling** - ต้องใช้ Cron แยกกันหลาย Scripts

### 1.2 Workflow Orchestration แก้ปัญหาอย่างไร?

**Workflow Orchestration** = การจัดการและควบคุม Data Pipeline แบบอัตโนมัติ

```
แนวคิด Orchestration:

┌─────────────────────────────────────────────────┐
│         Workflow Orchestrator                   │
│  (Apache Airflow, Prefect, Dagster, etc.)      │
├─────────────────────────────────────────────────┤
│                                                 │
│  ✅ Scheduling - กำหนดเวลารันอัตโนมัติ         │
│  ✅ Dependencies - จัดการลำดับการทำงาน         │
│  ✅ Monitoring - ดูสถานะแบบ Real-time           │
│  ✅ Retry Logic - ลองใหม่เมื่อ Fail             │
│  ✅ Alerting - แจ้งเตือนเมื่อมีปัญหา            │
│  ✅ Logging - บันทึกทุก Execution               │
│                                                 │
└─────────────────────────────────────────────────┘
```

### 1.3 ตัวอย่าง Use Cases

**1. ETL Pipeline:**
```
Extract Data → Transform Data → Load to Database
```

**2. IoT Data Pipeline:**
```
Wait for File → Validate Data → Process → Upload to Cloud
```

**3. Machine Learning Pipeline:**
```
Fetch Data → Train Model → Evaluate → Deploy
```

**4. Report Generation:**
```
Query Database → Generate Report → Send Email
```

---

## 2. Apache Airflow Introduction

### 2.1 Apache Airflow คืออะไร?

**Apache Airflow** = Open-source Platform สำหรับสร้างและจัดการ Workflows แบบ Programmatic

**ประวัติ:**
- 🏢 สร้างโดย Airbnb ปี 2014
- 🌐 เป็น Apache Project ปี 2019
- 🌟 ยอดนิยมที่สุดสำหรับ Data Engineering

**จุดเด่น:**
- ✅ **Python-based** - เขียน Workflows ด้วย Python
- ✅ **Dynamic** - สร้าง DAG แบบ Programmatic
- ✅ **Extensible** - มี Operators มากมาย, สร้าง Custom ได้
- ✅ **Scalable** - รองรับ Distributed Execution
- ✅ **Rich UI** - Web Interface สวยงามและใช้งานง่าย

### 2.2 Airflow Architecture

```
┌────────────────────────────────────────────────────────┐
│              Airflow Architecture                      │
└────────────────────────────────────────────────────────┘

    ┌──────────────────┐
    │   Airflow UI     │ ◄─── Users เข้าใช้งานผ่าน Browser
    │   (Web Server)   │
    └─────────┬────────┘
              │
    ┌─────────▼────────┐
    │   Scheduler      │ ◄─── จัดการ Scheduling และ Trigger Tasks
    └─────────┬────────┘
              │
    ┌─────────▼────────┐
    │   Metadata DB    │ ◄─── เก็บสถานะ DAGs, Tasks, Logs
    │  (PostgreSQL/    │
    │   MySQL)         │
    └─────────┬────────┘
              │
    ┌─────────▼────────┐
    │    Executor      │ ◄─── รัน Tasks (LocalExecutor, CeleryExecutor)
    └─────────┬────────┘
              │
    ┌─────────▼────────┐
    │    Workers       │ ◄─── ทำงานจริง (Execute Python code)
    └──────────────────┘
```

**Components อธิบาย:**

| Component | หน้าที่ |
|-----------|---------|
| **Web Server** | UI สำหรับดู DAGs, Logs, Monitoring |
| **Scheduler** | ตรวจสอบและ Trigger Tasks ตาม Schedule |
| **Metadata DB** | เก็บข้อมูล DAGs, Task Status, Connections |
| **Executor** | จัดสรร Tasks ไปยัง Workers |
| **Workers** | รัน Tasks จริง |

### 2.3 Airflow vs Cron

| Feature | Cron | Airflow |
|---------|------|---------|
| **Scheduling** | ⭐⭐⭐ | ⭐⭐⭐ |
| **Dependencies** | ❌ ไม่มี | ✅ รองรับ |
| **Monitoring** | ❌ ไม่มี | ✅ Rich UI |
| **Retry Logic** | ❌ ต้องเขียนเอง | ✅ Built-in |
| **Logging** | ⭐ พื้นฐาน | ⭐⭐⭐ ครบถ้วน |
| **Alerting** | ❌ ต้องเขียนเอง | ✅ Built-in |
| **Complexity** | 🟢 ง่าย | 🟡 ปานกลาง |

**💡 แนะนำ:**
- ใช้ **Cron** สำหรับ: Simple Tasks ที่รันอิสระ
- ใช้ **Airflow** สำหรับ: Complex Workflows ที่มี Dependencies

---

## 3. DAG: Directed Acyclic Graph

### 3.1 DAG คืออะไร?

**DAG (Directed Acyclic Graph)** = กราฟที่มีทิศทางและไม่มี Cycle

**คำอธิบาย:**
- **Directed** = ลูกศรชี้ทิศทาง (Task A → Task B)
- **Acyclic** = ไม่วนกลับมาจุดเดิม (ไม่มี Loop)
- **Graph** = โครงสร้างข้อมูลที่มี Nodes และ Edges

### 3.2 ภาพตัวอย่าง DAG

```
Simple ETL DAG:

    ┌──────────┐
    │ Extract  │
    │  Data    │
    └────┬─────┘
         │
         ▼
    ┌──────────┐
    │Transform │
    │  Data    │
    └────┬─────┘
         │
         ▼
    ┌──────────┐
    │  Load    │
    │  Data    │
    └──────────┘
```

```
Parallel Tasks DAG:

                ┌─────────────┐
                │    Start    │
                └──────┬──────┘
                       │
         ┌─────────────┼─────────────┐
         │             │             │
         ▼             ▼             ▼
    ┌────────┐   ┌────────┐   ┌────────┐
    │ Task A │   │ Task B │   │ Task C │
    └───┬────┘   └───┬────┘   └───┬────┘
        │            │            │
        └─────────┬──┴────────────┘
                  │
                  ▼
            ┌──────────┐
            │   End    │
            └──────────┘
```

```
Complex IoT Pipeline DAG:

    ┌──────────────┐
    │ FileSensor   │ ◄── รอไฟล์เข้ามา
    │ (Wait File)  │
    └──────┬───────┘
           │
           ▼
    ┌──────────────┐
    │  Validate    │ ◄── ตรวจสอบไฟล์
    │    File      │
    └──────┬───────┘
           │
           ▼
    ┌──────────────┐
    │   Extract    │ ◄── อ่านข้อมูล
    │   & Clean    │
    └──────┬───────┘
           │
           ├─────────────────┐
           │                 │
           ▼                 ▼
    ┌─────────────┐   ┌─────────────┐
    │ Transform   │   │  Generate   │ ◄── Parallel
    │   Data      │   │   Report    │
    └──────┬──────┘   └──────┬──────┘
           │                 │
           └────────┬────────┘
                    │
                    ▼
            ┌───────────────┐
            │  Upload to    │
            │    Cloud      │
            └───────────────┘
```

### 3.3 DAG Properties

```python
from airflow import DAG
from datetime import datetime, timedelta

# กำหนดค่าเริ่มต้นสำหรับ Tasks
default_args = {
    'owner': 'data-engineer',
    'depends_on_past': False,
    'email': ['alert@example.com'],
    'email_on_failure': True,
    'email_on_retry': False,
    'retries': 3,
    'retry_delay': timedelta(minutes=5),
}

# สร้าง DAG
dag = DAG(
    dag_id='example_etl_pipeline',
    default_args=default_args,
    description='A simple ETL pipeline',
    schedule_interval='0 2 * * *',  # ทุกวัน เวลา 02:00
    start_date=datetime(2025, 1, 1),
    catchup=False,
    tags=['etl', 'example'],
)
```

**Parameters อธิบาย:**

| Parameter | คำอธิบาย |
|-----------|----------|
| **dag_id** | ชื่อ DAG (ต้องไม่ซ้ำ) |
| **default_args** | ค่าเริ่มต้นสำหรับทุก Task |
| **description** | คำอธิบาย DAG |
| **schedule_interval** | ความถี่ในการรัน (Cron format) |
| **start_date** | วันที่เริ่มต้น |
| **catchup** | รัน Past Dates ไหม (False = ไม่รัน) |
| **tags** | ป้ายกำกับสำหรับจัดกลุ่ม |

---

## 4. Tasks and Operators

### 4.1 Task คืออะไร?

**Task** = หน่วยงานเล็กสุดใน DAG (Node ใน Graph)

**ตัวอย่าง:**
- อ่านไฟล์ CSV
- รัน Python function
- Execute SQL query
- ส่ง Email

### 4.2 Operator คืออะไร?

**Operator** = Template สำหรับสร้าง Task

**Operators ที่นิยมใช้:**

```
┌─────────────────────────────────────────┐
│         Airflow Operators               │
├─────────────────────────────────────────┤
│                                         │
│  📝 PythonOperator                      │
│     - รัน Python function               │
│                                         │
│  💻 BashOperator                        │
│     - รัน Shell commands                │
│                                         │
│  🔍 FileSensor                          │
│     - รอไฟล์ปรากฏ                       │
│                                         │
│  🗃️ PostgresOperator / MySQLOperator   │
│     - รัน SQL queries                   │
│                                         │
│  ☁️ S3Operator / GCSOperator            │
│     - Upload/Download จาก Cloud        │
│                                         │
│  📧 EmailOperator                       │
│     - ส่ง Email                         │
│                                         │
│  🕒 TimeSensor                          │
│     - รอถึงเวลาที่กำหนด                  │
│                                         │
└─────────────────────────────────────────┘
```

### 4.3 PythonOperator - พื้นฐาน

```python
from airflow.operators.python import PythonOperator

# 1. สร้าง Python function
def print_hello():
    print("Hello from Airflow!")
    return "Success"

# 2. สร้าง Task จาก PythonOperator
task_hello = PythonOperator(
    task_id='print_hello_task',
    python_callable=print_hello,
    dag=dag,
)
```

**พารามิเตอร์สำคัญ:**
- `task_id` - ชื่อ Task (ต้องไม่ซ้ำใน DAG)
- `python_callable` - Python function ที่จะรัน
- `op_kwargs` - Arguments ส่งให้ function (dict)
- `dag` - DAG ที่ Task นี้สังกัด

### 4.4 PythonOperator - ส่ง Arguments

```python
def process_data(sensor_id, date):
    print(f"Processing data for sensor: {sensor_id}")
    print(f"Date: {date}")
    return f"Processed {sensor_id}"

task_process = PythonOperator(
    task_id='process_sensor_data',
    python_callable=process_data,
    op_kwargs={
        'sensor_id': 'S001',
        'date': '2025-01-01'
    },
    dag=dag,
)
```

### 4.5 BashOperator - รัน Shell Commands

```python
from airflow.operators.bash import BashOperator

# สร้าง directory
task_mkdir = BashOperator(
    task_id='create_directory',
    bash_command='mkdir -p /tmp/airflow_data',
    dag=dag,
)

# รัน Python script
task_run_script = BashOperator(
    task_id='run_etl_script',
    bash_command='python /path/to/etl_script.py',
    dag=dag,
)

# Copy file
task_copy = BashOperator(
    task_id='copy_file',
    bash_command='cp /source/data.csv /destination/',
    dag=dag,
)
```

### 4.6 FileSensor - รอไฟล์ปรากฏ

```python
from airflow.sensors.filesystem import FileSensor

# รอไฟล์ CSV
task_wait_file = FileSensor(
    task_id='wait_for_sensor_file',
    filepath='/data/sensor_data_{{ ds }}.csv',  # ds = execution_date
    poke_interval=30,  # เช็คทุก 30 วินาที
    timeout=600,  # Timeout หลัง 10 นาที
    dag=dag,
)
```

**Parameters:**
- `filepath` - Path ของไฟล์ที่รอ
- `poke_interval` - ระยะเวลาเช็คซ้ำ (วินาที)
- `timeout` - Timeout (วินาที)
- `mode` - 'poke' (default) หรือ 'reschedule'

### 4.7 ตัวอย่าง Complete DAG

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.operators.bash import BashOperator
from datetime import datetime, timedelta

# Python functions
def extract_data():
    print("Extracting data from source...")
    return "data_extracted"

def transform_data():
    print("Transforming data...")
    return "data_transformed"

def load_data():
    print("Loading data to destination...")
    return "data_loaded"

# Default args
default_args = {
    'owner': 'airflow',
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
}

# DAG
dag = DAG(
    dag_id='simple_etl_dag',
    default_args=default_args,
    description='Simple ETL pipeline',
    schedule_interval='@daily',
    start_date=datetime(2025, 1, 1),
    catchup=False,
)

# Tasks
task_extract = PythonOperator(
    task_id='extract',
    python_callable=extract_data,
    dag=dag,
)

task_transform = PythonOperator(
    task_id='transform',
    python_callable=transform_data,
    dag=dag,
)

task_load = PythonOperator(
    task_id='load',
    python_callable=load_data,
    dag=dag,
)

# Dependencies (แบบที่ 1)
task_extract >> task_transform >> task_load
```

---

## 5. Dependencies and Scheduling

### 5.1 Task Dependencies

**Dependencies** = ความสัมพันธ์ระหว่าง Tasks (Task ไหนต้องรันก่อน-หลัง)

### 5.2 การกำหนด Dependencies

```python
# วิธีที่ 1: ใช้ >> operator (แนะนำ)
task_a >> task_b >> task_c

# วิธีที่ 2: ใช้ set_downstream()
task_a.set_downstream(task_b)
task_b.set_downstream(task_c)

# วิธีที่ 3: ใช้ set_upstream()
task_c.set_upstream(task_b)
task_b.set_upstream(task_a)

# วิธีที่ 4: ใช้ << operator
task_c << task_b << task_a
```

### 5.3 Dependency Patterns

**Pattern 1: Linear (Sequential)**
```python
# A → B → C → D
task_a >> task_b >> task_c >> task_d
```

```
    ┌───┐     ┌───┐     ┌───┐     ┌───┐
    │ A │ --> │ B │ --> │ C │ --> │ D │
    └───┘     └───┘     └───┘     └───┘
```

**Pattern 2: Parallel (Fan-out)**
```python
# A → [B, C, D] (B, C, D รันพร้อมกัน)
task_a >> [task_b, task_c, task_d]
```

```
              ┌───┐
         ┌--> │ B │
         │    └───┘
    ┌───┐│
    │ A │├--> ┌───┐
    └───┘│    │ C │
         │    └───┘
         └--> ┌───┐
              │ D │
              └───┘
```

**Pattern 3: Fan-in (รวมกลับ)**
```python
# [A, B, C] → D
[task_a, task_b, task_c] >> task_d
```

```
    ┌───┐
    │ A │ --┐
    └───┘   │
            │    ┌───┐
    ┌───┐   ├--> │ D │
    │ B │ --┘    └───┘
    └───┘   │
            │
    ┌───┐   │
    │ C │ --┘
    └───┘
```

**Pattern 4: Complex Dependencies**
```python
# A → B
# A → C
# [B, C] → D
task_a >> task_b >> task_d
task_a >> task_c >> task_d
```

```
         ┌───┐
         │ B │ --┐
    ┌───┐└───┘   │
    │ A │         ├--> ┌───┐
    └───┘┌───┐   │    │ D │
         │ C │ --┘    └───┘
         └───┘
```

### 5.4 Scheduling - Cron Expressions

Airflow ใช้ **Cron Expression** สำหรับกำหนดตาราง Schedule

**รูปแบบ Cron:**
```
 ┌─────── minute (0 - 59)
 │ ┌────── hour (0 - 23)
 │ │ ┌───── day of month (1 - 31)
 │ │ │ ┌──── month (1 - 12)
 │ │ │ │ ┌─── day of week (0 - 6) (0 = Sunday)
 │ │ │ │ │
 * * * * *
```

**ตัวอย่าง:**

| Cron Expression | ความหมาย |
|-----------------|----------|
| `0 2 * * *` | ทุกวัน เวลา 02:00 |
| `*/5 * * * *` | ทุก 5 นาที |
| `0 */2 * * *` | ทุก 2 ชั่วโมง |
| `0 0 * * 0` | ทุกวันอาทิตย์ เที่ยงคืน |
| `0 9 1 * *` | วันที่ 1 ของทุกเดือน เวลา 09:00 |
| `0 0 1 1 *` | วันที่ 1 มกราคม ทุกปี |

**Airflow Presets (ใช้แทน Cron):**

| Preset | Cron Equivalent | ความหมาย |
|--------|----------------|----------|
| `@once` | None | รันครั้งเดียว |
| `@hourly` | `0 * * * *` | ทุกชั่วโมง |
| `@daily` | `0 0 * * *` | ทุกวัน เที่ยงคืน |
| `@weekly` | `0 0 * * 0` | ทุกสัปดาห์ (วันอาทิตย์) |
| `@monthly` | `0 0 1 * *` | ทุกเดือน (วันที่ 1) |
| `@yearly` | `0 0 1 1 *` | ทุกปี (1 มกราคม) |

```python
# ตัวอย่างการใช้ schedule_interval
dag = DAG(
    dag_id='scheduled_dag',
    schedule_interval='@daily',  # หรือ '0 0 * * *'
    start_date=datetime(2025, 1, 1),
    catchup=False,
)
```

### 5.5 Execution Date vs Schedule Interval

**แนวคิดสำคัญ:**

```
Execution Date ≠ Runtime Date

Example: schedule_interval = @daily (เที่ยงคืน)

Timeline:
2025-01-01 00:00 ─┬─ Execution Date: 2025-01-01
                  │  (รันสำหรับข้อมูลวันที่ 1)
                  │  Runtime: 2025-01-02 00:00
                  │  (รันจริงวันที่ 2)
                  │
2025-01-02 00:00 ─┤
                  │  Execution Date: 2025-01-02
                  │  Runtime: 2025-01-03 00:00
                  ▼
```

**💡 เข้าใจง่าย:**
- **Execution Date** = วันที่ของข้อมูลที่เราประมวลผล
- **Runtime** = เวลาที่ DAG รันจริง

---

## 6. Airflow UI Navigation

### 6.1 Airflow UI Overview

```
┌────────────────────────────────────────────────────┐
│  Apache Airflow                          [User ▼] │
├────────────────────────────────────────────────────┤
│  DAGs | Data Profiling | Browse | Admin | Docs    │
├────────────────────────────────────────────────────┤
│                                                    │
│  DAGs List:                                        │
│                                                    │
│  🟢 iot_pipeline_dag        [Actions] [Graph]     │
│  🔴 etl_daily_job           [Actions] [Graph]     │
│  🟢 simple_hello_dag        [Actions] [Graph]     │
│                                                    │
└────────────────────────────────────────────────────┘
```

### 6.2 Main Views

**1. DAGs View (หน้าหลัก)**
- แสดง DAGs ทั้งหมด
- สถานะแต่ละ DAG
- Actions: Pause/Unpause, Trigger, Refresh

**2. Graph View**
```
แสดง DAG เป็นกราฟ:

    ┌─────────┐
    │ Task A  │ ✅ Success
    └────┬────┘
         │
    ┌────▼────┐
    │ Task B  │ ⏸️ Running
    └────┬────┘
         │
    ┌────▼────┐
    │ Task C  │ ⬜ Not Started
    └─────────┘
```

**3. Tree View**
```
แสดงประวัติการรัน:

DAG: iot_pipeline
├─ 2025-01-01  ✅✅✅
├─ 2025-01-02  ✅✅❌
├─ 2025-01-03  ✅⏸️⬜
└─ 2025-01-04  ⬜⬜⬜
```

**4. Gantt Chart**
```
แสดงระยะเวลาการทำงาน:

Task A  |████████|
Task B      |████|
Task C          |████████|
        12:00  12:05  12:10
```

**5. Calendar View**
- แสดงสถานะการรันแต่ละวัน
- เหมาะสำหรับดู Historical Data

### 6.3 Task States (สถานะ)

| State | Icon | ความหมาย |
|-------|------|----------|
| **success** | 🟢 | สำเร็จ |
| **running** | 🔵 | กำลังทำงาน |
| **failed** | 🔴 | ล้มเหลว |
| **skipped** | ⚪ | ข้าม (Conditional Task) |
| **queued** | 🟡 | รอคิว |
| **upstream_failed** | 🟠 | Task ก่อนหน้าล้มเหลว |
| **scheduled** | ⬜ | กำหนดเวลาแล้ว |
| **up_for_retry** | 🟣 | รอลองใหม่ |

### 6.4 Task Actions

**จาก Graph View สามารถ:**
- 👁️ **View Log** - ดู Logs
- 🔄 **Clear** - ล้างสถานะและรันใหม่
- ✅ **Mark Success** - บังคับให้สำเร็จ
- ❌ **Mark Failed** - บังคับให้ล้มเหลว
- ⏭️ **Skip** - ข้าม Task

### 6.5 Monitoring และ Troubleshooting

**เช็ค Logs:**
```
1. คลิก Task ใน Graph View
2. เลือก "View Log"
3. อ่าน Error Messages

ตัวอย่าง Log:
[2025-01-01 12:00:00] INFO - Starting task: extract_data
[2025-01-01 12:00:05] INFO - Extracting data...
[2025-01-01 12:00:10] ERROR - File not found: /data/sensor.csv
[2025-01-01 12:00:10] INFO - Task failed, will retry
```

**Common Issues:**

| Issue | สาเหตุ | แก้ไข |
|-------|--------|-------|
| **Import Error** | DAG file มี Syntax Error | ตรวจสอบ Code |
| **Task Timeout** | Task ทำงานนานเกินไป | เพิ่ม timeout parameter |
| **Dependency Cycle** | DAG มี Loop | ตรวจสอบ Dependencies |
| **Connection Failed** | ไม่เชื่อมต่อ Database/API | ตรวจสอบ Connections |

---

## 7. Installation Methods

### 7.1 วิธีติดตั้ง Airflow

มี 3 วิธีหลัก:

```
┌─────────────────────────────────────────┐
│       Airflow Installation Methods      │
├─────────────────────────────────────────┤
│                                         │
│  1️⃣ Docker (แนะนำสำหรับเริ่มต้น)       │
│     - ติดตั้งง่าย                       │
│     - มีทุกอย่างครบ                     │
│     - Isolated Environment              │
│                                         │
│  2️⃣ Virtual Environment (Local)         │
│     - Lightweight                       │
│     - เหมาะสำหรับ Development           │
│     - ควบคุมได้มาก                      │
│                                         │
│  3️⃣ Cloud Managed Services              │
│     - AWS MWAA, GCP Cloud Composer     │
│     - ไม่ต้องจัดการ Infrastructure      │
│     - เสียค่าใช้จ่าย                     │
│                                         │
└─────────────────────────────────────────┘
```

### 7.2 ติดตั้งด้วย Docker (แนะนำ)

```bash
# 1. ดาวน์โหลด docker-compose.yaml
curl -LfO 'https://airflow.apache.org/docs/apache-airflow/stable/docker-compose.yaml'

# 2. สร้าง directories
mkdir -p ./dags ./logs ./plugins ./config

# 3. สร้าง .env file
echo -e "AIRFLOW_UID=$(id -u)" > .env

# 4. Initialize Database
docker compose up airflow-init

# 5. Start Airflow
docker compose up

# เข้าใช้งาน: http://localhost:8080
# Username: airflow
# Password: airflow
```

**โครงสร้าง Folders:**
```
airflow/
├── dags/              ← เก็บ DAG files
├── logs/              ← Logs
├── plugins/           ← Custom Plugins
├── config/            ← Configuration
└── docker-compose.yaml
```

### 7.3 ติดตั้งด้วย Virtual Environment

```bash
# 1. สร้าง Virtual Environment
python -m venv airflow_env
source airflow_env/bin/activate  # macOS/Linux
# airflow_env\Scripts\activate  # Windows

# 2. ตั้งค่า AIRFLOW_HOME
export AIRFLOW_HOME=~/airflow

# 3. ติดตั้ง Airflow
AIRFLOW_VERSION=2.8.0
PYTHON_VERSION="$(python --version | cut -d " " -f 2 | cut -d "." -f 1-2)"
CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt"
pip install "apache-airflow==${AIRFLOW_VERSION}" --constraint "${CONSTRAINT_URL}"

# 4. Initialize Database
airflow db init

# 5. สร้าง Admin User
airflow users create \
    --username admin \
    --firstname Admin \
    --lastname User \
    --role Admin \
    --email admin@example.com \
    --password admin

# 6. Start Webserver และ Scheduler
airflow webserver --port 8080  # Terminal 1
airflow scheduler                # Terminal 2
```

### 7.4 เปรียบเทียบวิธีติดตั้ง

| Method | ความยาก | เวลา | Use Case |
|--------|----------|------|----------|
| **Docker** | 🟢 ง่าย | 10 นาที | Learning, Development |
| **Virtual Env** | 🟡 ปานกลาง | 15 นาที | Local Development |
| **Cloud** | 🟢 ง่าย | 5 นาที | Production |

---

## 8. สรุป Module 5

### 8.1 Key Takeaways

✅ **Workflow Orchestration** จัดการ Data Pipeline แบบอัตโนมัติ
✅ **Apache Airflow** เป็น Platform ยอดนิยมสำหรับ Data Engineering
✅ **DAG** คือ Directed Acyclic Graph (กราฟไม่มี Loop)
✅ **Tasks** สร้างจาก **Operators** (PythonOperator, BashOperator, etc.)
✅ **Dependencies** กำหนดด้วย `>>` operator
✅ **Scheduling** ใช้ Cron Expressions หรือ Presets (@daily, @hourly)
✅ **Airflow UI** มี Views หลากหลาย (Graph, Tree, Gantt)

### 8.2 Skills ที่ได้จาก Module นี้

| Skill | Level |
|-------|-------|
| Workflow Orchestration Concepts | ⭐⭐⭐ |
| DAG Design | ⭐⭐⭐ |
| Using Operators | ⭐⭐⭐ |
| Task Dependencies | ⭐⭐⭐ |
| Scheduling with Cron | ⭐⭐ |
| Airflow UI Navigation | ⭐⭐ |
| Installation & Setup | ⭐⭐ |

### 8.3 เตรียมพร้อมสำหรับ Module 6

ใน Module ถัดไป คุณจะได้เรียนรู้:
- สร้าง IoT Pipeline ด้วย Airflow
- ใช้ FileSensor รอไฟล์เข้ามา
- Integrate ETL code เข้า PythonOperator
- Upload ข้อมูลไปยัง Cloud Storage

---

## 📝 Challenge Questions

ก่อนไป Module 6 ลองตอบคำถามเหล่านี้:

1. **DAG vs Script:** อะไรคือข้อแตกต่างหลักระหว่าง Airflow DAG และ Python Script ธรรมดา?
2. **Dependencies:** ถ้ามี 3 Tasks (A, B, C) ที่ B และ C ต้องรอ A เสร็จก่อน แต่ B กับ C รันพร้อมกันได้ จะเขียน Dependencies อย่างไร?
3. **Cron:** ถ้าต้องการรัน DAG ทุกวันจันทร์ เวลา 08:00 ต้องใช้ Cron Expression อะไร?
4. **Operators:** PythonOperator กับ BashOperator ควรใช้ตัวไหน เมื่อไหร่?

---

## 🎯 ถัดไป: Labs & Practical Exercises

พร้อมทำ Lab แล้วหรือยัง?

**👉 [เริ่มทำ Lab Module 5](./labs/lab-module-5.md)**

ใน Lab คุณจะได้:
- ติดตั้ง Airflow ด้วย Docker
- สร้าง "Hello World" DAG แรก
- ทดสอบ Operators ต่างๆ
- ฝึกกำหนด Task Dependencies
- เรียนรู้การใช้งาน Airflow UI

---

**[⬅️ กลับไปที่ Wiki หลัก](../wiki.md)** | **[Module 6: Airflow - Building IoT Pipeline ➡️](../module-6/module-6.md)**
