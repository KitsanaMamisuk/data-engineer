# Lab Module 5: Airflow - Orchestration Basics

**🎯 วัตถุประสงค์:**
- ติดตั้ง Apache Airflow ด้วย Docker
- สร้าง DAG แรกของคุณ
- ฝึกใช้งาน Operators ต่างๆ
- เรียนรู้การกำหนด Task Dependencies
- นำทาง Airflow UI และ Monitoring

**⏱️ เวลาที่ใช้:** 2 ชั่วโมง

---

## 📋 Pre-requisites

ก่อนเริ่ม Lab ตรวจสอบว่าคุณมี:
- ✅ Docker Desktop ติดตั้งแล้ว
- ✅ Docker Compose (มาพร้อม Docker Desktop)
- ✅ Terminal/Command Prompt
- ✅ Text Editor (VS Code แนะนำ)
- ✅ เนื้อที่ Hard Disk อย่างน้อย 5 GB

**ตรวจสอบ Docker:**
```bash
docker --version
docker compose version
```

---

## Lab 5.1: Install Airflow via Docker (30 นาที)

### วัตถุประสงค์
ติดตั้ง Apache Airflow ด้วย Docker Compose

### ขั้นตอน

#### Step 1: สร้าง Project Directory

```bash
# สร้าง directory สำหรับ Airflow
mkdir ~/airflow-lab
cd ~/airflow-lab

# สร้าง subdirectories
mkdir -p ./dags ./logs ./plugins ./config
```

**โครงสร้าง:**
```
airflow-lab/
├── dags/       ← เก็บ DAG files
├── logs/       ← Logs
├── plugins/    ← Custom plugins
└── config/     ← Configuration
```

#### Step 2: ดาวน์โหลด docker-compose.yaml

```bash
# ดาวน์โหลด official docker-compose file
curl -LfO 'https://airflow.apache.org/docs/apache-airflow/2.8.0/docker-compose.yaml'

# หรือใช้ wget
# wget 'https://airflow.apache.org/docs/apache-airflow/2.8.0/docker-compose.yaml'
```

#### Step 3: สร้าง .env File

```bash
# สร้าง .env file สำหรับ User ID
echo -e "AIRFLOW_UID=$(id -u)" > .env

# บน Windows ให้ใช้:
# echo AIRFLOW_UID=50000 > .env
```

#### Step 4: Initialize Airflow Database

```bash
# Initialize database และสร้าง Admin user
docker compose up airflow-init

# รอจนเห็นข้อความ:
# airflow-init_1 exited with code 0
```

**Expected Output:**
```
[+] Running 1/0
 ⠿ Container airflow-lab-airflow-init-1  Created
[+] Running 1/1
 ⠿ Container airflow-lab-airflow-init-1  Started
...
Admin user airflow created
airflow-lab-airflow-init-1 exited with code 0
```

#### Step 5: Start Airflow Services

```bash
# Start ทุก services (detached mode)
docker compose up -d

# ตรวจสอบ services ทำงาน
docker compose ps
```

**Expected Output:**
```
NAME                                   STATUS    PORTS
airflow-lab-airflow-scheduler-1        Up        8080/tcp
airflow-lab-airflow-webserver-1        Up        0.0.0.0:8080->8080/tcp
airflow-lab-airflow-worker-1           Up        8080/tcp
airflow-lab-postgres-1                 Up        5432/tcp
airflow-lab-redis-1                    Up        6379/tcp
```

#### Step 6: เข้าใช้งาน Airflow UI

```bash
# เปิด Browser ไปที่:
http://localhost:8080

# Login Credentials:
Username: airflow
Password: airflow
```

**หน้า Login:**
```
┌─────────────────────────────┐
│    Apache Airflow           │
├─────────────────────────────┤
│                             │
│  Username: [airflow]        │
│  Password: [********]       │
│                             │
│  [        Login       ]     │
│                             │
└─────────────────────────────┘
```

#### Step 7: ตรวจสอบ Example DAGs

หลัง Login สำเร็จ คุณจะเห็น:
- 📊 DAGs List
- 🟢 Example DAGs (หลายตัว)
- 📈 Statistics

