# Lab Module 4: Cloud Integration & Storage

**🎯 วัตถุประสงค์:**
- ออกแบบ Data Partitioning Strategy สำหรับ Time-Series Data
- สร้าง File Naming Convention ที่เป็นมาตรฐาน
- ฝึกจัดระเบียบ Folder Structure แบบ Data Lake
- เขียน Python Script จำลองการจัดการไฟล์แบบ Cloud Storage

**⏱️ เวลาที่ใช้:** 1.5 ชั่วโมง

---

## 📋 Pre-requisites

ก่อนเริ่ม Lab ตรวจสอบว่าคุณมี:
- ✅ Python 3.8+ พร้อม Virtual Environment
- ✅ Pandas และ NumPy ติดตั้งแล้ว
- ✅ เข้าใจ Module 4 เรื่อง Partitioning และ Data Lake

---

## Lab 4.1: Define Time-Series Partitioning Strategy (30 นาที)

### วัตถุประสงค์
ออกแบบ Partitioning Strategy และ File Naming Convention สำหรับข้อมูล IoT Sensor

### ขั้นตอน

#### Step 1: วิเคราะห์ Requirements

สมมติเรามีระบบ IoT ที่มีคุณสมบัติดังนี้:
- มี Sensors จำนวน 100 เครื่อง (S001 - S100)
- แต่ละ Sensor ส่งข้อมูลทุก 5 นาที
- ข้อมูลเก็บใน JSON Format
- ต้องการ Query ข้อมูลตาม Date Range บ่อยๆ
- ต้องการลบข้อมูลเก่ากว่า 90 วันได้ง่าย

**คำถาม:** Partitioning Strategy ใดเหมาะสมที่สุด?

<details>
<summary>💡 คำตอบและเหตุผล (คลิกเพื่อดู)</summary>

**Strategy ที่เหมาะสม: Time-Based Partitioning (year/month/day)**

**เหตุผล:**
1. ✅ Query Pattern: ต้อง Query ตาม Date Range บ่อย
2. ✅ Data Management: ลบข้อมูลเก่าได้ง่าย (ลบทั้ง Partition)
3. ✅ Data Volume: 100 sensors × 12 records/hour × 24 hours = 28,800 records/day ต่อ Partition พอดี
4. ✅ Scalability: เพิ่ม Sensor ใหม่ไม่กระทบ Partition Structure

**ไม่เลือก Sensor-Based Partitioning เพราะ:**
- ❌ ต้อง Scan หลาย Partitions เวลา Query ตาม Date
- ❌ ลบข้อมูลเก่ายาก (ต้องเข้าไปลบในแต่ละ Sensor Partition)
</details>

#### Step 2: ออกแบบ Folder Structure

สร้างไฟล์ `partitioning_design.md` และออกแบบ Structure:

```markdown
# IoT Data Lake - Partitioning Design

## Folder Structure

```
s3://iot-data-lake/
│
├─ raw/
│   └─ sensors/
│       └─ year=2025/
│           └─ month=01/
│               └─ day=01/
│                   ├─ sensor_S001_20250101_000000.json
│                   ├─ sensor_S001_20250101_000500.json
│                   ├─ sensor_S002_20250101_000000.json
│                   └─ ...
│
├─ processed/
│   └─ sensors/
│       └─ year=2025/
│           └─ month=01/
│               └─ day=01/
│                   └─ sensors_20250101.parquet
│
└─ curated/
    └─ daily_summary/
        └─ year=2025/
            └─ month=01/
                └─ summary_20250101.parquet
```

## File Naming Convention

**Format:** `{source}_{entity}_{timestamp}.{extension}`

**Examples:**
- RAW: `sensor_S001_20250101_120530.json`
- PROCESSED: `sensors_20250101.parquet`
- CURATED: `summary_20250101.parquet`

**Components:**
- source: `sensor`, `processed`, `summary`
- entity: `S001`, `S002`, `all`
- timestamp: `YYYYMMDD_HHMMSS` or `YYYYMMDD`
- extension: `.json`, `.parquet`, `.csv`

## Query Examples

### Query 1: ดึงข้อมูลทั้งหมดวันที่ 2025-01-01
```
Path: s3://iot-data-lake/raw/sensors/year=2025/month=01/day=01/
```

### Query 2: ดึงข้อมูลของ Sensor S001 ในเดือน January 2025
```
Path: s3://iot-data-lake/raw/sensors/year=2025/month=01/day=*/
Filter: sensor_S001_*.json
```

### Query 3: ลบข้อมูลเก่ากว่า 90 วัน
```
Delete Partitions: year=2024/month=10/day=01 and older
```
```

