# Module 4: Cloud Integration & Storage

**🎯 วัตถุประสงค์การเรียนรู้:**
- เข้าใจความแตกต่างระหว่าง Cloud Storage และ On-Premise Storage
- ทำความรู้จักกับ Data Lake Architecture
- เรียนรู้ Object Storage (S3, GCS, Azure Blob)
- ออกแบบ Data Partitioning Strategy สำหรับ Time-Series Data
- ใช้ Cloud SDK พื้นฐาน (boto3, google-cloud-storage)

**⏱️ เวลาที่ใช้:** 3.5 ชั่วโมง (ทฤษฎี 2 ชม. + Lab 1.5 ชม.)

---

## 📚 สารบัญ

1. [Cloud Storage vs On-Premise](#1-cloud-storage-vs-on-premise)
2. [Data Lake Architecture](#2-data-lake-architecture)
3. [Object Storage Concepts](#3-object-storage-concepts)
4. [Data Partitioning Strategy](#4-data-partitioning-strategy)
5. [Cloud SDK Overview](#5-cloud-sdk-overview)
6. [Data Organization Best Practices](#6-data-organization-best-practices)
7. [Labs & Practical Exercises](./labs/lab-module-4.md)

---

## 1. Cloud Storage vs On-Premise

### 1.1 ทำไมต้องใช้ Cloud Storage?

ในยุคของ Big Data และ IoT การเก็บข้อมูลขนาดใหญ่บน On-Premise Server มีข้อจำกัดหลายประการ Cloud Storage จึงกลายเป็นทางเลือกหลักสำหรับ Data Engineering

**ข้อดีของ Cloud Storage:**

✅ **Scalability** - ขยาย Storage ได้ไม่จำกัดตามความต้องการ
✅ **Cost-Effective** - จ่ายตามการใช้งาน (Pay-as-you-go)
✅ **High Availability** - มี Redundancy และ Backup อัตโนมัติ
✅ **Accessibility** - เข้าถึงได้จากทุกที่ผ่าน Internet
✅ **No Maintenance** - ไม่ต้องดูแล Hardware และ Infrastructure

### 1.2 เปรียบเทียบ Cloud Storage vs On-Premise

```
┌───────────────────────────────────────────────────────────────┐
│              Cloud Storage vs On-Premise                      │
├───────────────────────────────────────────────────────────────┤
│                                                               │
│  ON-PREMISE STORAGE              CLOUD STORAGE                │
│  ┌─────────────────┐             ┌─────────────────┐         │
│  │  Data Center    │             │   AWS S3        │         │
│  │  ┌───────────┐  │             │   GCS           │         │
│  │  │  Server   │  │             │   Azure Blob    │         │
│  │  │  Rack     │  │             │                 │         │
│  │  │  [Disks]  │  │             │  ∞ Scalable     │         │
│  │  └───────────┘  │             │  99.99% Uptime  │         │
│  │                 │             │  Global CDN     │         │
│  └─────────────────┘             └─────────────────┘         │
│                                                               │
│  ❌ Fixed Capacity               ✅ Unlimited Scale           │
│  ❌ High Upfront Cost            ✅ Pay-as-you-go             │
│  ❌ Maintenance Required         ✅ Fully Managed             │
│  ❌ Single Location              ✅ Multi-Region               │
│  ✅ Full Control                 ❌ Vendor Lock-in            │
│  ✅ Low Latency (Local)          ❌ Network Dependency        │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

### 1.3 Use Cases สำหรับ Cloud Storage

| Use Case | เหมาะกับ Cloud Storage? | เหตุผล |
|----------|------------------------|--------|
| **IoT Sensor Data** | ✅ ใช่ | ข้อมูลเพิ่มขึ้นอย่างรวดเร็ว ต้องการ Scalability |
| **Log Files** | ✅ ใช่ | ข้อมูลมหาศาล ต้องการ Cost-Effective Storage |
| **Data Lake/Archive** | ✅ ใช่ | เก็บข้อมูลระยะยาว ไม่ค่อยเข้าถึง |
| **Real-time Trading** | ❌ ไม่ | ต้องการ Low Latency สูงมาก |
| **Sensitive Government Data** | ❓ ขึ้นอยู่กับ | ต้องพิจารณา Security และ Compliance |

---

## 2. Data Lake Architecture

### 2.1 Data Lake คืออะไร?

**Data Lake** = Central Repository ที่เก็บข้อมูลทุกชนิด (Structured, Semi-structured, Unstructured) ในรูปแบบ Raw Format

**ความแตกต่างจาก Data Warehouse:**

| Aspect | Data Lake | Data Warehouse |
|--------|-----------|----------------|
| **Data Type** | Raw, Unstructured, Semi-structured | Processed, Structured |
| **Schema** | Schema-on-Read | Schema-on-Write |
| **Users** | Data Scientists, ML Engineers | Business Analysts |
| **Cost** | ถูกกว่า (Object Storage) | แพงกว่า (Database) |
| **Agility** | ยืดหยุ่นสูง | ต้องออกแบบ Schema ก่อน |

### 2.2 Data Lake Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                 Data Lake Architecture                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  DATA SOURCES                                               │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                 │
│  │   IoT    │  │   Logs   │  │   Apps   │                 │
│  │ Sensors  │  │  Server  │  │   APIs   │                 │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘                 │
│       │             │             │                         │
│       └─────────────┴─────────────┘                         │
│                     │                                       │
│                     ▼                                       │
│  ┌─────────────────────────────────────────────┐           │
│  │          INGESTION LAYER                    │           │
│  │  - Streaming (Kafka, Kinesis)               │           │
│  │  - Batch (Airflow, ETL Jobs)                │           │
│  └─────────────────┬───────────────────────────┘           │
│                    │                                        │
│                    ▼                                        │
│  ┌─────────────────────────────────────────────┐           │
│  │          STORAGE LAYER (Data Lake)          │           │
│  │                                             │           │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  │           │
│  │  │   RAW    │  │ PROCESSED│  │ CURATED  │  │           │
│  │  │  Zone    │  │   Zone   │  │   Zone   │  │           │
│  │  │          │  │          │  │          │  │           │
│  │  │ Raw JSON │→ │ Parquet  │→ │ Analytics│  │           │
│  │  │ Raw CSV  │  │ Cleaned  │  │  Ready   │  │           │
│  │  └──────────┘  └──────────┘  └──────────┘  │           │
│  │        S3 / GCS / Azure Blob Storage        │           │
│  └─────────────────┬───────────────────────────┘           │
│                    │                                        │
│                    ▼                                        │
│  ┌─────────────────────────────────────────────┐           │
│  │         PROCESSING LAYER                    │           │
│  │  - Spark, Pandas                            │           │
│  │  - Data Transformation                      │           │
│  │  - Feature Engineering                      │           │
│  └─────────────────┬───────────────────────────┘           │
│                    │                                        │
│                    ▼                                        │
│  ┌─────────────────────────────────────────────┐           │
│  │         CONSUMPTION LAYER                   │           │
│  │  - BI Tools (Tableau, Power BI)             │           │
│  │  - ML Models                                │           │
│  │  - Data APIs                                │           │
│  └─────────────────────────────────────────────┘           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 2.3 Data Lake Zones (3-Tier Architecture)

#### Zone 1: RAW Zone (Bronze)
- **วัตถุประสงค์:** เก็บข้อมูล Raw ที่ยังไม่ผ่านการ Process
- **Format:** ตามที่ได้รับมา (JSON, CSV, Log Files)
- **Retention:** เก็บถาวร (Immutable)
- **Example:** `s3://datalake/raw/sensor_data/2025/01/01/sensor_001.json`

#### Zone 2: PROCESSED Zone (Silver)
- **วัตถุประสงค์:** ข้อมูลที่ผ่านการ Clean และ Transform แล้ว
- **Format:** Columnar Format (Parquet, ORC) เพื่อ Query ได้เร็ว
- **Retention:** เก็บตาม Policy
- **Example:** `s3://datalake/processed/sensor_data/year=2025/month=01/day=01/`

#### Zone 3: CURATED Zone (Gold)
- **วัตถุประสงค์:** ข้อมูลพร้อมใช้งาน (Analytics-Ready)
- **Format:** Aggregated, Denormalized
- **Retention:** เก็บตามความต้องการทางธุรกิจ
- **Example:** `s3://datalake/curated/daily_sensor_summary/2025/01/`

---

## 3. Object Storage Concepts

### 3.1 Object Storage คืออะไร?

**Object Storage** เป็นสถาปัตยกรรมการเก็บข้อมูลที่เก็บข้อมูลในรูปแบบ Objects แทนที่จะเป็น Files (File System) หรือ Blocks (Block Storage)

**Object ประกอบด้วย:**
1. **Data** - เนื้อหาของไฟล์
2. **Metadata** - ข้อมูลเกี่ยวกับไฟล์ (Size, Type, Timestamp)
3. **Unique Identifier (Object Key)** - ชื่อเฉพาะสำหรับเข้าถึง Object

### 3.2 Cloud Object Storage Providers

```
┌─────────────────────────────────────────────────────────────┐
│            Cloud Object Storage Services                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  AWS                   GCP                   AZURE          │
│  ┌─────────────┐      ┌─────────────┐      ┌─────────────┐ │
│  │  Amazon S3  │      │     GCS     │      │  Blob       │ │
│  │  (Simple    │      │  (Google    │      │  Storage    │ │
│  │   Storage   │      │   Cloud     │      │             │ │
│  │   Service)  │      │   Storage)  │      │             │ │
│  ├─────────────┤      ├─────────────┤      ├─────────────┤ │
│  │ Buckets     │      │ Buckets     │      │ Containers  │ │
│  │ ├─ Objects  │      │ ├─ Objects  │      │ ├─ Blobs    │ │
│  │ └─ Keys     │      │ └─ Keys     │      │ └─ Keys     │ │
│  ├─────────────┤      ├─────────────┤      ├─────────────┤ │
│  │ S3 Standard │      │ Standard    │      │ Hot         │ │
│  │ S3 IA       │      │ Nearline    │      │ Cool        │ │
│  │ Glacier     │      │ Coldline    │      │ Archive     │ │
│  └─────────────┘      └─────────────┘      └─────────────┘ │
│                                                             │
│  Market Leader         AI/ML Focus          Enterprise     │
│  Most Mature           Cheap Bandwidth      Microsoft      │
│                                              Integration    │
└─────────────────────────────────────────────────────────────┘
```

### 3.3 Object Storage Structure

```
Bucket/Container (Top-level namespace)
│
├─ data/
│  ├─ raw/
│  │  ├─ sensors/
│  │  │  ├─ 2025/
│  │  │  │  ├─ 01/
│  │  │  │  │  ├─ 01/
│  │  │  │  │  │  ├─ sensor_001_20250101_0000.json
│  │  │  │  │  │  ├─ sensor_001_20250101_0005.json
│  │  │  │  │  │  └─ sensor_002_20250101_0000.json
│  │  │  │  │  └─ 02/
│  │  │  │  └─ 02/
│  │  │  └─ 2024/
│  │  └─ logs/
│  └─ processed/
└─ models/

Object Key Example:
data/raw/sensors/2025/01/01/sensor_001_20250101_0000.json
```

**💡 สำคัญ:** Object Storage ไม่มี Folder จริงๆ แต่ใช้ `/` ใน Object Key เพื่อจำลอง Hierarchical Structure

### 3.4 Storage Classes (Tiers)

Cloud Providers มี Storage Classes หลายแบบให้เลือก ตาม Access Pattern:

| Storage Class | Use Case | Retrieval Time | Cost |
|---------------|----------|----------------|------|
| **Standard/Hot** | Frequently accessed | Instant | $$$ |
| **Infrequent Access (IA)** | Monthly access | Instant | $$ |
| **Archive/Glacier** | Rarely accessed | Minutes-Hours | $ |

**ตัวอย่าง Strategy:**
- **RAW Zone:** S3 Standard (ต้องเข้าถึงบ่อยสำหรับ ETL)
- **PROCESSED Zone (Old Data):** S3 IA (เข้าถึงนาน ๆ ครั้ง)
- **Archive (1 year+):** Glacier (เก็บเผื่อไว้ตาม Compliance)

---

## 4. Data Partitioning Strategy

### 4.1 ทำไมต้อง Partition ข้อมูล?

การ Partition คือการแบ่งข้อมูลออกเป็นส่วนย่อยๆ ตาม Criteria ที่กำหนด (เช่น วันที่, Region, Sensor ID)

**ข้อดีของ Partitioning:**

✅ **Query Performance** - อ่านเฉพาะ Partition ที่ต้องการ (ไม่ต้อง Scan ทั้งหมด)
✅ **Cost Reduction** - ลดค่าใช้จ่าย Data Transfer และ Compute
✅ **Data Management** - ลบ/Backup Partition ง่าย
✅ **Parallel Processing** - ประมวลผลหลาย Partition พร้อมกันได้

### 4.2 Partitioning Strategies สำหรับ Time-Series Data

```
┌─────────────────────────────────────────────────────────────┐
│         Time-Series Partitioning Strategies                 │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. TIME-BASED PARTITIONING (ตาม Timestamp)                │
│                                                             │
│     /data/sensors/                                          │
│         ├─ year=2025/                                       │
│         │   ├─ month=01/                                    │
│         │   │   ├─ day=01/                                  │
│         │   │   │   ├─ sensor_001.parquet                   │
│         │   │   │   └─ sensor_002.parquet                   │
│         │   │   ├─ day=02/                                  │
│         │   │   └─ day=03/                                  │
│         │   └─ month=02/                                    │
│         └─ year=2024/                                       │
│                                                             │
│     ✅ เหมาะสำหรับ: Time-Series Analysis, Historical Data  │
│     ✅ Query Pattern: "ดึงข้อมูลวันที่ 2025-01-01"         │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  2. ENTITY-BASED PARTITIONING (ตาม Entity เช่น Sensor)     │
│                                                             │
│     /data/sensors/                                          │
│         ├─ sensor_id=S001/                                  │
│         │   ├─ 2025-01-01.parquet                           │
│         │   ├─ 2025-01-02.parquet                           │
│         │   └─ 2025-01-03.parquet                           │
│         ├─ sensor_id=S002/                                  │
│         └─ sensor_id=S003/                                  │
│                                                             │
│     ✅ เหมาะสำหรับ: Per-Sensor Analysis                    │
│     ✅ Query Pattern: "ดึงข้อมูลทั้งหมดของ Sensor S001"    │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  3. HYBRID PARTITIONING (รวมทั้ง Time + Entity)            │
│                                                             │
│     /data/sensors/                                          │
│         ├─ year=2025/                                       │
│         │   ├─ month=01/                                    │
│         │   │   ├─ day=01/                                  │
│         │   │   │   ├─ sensor_id=S001/                      │
│         │   │   │   │   └─ data.parquet                     │
│         │   │   │   ├─ sensor_id=S002/                      │
│         │   │   │   └─ sensor_id=S003/                      │
│         │   │   └─ day=02/                                  │
│         │   └─ month=02/                                    │
│         └─ year=2024/                                       │
│                                                             │
│     ✅ เหมาะสำหรับ: Complex Queries                        │
│     ✅ Query Pattern: "ดึงข้อมูล S001 ช่วง 2025-01"        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 4.3 Recommended Partitioning Strategy สำหรับ IoT Data

สำหรับโปรเจค IoT Pipeline เราแนะนำ **Time-Based Partitioning** เป็นหลัก:

```
Recommended Structure:

s3://iot-data-lake/
    ├─ raw/
    │   └─ sensors/
    │       └─ year=2025/
    │           └─ month=01/
    │               └─ day=01/
    │                   ├─ sensor_S001_20250101_000000.json
    │                   ├─ sensor_S001_20250101_000500.json
    │                   └─ sensor_S002_20250101_000000.json
    │
    └─ processed/
        └─ sensors/
            └─ year=2025/
                └─ month=01/
                    └─ day=01/
                        └─ data.parquet
```

**เหตุผล:**
- IoT Data มีลักษณะ Time-Series ต้อง Query ตาม Time Range บ่อย
- ง่ายต่อการ Lifecycle Management (เช่น ลบข้อมูลเก่ากว่า 90 วัน)
- รองรับ Incremental Load (โหลดเฉพาะวันใหม่ๆ)

### 4.4 File Naming Convention

**Best Practice สำหรับตั้งชื่อไฟล์:**

```
Format: {source}_{entity}_{timestamp}_{version}.{extension}

Examples:
✅ sensor_S001_20250101_000000_v1.json
✅ processed_daily_20250101_v2.parquet
✅ raw_temperature_20250101_120000.csv

❌ data.json                    (ไม่บอกรายละเอียด)
❌ sensor1_data_01012025.json   (Format วันที่ไม่ Standard)
❌ output_final_final_v3.csv    (ไม่มีมาตรฐาน)
```

**Naming Convention Components:**

| Component | Description | Example |
|-----------|-------------|---------|
| **Source** | แหล่งที่มาของข้อมูล | `sensor`, `api`, `manual` |
| **Entity** | หน่วยข้อมูล | `S001`, `temperature`, `daily_summary` |
| **Timestamp** | วันเวลา (ISO 8601) | `20250101_120000` หรือ `2025-01-01T12:00:00Z` |
| **Version** | Version (ถ้ามี) | `v1`, `v2` |
| **Extension** | ประเภทไฟล์ | `.json`, `.parquet`, `.csv` |

---

## 5. Cloud SDK Overview

### 5.1 ทำไมต้องใช้ Cloud SDK?

Cloud SDK (Software Development Kit) เป็น Library ที่ให้เราเขียนโค้ดเพื่อจัดการ Cloud Services

**ตัวอย่าง Use Cases:**
- Upload/Download ไฟล์จาก S3, GCS
- List Objects ใน Bucket
- จัดการ Permissions
- Automate Cloud Operations

### 5.2 Popular Cloud SDKs

```
┌─────────────────────────────────────────────────────────────┐
│               Cloud SDKs for Python                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  AWS                                                        │
│  ┌───────────────────────────────────────┐                 │
│  │  boto3 (Python SDK for AWS)           │                 │
│  │                                       │                 │
│  │  import boto3                         │                 │
│  │  s3 = boto3.client('s3')              │                 │
│  │  s3.upload_file('file.csv',           │                 │
│  │                 'bucket', 'key')      │                 │
│  └───────────────────────────────────────┘                 │
│                                                             │
│  GCP                                                        │
│  ┌───────────────────────────────────────┐                 │
│  │  google-cloud-storage                 │                 │
│  │                                       │                 │
│  │  from google.cloud import storage     │                 │
│  │  client = storage.Client()            │                 │
│  │  bucket = client.bucket('bucket-name')│                 │
│  │  blob = bucket.blob('file.csv')       │                 │
│  │  blob.upload_from_filename('file.csv')│                 │
│  └───────────────────────────────────────┘                 │
│                                                             │
│  AZURE                                                      │
│  ┌───────────────────────────────────────┐                 │
│  │  azure-storage-blob                   │                 │
│  │                                       │                 │
│  │  from azure.storage.blob import       │                 │
│  │      BlobServiceClient                │                 │
│  │  client = BlobServiceClient(...)      │                 │
│  │  blob_client = client.get_blob_client │                 │
│  │      (container, blob)                │                 │
│  │  blob_client.upload_blob(data)        │                 │
│  └───────────────────────────────────────┘                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 5.3 AWS boto3 - Conceptual Overview

**boto3** เป็น Python SDK สำหรับ AWS Services (S3, EC2, Lambda, etc.)

#### 5.3.1 การติดตั้ง

```bash
pip install boto3
```

#### 5.3.2 Basic Operations (Conceptual)

**Example 1: Upload ไฟล์ไปยัง S3**

```python
import boto3

# สร้าง S3 Client
s3_client = boto3.client('s3',
                         aws_access_key_id='YOUR_ACCESS_KEY',
                         aws_secret_access_key='YOUR_SECRET_KEY')

# Upload ไฟล์
s3_client.upload_file(
    Filename='local_file.csv',          # ไฟล์ในเครื่อง
    Bucket='my-iot-bucket',             # S3 Bucket Name
    Key='data/raw/sensor_001.csv'       # Object Key ใน S3
)

print("Upload สำเร็จ!")
```

**Example 2: Download ไฟล์จาก S3**

```python
import boto3

s3_client = boto3.client('s3')

# Download ไฟล์
s3_client.download_file(
    Bucket='my-iot-bucket',
    Key='data/raw/sensor_001.csv',
    Filename='downloaded_file.csv'
)

print("Download สำเร็จ!")
```

**Example 3: List Objects ใน Bucket**

```python
import boto3

s3_client = boto3.client('s3')

# List Objects with Prefix (เหมือน List ไฟล์ใน Folder)
response = s3_client.list_objects_v2(
    Bucket='my-iot-bucket',
    Prefix='data/raw/sensors/2025/01/'
)

# แสดงรายชื่อไฟล์
for obj in response.get('Contents', []):
    print(f"- {obj['Key']} (Size: {obj['Size']} bytes)")
```

### 5.4 GCP google-cloud-storage - Conceptual Overview

**google-cloud-storage** เป็น Python Library สำหรับ Google Cloud Storage

#### 5.4.1 การติดตั้ง

```bash
pip install google-cloud-storage
```

#### 5.4.2 Basic Operations (Conceptual)

**Example 1: Upload ไฟล์ไปยัง GCS**

```python
from google.cloud import storage

# สร้าง Storage Client
client = storage.Client(project='your-project-id')

# เลือก Bucket
bucket = client.bucket('my-iot-bucket')

# Upload ไฟล์
blob = bucket.blob('data/raw/sensors/sensor_001.csv')
blob.upload_from_filename('local_file.csv')

print("Upload สำเร็จ!")
```

**Example 2: Download ไฟล์จาก GCS**

```python
from google.cloud import storage

client = storage.Client()
bucket = client.bucket('my-iot-bucket')

# Download ไฟล์
blob = bucket.blob('data/raw/sensors/sensor_001.csv')
blob.download_to_filename('downloaded_file.csv')

print("Download สำเร็จ!")
```

### 5.5 เปรียบเทียบ boto3 vs google-cloud-storage

| Operation | boto3 (AWS S3) | google-cloud-storage (GCS) |
|-----------|----------------|----------------------------|
| **Client** | `boto3.client('s3')` | `storage.Client()` |
| **Upload** | `s3_client.upload_file()` | `blob.upload_from_filename()` |
| **Download** | `s3_client.download_file()` | `blob.download_to_filename()` |
| **List** | `s3_client.list_objects_v2()` | `bucket.list_blobs()` |

**💡 Note:** ใน Module นี้เราจะเรียนรู้แค่ Conceptual Overview เท่านั้น ใน Production จริงต้อง Setup Credentials และ Permissions ด้วย

---

## 6. Data Organization Best Practices

### 6.1 Folder Structure Best Practices

**แนะนำ 3-Tier Structure:**

```
s3://iot-data-lake/
│
├─ raw/                              # RAW Zone (Bronze)
│   ├─ sensors/
│   │   └─ year=2025/
│   │       └─ month=01/
│   │           └─ day=01/
│   │               └─ *.json
│   ├─ logs/
│   └─ events/
│
├─ processed/                        # PROCESSED Zone (Silver)
│   ├─ sensors_cleaned/
│   │   └─ year=2025/
│   │       └─ month=01/
│   │           └─ day=01/
│   │               └─ *.parquet
│   └─ aggregated/
│
├─ curated/                          # CURATED Zone (Gold)
│   ├─ daily_summary/
│   ├─ sensor_stats/
│   └─ ml_features/
│
├─ models/                           # ML Models
│   └─ temperature_prediction/
│       └─ v1.0/
│
└─ archive/                          # Archived Data
    └─ year=2024/
```

### 6.2 Data Quality Checks

ก่อน Upload ข้อมูลไป Cloud ควรตรวจสอบ:

```python
import pandas as pd

def validate_sensor_data(df):
    """
    Validate sensor data before uploading to cloud
    """
    checks = {
        'has_timestamp': 'timestamp' in df.columns,
        'has_sensor_id': 'sensor_id' in df.columns,
        'no_duplicates': df.duplicated().sum() == 0,
        'no_nulls_critical': df[['timestamp', 'sensor_id']].isnull().sum().sum() == 0,
        'valid_temp_range': df['temperature'].between(-50, 100).all()
    }

    passed = all(checks.values())

    if not passed:
        print("❌ Validation Failed:")
        for check, result in checks.items():
            if not result:
                print(f"  - {check}: FAILED")
    else:
        print("✅ All validation checks passed!")

    return passed

# Example Usage
df = pd.read_csv('sensor_data.csv')
if validate_sensor_data(df):
    # Upload to cloud
    print("Ready to upload!")
```

### 6.3 Metadata Management

เก็บ Metadata เพื่อช่วยในการจัดการข้อมูล:

**ตัวอย่าง Metadata File (`_metadata.json`):**

```json
{
  "dataset_name": "iot_sensor_data",
  "partition": "year=2025/month=01/day=01",
  "created_at": "2025-01-01T00:00:00Z",
  "record_count": 288,
  "file_format": "parquet",
  "schema_version": "v1.0",
  "columns": [
    {"name": "timestamp", "type": "datetime64"},
    {"name": "sensor_id", "type": "string"},
    {"name": "temperature", "type": "float64"},
    {"name": "humidity", "type": "float64"}
  ],
  "quality_checks": {
    "null_count": 0,
    "duplicate_count": 0,
    "validation_passed": true
  }
}
```

### 6.4 Data Lifecycle Management

กำหนด Lifecycle Policy สำหรับข้อมูลแต่ละ Zone:

```
Lifecycle Policy Example:

RAW Zone (Bronze):
├─ 0-30 days:    S3 Standard (Hot)
├─ 31-90 days:   S3 Infrequent Access
└─ 90+ days:     S3 Glacier (Archive)

PROCESSED Zone (Silver):
├─ 0-60 days:    S3 Standard
└─ 60+ days:     S3 Infrequent Access

CURATED Zone (Gold):
└─ Always:       S3 Standard (ต้องเข้าถึงบ่อย)
```

### 6.5 Security Best Practices

✅ **DO:**
- ใช้ IAM Roles และ Policies ควบคุม Access
- Enable Encryption (at rest และ in transit)
- ใช้ Bucket Versioning ป้องกันข้อมูลหาย
- Enable Access Logging
- ใช้ Least Privilege Principle

❌ **DON'T:**
- เก็บ Access Keys ใน Source Code
- ตั้ง Bucket เป็น Public โดยไม่จำเป็น
- ใช้ Root Account Credentials
- ทำ Hardcode Secrets ในโค้ด

---

## 7. สรุป Module 4

### 7.1 Key Takeaways

✅ **Cloud Storage** มีข้อดีกว่า On-Premise ในด้าน Scalability, Cost, Availability
✅ **Data Lake** เป็น Central Repository สำหรับเก็บข้อมูล Raw ทุกชนิด
✅ **Object Storage** (S3, GCS, Azure Blob) เหมาะสำหรับ Big Data และ Data Lake
✅ **Partitioning** ตาม Time (year/month/day) เหมาะสำหรับ Time-Series Data
✅ **File Naming Convention** ต้องมีมาตรฐาน ระบุ Source, Entity, Timestamp
✅ **Cloud SDK** (boto3, google-cloud-storage) ใช้จัดการ Cloud Storage ด้วยโค้ด
✅ **3-Tier Architecture** (Raw, Processed, Curated) ช่วยจัดการข้อมูลได้ดีขึ้น

### 7.2 Skills ที่ได้จาก Module นี้

| Skill | Level |
|-------|-------|
| Cloud Storage Concepts | ⭐⭐⭐ |
| Data Lake Architecture | ⭐⭐⭐ |
| Partitioning Strategy | ⭐⭐⭐ |
| Cloud SDK Basics (Conceptual) | ⭐⭐ |
| Data Organization | ⭐⭐⭐ |

### 7.3 เตรียมพร้อมสำหรับ Module 5

ใน Module ถัดไป คุณจะได้เรียนรู้:
- Apache Airflow Architecture
- DAG (Directed Acyclic Graph)
- Tasks และ Operators
- Airflow UI Navigation
- การติดตั้งและ Setup Airflow

---

## 📝 Challenge Questions

ก่อนไป Module 5 ลองตอบคำถามเหล่านี้:

1. **Cloud vs On-Premise:** ข้อมูล IoT ขนาดใหญ่ ทำไมถึงเหมาะกับ Cloud Storage?
2. **Data Lake Zones:** ข้อมูล RAW, PROCESSED, CURATED แตกต่างกันอย่างไร?
3. **Partitioning:** IoT Sensor ส่งข้อมูลทุก 5 นาที จะ Partition ตาม year/month/day หรือ sensor_id ดีกว่า? ทำไม?
4. **File Naming:** ชื่อไฟล์ `data.json` ดีหรือไม่? ควรตั้งชื่อว่าอะไรดีกว่า?
5. **Storage Class:** ข้อมูลเก่ากว่า 90 วันควรเก็บใน Storage Class ใด?

---

## 🎯 ถัดไป: Labs & Practical Exercises

พร้อมทำ Lab แล้วหรือยัง?

**👉 [เริ่มทำ Lab Module 4](./labs/lab-module-4.md)**

ใน Lab คุณจะได้:
- ออกแบบ Partitioning Strategy สำหรับข้อมูล IoT
- สร้าง File Naming Convention
- ฝึกจัดระเบียบ Folder Structure
- เขียน Python Script จำลองการ Upload ไปยัง Cloud (Local Simulation)

---

**[⬅️ Module 3: ETL - Transformation & Optimization](../module-3/module-3.md)** | **[Module 5: Airflow - Orchestration Basics ➡️](../module-5/module-5.md)**