### ✅ Checkpoint 5.1

ตรวจสอบว่า:
- [ ] Docker services ทำงานทั้งหมด
- [ ] เข้า Airflow UI ได้ที่ http://localhost:8080
- [ ] Login ด้วย airflow/airflow สำเร็จ
- [ ] เห็น Example DAGs ในหน้าแรก

### 🔧 Troubleshooting

**หาก Port 8080 ถูกใช้แล้ว:**
```bash
# แก้ไข docker-compose.yaml
# เปลี่ยน "8080:8080" เป็น "8081:8080"
# แล้ว restart
docker compose down
docker compose up -d
```

**หาก Services ไม่ขึ้น:**
```bash
# ดู logs
docker compose logs airflow-webserver

# Restart services
docker compose down
docker compose up -d
```

---

## Lab 5.2: Create "Hello World" DAG (20 นาที)

### วัตถุประสงค์
สร้าง DAG แรกด้วย PythonOperator

### ขั้นตอน

#### Step 1: สร้างไฟล์ DAG

สร้างไฟล์ `hello_world_dag.py` ใน `dags/` folder:

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime, timedelta

# Default arguments
default_args = {
    'owner': 'data-engineer',
    'depends_on_past': False,
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
}

# Python function
def print_hello():
    print("Hello World from Airflow!")
    print("This is my first DAG!")
    return "Success"

# Create DAG
dag = DAG(
    dag_id='hello_world_dag',
    default_args=default_args,
    description='My first Airflow DAG',
    schedule_interval='@daily',
    start_date=datetime(2025, 1, 1),
    catchup=False,
    tags=['tutorial', 'hello-world'],
)

# Create Task
task_hello = PythonOperator(
    task_id='say_hello',
    python_callable=print_hello,
    dag=dag,
)
```

**บันทึกไฟล์ที่:**
```
~/airflow-lab/dags/hello_world_dag.py
```

#### Step 2: รอ Airflow โหลด DAG

```bash
# Airflow จะสแกน dags/ folder ทุก 30 วินาที
# รอสักครู่แล้ว Refresh Browser