#### Step 3: คำนวณ Data Volume

สร้างไฟล์ `calculate_partition_size.py`:

```python
"""
คำนวณขนาดข้อมูลใน Partition แต่ละวัน
"""

# Configuration
NUM_SENSORS = 100
RECORDS_PER_HOUR_PER_SENSOR = 12  # ทุก 5 นาที = 12 records/hour
HOURS_PER_DAY = 24
AVG_RECORD_SIZE_BYTES = 500  # JSON record ~500 bytes

# Calculate
records_per_day_per_sensor = RECORDS_PER_HOUR_PER_SENSOR * HOURS_PER_DAY
total_records_per_day = records_per_day_per_sensor * NUM_SENSORS

# Storage size
total_size_bytes = total_records_per_day * AVG_RECORD_SIZE_BYTES
total_size_mb = total_size_bytes / (1024 * 1024)
total_size_gb = total_size_mb / 1024

print("=== IoT Data Volume Calculation ===\n")
print(f"Sensors: {NUM_SENSORS}")
print(f"Records per hour per sensor: {RECORDS_PER_HOUR_PER_SENSOR}")
print(f"Records per day per sensor: {records_per_day_per_sensor}")
print(f"\n--- Daily Partition Size ---")
print(f"Total records per day: {total_records_per_day:,}")
print(f"Total size: {total_size_mb:.2f} MB ({total_size_gb:.4f} GB)")

# Monthly and Yearly estimates
monthly_size_gb = total_size_gb * 30
yearly_size_gb = total_size_gb * 365

print(f"\n--- Estimates ---")
print(f"Monthly storage: {monthly_size_gb:.2f} GB")
print(f"Yearly storage: {yearly_size_gb:.2f} GB")

# Cost estimation (example: S3 Standard ~$0.023/GB/month)
s3_cost_per_gb = 0.023
monthly_cost = monthly_size_gb * s3_cost_per_gb * 30  # 30 days retention
print(f"\n--- Estimated S3 Cost (Standard, 30-day retention) ---")
print(f"Monthly cost: ${monthly_cost:.2f}")
```

รัน Script:

```bash
python calculate_partition_size.py
```

**Expected Output:**
```
=== IoT Data Volume Calculation ===

Sensors: 100
Records per hour per sensor: 12
Records per day per sensor: 288

--- Daily Partition Size ---
Total records per day: 28,800
Total size: 13.73 MB (0.0134 GB)

--- Estimates ---
Monthly storage: 0.40 GB
Yearly storage: 4.89 GB

--- Estimated S3 Cost (Standard, 30-day retention) ---
Monthly cost: $0.28
```

### ✅ Checkpoint 4.1

ตรวจสอบว่า:
- [ ] ออกแบบ Partitioning Strategy ได้ (year/month/day)
- [ ] สร้าง File Naming Convention แล้ว
- [ ] คำนวณ Data Volume และ Storage Cost ได้
- [ ] เข้าใจ Trade-offs ของแต่ละ Strategy

---

## Lab 4.2: Implement Local Data Lake Simulator (40 นาที)

### วัตถุประสงค์
สร้าง Python Script จำลอง Data Lake Structure แบบ Local (ไม่ต้องมี Cloud Account)

### ขั้นตอน

#### Step 1: สร้าง Folder Structure Generator

สร้างไฟล์ `create_datalake_structure.py`:

```python
"""
สร้าง Data Lake Folder Structure แบบ Local
"""
import os
from pathlib import Path
from datetime import datetime

def create_datalake_structure(base_path='./local_datalake'):
    """
    สร้าง Folder Structure สำหรับ Data Lake
    """
    # Define structure
    zones = {
        'raw': ['sensors', 'logs', 'events'],
        'processed': ['sensors_cleaned', 'aggregated'],
        'curated': ['daily_summary', 'sensor_stats', 'ml_features']
    }

    print(f"Creating Data Lake structure at: {base_path}\n")

    for zone, folders in zones.items():
        for folder in folders:
            # สร้าง Partitions สำหรับ 3 วันล่าสุด
            for day in range(1, 4):
                partition_path = Path(base_path) / zone / folder / \
                                 f'year=2025/month=01/day={day:02d}'
                partition_path.mkdir(parents=True, exist_ok=True)
                print(f"✅ Created: {partition_path}")

    # สร้าง metadata folder
    metadata_path = Path(base_path) / '_metadata'
    metadata_path.mkdir(parents=True, exist_ok=True)
    print(f"✅ Created: {metadata_path}")

    print("\n✅ Data Lake structure created successfully!")

if __name__ == '__main__':
    create_datalake_structure()
```

