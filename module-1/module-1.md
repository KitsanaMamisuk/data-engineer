# Module 1: Python Fundamentals & Environment

**🎯 วัตถุประสงค์การเรียนรู้:**
- เข้าใจ Data Structures สำคัญใน Python (Pandas & NumPy)
- ตั้งค่า Virtual Environment สำหรับโปรเจค
- ทำความรู้จักกับ Time-Series Data
- เตรียมพร้อมสำหรับการทำงานกับข้อมูล IoT

**⏱️ เวลาที่ใช้:** 3 ชั่วโมง (ทฤษฎี 2 ชม. + Lab 1 ชม.)

---

## 📚 สารบัญ

1. [Python สำหรับ Data Engineering](#1-python-สำหรับ-data-engineering)
2. [Data Structures: Pandas & NumPy](#2-data-structures-pandas--numpy)
3. [Virtual Environment Setup](#3-virtual-environment-setup)
4. [Time-Series Data Introduction](#4-time-series-data-introduction)
5. [Labs & Practical Exercises](./labs/lab-module-1.md)

---

## 1. Python สำหรับ Data Engineering

### 1.1 ทำไมต้องใช้ Python?

Python เป็นภาษายอดนิยมสำหรับ Data Engineering เพราะ:

✅ **Rich Ecosystem** - Libraries มากมายสำหรับ Data Processing
✅ **Readable & Maintainable** - Syntax อ่านง่าย เหมาะกับทีม
✅ **Community Support** - มีชุมชนใหญ่ รองรับการแก้ปัญหา
✅ **Integration** - เชื่อมต่อกับเครื่องมืออื่นได้ง่าย

### 1.2 Python Version สำหรับ Data Engineering

```bash
# ตรวจสอบ Python version
python --version  # ควรเป็น 3.8 ขึ้นไป

# แนะนำ:
Python 3.8+  (รองรับ Type Hints, Performance Improvements)
```

### 1.3 ภาพรวม Python Data Stack

```
┌─────────────────────────────────────────────┐
│         Python Data Engineering Stack       │
├─────────────────────────────────────────────┤
│                                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐ │
│  │  NumPy   │  │  Pandas  │  │ Jupyter  │ │
│  │ (Array)  │  │(DataFrame)│  │(Notebook)│ │
│  └────┬─────┘  └─────┬────┘  └──────────┘ │
│       │              │                     │
│       └──────┬───────┘                     │
│              │                             │
│      ┌───────▼────────┐                    │
│      │  Python Core   │                    │
│      │  (3.8+)        │                    │
│      └────────────────┘                    │
│                                             │
└─────────────────────────────────────────────┘
```

---

## 2. Data Structures: Pandas & NumPy

### 2.1 NumPy Arrays - Foundation สำหรับ Numerical Computing

NumPy เป็น Library พื้นฐานสำหรับการคำนวณทางคณิตศาสตร์

**ความสำคัญ:**
- ⚡ **Performance:** เร็วกว่า Python Lists มาก
- 📊 **Vectorized Operations:** คำนวณทั้ง Array ได้ในคำสั่งเดียว
- 💾 **Memory Efficient:** ใช้หน่วยความจำน้อยกว่า

#### 2.1.1 การสร้าง NumPy Array

```python
import numpy as np

# สร้าง Array จาก List
temperatures = np.array([25.5, 26.0, 24.8, 25.2, 26.5])
print(f"Temperatures: {temperatures}")
print(f"Data Type: {temperatures.dtype}")  # float64

# สร้าง Array ด้วย dtype ที่กำหนด (Memory Optimization)
temp_optimized = np.array([25, 26, 24, 25, 26], dtype=np.int8)
print(f"Memory (float64): {temperatures.nbytes} bytes")
print(f"Memory (int8): {temp_optimized.nbytes} bytes")
```

**Output:**
```
Temperatures: [25.5 26.  24.8 25.2 26.5]
Data Type: float64
Memory (float64): 40 bytes
Memory (int8): 5 bytes
```

#### 2.1.2 Vectorized Operations

```python
# คำนวณทั้ง Array (ไม่ต้องใช้ Loop)
celsius = np.array([0, 10, 20, 30, 40])
fahrenheit = (celsius * 9/5) + 32

print(f"Celsius: {celsius}")
print(f"Fahrenheit: {fahrenheit}")

# เปรียบเทียบกับ Python Loop (ช้ากว่า)
# fahrenheit_loop = [(c * 9/5) + 32 for c in celsius]
```

**Output:**
```
Celsius: [ 0 10 20 30 40]
Fahrenheit: [ 32.  50.  68.  86. 104.]
```

### 2.2 Pandas DataFrame - ตารางข้อมูลสำหรับ Data Engineering

Pandas ให้โครงสร้างข้อมูลแบบตาราง (Tabular) เหมาะสำหรับ ETL

**ความสามารถหลัก:**
- 📋 **Labeled Data:** มี Column Names และ Index
- 🔍 **Filtering & Querying:** กรองข้อมูลได้สะดวก
- 🔄 **Missing Data Handling:** จัดการข้อมูลหายได้ดี
- 📊 **Aggregation:** คำนวณค่าสถิติได้ง่าย

#### 2.2.1 การสร้าง DataFrame

```python
import pandas as pd

# สร้าง DataFrame จาก Dictionary
sensor_data = {
    'timestamp': ['2025-01-01 00:00', '2025-01-01 00:05', '2025-01-01 00:10'],
    'sensor_id': ['S001', 'S001', 'S001'],
    'temperature': [25.5, 26.0, 24.8],
    'humidity': [60, 62, 58]
}

df = pd.DataFrame(sensor_data)
print(df)
print(f"\nData Types:\n{df.dtypes}")
```

**Output:**
```
            timestamp sensor_id  temperature  humidity
0  2025-01-01 00:00      S001         25.5        60
1  2025-01-01 00:05      S001         26.0        62
2  2025-01-01 00:10      S001         24.8        58

Data Types:
timestamp       object
sensor_id       object
temperature    float64
humidity         int64
dtype: object
```

#### 2.2.2 DataFrame Operations

```python
# การเลือกข้อมูล (Selection)
print(df['temperature'])  # เลือก Column
print(df[df['temperature'] > 25])  # Filter Rows

# การคำนวณค่าสถิติ (Aggregation)
print(f"Average Temperature: {df['temperature'].mean():.2f}°C")
print(f"Max Humidity: {df['humidity'].max()}%")

# การเพิ่ม Column ใหม่ (Feature Engineering)
df['temp_fahrenheit'] = (df['temperature'] * 9/5) + 32
print(df)
```

### 2.3 เปรียบเทียบ NumPy vs Pandas

| Feature | NumPy | Pandas |
|---------|-------|--------|
| **โครงสร้างข้อมูล** | Array (Homogeneous) | DataFrame (Heterogeneous) |
| **Labels** | Index เป็นตัวเลข | Column Names + Index |
| **ความเร็ว** | ⚡⚡⚡ เร็วมาก | ⚡⚡ เร็ว |
| **Use Case** | Numerical Computation | Data Analysis & ETL |
| **Missing Data** | ไม่รองรับดี | รองรับดี (NaN, None) |

**💡 แนะนำ:**
- ใช้ **NumPy** สำหรับ: การคำนวณทางคณิตศาสตร์, Matrix Operations
- ใช้ **Pandas** สำหรับ: ETL, Data Cleaning, Analysis

---

## 3. Virtual Environment Setup

### 3.1 ทำไมต้องใช้ Virtual Environment?

Virtual Environment แยก Dependencies ของแต่ละโปรเจค

**ประโยชน์:**
- ✅ **Isolation:** แต่ละโปรเจคมี Libraries เป็นของตัวเอง
- ✅ **Reproducibility:** ติดตั้ง Environment เดิมได้ทุกเครื่อง
- ✅ **No Conflicts:** ป้องกัน Version Conflicts ระหว่างโปรเจค

### 3.2 ภาพแสดง Virtual Environment Concept

```
┌─────────────────────────────────────────────────┐
│          System Python (Global)                 │
│  - python 3.10                                  │
│  - pip                                          │
└──────────────────┬──────────────────────────────┘
                   │
         ┌─────────┴─────────┬─────────────────┐
         │                   │                 │
    ┌────▼─────┐      ┌──────▼────┐     ┌──────▼────┐
    │ Project A │      │ Project B │     │ Project C │
    │ (venv)    │      │ (venv)    │     │ (venv)    │
    ├───────────┤      ├───────────┤     ├───────────┤
    │pandas 1.3 │      │pandas 2.0 │     │pandas 1.5 │
    │numpy 1.21 │      │numpy 1.24 │     │numpy 1.23 │
    │airflow 2.0│      │flask 2.0  │     │django 4.0 │
    └───────────┘      └───────────┘     └───────────┘
```

### 3.3 การสร้างและใช้งาน Virtual Environment

#### 3.3.1 ใช้ venv (Built-in Python)

```bash
# 1. สร้าง Virtual Environment
python -m venv iot_pipeline_env

# 2. Activate Environment

# บน macOS/Linux:
source iot_pipeline_env/bin/activate

# บน Windows:
iot_pipeline_env\Scripts\activate

# 3. ตรวจสอบว่า Activate แล้ว
which python  # macOS/Linux
where python  # Windows
# ควรแสดง path ใน iot_pipeline_env

# 4. ติดตั้ง Libraries
pip install pandas numpy jupyter

# 5. ตรวจสอบ Libraries ที่ติดตั้ง
pip list

# 6. Deactivate (เมื่อเสร็จงาน)
deactivate
```

#### 3.3.2 การจัดการ Dependencies

```bash
# บันทึก Dependencies ลงไฟล์
pip freeze > requirements.txt

# ติดตั้งจากไฟล์ (บนเครื่องใหม่)
pip install -r requirements.txt
```

**ตัวอย่าง requirements.txt:**
```txt
pandas==1.3.5
numpy==1.21.6
jupyter==1.0.0
```

### 3.4 Best Practices สำหรับ Virtual Environment

✅ **DO:**
- สร้าง Environment ใหม่สำหรับทุกโปรเจค
- ใช้ `requirements.txt` เก็บ Dependencies
- เพิ่ม `venv/` หรือ `iot_pipeline_env/` ใน `.gitignore`

❌ **DON'T:**
- ติดตั้ง Libraries ใน Global Python
- Commit Virtual Environment ลง Git
- ใช้ Environment เดียวกันหลายโปรเจค

---

## 4. Time-Series Data Introduction

### 4.1 Time-Series Data คืออะไร?

**Time-Series Data** = ข้อมูลที่มี Timestamp แสดงลำดับเวลา

**ตัวอย่าง:**
- 📊 ข้อมูล Sensor อุณหภูมิทุก 5 นาที
- 💹 ราคาหุ้นทุกวินาที
- 🌡️ ข้อมูลสภาพอากาศรายชั่วโมง

### 4.2 คุณสมบัติของ Time-Series Data

```
Time-Series Data Characteristics:

1. Ordered by Time
   ├─▶ Sequential (มีลำดับเวลา)
   └─▶ Cannot shuffle randomly

2. Temporal Dependencies
   ├─▶ Current value อาจขึ้นกับ Past values
   └─▶ มี Trends, Seasonality, Patterns

3. Frequency/Interval
   ├─▶ Regular: ทุก 5 นาที, ทุกชั่วโมง
   └─▶ Irregular: Event-driven
```

### 4.3 ตัวอย่างโครงสร้าง IoT Time-Series Data

```
Timestamp            | Sensor ID | Temperature | Humidity | Pressure
---------------------|-----------|-------------|----------|----------
2025-01-01 00:00:00  | S001      | 25.5        | 60       | 1013
2025-01-01 00:05:00  | S001      | 26.0        | 62       | 1012
2025-01-01 00:10:00  | S001      | 24.8        | 58       | 1014
2025-01-01 00:00:00  | S002      | 22.3        | 55       | 1013
...
```

### 4.4 การทำงานกับ Timestamp ใน Pandas

```python
import pandas as pd

# สร้าง DataFrame พร้อม Timestamp
data = {
    'timestamp': ['2025-01-01 00:00', '2025-01-01 00:05', '2025-01-01 00:10'],
    'temperature': [25.5, 26.0, 24.8]
}
df = pd.DataFrame(data)

# แปลง timestamp เป็น datetime type
df['timestamp'] = pd.to_datetime(df['timestamp'])
print(df.dtypes)

# ตั้ง timestamp เป็น Index
df.set_index('timestamp', inplace=True)
print(df)

# Resampling - คำนวณค่าเฉลี่ยทุก 10 นาที
df_10min = df.resample('10min').mean()
print(df_10min)
```

### 4.5 ภาพแสดง Time-Series Data Flow

```
IoT Sensor Data Flow:

┌──────────┐
│  Sensor  │
│  (S001)  │
└────┬─────┘
     │ ทุก 5 นาที
     ▼
┌────────────────────────────┐
│    Raw Time-Series Data    │
│                            │
│  2025-01-01 00:00  25.5°C  │
│  2025-01-01 00:05  26.0°C  │
│  2025-01-01 00:10  24.8°C  │
│  ...                       │
└──────────┬─────────────────┘
           │
           ▼
    ┌──────────────┐
    │ ETL Pipeline │
    │  (Pandas)    │
    └──────┬───────┘
           │
           ▼
┌──────────────────────────┐
│  Processed Data          │
│  - Cleaned               │
│  - Aggregated            │
│  - Feature Engineered    │
└──────────────────────────┘
```

---

## 5. สรุป Module 1

### 5.1 Key Takeaways

✅ **Python** เป็นภาษายอดนิยมสำหรับ Data Engineering
✅ **NumPy** ใช้สำหรับ Numerical Computing (เร็วและประหยัด Memory)
✅ **Pandas** ใช้สำหรับ Data Analysis และ ETL (รองรับ Missing Data, Aggregation)
✅ **Virtual Environment** แยก Dependencies แต่ละโปรเจค
✅ **Time-Series Data** มี Timestamp และต้องเรียงตามเวลา

### 5.2 Skills ที่ได้จาก Module นี้

| Skill | Level |
|-------|-------|
| NumPy Arrays Operations | ⭐⭐⭐ |
| Pandas DataFrame Basics | ⭐⭐⭐ |
| Virtual Environment Setup | ⭐⭐⭐ |
| Time-Series Data Concepts | ⭐⭐ |

### 5.3 เตรียมพร้อมสำหรับ Module 2

ใน Module ถัดไป คุณจะได้เรียนรู้:
- การโหลดข้อมูล CSV ขนาดใหญ่แบบ Optimized
- การจัดการ Missing Values
- Data Integrity Checks

---

## 📝 Challenge Questions

ก่อนไป Module 2 ลองตอบคำถามเหล่านี้:

1. **NumPy vs Pandas:** เมื่อไหร่ควรใช้ NumPy? เมื่อไหร่ควรใช้ Pandas?
2. **Memory Optimization:** ถ้ามี Array ของตัวเลข 0-100 ควรใช้ dtype อะไร? ทำไม?
3. **Virtual Environment:** ทำไมไม่ควร Commit folder `venv/` ลง Git?
4. **Time-Series:** IoT Sensor ส่งข้อมูลทุก 5 นาที ใน 1 วันจะมีกี่ records?

---

## 🎯 ถัดไป: Labs & Practical Exercises

พร้อมทำ Lab แล้วหรือยัง?

**👉 [เริ่มทำ Lab Module 1](./labs/lab-module-1.md)**

ใน Lab คุณจะได้:
- ติดตั้ง Virtual Environment
- ทดสอบ Pandas และ NumPy
- โหลดและวิเคราะห์ข้อมูล Time-Series ตัวอย่าง

---

**[⬅️ กลับไปที่ Wiki หลัก](../wiki.md)** | **[Module 2: ETL - Data Loading & Cleansing ➡️](../module-2/module-2.md)**