# หรือ restart scheduler เพื่อโหลดเร็วขึ้น:
docker compose restart airflow-scheduler
```

#### Step 3: เปิดใช้งาน DAG

ใน Airflow UI:
1. หา DAG ชื่อ `hello_world_dag`
2. คลิก Toggle (สวิตช์) เพื่อเปิดใช้งาน (เปลี่ยนจาก ⚫ เป็น 🟢)

#### Step 4: Trigger DAG ด้วยมือ

```
1. คลิกปุ่ม "▶️ Trigger DAG" (ขวามือ)
2. ยืนยัน "Trigger"
3. รอ DAG รันเสร็จ (refresh หน้าเว็บ)
```

#### Step 5: ดู Logs

```
1. คลิกชื่อ DAG "hello_world_dag"
2. คลิก Graph View
3. คลิก Task "say_hello"
4. เลือก "Log"
```

**Expected Log Output:**
```
[2025-01-01 12:00:00] INFO - Starting task: say_hello
[2025-01-01 12:00:01] INFO - Executing: print_hello()
[2025-01-01 12:00:01] INFO - Hello World from Airflow!
[2025-01-01 12:00:01] INFO - This is my first DAG!
[2025-01-01 12:00:01] INFO - Task returned: Success
[2025-01-01 12:00:01] INFO - Task exited with return code 0
```

### ✅ Checkpoint 5.2

ตรวจสอบว่า:
- [ ] สร้างไฟล์ `hello_world_dag.py` ใน `dags/` folder
- [ ] เห็น DAG ใน Airflow UI
- [ ] Trigger DAG และรันสำเร็จ (🟢)
- [ ] อ่าน Logs และเห็นข้อความ "Hello World from Airflow!"

---

## Lab 5.3: Multiple Tasks with Dependencies (25 นาที)

### วัตถุประสงค์
สร้าง DAG ที่มีหลาย Tasks และกำหนด Dependencies

### ขั้นตอน

#### Step 1: สร้าง ETL DAG

สร้างไฟล์ `simple_etl_dag.py`:

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.operators.bash import BashOperator
from datetime import datetime, timedelta

# Python functions
def extract_data():
    print("=" * 50)
    print("EXTRACT: Reading data from source...")
    data = [
        {'sensor_id': 'S001', 'temperature': 25.5},
        {'sensor_id': 'S002', 'temperature': 26.0},
        {'sensor_id': 'S003', 'temperature': 24.8},
    ]
    print(f"Extracted {len(data)} records")
    print("=" * 50)
    return data

def transform_data(**context):
    print("=" * 50)
    print("TRANSFORM: Converting Celsius to Fahrenheit...")

    # ดึงข้อมูลจาก previous task (XCom)
    ti = context['ti']
    data = ti.xcom_pull(task_ids='extract')

    transformed = []
    for record in data:
        celsius = record['temperature']
        fahrenheit = (celsius * 9/5) + 32
        transformed.append({
            'sensor_id': record['sensor_id'],
            'celsius': celsius,
            'fahrenheit': fahrenheit
        })

    print(f"Transformed {len(transformed)} records")
    for rec in transformed:
        print(f"  {rec['sensor_id']}: {rec['celsius']}°C = {rec['fahrenheit']}°F")
    print("=" * 50)
    return transformed

def load_data(**context):
    print("=" * 50)
    print("LOAD: Saving data to destination...")

    ti = context['ti']
    data = ti.xcom_pull(task_ids='transform')

    print(f"Loading {len(data)} records to database...")
    print("✅ Data loaded successfully!")
    print("=" * 50)
    return "Load completed"

# Default args
default_args = {
    'owner': 'data-engineer',
    'retries': 2,
    'retry_delay': timedelta(minutes=3),
}

# Create DAG
dag = DAG(
    dag_id='simple_etl_dag',
    default_args=default_args,
    description='Simple ETL pipeline with 3 tasks',
    schedule_interval='@daily',
    start_date=datetime(2025, 1, 1),
    catchup=False,
    tags=['etl', 'tutorial'],
)

# Create Tasks
task_start = BashOperator(
    task_id='start',
    bash_command='echo "Starting ETL Pipeline..."',
    dag=dag,
)

task_extract = PythonOperator(
    task_id='extract',
    python_callable=extract_data,
    dag=dag,
)

task_transform = PythonOperator(
    task_id='transform',
    python_callable=transform_data,
    provide_context=True,
    dag=dag,
)

task_load = PythonOperator(
    task_id='load',
    python_callable=load_data,
    provide_context=True,
    dag=dag,
)

task_end = BashOperator(
    task_id='end',
    bash_command='echo "ETL Pipeline completed successfully!"',
    dag=dag,
)

# Define Dependencies
task_start >> task_extract >> task_transform >> task_load >> task_end
```

**บันทึกไฟล์ที่:**
```
~/airflow-lab/dags/simple_etl_dag.py
```

#### Step 2: Trigger DAG

1. Refresh Airflow UI
2. หา DAG `simple_etl_dag`
3. เปิดใช้งาน (Toggle on)
4. Trigger DAG

#### Step 3: ดู Graph View

```
1. คลิก DAG "simple_etl_dag"
2. เลือก Graph View
3. สังเกต Task Dependencies:

   ┌───────┐
   │ start │
   └───┬───┘
       │
   ┌───▼───────┐
   │ extract   │
   └───┬───────┘
       │
   ┌───▼──────────┐
   │ transform    │
   └───┬──────────┘
       │
   ┌───▼──────┐
   │  load    │
   └───┬──────┘
       │
   ┌───▼───┐
   │  end  │
   └───────┘
```

#### Step 4: ดู Logs แต่ละ Task

ดู Logs ของ `transform` task:
```
[2025-01-01 12:00:05] INFO - TRANSFORM: Converting Celsius to Fahrenheit...
[2025-01-01 12:00:05] INFO -   S001: 25.5°C = 77.9°F
[2025-01-01 12:00:05] INFO -   S002: 26.0°C = 78.8°F
[2025-01-01 12:00:05] INFO -   S003: 24.8°C = 76.64°F
```