รัน Script:

```bash
python create_datalake_structure.py
```

ตรวจสอบว่า Folder Structure ถูกสร้างขึ้น:

```bash
tree local_datalake -L 4
# หรือบน Windows: dir local_datalake /s
```

**Expected Output:**
```
local_datalake/
├── raw/
│   ├── sensors/
│   │   └── year=2025/
│   │       └── month=01/
│   │           ├── day=01/
│   │           ├── day=02/
│   │           └── day=03/
│   ├── logs/
│   └── events/
├── processed/
│   ├── sensors_cleaned/
│   └── aggregated/
├── curated/
│   ├── daily_summary/
│   ├── sensor_stats/
│   └── ml_features/
└── _metadata/
```

#### Step 2: Generate Sample Sensor Data

สร้างไฟล์ `generate_sensor_data.py`:

```python
"""
Generate Sample IoT Sensor Data with Partitioning
"""
import json
import pandas as pd
import numpy as np
from pathlib import Path
from datetime import datetime, timedelta

def generate_sensor_record(sensor_id, timestamp):
    """
    สร้าง Sensor Record 1 รายการ
    """
    return {
        'timestamp': timestamp.isoformat(),
        'sensor_id': sensor_id,
        'temperature': round(25 + np.random.normal(0, 2), 2),
        'humidity': round(60 + np.random.normal(0, 5), 2),
        'pressure': round(1013 + np.random.normal(0, 2), 2),
        'battery_level': round(100 - np.random.uniform(0, 5), 2)
    }

def generate_partitioned_data(base_path='./local_datalake',
                               num_sensors=5,
                               num_days=3):
    """
    Generate ข้อมูล Sensor แบบ Partitioned
    """
    print(f"Generating sensor data for {num_sensors} sensors, {num_days} days\n")

    # Start date
    start_date = datetime(2025, 1, 1, 0, 0, 0)

    for day in range(num_days):
        current_date = start_date + timedelta(days=day)
        partition_path = Path(base_path) / 'raw' / 'sensors' / \
                         f'year={current_date.year}' / \
                         f'month={current_date.month:02d}' / \
                         f'day={current_date.day:02d}'

        # สร้าง records สำหรับ 1 ชั่วโมงแรก (เพื่อประหยัดเวลา)
        for sensor_id in range(1, num_sensors + 1):
            sensor_name = f'S{sensor_id:03d}'
            records = []

            # Generate 12 records (ทุก 5 นาที ใน 1 ชั่วโมง)
            for minute in range(0, 60, 5):
                timestamp = current_date + timedelta(minutes=minute)
                record = generate_sensor_record(sensor_name, timestamp)
                records.append(record)

            # Save to JSON file
            filename = f'sensor_{sensor_name}_{current_date.strftime("%Y%m%d")}_000000.json'
            file_path = partition_path / filename

            with open(file_path, 'w') as f:
                json.dump(records, f, indent=2)

            print(f"✅ Created: {file_path.relative_to(base_path)} ({len(records)} records)")

    print(f"\n✅ Data generation complete!")

def list_partition_contents(base_path='./local_datalake'):
    """
    แสดงรายการไฟล์ใน Partitions
    """
    print("\n=== Partition Contents ===\n")

    raw_path = Path(base_path) / 'raw' / 'sensors'

    for partition_dir in sorted(raw_path.rglob('day=*')):
        files = list(partition_dir.glob('*.json'))
        print(f"📁 {partition_dir.relative_to(base_path)}/")
        print(f"   Files: {len(files)}")

        if files:
            # Show file sizes
            total_size = sum(f.stat().st_size for f in files)
            print(f"   Size: {total_size / 1024:.2f} KB")
        print()

if __name__ == '__main__':
    # Generate data
    generate_partitioned_data(num_sensors=5, num_days=3)

    # List contents
    list_partition_contents()
```

