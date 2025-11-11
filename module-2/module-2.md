# Module 2: ETL - Data Loading & Cleansing

**🎯 วัตถุประสงค์การเรียนรู้:**
- เข้าใจกระบวนการ ETL (Extract, Transform, Load)
- โหลดข้อมูล CSV ขนาดใหญ่แบบ Optimized
- ใช้ dtype optimization เพื่อประหยัด Memory
- จัดการ Missing Values ด้วยเทคนิคต่างๆ (Ffill, Bfill, Mean, Median)
- ตรวจสอบ Data Integrity และคุณภาพข้อมูล

**⏱️ เวลาที่ใช้:** 3.5 ชั่วโมง (ทฤษฎี 2 ชม. + Lab 1.5 ชม.)

---

## 📚 สารบัญ

1. [ETL Pipeline Overview](#1-etl-pipeline-overview)
2. [Data Loading - Optimized CSV Reading](#2-data-loading---optimized-csv-reading)
3. [dtype Optimization & Memory Management](#3-dtype-optimization--memory-management)
4. [Handling Missing Values](#4-handling-missing-values)
5. [Data Integrity Checks](#5-data-integrity-checks)
6. [Labs & Practical Exercises](./labs/lab-module-2.md)

---

## 1. ETL Pipeline Overview

### 1.1 ETL คืออะไร?

**ETL** = Extract, Transform, Load - กระบวนการพื้นฐานของ Data Engineering

```
┌─────────────────────────────────────────────────────┐
│               ETL Pipeline Process                  │
└─────────────────────────────────────────────────────┘

┌──────────┐       ┌──────────┐       ┌──────────┐
│ EXTRACT  │  ───▶ │TRANSFORM │  ───▶ │  LOAD    │
└──────────┘       └──────────┘       └──────────┘
    │                   │                   │
    │                   │                   │
    ▼                   ▼                   ▼
  ┌─────┐           ┌─────┐           ┌─────┐
  │Data │           │Clean│           │Store│
  │Source│          │+Valid│          │Target│
  └─────┘           └─────┘           └─────┘

Examples:
  CSV File         - Remove NULL      Database
  API              - Type Convert     Data Lake
  Database         - Validate Range   Data Warehouse
  Sensor           - Calculate        Cloud Storage
```

### 1.2 แต่ละ Stage ของ ETL

#### **EXTRACT (ดึงข้อมูล)**
- อ่านข้อมูลจาก Source (CSV, Database, API, Sensor)
- เลือกเฉพาะข้อมูลที่ต้องการ
- **ใน Module นี้:** โหลด CSV แบบ Optimized

#### **TRANSFORM (แปลงข้อมูล)**
- ทำความสะอาดข้อมูล (Cleansing)
- จัดการ Missing Values
- แปลง Data Types
- สร้าง Features ใหม่
- **ใน Module นี้:** Cleansing + Missing Value Handling

#### **LOAD (โหลดเข้าระบบ)**
- บันทึกข้อมูลลง Target System
- ตรวจสอบ Data Integrity
- **ใน Module ถัดไป:** Transformation & Storage

### 1.3 ETL สำหรับ IoT Sensor Data

```
IoT ETL Workflow Example:

┌──────────────────┐
│  sensor_raw.csv  │  ← Raw sensor data (50 MB)
└────────┬─────────┘
         │ EXTRACT
         ▼
┌──────────────────────────────┐
│  df = pd.read_csv(...)       │
│  - Specify dtype             │  ← Memory Optimization
│  - Parse dates               │
└────────┬─────────────────────┘
         │ TRANSFORM
         ▼
┌──────────────────────────────┐
│  Data Cleansing              │
│  - Handle Missing Values     │  ← Ffill, Mean, Drop
│  - Remove Outliers           │
│  - Validate Ranges           │
└────────┬─────────────────────┘
         │ LOAD
         ▼
┌──────────────────────────────┐
│  df.to_parquet(...)          │
│  df.to_sql(...)              │  ← Store to Target
└──────────────────────────────┘
```

### 1.4 ทำไม ETL สำคัญ?

✅ **Data Quality:** ข้อมูลสะอาด พร้อมใช้งาน
✅ **Performance:** Optimized Loading ลด Memory และเวลา
✅ **Reliability:** ตรวจสอบ Integrity ก่อนนำไปใช้
✅ **Scalability:** รองรับข้อมูลขนาดใหญ่

---

## 2. Data Loading - Optimized CSV Reading

### 2.1 การโหลด CSV แบบปกติ vs Optimized

#### 2.1.1 วิธีปกติ (ไม่ Optimize)

```python
import pandas as pd

# โหลดแบบปกติ - Pandas จะเดา dtype เอง
df = pd.read_csv('sensor_data.csv')

print(df.info(memory_usage='deep'))
```

**ปัญหา:**
- 📈 ใช้ Memory มากเกินจำเป็น
- ⏱️ ช้า (ต้องเดา dtype)
- ❌ ไม่เหมาะกับข้อมูลขนาดใหญ่

#### 2.1.2 วิธี Optimized (กำหนด dtype)

```python
import pandas as pd

# กำหนด dtype ตั้งแต่ตอนโหลด
dtype_spec = {
    'sensor_id': 'category',      # จำกัดค่า → category
    'temperature': 'float32',     # ไม่ต้องการความละเอียดสูง → float32
    'humidity': 'int16',          # ค่า 0-100 → int16
    'status': 'category'          # จำกัดค่า (OK/ERROR) → category
}

df = pd.read_csv(
    'sensor_data.csv',
    dtype=dtype_spec,
    parse_dates=['timestamp']     # แปลง timestamp เป็น datetime
)

print(df.info(memory_usage='deep'))
```

**ข้อดี:**
- ✅ ลด Memory ได้ 50-80%
- ✅ เร็วขึ้น
- ✅ ควบคุม Data Type ได้

### 2.2 เปรียบเทียบ Memory Usage

```python
import pandas as pd
import numpy as np

# สร้างข้อมูลตัวอย่าง
data = {
    'sensor_id': ['S001', 'S002', 'S001'] * 10000,
    'temperature': np.random.uniform(20, 30, 30000),
    'humidity': np.random.randint(40, 80, 30000)
}

# แบบปกติ
df_normal = pd.DataFrame(data)
memory_normal = df_normal.memory_usage(deep=True).sum() / 1024**2  # MB

# แบบ Optimized
df_optimized = pd.DataFrame(data)
df_optimized['sensor_id'] = df_optimized['sensor_id'].astype('category')
df_optimized['temperature'] = df_optimized['temperature'].astype('float32')
df_optimized['humidity'] = df_optimized['humidity'].astype('int16')
memory_optimized = df_optimized.memory_usage(deep=True).sum() / 1024**2  # MB

# คำนวณ Saving
saving_percent = ((memory_normal - memory_optimized) / memory_normal) * 100

print(f"Memory (Normal):    {memory_normal:.2f} MB")
print(f"Memory (Optimized): {memory_optimized:.2f} MB")
print(f"Saving:             {saving_percent:.1f}%")
```

**Expected Output:**
```
Memory (Normal):    2.29 MB
Memory (Optimized): 0.52 MB
Saving:             77.3%
```

### 2.3 การเลือก dtype ที่เหมาะสม

| Data Type | Range | Memory | Use Case |
|-----------|-------|--------|----------|
| **int8** | -128 to 127 | 1 byte | สถานะ, Flag (0/1) |
| **int16** | -32,768 to 32,767 | 2 bytes | Humidity (0-100), Small counts |
| **int32** | -2B to 2B | 4 bytes | IDs, Large counts |
| **int64** | -9E18 to 9E18 | 8 bytes | Default (ใหญ่เกินไป) |
| **float32** | ~7 decimals | 4 bytes | Temperature, Sensor values |
| **float64** | ~15 decimals | 8 bytes | Default (ละเอียดเกินไป) |
| **category** | Varies | Optimized | Repeated strings (sensor_id, status) |

### 2.4 Best Practices สำหรับการโหลด CSV

```python
import pandas as pd

# ✅ Recommended Approach
df = pd.read_csv(
    'sensor_large.csv',

    # 1. กำหนด dtype
    dtype={
        'sensor_id': 'category',
        'temperature': 'float32',
        'humidity': 'int16',
        'pressure': 'float32',
        'status': 'category'
    },

    # 2. Parse datetime columns
    parse_dates=['timestamp'],

    # 3. Set index (ถ้าต้องการ)
    # index_col='timestamp',

    # 4. เลือกเฉพาะ columns ที่ต้องการ
    usecols=['timestamp', 'sensor_id', 'temperature', 'humidity'],

    # 5. จัดการ Missing Values ตอนโหลด (ถ้ารู้แน่ชัด)
    # na_values=['NA', 'null', ''],

    # 6. ใช้ chunksize สำหรับไฟล์ใหญ่มาก
    # chunksize=10000
)

print(f"Loaded {len(df):,} rows")
print(f"Memory: {df.memory_usage(deep=True).sum() / 1024**2:.2f} MB")
```

---

## 3. dtype Optimization & Memory Management

### 3.1 การวัด Memory Usage

```python
import pandas as pd

def memory_usage_mb(df):
    """คำนวณ Memory ที่ DataFrame ใช้ (MB)"""
    return df.memory_usage(deep=True).sum() / 1024**2

# โหลดข้อมูล
df = pd.read_csv('sensor_data.csv')

# แสดง Memory แต่ละ Column
print(df.memory_usage(deep=True))
print(f"\nTotal: {memory_usage_mb(df):.2f} MB")
```

**Output:**
```
Index          128
sensor_id    24000
temperature  24000
humidity     24000
dtype: int64

Total: 2.29 MB
```

### 3.2 Automatic dtype Optimization

```python
def optimize_dtypes(df):
    """
    Optimize DataFrame dtypes automatically

    Returns:
        Optimized DataFrame และรายงาน Memory Saving
    """
    memory_before = df.memory_usage(deep=True).sum() / 1024**2

    # Optimize ตาม Column Type
    for col in df.columns:
        col_type = df[col].dtype

        # 1. Optimize Integer Columns
        if col_type == 'int64':
            c_min = df[col].min()
            c_max = df[col].max()

            if c_min > np.iinfo(np.int8).min and c_max < np.iinfo(np.int8).max:
                df[col] = df[col].astype(np.int8)
            elif c_min > np.iinfo(np.int16).min and c_max < np.iinfo(np.int16).max:
                df[col] = df[col].astype(np.int16)
            elif c_min > np.iinfo(np.int32).min and c_max < np.iinfo(np.int32).max:
                df[col] = df[col].astype(np.int32)

        # 2. Optimize Float Columns
        elif col_type == 'float64':
            df[col] = df[col].astype(np.float32)

        # 3. Convert Object to Category (ถ้าค่าซ้ำกันเยอะ)
        elif col_type == 'object':
            num_unique = df[col].nunique()
            num_total = len(df[col])

            # ถ้าค่าซ้ำกันมากกว่า 50% → ใช้ category
            if num_unique / num_total < 0.5:
                df[col] = df[col].astype('category')

    memory_after = df.memory_usage(deep=True).sum() / 1024**2
    saving = ((memory_before - memory_after) / memory_before) * 100

    print(f"Memory Before: {memory_before:.2f} MB")
    print(f"Memory After:  {memory_after:.2f} MB")
    print(f"Saving:        {saving:.1f}%")

    return df

# ใช้งาน
df_optimized = optimize_dtypes(df)
```

### 3.3 Memory Profiling Workflow

```
Memory Optimization Process:

┌─────────────────────┐
│ 1. Load Data        │
│    (without dtype)  │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ 2. Check Memory     │
│    df.info()        │
│    .memory_usage()  │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ 3. Analyze Columns  │
│    - Value ranges   │
│    - Unique counts  │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ 4. Choose dtype     │
│    int8/16/32       │
│    float32          │
│    category         │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ 5. Reload with      │
│    dtype spec       │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ 6. Verify Saving    │
│    Compare memory   │
└─────────────────────┘
```

### 3.4 Category dtype - เมื่อไหร่ควรใช้?

**Category dtype** เหมาะสำหรับ Columns ที่มีค่าซ้ำกันเยอะ

```python
import pandas as pd

# สร้างข้อมูลตัวอย่าง
sensor_ids = ['S001', 'S002', 'S003', 'S004', 'S005'] * 10000  # 50,000 rows

# แบบ object (string)
df_object = pd.DataFrame({'sensor_id': sensor_ids})
memory_object = df_object.memory_usage(deep=True).sum() / 1024

# แบบ category
df_category = pd.DataFrame({'sensor_id': pd.Categorical(sensor_ids)})
memory_category = df_category.memory_usage(deep=True).sum() / 1024

print(f"Memory (object):   {memory_object:.2f} KB")
print(f"Memory (category): {memory_category:.2f} KB")
print(f"Saving: {((memory_object - memory_category) / memory_object * 100):.1f}%")
```

**Output:**
```
Memory (object):   2929.69 KB
Memory (category): 49.84 KB
Saving: 98.3%
```

**💡 แนะนำใช้ category เมื่อ:**
- ค่าซ้ำกันมากกว่า 50% (เช่น sensor_id, status, location)
- จำนวนค่า unique น้อย (< 50% ของทั้งหมด)

---

## 4. Handling Missing Values

### 4.1 Missing Values คืออะไร?

**Missing Values** = ข้อมูลที่ขาดหายไป แสดงด้วย `NaN`, `None`, `NULL`

**สาเหตุ:**
- 🔌 Sensor ขัดข้อง
- 📡 Network ขาดหาย
- 💾 Storage Error
- 🐛 Software Bug

### 4.2 ตรวจสอบ Missing Values

```python
import pandas as pd
import numpy as np

# สร้างข้อมูลตัวอย่างที่มี Missing Values
data = {
    'timestamp': pd.date_range('2025-01-01', periods=10, freq='5min'),
    'sensor_id': ['S001'] * 10,
    'temperature': [25.5, np.nan, 26.0, 24.8, np.nan, 25.2, 26.5, np.nan, 25.0, 24.5],
    'humidity': [60, 62, np.nan, 58, 61, np.nan, 63, 59, np.nan, 60]
}
df = pd.DataFrame(data)

# 1. นับจำนวน Missing Values แต่ละ Column
print("Missing Values Count:")
print(df.isnull().sum())

# 2. เปอร์เซ็นต์ Missing
print("\nMissing Values Percentage:")
print((df.isnull().sum() / len(df) * 100).round(2))

# 3. แสดงแถวที่มี Missing Values
print("\nRows with Missing Values:")
print(df[df.isnull().any(axis=1)])

# 4. ตรวจสอบ Missing Pattern
import matplotlib.pyplot as plt
import seaborn as sns

# Visualize Missing Pattern (สำหรับข้อมูลใหญ่)
# sns.heatmap(df.isnull(), cbar=False, cmap='viridis')
```

**Output:**
```
Missing Values Count:
timestamp      0
sensor_id      0
temperature    3
humidity       3
dtype: int64

Missing Values Percentage:
timestamp      0.0
sensor_id      0.0
temperature   30.0
humidity      30.0
dtype: float64
```

### 4.3 เทคนิคการจัดการ Missing Values

```
Missing Value Handling Strategies:

┌─────────────────────────────────────┐
│    1. DROP (ลบทิ้ง)                 │
├─────────────────────────────────────┤
│  ✅ ใช้เมื่อ: Missing < 5%          │
│  ❌ ข้อเสีย: สูญเสียข้อมูล           │
└─────────────────────────────────────┘

┌─────────────────────────────────────┐
│    2. FILL (เติมค่า)                │
├─────────────────────────────────────┤
│  a) Forward Fill (ffill)            │
│     ใช้ค่าก่อนหน้ามาเติม             │
│  b) Backward Fill (bfill)           │
│     ใช้ค่าถัดไปมาเติม                │
│  c) Mean/Median/Mode                │
│     ใช้ค่าสถิติเติม                  │
│  d) Interpolate                     │
│     คำนวณค่าระหว่าง                  │
└─────────────────────────────────────┘

┌─────────────────────────────────────┐
│    3. FLAG (ทำเครื่องหมาย)          │
├─────────────────────────────────────┤
│  สร้าง Column ใหม่บอกว่า Missing   │
│  เพื่อใช้ใน Machine Learning        │
└─────────────────────────────────────┘
```

#### 4.3.1 Method 1: Drop Missing Values

```python
# ลบแถวที่มี Missing Values
df_dropped = df.dropna()
print(f"Original: {len(df)} rows")
print(f"After Drop: {len(df_dropped)} rows")

# ลบเฉพาะ Column ที่มี Missing > 50%
threshold = len(df) * 0.5
df_cleaned = df.dropna(thresh=threshold, axis=1)
```

#### 4.3.2 Method 2: Forward Fill (Ffill)

**เหมาะสำหรับ Time-Series:** ใช้ค่าก่อนหน้ามาเติม

```python
# Forward Fill - เติมด้วยค่าก่อนหน้า
df_ffill = df.copy()
df_ffill['temperature'] = df_ffill['temperature'].fillna(method='ffill')
df_ffill['humidity'] = df_ffill['humidity'].fillna(method='ffill')

print(df_ffill)
```

**Before:**
```
   timestamp  temperature  humidity
0  00:00:00         25.5      60.0
1  00:05:00          NaN      62.0
2  00:10:00         26.0       NaN
```

**After Ffill:**
```
   timestamp  temperature  humidity
0  00:00:00         25.5      60.0
1  00:05:00         25.5      62.0  ← ใช้ 25.5 จากบนมา
2  00:10:00         26.0      62.0  ← ใช้ 62 จากบนมา
```

#### 4.3.3 Method 3: Mean/Median Fill

**เหมาะสำหรับ:** ข้อมูลที่ไม่มี Time Dependency

```python
# Mean Fill - เติมด้วยค่าเฉลี่ย
df_mean = df.copy()
df_mean['temperature'] = df_mean['temperature'].fillna(df_mean['temperature'].mean())
df_mean['humidity'] = df_mean['humidity'].fillna(df_mean['humidity'].mean())

print(f"Temperature Mean: {df['temperature'].mean():.2f}")
print(df_mean)

# Median Fill - เติมด้วย Median (ดีกว่าถ้ามี Outliers)
df_median = df.copy()
df_median['temperature'] = df_median['temperature'].fillna(df_median['temperature'].median())
```

#### 4.3.4 Method 4: Interpolation

**เหมาะสำหรับ Time-Series:** คำนวณค่าระหว่าง

```python
# Linear Interpolation
df_interpolated = df.copy()
df_interpolated['temperature'] = df_interpolated['temperature'].interpolate(method='linear')
df_interpolated['humidity'] = df_interpolated['humidity'].interpolate(method='linear')

print(df_interpolated)
```

**Before:**
```
   temperature
0         25.5
1          NaN
2         26.0
```

**After Interpolation:**
```
   temperature
0         25.5
1         25.75  ← (25.5 + 26.0) / 2
2         26.0
```

### 4.4 เปรียบเทียบ Methods

| Method | Use Case | Pros | Cons |
|--------|----------|------|------|
| **Drop** | Missing < 5% | ข้อมูลสะอาด 100% | สูญเสียข้อมูล |
| **Ffill** | Time-Series Sequential | รักษาแนวโน้ม | ไม่เหมาะถ้า Missing นาน |
| **Mean** | Non-temporal | ง่าย, รวดเร็ว | ไม่สะท้อนแนวโน้ม |
| **Median** | Data มี Outliers | Robust to outliers | ไม่สะท้อนแนวโน้ม |
| **Interpolate** | Time-Series | คำนวณค่าระหว่าง | ซับซ้อนกว่า |

### 4.5 Best Practices

```python
def handle_missing_values(df, strategy='auto'):
    """
    จัดการ Missing Values อัตโนมัติ

    Parameters:
        df: DataFrame
        strategy: 'auto', 'drop', 'ffill', 'mean'

    Returns:
        DataFrame ที่จัดการ Missing Values แล้ว
    """
    df_clean = df.copy()

    # ตรวจสอบ Missing Values
    missing_percent = (df_clean.isnull().sum() / len(df_clean) * 100)

    print("Missing Values Report:")
    print(missing_percent[missing_percent > 0])

    if strategy == 'auto':
        # ถ้า Missing < 5% → Drop
        # ถ้า Time-Series → Ffill
        # ถ้าไม่ใช่ → Mean

        for col in df_clean.columns:
            missing_pct = missing_percent[col]

            if missing_pct == 0:
                continue
            elif missing_pct < 5:
                print(f"{col}: Drop (Missing {missing_pct:.1f}%)")
                df_clean = df_clean.dropna(subset=[col])
            elif pd.api.types.is_datetime64_any_dtype(df_clean.index):
                print(f"{col}: Forward Fill (Time-Series)")
                df_clean[col] = df_clean[col].fillna(method='ffill')
            else:
                print(f"{col}: Mean Fill")
                df_clean[col] = df_clean[col].fillna(df_clean[col].mean())

    elif strategy == 'drop':
        df_clean = df_clean.dropna()
    elif strategy == 'ffill':
        df_clean = df_clean.fillna(method='ffill')
    elif strategy == 'mean':
        df_clean = df_clean.fillna(df_clean.mean())

    print(f"\nOriginal: {len(df)} rows")
    print(f"Cleaned: {len(df_clean)} rows")

    return df_clean

# ใช้งาน
df_cleaned = handle_missing_values(df, strategy='auto')
```

---

## 5. Data Integrity Checks

### 5.1 Data Integrity คืออะไร?

**Data Integrity** = ความถูกต้องและความสมบูรณ์ของข้อมูล

**ตรวจสอบ:**
- ✅ Data Types ถูกต้อง
- ✅ Ranges ตรงตามที่กำหนด
- ✅ ไม่มี Duplicates ที่ไม่ควรมี
- ✅ Relationships ถูกต้อง

### 5.2 ประเภทของ Data Integrity Checks

```
Data Integrity Check Types:

┌──────────────────────────────────┐
│ 1. Type Check                    │
│    ตรวจสอบ Data Type             │
├──────────────────────────────────┤
│  - temperature → float           │
│  - timestamp → datetime          │
│  - sensor_id → string/category   │
└──────────────────────────────────┘

┌──────────────────────────────────┐
│ 2. Range Check                   │
│    ตรวจสอบช่วงค่า                │
├──────────────────────────────────┤
│  - temperature: -50°C to 100°C   │
│  - humidity: 0% to 100%          │
│  - pressure: 900 to 1100 hPa     │
└──────────────────────────────────┘

┌──────────────────────────────────┐
│ 3. Null Check                    │
│    ตรวจสอบค่า Missing            │
├──────────────────────────────────┤
│  - Critical columns ต้องไม่ NULL │
│  - timestamp, sensor_id          │
└──────────────────────────────────┘

┌──────────────────────────────────┐
│ 4. Duplicate Check               │
│    ตรวจสอบข้อมูลซ้ำ              │
├──────────────────────────────────┤
│  - (timestamp, sensor_id) unique │
└──────────────────────────────────┘

┌──────────────────────────────────┐
│ 5. Consistency Check             │
│    ตรวจสอบความสอดคล้อง           │
├──────────────────────────────────┤
│  - Timestamp sequence            │
│  - Related values logic          │
└──────────────────────────────────┘
```

### 5.3 การตรวจสอบ Data Types

```python
import pandas as pd

def check_data_types(df, expected_types):
    """
    ตรวจสอบว่า Data Types ตรงกับที่กำหนดไหม

    Parameters:
        df: DataFrame
        expected_types: dict ของ column: expected_type

    Returns:
        dict ของ validation results
    """
    results = {}

    for col, expected_type in expected_types.items():
        if col not in df.columns:
            results[col] = {'status': 'MISSING', 'message': 'Column not found'}
            continue

        actual_type = df[col].dtype

        # ตรวจสอบ type
        if expected_type == 'datetime':
            is_valid = pd.api.types.is_datetime64_any_dtype(df[col])
        elif expected_type == 'numeric':
            is_valid = pd.api.types.is_numeric_dtype(df[col])
        elif expected_type == 'category':
            is_valid = df[col].dtype.name == 'category'
        else:
            is_valid = str(actual_type) == expected_type

        results[col] = {
            'status': 'PASS' if is_valid else 'FAIL',
            'expected': expected_type,
            'actual': str(actual_type)
        }

    return results

# ใช้งาน
expected = {
    'timestamp': 'datetime',
    'sensor_id': 'category',
    'temperature': 'float32',
    'humidity': 'int16'
}

type_check = check_data_types(df, expected)
for col, result in type_check.items():
    print(f"{col}: {result['status']} - Expected: {result['expected']}, Actual: {result['actual']}")
```

### 5.4 การตรวจสอบ Ranges

```python
def check_ranges(df, range_rules):
    """
    ตรวจสอบว่าค่าอยู่ในช่วงที่กำหนดไหม

    Parameters:
        df: DataFrame
        range_rules: dict ของ column: (min, max)

    Returns:
        DataFrame ของแถวที่ละเมิด rules
    """
    violations = {}

    for col, (min_val, max_val) in range_rules.items():
        if col not in df.columns:
            continue

        # หาแถวที่อยู่นอกช่วง
        out_of_range = df[
            (df[col] < min_val) | (df[col] > max_val)
        ]

        if len(out_of_range) > 0:
            violations[col] = {
                'count': len(out_of_range),
                'expected_range': (min_val, max_val),
                'actual_range': (df[col].min(), df[col].max()),
                'violations': out_of_range
            }
            print(f"❌ {col}: {len(out_of_range)} violations")
            print(f"   Expected: [{min_val}, {max_val}]")
            print(f"   Found: [{df[col].min():.2f}, {df[col].max():.2f}]")
        else:
            print(f"✅ {col}: All values in range [{min_val}, {max_val}]")

    return violations

# ใช้งาน
range_rules = {
    'temperature': (-50, 100),   # °C
    'humidity': (0, 100),        # %
    'pressure': (900, 1100)      # hPa
}

violations = check_ranges(df, range_rules)
```

### 5.5 การตรวจสอบ Duplicates

```python
def check_duplicates(df, subset=None, keep='first'):
    """
    ตรวจสอบและจัดการข้อมูลซ้ำ

    Parameters:
        df: DataFrame
        subset: columns ที่ใช้ตรวจสอบ (ถ้าไม่ระบุ = ทุก column)
        keep: 'first', 'last', False

    Returns:
        DataFrame ที่ไม่มีข้อมูลซ้ำ
    """
    # หา Duplicates
    duplicates = df[df.duplicated(subset=subset, keep=False)]
    num_duplicates = len(duplicates)

    if num_duplicates > 0:
        print(f"❌ Found {num_duplicates} duplicate rows")
        print("\nDuplicate Examples:")
        print(duplicates.head())

        # ลบ Duplicates
        df_cleaned = df.drop_duplicates(subset=subset, keep=keep)
        print(f"\n✅ Removed {len(df) - len(df_cleaned)} duplicates")
        return df_cleaned
    else:
        print("✅ No duplicates found")
        return df

# ใช้งาน - ตรวจสอบ (timestamp, sensor_id) ต้องไม่ซ้ำ
df_no_dup = check_duplicates(df, subset=['timestamp', 'sensor_id'])
```

### 5.6 Data Quality Report

```python
def generate_data_quality_report(df):
    """
    สร้างรายงาน Data Quality ครบถ้วน

    Returns:
        dict ของ quality metrics
    """
    report = {
        'total_rows': len(df),
        'total_columns': len(df.columns),
        'memory_usage_mb': df.memory_usage(deep=True).sum() / 1024**2,
        'missing_values': {},
        'duplicates': 0,
        'data_types': {},
        'numeric_summary': {}
    }

    # 1. Missing Values
    for col in df.columns:
        missing_count = df[col].isnull().sum()
        if missing_count > 0:
            report['missing_values'][col] = {
                'count': int(missing_count),
                'percentage': float((missing_count / len(df) * 100).round(2))
            }

    # 2. Duplicates
    report['duplicates'] = int(df.duplicated().sum())

    # 3. Data Types
    for col in df.columns:
        report['data_types'][col] = str(df[col].dtype)

    # 4. Numeric Summary
    numeric_cols = df.select_dtypes(include=['number']).columns
    for col in numeric_cols:
        report['numeric_summary'][col] = {
            'min': float(df[col].min()),
            'max': float(df[col].max()),
            'mean': float(df[col].mean()),
            'median': float(df[col].median()),
            'std': float(df[col].std())
        }

    return report

# ใช้งาน
import json

quality_report = generate_data_quality_report(df)
print(json.dumps(quality_report, indent=2))
```

### 5.7 Complete Data Validation Pipeline

```python
def validate_sensor_data(df):
    """
    Validation Pipeline สำหรับ IoT Sensor Data

    Returns:
        tuple: (is_valid, cleaned_df, report)
    """
    report = {
        'checks': [],
        'errors': [],
        'warnings': []
    }

    df_clean = df.copy()
    is_valid = True

    # Check 1: Required Columns
    required_cols = ['timestamp', 'sensor_id', 'temperature', 'humidity']
    missing_cols = [col for col in required_cols if col not in df_clean.columns]

    if missing_cols:
        report['errors'].append(f"Missing columns: {missing_cols}")
        is_valid = False
        return is_valid, df_clean, report

    report['checks'].append("✅ All required columns present")

    # Check 2: Data Types
    if not pd.api.types.is_datetime64_any_dtype(df_clean['timestamp']):
        report['warnings'].append("timestamp is not datetime type")
        df_clean['timestamp'] = pd.to_datetime(df_clean['timestamp'])

    report['checks'].append("✅ Data types validated")

    # Check 3: Missing Values
    critical_cols = ['timestamp', 'sensor_id']
    for col in critical_cols:
        missing = df_clean[col].isnull().sum()
        if missing > 0:
            report['errors'].append(f"{col} has {missing} NULL values")
            is_valid = False

    report['checks'].append("✅ Critical columns checked")

    # Check 4: Value Ranges
    temp_out = df_clean[(df_clean['temperature'] < -50) | (df_clean['temperature'] > 100)]
    if len(temp_out) > 0:
        report['warnings'].append(f"Temperature out of range: {len(temp_out)} rows")

    hum_out = df_clean[(df_clean['humidity'] < 0) | (df_clean['humidity'] > 100)]
    if len(hum_out) > 0:
        report['warnings'].append(f"Humidity out of range: {len(hum_out)} rows")

    report['checks'].append("✅ Value ranges checked")

    # Check 5: Duplicates
    dup_count = df_clean.duplicated(subset=['timestamp', 'sensor_id']).sum()
    if dup_count > 0:
        report['warnings'].append(f"Found {dup_count} duplicate records")
        df_clean = df_clean.drop_duplicates(subset=['timestamp', 'sensor_id'])

    report['checks'].append("✅ Duplicates checked")

    return is_valid, df_clean, report

# ใช้งาน
is_valid, df_validated, validation_report = validate_sensor_data(df)

print("Validation Report:")
for check in validation_report['checks']:
    print(check)

if validation_report['errors']:
    print("\nErrors:")
    for error in validation_report['errors']:
        print(f"  ❌ {error}")

if validation_report['warnings']:
    print("\nWarnings:")
    for warning in validation_report['warnings']:
        print(f"  ⚠️  {warning}")

if is_valid:
    print("\n✅ Data validation PASSED")
else:
    print("\n❌ Data validation FAILED")
```

---

## 6. สรุป Module 2

### 6.1 Key Takeaways

✅ **ETL Pipeline:** Extract → Transform → Load กระบวนการพื้นฐานของ Data Engineering
✅ **Optimized Loading:** ใช้ dtype specification ลด Memory ได้ 50-80%
✅ **Missing Values:** เลือก Strategy ที่เหมาะสม (Drop, Ffill, Mean, Interpolate)
✅ **Data Integrity:** ตรวจสอบ Types, Ranges, Nulls, Duplicates ก่อนใช้งาน
✅ **Automation:** สร้าง Functions เพื่อ Validate และ Clean ข้อมูลอัตโนมัติ

### 6.2 Memory Optimization Summary

```
Memory Optimization Techniques:

Before:
  int64    → 8 bytes
  float64  → 8 bytes
  object   → Varies (high)

After:
  int8/16  → 1-2 bytes    ✅ 75-87.5% saving
  float32  → 4 bytes      ✅ 50% saving
  category → Optimized    ✅ 90%+ saving

Total Saving: 50-80% 🎉
```

### 6.3 Missing Value Strategy Decision Tree

```
Missing Value Decision:

              Missing Values?
                    │
        ┌───────────┴───────────┐
        │                       │
    Missing < 5%           Missing > 5%
        │                       │
      DROP                 Time-Series?
                          │           │
                         Yes         No
                          │           │
                      FFILL/     MEAN/MEDIAN
                   INTERPOLATE
```

### 6.4 Skills ที่ได้จาก Module นี้

| Skill | Level |
|-------|-------|
| ETL Pipeline Concepts | ⭐⭐⭐ |
| Optimized CSV Loading | ⭐⭐⭐ |
| dtype Optimization | ⭐⭐⭐⭐ |
| Missing Value Handling | ⭐⭐⭐⭐ |
| Data Validation | ⭐⭐⭐ |
| Memory Profiling | ⭐⭐ |

### 6.5 เตรียมพร้อมสำหรับ Module 3

ใน Module ถัดไป คุณจะได้เรียนรู้:
- **Feature Engineering:** สร้าง Rolling Average, Time-based Features
- **Performance Benchmarking:** Pandas vs NumPy
- **Chunking:** จัดการไฟล์ขนาดใหญ่มากๆ
- **Data Transformation Patterns**

---

## 📝 Challenge Questions

ก่อนไป Module 3 ลองตอบคำถามเหล่านี้:

1. **Memory Optimization:** ถ้ามีข้อมูล humidity 0-100 ควรใช้ dtype อะไร? คำนวณ Memory Saving
   ```python
   # 100,000 rows
   # Original: int64
   # Optimized: ?
   # Saving = ?%
   ```

2. **Missing Values:** Sensor ขาดข้อมูล 2 ชั่วโมง (24 records) ควรใช้วิธีไหน? ทำไม?
   - a) Drop
   - b) Ffill
   - c) Mean
   - d) Interpolate

3. **Data Validation:** อะไรคือ Critical Columns ที่ต้องไม่มี NULL ใน IoT Data?

4. **Range Check:** Temperature sensor ควรมี valid range เท่าไหร่? Humidity?

5. **Category dtype:** ถ้ามี sensor_id 1000 ค่า ซ้ำกัน 40% ควรใช้ category ไหม?

---

## 🎯 ถัดไป: Labs & Practical Exercises

พร้อมทำ Lab แล้วหรือยัง?

**👉 [เริ่มทำ Lab Module 2](./labs/lab-module-2.md)**

ใน Lab คุณจะได้:
- **Lab 2.1:** โหลด `sensor_large.csv` แบบ Optimized และคำนวณ Memory Saving
- **Lab 2.2:** จัดการ Missing Values ด้วยเทคนิคต่างๆ
- **Lab 2.3:** ตรวจสอบ Data Integrity และสร้าง Quality Report
- **Challenge:** สร้าง Complete ETL Pipeline

**⏱️ เวลา Lab:** 1.5 ชั่วโมง

---

**[⬅️ Module 1: Python Fundamentals](../module-1/module-1.md)** | **[กลับไปที่ Wiki หลัก](../wiki.md)** | **[Module 3: ETL - Transformation & Optimization ➡️](../module-3/module-3.md)**