### ✅ Checkpoint 5.3

ตรวจสอบว่า:
- [ ] สร้าง `simple_etl_dag.py` สำเร็จ
- [ ] DAG มี 5 Tasks (start, extract, transform, load, end)
- [ ] Tasks รันตามลำดับที่กำหนด
- [ ] ดู Logs และเห็นข้อมูลถูกแปลงจาก Celsius → Fahrenheit

---

## Lab 5.4: Parallel Tasks (20 นาที)

### วัตถุประสงค์
เรียนรู้การสร้าง Tasks ที่รันพร้อมกัน (Parallel)

### ขั้นตอน

#### Step 1: สร้าง Parallel DAG

สร้างไฟล์ `parallel_tasks_dag.py`:

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime, timedelta
import time

# Python functions
def process_sensor_a():
    print("Processing Sensor A data...")
    time.sleep(5)  # Simulate processing time
    print("✅ Sensor A completed")
    return "Sensor A: 100 records"

def process_sensor_b():
    print("Processing Sensor B data...")
    time.sleep(3)
    print("✅ Sensor B completed")
    return "Sensor B: 150 records"

def process_sensor_c():
    print("Processing Sensor C data...")
    time.sleep(4)
    print("✅ Sensor C completed")
    return "Sensor C: 120 records"

def aggregate_results(**context):
    ti = context['ti']

    # ดึงผลลัพธ์จาก 3 tasks
    result_a = ti.xcom_pull(task_ids='process_a')
    result_b = ti.xcom_pull(task_ids='process_b')
    result_c = ti.xcom_pull(task_ids='process_c')

    print("=" * 50)
    print("AGGREGATING RESULTS:")
    print(f"  - {result_a}")
    print(f"  - {result_b}")
    print(f"  - {result_c}")
    print("  - Total: 370 records")
    print("=" * 50)
    return "Aggregation completed"

# Default args
default_args = {
    'owner': 'data-engineer',
    'retries': 1,
    'retry_delay': timedelta(minutes=2),
}

# Create DAG
dag = DAG(
    dag_id='parallel_tasks_dag',
    default_args=default_args,
    description='DAG with parallel task execution',
    schedule_interval='@hourly',
    start_date=datetime(2025, 1, 1),
    catchup=False,
    tags=['parallel', 'tutorial'],
)

# Create Tasks
task_start = PythonOperator(
    task_id='start',
    python_callable=lambda: print("Starting parallel processing..."),
    dag=dag,
)

task_a = PythonOperator(
    task_id='process_a',
    python_callable=process_sensor_a,
    dag=dag,
)

task_b = PythonOperator(
    task_id='process_b',
    python_callable=process_sensor_b,
    dag=dag,
)

task_c = PythonOperator(
    task_id='process_c',
    python_callable=process_sensor_c,
    dag=dag,
)

task_aggregate = PythonOperator(
    task_id='aggregate',
    python_callable=aggregate_results,
    provide_context=True,
    dag=dag,
)

# Define Dependencies
# start → [a, b, c] → aggregate (a, b, c รันพร้อมกัน)
task_start >> [task_a, task_b, task_c] >> task_aggregate
```

#### Step 2: Trigger และสังเกต

1. Trigger `parallel_tasks_dag`
2. เปิด Graph View
3. สังเกตว่า Tasks A, B, C เปลี่ยนเป็น Running (🔵) พร้อมกัน

**Graph View:**
```
              ┌───────────┐
         ┌──> │process_a  │ ──┐
         │    └───────────┘   │
    ┌────┴───┐                │
    │ start  │                ├──> ┌─────────────┐
    └────┬───┘                │    │ aggregate   │
         │    ┌───────────┐   │    └─────────────┘
         ├──> │process_b  │ ──┤
         │    └───────────┘   │
         │                    │
         │    ┌───────────┐   │
         └──> │process_c  │ ──┘
              └───────────┘
```

#### Step 3: ดู Gantt Chart

1. เปิด Gantt Chart View
2. สังเกตว่า Tasks A, B, C รันทับกัน (แสดง Parallel Execution)

```
Gantt Chart:

start        |██|
process_a       |████████████|
process_b       |████████|
process_c       |██████████|
aggregate                    |██|
           12:00      12:05
```

### ✅ Checkpoint 5.4

ตรวจสอบว่า:
- [ ] สร้าง `parallel_tasks_dag.py` สำเร็จ
- [ ] Tasks A, B, C รันพร้อมกัน
- [ ] Task aggregate รอให้ทั้ง 3 tasks เสร็จก่อน
- [ ] เห็น Parallel execution ใน Gantt Chart

---

## Lab 5.5: Scheduling & Monitoring (15 นาที)

### วัตถุประสงค์
ฝึกกำหนด Schedule และ Monitor DAGs

### ขั้นตอน

#### Step 1: สร้าง Scheduled DAG

สร้างไฟล์ `scheduled_dag.py`:

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime, timedelta

def daily_report():
    now = datetime.now()
    print("=" * 60)
    print(f"📊 DAILY REPORT - {now.strftime('%Y-%m-%d %H:%M:%S')}")
    print("=" * 60)
    print("Execution Summary:")
    print("  - Total Sensors: 10")
    print("  - Records Processed: 14,400")
    print("  - Success Rate: 99.5%")
    print("  - Average Temperature: 25.3°C")
    print("=" * 60)
    return "Report generated"

# Default args
default_args = {
    'owner': 'data-engineer',
    'retries': 1,
}

# DAG with different schedules
dag_daily = DAG(
    dag_id='daily_report_dag',
    default_args=default_args,
    description='Runs daily at 2 AM',
    schedule_interval='0 2 * * *',  # Cron: ทุกวัน 02:00
    start_date=datetime(2025, 1, 1),
    catchup=False,
    tags=['scheduled', 'report'],
)

task_daily = PythonOperator(
    task_id='generate_daily_report',
    python_callable=daily_report,
    dag=dag_daily,
)

# DAG ทุก 5 นาที (เพื่อทดสอบ)
dag_frequent = DAG(
    dag_id='frequent_check_dag',
    default_args=default_args,
    description='Runs every 5 minutes',
    schedule_interval='*/5 * * * *',  # Cron: ทุก 5 นาที
    start_date=datetime(2025, 1, 1),
    catchup=False,
    tags=['scheduled', 'monitoring'],
)

task_frequent = PythonOperator(
    task_id='health_check',
    python_callable=lambda: print("✅ System health check passed"),
    dag=dag_frequent,
)
```

#### Step 2: ทดสอบ Schedule Intervals

**ดู Schedule:**
1. ไปที่ DAGs List
2. ดูคอลัมน์ "Schedule"
3. สังเกต:
   - `daily_report_dag`: `0 2 * * *`
   - `frequent_check_dag`: `*/5 * * * *`

#### Step 3: ดู Tree View

1. คลิก DAG `frequent_check_dag`
2. เปิด Tree View
3. รอ 5-10 นาที
4. Refresh และสังเกต DAG รันอัตโนมัติ

**Tree View:**
```
frequent_check_dag
├─ 2025-01-01 12:00  ✅
├─ 2025-01-01 12:05  ✅
├─ 2025-01-01 12:10  ✅
└─ 2025-01-01 12:15  ⏸️ Running
```

#### Step 4: ฝึกใช้ UI Features

**Pause/Unpause DAG:**
- คลิก Toggle เพื่อหยุดหรือเปิด Scheduling

**Browse → Task Instances:**
- ดู Task Instances ทั้งหมดในระบบ
- Filter by State, DAG, Date

**Browse → DAG Runs:**
- ดู DAG Run history
- สถานะแต่ละ run

**Admin → Connections:**
- ดู Connections ที่กำหนดไว้
- สำหรับเชื่อมต่อ Database, Cloud, APIs

### ✅ Checkpoint 5.5

ตรวจสอบว่า:
- [ ] สร้าง DAGs ที่มี schedule_interval ต่างกัน
- [ ] เข้าใจ Cron Expressions
- [ ] ดู Tree View และเห็น execution history
- [ ] ใช้ Pause/Unpause DAG ได้

---