รัน Script:

```bash
python generate_sensor_data.py
```

#### Step 3: Query Simulator

สร้างไฟล์ `query_partitioned_data.py`:

```python
"""
จำลอง Query ข้อมูลจาก Partitioned Data Lake
"""
import json
import pandas as pd
from pathlib import Path
from datetime import datetime

def query_by_date(base_path='./local_datalake', year=2025, month=1, day=1):
    """
    Query ข้อมูลตาม Date Partition
    """
    partition_path = Path(base_path) / 'raw' / 'sensors' / \
                     f'year={year}' / f'month={month:02d}' / f'day={day:02d}'

    print(f"=== Query: Date {year}-{month:02d}-{day:02d} ===\n")
    print(f"Partition: {partition_path}\n")

    if not partition_path.exists():
        print("❌ Partition not found!")
        return None

    # List files
    json_files = list(partition_path.glob('*.json'))
    print(f"Found {len(json_files)} files\n")

    # Load all data
    all_records = []
    for json_file in json_files:
        with open(json_file, 'r') as f:
            records = json.load(f)
            all_records.extend(records)

    # Convert to DataFrame
    df = pd.DataFrame(all_records)
    df['timestamp'] = pd.to_datetime(df['timestamp'])

    print("--- Sample Data ---")
    print(df.head())

    print("\n--- Statistics ---")
    print(f"Total records: {len(df)}")
    print(f"Unique sensors: {df['sensor_id'].nunique()}")
    print(f"Avg temperature: {df['temperature'].mean():.2f}°C")
    print(f"Avg humidity: {df['humidity'].mean():.2f}%")

    return df

def query_by_sensor(base_path='./local_datalake', sensor_id='S001',
                    year=2025, month=1):
    """
    Query ข้อมูลของ Sensor ใดๆ ในเดือนที่ระบุ
    """
    print(f"=== Query: Sensor {sensor_id} in {year}-{month:02d} ===\n")

    sensors_path = Path(base_path) / 'raw' / 'sensors' / \
                   f'year={year}' / f'month={month:02d}'

    if not sensors_path.exists():
        print("❌ Month partition not found!")
        return None

    # Scan all days in the month
    all_records = []
    for day_dir in sorted(sensors_path.glob('day=*')):
        # Find files for the specific sensor
        sensor_files = list(day_dir.glob(f'sensor_{sensor_id}_*.json'))

        for sensor_file in sensor_files:
            with open(sensor_file, 'r') as f:
                records = json.load(f)
                all_records.extend(records)

    if not all_records:
        print(f"❌ No data found for sensor {sensor_id}")
        return None

    df = pd.DataFrame(all_records)
    df['timestamp'] = pd.to_datetime(df['timestamp'])

    print("--- Sample Data ---")
    print(df.head())

    print("\n--- Statistics ---")
    print(f"Total records: {len(df)}")
    print(f"Date range: {df['timestamp'].min()} to {df['timestamp'].max()}")
    print(f"Avg temperature: {df['temperature'].mean():.2f}°C")

    return df

def compare_partition_scan():
    """
    เปรียบเทียบการ Scan ข้อมูล: With Partitioning vs Without
    """
    print("=== Partitioning Benefits ===\n")

    base_path = Path('./local_datalake/raw/sensors')

    # Count total files
    all_files = list(base_path.rglob('*.json'))
    print(f"Total files in Data Lake: {len(all_files)}")

    # Simulate query for one day
    day1_path = base_path / 'year=2025/month=01/day=01'
    day1_files = list(day1_path.glob('*.json'))

    print(f"\n--- Query: Only 2025-01-01 ---")
    print(f"Files scanned WITH partitioning: {len(day1_files)}")
    print(f"Files scanned WITHOUT partitioning: {len(all_files)}")

    savings = (1 - len(day1_files) / len(all_files)) * 100
    print(f"\n✅ Partition Scan Reduction: {savings:.1f}%")

if __name__ == '__main__':
    # Query 1: By Date
    print("\n" + "="*60)
    df1 = query_by_date(year=2025, month=1, day=1)

    print("\n" + "="*60)
    # Query 2: By Sensor
    df2 = query_by_sensor(sensor_id='S001', year=2025, month=1)

    print("\n" + "="*60)
    # Query 3: Compare scan efficiency
    compare_partition_scan()
```

รัน Script:

```bash
python query_partitioned_data.py
```

### ✅ Checkpoint 4.2

ตรวจสอบว่า:
- [ ] สร้าง Local Data Lake Structure สำเร็จ
- [ ] Generate Partitioned Sensor Data ได้
- [ ] Query ข้อมูลตาม Date Partition ได้
- [ ] Query ข้อมูลตาม Sensor ID ได้
- [ ] เข้าใจประโยชน์ของ Partitioning

---

## Lab 4.3: File Naming & Validation (20 นาที)

### วัตถุประสงค์
สร้าง Utility Functions สำหรับ File Naming และ Validation

### ขั้นตอน

#### Step 1: File Naming Helper

สร้างไฟล์ `file_naming_utils.py`:

```python
"""
Utility functions สำหรับ File Naming Convention
"""
from datetime import datetime
from pathlib import Path
import re

class DataLakeFileNaming:
    """
    Helper class สำหรับ File Naming Convention
    """

    @staticmethod
    def generate_filename(source, entity, timestamp, extension, version=None):
        """
        Generate filename ตาม Convention

        Args:
            source: แหล่งข้อมูล (sensor, api, manual)
            entity: หน่วยข้อมูล (S001, daily_summary)
            timestamp: datetime object หรือ string
            extension: .json, .parquet, .csv
            version: v1, v2 (optional)

        Returns:
            Filename string
        """
        if isinstance(timestamp, datetime):
            ts_str = timestamp.strftime('%Y%m%d_%H%M%S')
        else:
            ts_str = timestamp

        parts = [source, entity, ts_str]

        if version:
            parts.append(version)

        filename = '_'.join(parts) + extension

        return filename

    @staticmethod
    def parse_filename(filename):
        """
        Parse filename และ Extract components
        """
        # Remove extension
        name_without_ext = Path(filename).stem

        # Split by underscore
        parts = name_without_ext.split('_')

        if len(parts) < 3:
            raise ValueError(f"Invalid filename format: {filename}")

        result = {
            'source': parts[0],
            'entity': parts[1],
            'timestamp': parts[2] if len(parts) > 2 else None,
            'version': parts[3] if len(parts) > 3 else None,
            'extension': Path(filename).suffix
        }

        return result

    @staticmethod
    def validate_filename(filename):
        """
        Validate ว่า Filename ถูกต้องตาม Convention หรือไม่
        """
        try:
            parsed = DataLakeFileNaming.parse_filename(filename)

            # Check timestamp format (YYYYMMDD_HHMMSS or YYYYMMDD)
            ts = parsed['timestamp']
            ts_pattern = r'^\d{8}(_\d{6})?$'

            if not re.match(ts_pattern, ts):
                return False, "Invalid timestamp format"

            # Check extension
            valid_extensions = ['.json', '.parquet', '.csv', '.txt']
            if parsed['extension'] not in valid_extensions:
                return False, f"Invalid extension: {parsed['extension']}"

            return True, "Valid filename"

        except Exception as e:
            return False, str(e)

# Test functions
if __name__ == '__main__':
    naming = DataLakeFileNaming()

    print("=== File Naming Convention Tests ===\n")

    # Test 1: Generate filename
    print("--- Test 1: Generate Filename ---")
    filename1 = naming.generate_filename(
        source='sensor',
        entity='S001',
        timestamp=datetime(2025, 1, 1, 12, 30, 45),
        extension='.json'
    )
    print(f"Generated: {filename1}")

    filename2 = naming.generate_filename(
        source='processed',
        entity='daily_summary',
        timestamp='20250101',
        extension='.parquet',
        version='v2'
    )
    print(f"Generated: {filename2}")

    # Test 2: Parse filename
    print("\n--- Test 2: Parse Filename ---")
    parsed = naming.parse_filename(filename1)
    print(f"Parsed: {parsed}")

    # Test 3: Validate filename
    print("\n--- Test 3: Validate Filename ---")

    test_files = [
        'sensor_S001_20250101_120000.json',          # ✅ Valid
        'processed_daily_20250101_v2.parquet',       # ✅ Valid
        'sensor_S001_20250101.csv',                  # ✅ Valid
        'data.json',                                 # ❌ Invalid
        'sensor_S001_2025-01-01.json',               # ❌ Invalid timestamp
        'sensor_S001_20250101_120000.txt',           # ✅ Valid
    ]

    for filename in test_files:
        valid, message = naming.validate_filename(filename)
        status = "✅" if valid else "❌"
        print(f"{status} {filename:45s} -> {message}")
```