## 🏆 Challenge Exercise (เพิ่มเติม)

### Challenge 1: IoT Data Processor DAG

สร้าง DAG สำหรับประมวลผลข้อมูล IoT ที่มี:

**Requirements:**
1. Task: `check_file` (FileSensor จำลอง) - เช็คว่ามีไฟล์ไหม
2. Task: `validate_data` - ตรวจสอบข้อมูล
3. Tasks: `process_temperature`, `process_humidity` (Parallel)
4. Task: `send_notification` - แจ้งเตือนเมื่อเสร็จ
5. Schedule: ทุก 30 นาที

**DAG Structure:**
```
check_file → validate_data → [process_temp, process_humidity] → send_notification
```

<details>
<summary>💡 Solution (คลิกเพื่อดู)</summary>

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime, timedelta

def check_file():
    print("✅ Checking for new IoT data files...")
    print("File found: sensor_data_2025_01_01.csv")
    return True

def validate_data():
    print("🔍 Validating data integrity...")
    print("✅ Validation passed: 1440 records")
    return True

def process_temperature():
    print("🌡️ Processing temperature data...")
    print("Average: 25.5°C, Min: 20°C, Max: 30°C")
    return "Temperature processing completed"

def process_humidity():
    print("💧 Processing humidity data...")
    print("Average: 62%, Min: 45%, Max: 80%")
    return "Humidity processing completed"

def send_notification(**context):
    ti = context['ti']
    temp_result = ti.xcom_pull(task_ids='process_temp')
    humidity_result = ti.xcom_pull(task_ids='process_humidity')

    print("📧 Sending notification...")
    print(f"  - {temp_result}")
    print(f"  - {humidity_result}")
    print("✅ Notification sent successfully!")

default_args = {
    'owner': 'data-engineer',
    'retries': 2,
    'retry_delay': timedelta(minutes=5),
}

dag = DAG(
    dag_id='iot_data_processor_dag',
    default_args=default_args,
    description='IoT data processing pipeline',
    schedule_interval='*/30 * * * *',
    start_date=datetime(2025, 1, 1),
    catchup=False,
    tags=['iot', 'challenge'],
)

# Create tasks
task_check = PythonOperator(
    task_id='check_file',
    python_callable=check_file,
    dag=dag,
)

task_validate = PythonOperator(
    task_id='validate_data',
    python_callable=validate_data,
    dag=dag,
)

task_process_temp = PythonOperator(
    task_id='process_temp',
    python_callable=process_temperature,
    dag=dag,
)

task_process_humidity = PythonOperator(
    task_id='process_humidity',
    python_callable=process_humidity,
    dag=dag,
)

task_notify = PythonOperator(
    task_id='send_notification',
    python_callable=send_notification,
    provide_context=True,
    dag=dag,
)