รัน Script:

```bash
python file_naming_utils.py
```

#### Step 2: Data Validation

สร้างไฟล์ `data_validation.py`:

```python
"""
Data Validation สำหรับข้อมูลก่อน Upload ไป Data Lake
"""
import pandas as pd
import json
from pathlib import Path

class DataLakeValidator:
    """
    Validate ข้อมูลก่อน Upload ไป Data Lake
    """

    @staticmethod
    def validate_sensor_data(df):
        """
        Validate Sensor DataFrame
        """
        print("=== Data Validation ===\n")

        checks = {}

        # Check 1: Required columns
        required_cols = ['timestamp', 'sensor_id', 'temperature']
        checks['required_columns'] = all(col in df.columns for col in required_cols)

        if not checks['required_columns']:
            missing = [col for col in required_cols if col not in df.columns]
            print(f"❌ Missing columns: {missing}")
        else:
            print(f"✅ Required columns present")

        # Check 2: No duplicates
        checks['no_duplicates'] = not df.duplicated(subset=['timestamp', 'sensor_id']).any()
        dup_count = df.duplicated(subset=['timestamp', 'sensor_id']).sum()

        if checks['no_duplicates']:
            print(f"✅ No duplicate records")
        else:
            print(f"❌ Found {dup_count} duplicate records")

        # Check 3: No nulls in critical columns
        critical_nulls = df[['timestamp', 'sensor_id']].isnull().sum().sum()
        checks['no_critical_nulls'] = critical_nulls == 0

        if checks['no_critical_nulls']:
            print(f"✅ No nulls in critical columns")
        else:
            print(f"❌ Found {critical_nulls} nulls in critical columns")

        # Check 4: Valid temperature range
        if 'temperature' in df.columns:
            valid_temp = df['temperature'].between(-50, 100).all()
            checks['valid_temperature_range'] = valid_temp

            if valid_temp:
                print(f"✅ Temperature values in valid range (-50 to 100°C)")
            else:
                invalid_count = (~df['temperature'].between(-50, 100)).sum()
                print(f"❌ Found {invalid_count} invalid temperature values")

        # Check 5: Timestamp format
        try:
            pd.to_datetime(df['timestamp'])
            checks['valid_timestamp'] = True
            print(f"✅ Timestamp format is valid")
        except:
            checks['valid_timestamp'] = False
            print(f"❌ Invalid timestamp format")

        # Summary
        print(f"\n--- Validation Summary ---")
        passed = sum(checks.values())
        total = len(checks)
        print(f"Passed: {passed}/{total} checks")

        all_passed = all(checks.values())

        if all_passed:
            print("\n✅ All validations passed! Data is ready for upload.")
        else:
            print("\n❌ Validation failed! Please fix errors before uploading.")

        return all_passed, checks

# Test
if __name__ == '__main__':
    validator = DataLakeValidator()

    # Test 1: Valid data
    print("--- Test 1: Valid Data ---")
    valid_data = {
        'timestamp': ['2025-01-01 00:00:00', '2025-01-01 00:05:00', '2025-01-01 00:10:00'],
        'sensor_id': ['S001', 'S001', 'S002'],
        'temperature': [25.5, 26.0, 24.8],
        'humidity': [60, 62, 58]
    }
    df_valid = pd.DataFrame(valid_data)
    validator.validate_sensor_data(df_valid)

    print("\n" + "="*60)

    # Test 2: Invalid data
    print("\n--- Test 2: Invalid Data ---")
    invalid_data = {
        'timestamp': ['2025-01-01 00:00:00', '2025-01-01 00:05:00', '2025-01-01 00:00:00'],
        'sensor_id': ['S001', 'S001', 'S001'],  # Duplicate
        'temperature': [25.5, 150.0, 24.8],     # Invalid temp
        'humidity': [60, None, 58]              # Null value
    }
    df_invalid = pd.DataFrame(invalid_data)
    validator.validate_sensor_data(df_invalid)
```

รัน Script:

```bash
python data_validation.py
```

### ✅ Checkpoint 4.3

ตรวจสอบว่า:
- [ ] สร้าง File Naming Utility สำเร็จ
- [ ] Generate Filename ตาม Convention ได้
- [ ] Parse และ Validate Filename ได้
- [ ] Validate Data Quality ก่อน Upload ได้

---

## 🏆 Challenge Exercise (เพิ่มเติม)

### Challenge 1: Cloud Upload Simulator (Advanced)

สร้าง Python Class ที่จำลองการ Upload ไฟล์ไปยัง Cloud Storage (แต่เก็บที่ Local)

**Requirements:**
- รองรับ Partitioned Upload
- มี Progress Indicator
- Validate ข้อมูลก่อน Upload อัตโนมัติ
- Generate Metadata File

<details>
<summary>💡 Hint (คลิกเพื่อดู)</summary>

```python
class CloudStorageSimulator:
    def __init__(self, base_path):
        self.base_path = Path(base_path)

    def upload_file(self, local_file, destination_key, validate=True):
        """
        Simulate file upload
        """
        # 1. Validate data if needed
        # 2. Create destination directory
        # 3. Copy file
        # 4. Generate metadata
        # 5. Return upload status
        pass

    def list_objects(self, prefix):
        """
        List objects with prefix
        """
        pass

    def download_file(self, key, local_path):
        """
        Simulate download
        """
        pass
```
</details>

### Challenge 2: Partition Lifecycle Manager

สร้าง Script ที่จัดการ Partition Lifecycle:
- ลบ Partitions เก่ากว่า N วัน
- ย้าย Partitions เก่าไป Archive Zone
- Generate Report แสดง Storage Usage

### Challenge 3: Query Performance Benchmark

เปรียบเทียบ Query Performance ระหว่าง:
- Partitioned vs Non-partitioned Data
- Different Partition Strategies (by time vs by entity)
- Different File Formats (JSON vs Parquet)

---

## 📊 Lab Summary

### สิ่งที่คุณได้เรียนรู้

✅ **Lab 4.1:** ออกแบบ Time-Series Partitioning Strategy (year/month/day)
✅ **Lab 4.2:** สร้าง Local Data Lake Simulator และ Query ข้อมูล
✅ **Lab 4.3:** สร้าง File Naming Convention และ Data Validation

### Skills Acquired

| Skill | Confidence Level |
|-------|------------------|
| Partitioning Strategy Design | ⭐⭐⭐ |
| File Organization | ⭐⭐⭐ |
| Data Validation | ⭐⭐⭐ |
| Local Data Lake Simulation | ⭐⭐ |

### Key Files Created

```
lab_module_4/
├── partitioning_design.md
├── calculate_partition_size.py
├── create_datalake_structure.py
├── generate_sensor_data.py
├── query_partitioned_data.py
├── file_naming_utils.py
├── data_validation.py
└── local_datalake/
    ├── raw/
    ├── processed/
    └── curated/
```

---

## 🎯 ถัดไป

คุณพร้อมสำหรับ Module 5 แล้ว!

**👉 [Module 5: Airflow - Orchestration Basics](../../module-5/module-5.md)**

ใน Module ถัดไป คุณจะได้เรียนรู้:
- Apache Airflow Architecture
- DAG (Directed Acyclic Graph)
- Tasks และ Dependencies
- Airflow Installation

---

## 💡 Real-World Application

ในโปรเจคจริง Partitioning Strategy ที่ดีจะช่วย:
- **ลดค่าใช้จ่าย:** Query เฉพาะ Partition ที่ต้องการ ลด Data Scan
- **เพิ่มความเร็ว:** Parallel Processing หลาย Partitions พร้อมกัน
- **ง่ายต่อการจัดการ:** ลบ/Archive ข้อมูลเก่าได้ง่าย
- **Compliance:** เก็บข้อมูลตาม Policy (เช่น GDPR: ลบข้อมูลหลัง 90 วัน)

---

## 📞 Need Help?

หากติดปัญหา:
1. ตรวจสอบ Folder Structure ว่าถูกต้องตาม Convention
2. ตรวจสอบ File Naming ว่าตรงกับ Pattern
3. ตรวจสอบ Data Validation ว่าผ่านทุก Checks

---

**[⬅️ กลับไป Module 4](../module-4.md)** | **[🏠 กลับไปหน้าหลัก](../../wiki.md)**