# Define dependencies
task_check >> task_validate >> [task_process_temp, task_process_humidity] >> task_notify
```
</details>

### Challenge 2: Dynamic DAG with Variable Number of Sensors

สร้าง DAG ที่สร้าง Tasks แบบ Dynamic สำหรับหลาย Sensors

**Requirements:**
- รองรับ Sensor IDs แบบ Dynamic (S001, S002, S003, ...)
- สร้าง Task สำหรับแต่ละ Sensor อัตโนมัติ
- Aggregate ผลลัพธ์จากทุก Sensors

<details>
<summary>💡 Solution (คลิกเพื่อดู)</summary>

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime, timedelta

# Sensor IDs (สามารถเปลี่ยนได้)
SENSOR_IDS = ['S001', 'S002', 'S003', 'S004', 'S005']

def process_sensor(sensor_id):
    def _process():
        print(f"📊 Processing data for {sensor_id}...")
        # Simulate processing
        result = f"{sensor_id}: 288 records processed"
        print(f"✅ {result}")
        return result
    return _process

def aggregate_all(**context):
    ti = context['ti']
    print("=" * 60)
    print("📈 AGGREGATING ALL SENSOR DATA")
    print("=" * 60)

    total = 0
    for sensor_id in SENSOR_IDS:
        result = ti.xcom_pull(task_ids=f'process_{sensor_id}')
        print(f"  - {result}")
        total += 288

    print(f"\n  Total Records: {total}")
    print("=" * 60)
    return f"Aggregated {total} records from {len(SENSOR_IDS)} sensors"

default_args = {
    'owner': 'data-engineer',
    'retries': 1,
    'retry_delay': timedelta(minutes=3),
}

dag = DAG(
    dag_id='dynamic_sensor_dag',
    default_args=default_args,
    description='Dynamic DAG for multiple sensors',
    schedule_interval='@hourly',
    start_date=datetime(2025, 1, 1),
    catchup=False,
    tags=['dynamic', 'challenge'],
)

# Start task
task_start = PythonOperator(
    task_id='start',
    python_callable=lambda: print("Starting sensor processing..."),
    dag=dag,
)

# Dynamically create tasks for each sensor
sensor_tasks = []
for sensor_id in SENSOR_IDS:
    task = PythonOperator(
        task_id=f'process_{sensor_id}',
        python_callable=process_sensor(sensor_id),
        dag=dag,
    )
    sensor_tasks.append(task)

# Aggregate task
task_aggregate = PythonOperator(
    task_id='aggregate',
    python_callable=aggregate_all,
    provide_context=True,
    dag=dag,
)

# Define dependencies
# start → [all sensor tasks] → aggregate
task_start >> sensor_tasks >> task_aggregate
```
</details>

---

## 📊 Lab Summary

### สิ่งที่คุณได้เรียนรู้

✅ **Lab 5.1:** ติดตั้ง Airflow ด้วย Docker Compose
✅ **Lab 5.2:** สร้าง "Hello World" DAG แรก
✅ **Lab 5.3:** สร้าง ETL Pipeline ด้วย Multiple Tasks
✅ **Lab 5.4:** ใช้งาน Parallel Tasks
✅ **Lab 5.5:** กำหนด Scheduling และ Monitoring

### Skills Acquired

| Skill | Confidence Level |
|-------|------------------|
| Airflow Installation | ⭐⭐⭐ |
| DAG Creation | ⭐⭐⭐ |
| Using Operators | ⭐⭐⭐ |
| Task Dependencies | ⭐⭐⭐ |
| Scheduling | ⭐⭐ |
| UI Navigation | ⭐⭐⭐ |

---

## 🎯 ถัดไป

คุณพร้อมสำหรับ Module 6 แล้ว!

**👉 [Module 6: Airflow - Building the IoT Pipeline](../../module-6/module-6.md)**

ใน Module 6 คุณจะได้:
- ใช้ FileSensor จริงเพื่อรอไฟล์
- Integrate ETL code จาก Module 2-3
- Upload ข้อมูลไป Cloud Storage
- สร้าง Complete IoT Pipeline

---

## 🛠️ Docker Commands Cheat Sheet

```bash
# Start Airflow
docker compose up -d

# Stop Airflow
docker compose down

# View logs
docker compose logs -f airflow-scheduler
docker compose logs -f airflow-webserver

# Restart services
docker compose restart

# Remove all (including volumes)
docker compose down -v

# Check container status
docker compose ps

# Execute command in container
docker compose exec airflow-scheduler bash
```

---

## 📞 Need Help?

หากติดปัญหา:

1. **DAG ไม่ปรากฏใน UI:**
   - ตรวจสอบไฟล์อยู่ใน `dags/` folder
   - ตรวจสอบ Syntax Error (Browse → Import Errors)
   - Restart scheduler: `docker compose restart airflow-scheduler`

2. **Task ล้มเหลว:**
   - ดู Logs ของ Task
   - ตรวจสอบ Python function มี Error ไหม
   - ลอง Clear Task และรันใหม่

3. **Port Conflict:**
   - แก้ไข port ใน docker-compose.yaml
   - หรือหยุด service ที่ใช้ port 8080

4. **Out of Memory:**
   - ปิด DAGs ที่ไม่ใช้
   - เพิ่ม Memory ให้ Docker Desktop

---

**[⬅️ กลับไป Module 5](../module-5.md)** | **[🏠 กลับไปหน้าหลัก](../../wiki.md)**
