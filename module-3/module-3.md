# Module 3: ETL - Transformation & Optimization

**🎯 วัตถุประสงค์การเรียนรู้:**
- เข้าใจหลักการ Data Transformation และ Feature Engineering
- ใช้งาน Rolling Window Operations สำหรับ Time-Series
- เปรียบเทียบและวัดผล Performance ระหว่าง Pandas และ NumPy
- ใช้เทคนิค Chunking สำหรับจัดการข้อมูลขนาดใหญ่
- เพิ่มประสิทธิภาพการประมวลผลด้วย Vectorized Operations

**⏱️ เวลาที่ใช้:** 4.5 ชั่วโมง (ทฤษฎี 2.5 ชม. + Lab 2 ชม.)

---

## 📚 สารบัญ

1. [Data Transformation Overview](#1-data-transformation-overview)
2. [Feature Engineering สำหรับ Time-Series](#2-feature-engineering-สำหรับ-time-series)
3. [Rolling Window Operations](#3-rolling-window-operations)
4. [Performance Optimization Techniques](#4-performance-optimization-techniques)
5. [Pandas vs NumPy Performance](#5-pandas-vs-numpy-performance)
6. [Chunking Strategy สำหรับ Big Data](#6-chunking-strategy-สำหรับ-big-data)
7. [Vectorized Operations](#7-vectorized-operations)
8. [Labs & Practical Exercises](./labs/lab-module-3.md)

---

## 1. Data Transformation Overview

### 1.1 Data Transformation คืออะไร?

**Data Transformation** เป็นขั้นตอนสำคัญใน ETL Pipeline ที่แปลงข้อมูลดิบให้อยู่ในรูปแบบที่เหมาะสมสำหรับการวิเคราะห์

```
ETL Pipeline:

Extract          Transform              Load
(โหลดข้อมูล)  →  (แปลงข้อมูล)      →  (บันทึกข้อมูล)
   ↓                  ↓                     ↓
Raw Data    →   Processing Steps   →   Clean Data
                • Cleaning                  ↓
                • Feature Eng.          Analytics
                • Aggregation           ML Models
                • Optimization          Reports
```

### 1.2 ประเภทของ Transformation

**1. Data Cleaning Transformations**
- จัดการ Missing Values (Ffill, Interpolate)
- ลบ Outliers
- แก้ไข Data Types

**2. Feature Engineering Transformations**
- สร้าง Features ใหม่จากข้อมูลเดิม
- Rolling Statistics (Mean, Std, Min, Max)
- Time-based Features (Hour, Day of Week)

**3. Aggregation Transformations**
- Group by operations
- Resampling (เปลี่ยน Frequency)
- Pivot tables

**4. Performance Optimization Transformations**
- Vectorization
- Chunking
- Efficient Data Types

### 1.3 Transformation Workflow ใน IoT Data Pipeline

```python
import pandas as pd
import numpy as np

# Raw Data
df = pd.read_csv('sensor_raw.csv')

# 1. Data Cleaning
df['timestamp'] = pd.to_datetime(df['timestamp'])
df['temperature'].fillna(method='ffill', inplace=True)

# 2. Feature Engineering
df['temp_rolling_5min'] = df['temperature'].rolling(window=5).mean()
df['hour'] = df['timestamp'].dt.hour

# 3. Aggregation
hourly_avg = df.groupby('hour')['temperature'].mean()

# 4. Optimization
df['temperature'] = df['temperature'].astype(np.float32)
```

---

## 2. Feature Engineering สำหรับ Time-Series

### 2.1 Feature Engineering คืออะไร?

**Feature Engineering** = การสร้าง Features (คุณสมบัติ) ใหม่จากข้อมูลที่มีอยู่เพื่อปรับปรุงการวิเคราะห์หรือ ML Model

**ตัวอย่าง:**
- จาก `timestamp` → สร้าง `hour`, `day_of_week`, `is_weekend`
- จาก `temperature` → สร้าง `temp_rolling_avg`, `temp_change`

### 2.2 Time-Series Features ที่สำคัญ

#### 2.2.1 Time-based Features

```python
import pandas as pd

# สร้าง Sample Data
df = pd.DataFrame({
    'timestamp': pd.date_range('2025-01-01', periods=100, freq='5min'),
    'temperature': np.random.normal(25, 2, 100)
})

df['timestamp'] = pd.to_datetime(df['timestamp'])

# Extract Time-based Features
df['hour'] = df['timestamp'].dt.hour
df['day_of_week'] = df['timestamp'].dt.dayofweek  # 0=Monday
df['is_weekend'] = df['timestamp'].dt.dayofweek >= 5
df['month'] = df['timestamp'].dt.month
df['date'] = df['timestamp'].dt.date

print(df[['timestamp', 'hour', 'day_of_week', 'is_weekend']].head())
```

**Output:**
```
            timestamp  hour  day_of_week  is_weekend
0 2025-01-01 00:00:00     0            2       False
1 2025-01-01 00:05:00     0            2       False
2 2025-01-01 00:10:00     0            2       False
3 2025-01-01 00:15:00     0            2       False
4 2025-01-01 00:20:00     0            2       False
```

#### 2.2.2 Lag Features (ข้อมูลย้อนหลัง)

```python
# Previous values
df['temp_lag_1'] = df['temperature'].shift(1)  # 1 period ก่อน
df['temp_lag_2'] = df['temperature'].shift(2)  # 2 periods ก่อน

# Change from previous
df['temp_change'] = df['temperature'] - df['temp_lag_1']
df['temp_change_pct'] = (df['temperature'] - df['temp_lag_1']) / df['temp_lag_1'] * 100

print(df[['temperature', 'temp_lag_1', 'temp_change', 'temp_change_pct']].head())
```

**Use Cases:**
- **Anomaly Detection:** ตรวจจับการเปลี่ยนแปลงผิดปกติ
- **Trend Analysis:** วิเคราะห์แนวโน้ม
- **ML Models:** Features สำหรับ Prediction

#### 2.2.3 Rolling Statistics Features

```python
# Rolling Mean (Moving Average)
df['temp_ma_5'] = df['temperature'].rolling(window=5).mean()
df['temp_ma_10'] = df['temperature'].rolling(window=10).mean()

# Rolling Std (Volatility)
df['temp_std_5'] = df['temperature'].rolling(window=5).std()

# Rolling Min/Max
df['temp_min_5'] = df['temperature'].rolling(window=5).min()
df['temp_max_5'] = df['temperature'].rolling(window=5).max()

# Rolling Range
df['temp_range_5'] = df['temp_max_5'] - df['temp_min_5']

print(df[['temperature', 'temp_ma_5', 'temp_std_5', 'temp_range_5']].head(10))
```

### 2.3 ภาพแสดง Feature Engineering Flow

```
Raw Time-Series Data
        ↓
┌───────────────────────────────────────┐
│  Original Features                    │
│  • timestamp: 2025-01-01 00:00:00    │
│  • temperature: 25.5                  │
│  • humidity: 60                       │
└───────────────┬───────────────────────┘
                │ Feature Engineering
                ↓
┌───────────────────────────────────────┐
│  Engineered Features                  │
│                                       │
│  Time-based:                          │
│  • hour: 0                            │
│  • day_of_week: 2                     │
│  • is_weekend: False                  │
│                                       │
│  Lag Features:                        │
│  • temp_lag_1: 25.3                   │
│  • temp_change: +0.2                  │
│                                       │
│  Rolling Features:                    │
│  • temp_rolling_5min: 25.4            │
│  • temp_std_5min: 0.3                 │
│                                       │
│  Domain Features:                     │
│  • temp_category: Normal              │
│  • temp_fahrenheit: 77.9              │
└───────────────────────────────────────┘
```

### 2.4 Domain-Specific Features สำหรับ IoT

```python
# Temperature Category
def categorize_temp(temp):
    if temp < 20:
        return 'Cold'
    elif temp < 25:
        return 'Cool'
    elif temp < 28:
        return 'Normal'
    else:
        return 'Hot'

df['temp_category'] = df['temperature'].apply(categorize_temp)

# Temperature Comfort Index (สมมติ)
df['comfort_index'] = df['temperature'] - (0.55 * (1 - df['humidity']/100) * (df['temperature'] - 14.5))

# Celsius to Fahrenheit
df['temp_fahrenheit'] = (df['temperature'] * 9/5) + 32

print(df[['temperature', 'humidity', 'temp_category', 'comfort_index']].head())
```

---

## 3. Rolling Window Operations

### 3.1 Rolling Window คืออะไร?

**Rolling Window** (หรือ Moving Window) คือการคำนวณค่าสถิติจาก Subset ของข้อมูลที่เลื่อนไปตามลำดับเวลา

**ประโยชน์:**
- ✅ **Smoothing:** ลดความผันผวนของข้อมูล
- ✅ **Trend Detection:** มองเห็นแนวโน้มได้ชัดเจนขึ้น
- ✅ **Noise Reduction:** ลดสัญญาณรบกวน

### 3.2 ภาพแสดง Rolling Window Concept

```
Temperature Data: [22, 24, 23, 25, 26, 24, 23, 22, 21, 23]

Rolling Window = 3:

Window 1: [22, 24, 23] → Mean = 23.0
Window 2:     [24, 23, 25] → Mean = 24.0
Window 3:         [23, 25, 26] → Mean = 24.7
Window 4:             [25, 26, 24] → Mean = 25.0
Window 5:                 [26, 24, 23] → Mean = 24.3
...

Result: [NaN, NaN, 23.0, 24.0, 24.7, 25.0, 24.3, ...]
```

### 3.3 การใช้งาน Rolling Window ใน Pandas

#### 3.3.1 Rolling Mean (Moving Average)

```python
import pandas as pd
import numpy as np

# สร้าง Sample Data
np.random.seed(42)
df = pd.DataFrame({
    'timestamp': pd.date_range('2025-01-01', periods=50, freq='5min'),
    'temperature': 25 + np.random.normal(0, 2, 50)
})

# Rolling Mean
df['temp_ma_5'] = df['temperature'].rolling(window=5).mean()
df['temp_ma_10'] = df['temperature'].rolling(window=10).mean()

print("Rolling Mean Example:")
print(df[['temperature', 'temp_ma_5', 'temp_ma_10']].head(15))
```

**Parameters:**
- `window`: จำนวน periods ที่ต้องการคำนวณ
- `min_periods`: จำนวน periods ขั้นต่ำที่ต้องมีเพื่อคำนวณ (default = window)
- `center`: ถ้า True จะคำนวณจากตรงกลาง window

#### 3.3.2 Rolling Standard Deviation

```python
# Rolling Std (วัดความผันผวน)
df['temp_volatility_5'] = df['temperature'].rolling(window=5).std()

# Rolling Coefficient of Variation
df['temp_cv_5'] = df['temp_volatility_5'] / df['temp_ma_5']

print("Rolling Std Example:")
print(df[['temperature', 'temp_ma_5', 'temp_volatility_5']].head(15))
```

#### 3.3.3 Rolling Min/Max (Range)

```python
# Rolling Min/Max
df['temp_min_10'] = df['temperature'].rolling(window=10).min()
df['temp_max_10'] = df['temperature'].rolling(window=10).max()
df['temp_range_10'] = df['temp_max_10'] - df['temp_min_10']

print("Rolling Range Example:")
print(df[['temperature', 'temp_min_10', 'temp_max_10', 'temp_range_10']].tail(10))
```

### 3.4 Time-based Rolling Windows

สำหรับ Time-Series ควรใช้ Time-based Window แทน Count-based

```python
# ตั้ง timestamp เป็น index
df.set_index('timestamp', inplace=True)

# Rolling Window 30 นาที (แทนที่จะใช้ window=6)
df['temp_ma_30min'] = df['temperature'].rolling('30min').mean()

# Rolling Window 1 ชั่วโมง
df['temp_ma_1h'] = df['temperature'].rolling('1h').mean()

# Rolling Window 1 วัน
df['temp_ma_1d'] = df['temperature'].rolling('1D').mean()

print("Time-based Rolling Example:")
print(df[['temperature', 'temp_ma_30min', 'temp_ma_1h']].head(20))
```

**ข้อดีของ Time-based Rolling:**
- ✅ รองรับข้อมูล Irregular frequency
- ✅ ความหมายชัดเจนกว่า (30 นาทีแทน 6 periods)
- ✅ ไม่เปลี่ยนแปลงเมื่อ frequency เปลี่ยน

### 3.5 Rolling Window สำหรับ Anomaly Detection

```python
# คำนวณ Upper/Lower Bounds (Mean ± 2*Std)
window_size = 20
df['temp_ma'] = df['temperature'].rolling(window=window_size).mean()
df['temp_std'] = df['temperature'].rolling(window=window_size).std()

df['upper_bound'] = df['temp_ma'] + (2 * df['temp_std'])
df['lower_bound'] = df['temp_ma'] - (2 * df['temp_std'])

# ตรวจจับ Anomaly
df['is_anomaly'] = (df['temperature'] > df['upper_bound']) | \
                   (df['temperature'] < df['lower_bound'])

print(f"Anomalies detected: {df['is_anomaly'].sum()}")
print("\nAnomalies:")
print(df[df['is_anomaly']][['temperature', 'temp_ma', 'upper_bound', 'lower_bound']])
```

---

## 4. Performance Optimization Techniques

### 4.1 ทำไมต้อง Optimize Performance?

เมื่อข้อมูลมีขนาดใหญ่ขึ้น Performance กลายเป็นปัญหาสำคัญ

**ปัญหาที่พบ:**
- ⏱️ **Slow Processing:** ใช้เวลานานเกินไป
- 💾 **Memory Error:** RAM ไม่พอ
- 🔥 **High CPU Usage:** ใช้ทรัพยากรมากเกินไป

### 4.2 Performance Optimization Strategies

```
Optimization Techniques:

1. Data Type Optimization
   ├─▶ ใช้ dtype ที่เหมาะสม (float32 แทน float64)
   ├─▶ ใช้ category สำหรับ String ซ้ำๆ
   └─▶ ลด Memory Usage 50-90%

2. Vectorization
   ├─▶ ใช้ NumPy operations
   ├─▶ หลีกเลี่ยง Python loops
   └─▶ เร็วขึ้น 10-100 เท่า

3. Chunking
   ├─▶ แบ่งข้อมูลเป็นส่วนๆ
   ├─▶ ประมวลผลทีละ Chunk
   └─▶ รองรับข้อมูลขนาดใหญ่

4. Efficient Algorithms
   ├─▶ เลือก Algorithm ที่เหมาะสม
   ├─▶ หลีกเลี่ยง Nested Loops
   └─▶ ใช้ Built-in Functions
```

### 4.3 Data Type Optimization

#### 4.3.1 Numeric Data Types

```python
import pandas as pd
import numpy as np

# สร้าง Sample Data
df = pd.DataFrame({
    'sensor_id': ['S001'] * 1000,
    'temperature': np.random.uniform(20, 30, 1000),
    'humidity': np.random.randint(40, 80, 1000)
})

print("=== Before Optimization ===")
print(df.dtypes)
print(f"Memory Usage: {df.memory_usage(deep=True).sum() / 1024:.2f} KB\n")

# Optimize
df_optimized = df.copy()
df_optimized['sensor_id'] = df_optimized['sensor_id'].astype('category')
df_optimized['temperature'] = df_optimized['temperature'].astype(np.float32)
df_optimized['humidity'] = df_optimized['humidity'].astype(np.int8)

print("=== After Optimization ===")
print(df_optimized.dtypes)
print(f"Memory Usage: {df_optimized.memory_usage(deep=True).sum() / 1024:.2f} KB")

memory_saved = df.memory_usage(deep=True).sum() - df_optimized.memory_usage(deep=True).sum()
percent_saved = (memory_saved / df.memory_usage(deep=True).sum()) * 100
print(f"Memory Saved: {memory_saved / 1024:.2f} KB ({percent_saved:.1f}%)")
```

#### 4.3.2 ตารางเปรียบเทียบ Data Types

| Type | Size | Range | Use Case |
|------|------|-------|----------|
| **int8** | 1 byte | -128 to 127 | Temperature (°C), Small integers |
| **int16** | 2 bytes | -32,768 to 32,767 | Medium integers |
| **int32** | 4 bytes | -2B to 2B | Large integers |
| **float32** | 4 bytes | ±3.4e38 (7 digits) | Temperature (decimal) |
| **float64** | 8 bytes | ±1.7e308 (15 digits) | High precision (default) |
| **category** | Variable | N/A | Repeated strings |

### 4.4 Function-based Optimization

```python
import time

# Sample Data
n = 1_000_000
temperatures = np.random.uniform(20, 30, n)

# ❌ Bad: Python Loop
start = time.time()
fahrenheit_loop = []
for temp in temperatures:
    fahrenheit_loop.append((temp * 9/5) + 32)
time_loop = time.time() - start

# ❌ Bad: Apply Function
df = pd.DataFrame({'temperature': temperatures})
start = time.time()
df['fahrenheit'] = df['temperature'].apply(lambda x: (x * 9/5) + 32)
time_apply = time.time() - start

# ✅ Good: Vectorized Operation
start = time.time()
fahrenheit_vectorized = (temperatures * 9/5) + 32
time_vectorized = time.time() - start

print(f"Python Loop:  {time_loop:.4f} sec")
print(f"Pandas Apply: {time_apply:.4f} sec")
print(f"Vectorized:   {time_vectorized:.4f} sec")
print(f"\nSpeedup (Loop → Vectorized): {time_loop / time_vectorized:.1f}x")
```

**Expected Output:**
```
Python Loop:  0.5234 sec
Pandas Apply: 0.3421 sec
Vectorized:   0.0045 sec

Speedup (Loop → Vectorized): 116.3x
```

---

## 5. Pandas vs NumPy Performance

### 5.1 Pandas vs NumPy: เมื่อไหร่ควรใช้อะไร?

| Aspect | Pandas | NumPy |
|--------|--------|-------|
| **ความเร็ว** | ⚡⚡ ค่อนข้างเร็ว | ⚡⚡⚡ เร็วมาก |
| **Use Case** | Data Analysis, ETL | Numerical Computing |
| **โครงสร้าง** | DataFrame (Labels) | Array (No Labels) |
| **Operations** | Rich (groupby, merge) | Basic Math Operations |
| **Memory** | ใช้มากกว่า | ใช้น้อยกว่า |

### 5.2 Performance Comparison: Basic Operations

```python
import pandas as pd
import numpy as np
import time

# Prepare Data
n = 10_000_000
data = np.random.uniform(0, 100, n)
df = pd.DataFrame({'value': data})

print(f"Data Size: {n:,} records\n")

# Test 1: Addition
start = time.time()
result_pandas = df['value'] + 10
time_pandas_add = time.time() - start

start = time.time()
result_numpy = data + 10
time_numpy_add = time.time() - start

print("=== Addition (x + 10) ===")
print(f"Pandas: {time_pandas_add:.4f} sec")
print(f"NumPy:  {time_numpy_add:.4f} sec")
print(f"NumPy is {time_pandas_add / time_numpy_add:.1f}x faster\n")

# Test 2: Mathematical Operations
start = time.time()
result_pandas = (df['value'] * 2) - 5
time_pandas_math = time.time() - start

start = time.time()
result_numpy = (data * 2) - 5
time_numpy_math = time.time() - start

print("=== Math Operations (x * 2 - 5) ===")
print(f"Pandas: {time_pandas_math:.4f} sec")
print(f"NumPy:  {time_numpy_math:.4f} sec")
print(f"NumPy is {time_pandas_math / time_numpy_math:.1f}x faster\n")

# Test 3: Statistical Operations
start = time.time()
mean_pandas = df['value'].mean()
time_pandas_mean = time.time() - start

start = time.time()
mean_numpy = data.mean()
time_numpy_mean = time.time() - start

print("=== Mean Calculation ===")
print(f"Pandas: {time_pandas_mean:.6f} sec (mean = {mean_pandas:.2f})")
print(f"NumPy:  {time_numpy_mean:.6f} sec (mean = {mean_numpy:.2f})")
print(f"NumPy is {time_pandas_mean / time_numpy_mean:.1f}x faster")
```

### 5.3 Benchmark: Array Subtraction

การเปรียบเทียบการ Subtract ระหว่าง Arrays

```python
import numpy as np
import pandas as pd
import time

# Prepare Data
size = 5_000_000
arr1 = np.random.uniform(0, 100, size)
arr2 = np.random.uniform(0, 100, size)

s1 = pd.Series(arr1)
s2 = pd.Series(arr2)

# NumPy Subtraction
start = time.time()
result_numpy = arr1 - arr2
time_numpy = time.time() - start

# Pandas Series Subtraction
start = time.time()
result_pandas = s1 - s2
time_pandas = time.time() - start

print("=== Array/Series Subtraction Benchmark ===")
print(f"Data Size: {size:,} elements\n")
print(f"NumPy Array:   {time_numpy:.6f} sec")
print(f"Pandas Series: {time_pandas:.6f} sec")
print(f"\nSpeedup: NumPy is {time_pandas / time_numpy:.2f}x faster")
```

**Expected Output:**
```
=== Array/Series Subtraction Benchmark ===
Data Size: 5,000,000 elements

NumPy Array:   0.008234 sec
Pandas Series: 0.015678 sec

Speedup: NumPy is 1.90x faster
```

### 5.4 เมื่อไหร่ควรใช้ NumPy?

**✅ ใช้ NumPy เมื่อ:**
- การคำนวณทางคณิตศาสตร์บริสุทธิ์
- ต้องการ Performance สูงสุด
- ไม่ต้องการ Labels (Column Names, Index)
- ทำงานกับ Multi-dimensional Arrays

**✅ ใช้ Pandas เมื่อ:**
- ต้องการ Labels และ Column Names
- ทำ ETL, Data Cleaning
- ใช้ groupby, merge, pivot operations
- อ่าน/เขียนไฟล์ (CSV, Excel, SQL)

### 5.5 Best Practice: ใช้ร่วมกัน

```python
# ใช้ Pandas สำหรับ Data Structure
df = pd.read_csv('sensor_data.csv')

# แปลงเป็น NumPy สำหรับการคำนวณหนัก
temperatures = df['temperature'].values  # NumPy array
humidities = df['humidity'].values

# ใช้ NumPy คำนวณ (เร็ว)
comfort_index = temperatures - (0.55 * (1 - humidities/100) * (temperatures - 14.5))

# เอาผลลัพธ์กลับไปใน Pandas
df['comfort_index'] = comfort_index
```

---

## 6. Chunking Strategy สำหรับ Big Data

### 6.1 Chunking คืออะไร?

**Chunking** = แบ่งข้อมูลขนาดใหญ่เป็นส่วนเล็กๆ (Chunks) แล้วประมวลผลทีละส่วน

**ประโยชน์:**
- ✅ **ลด Memory Usage:** ไม่ต้องโหลดข้อมูลทั้งหมดพร้อมกัน
- ✅ **รองรับข้อมูลขนาดใหญ่:** ประมวลผลได้แม้ RAM จำกัด
- ✅ **Streaming Processing:** ประมวลผลได้ทันทีที่ข้อมูลมา

### 6.2 ภาพแสดง Chunking Concept

```
Large CSV File (1 GB):
┌─────────────────────────────────────┐
│  Row 1 - 100,000                    │ ← Chunk 1
├─────────────────────────────────────┤
│  Row 100,001 - 200,000              │ ← Chunk 2
├─────────────────────────────────────┤
│  Row 200,001 - 300,000              │ ← Chunk 3
├─────────────────────────────────────┤
│  ...                                │
└─────────────────────────────────────┘

Process Flow:
Chunk 1 → Process → Write
Chunk 2 → Process → Write
Chunk 3 → Process → Write
...

Memory Usage: แทนที่จะใช้ 1 GB → ใช้แค่ ~10 MB/chunk
```

### 6.3 การใช้ Chunking ใน Pandas

#### 6.3.1 อ่าน CSV แบบ Chunking

```python
import pandas as pd

# อ่าน CSV ทีละ Chunk
chunk_size = 100_000
chunks_processed = 0

for chunk in pd.read_csv('sensor_large.csv', chunksize=chunk_size):
    # ประมวลผลแต่ละ chunk
    chunk['timestamp'] = pd.to_datetime(chunk['timestamp'])
    chunk['temp_celsius'] = chunk['temperature']
    chunk['temp_fahrenheit'] = (chunk['temperature'] * 9/5) + 32

    # บันทึกผลลัพธ์
    mode = 'w' if chunks_processed == 0 else 'a'
    header = True if chunks_processed == 0 else False
    chunk.to_csv('processed_output.csv', mode=mode, header=header, index=False)

    chunks_processed += 1
    print(f"Processed chunk {chunks_processed}: {len(chunk)} records")

print(f"\nTotal chunks processed: {chunks_processed}")
```

#### 6.3.2 Chunking สำหรับ Aggregation

```python
# คำนวณ Statistics จากไฟล์ขนาดใหญ่
chunk_size = 100_000
temp_sum = 0
temp_count = 0
temp_min = float('inf')
temp_max = float('-inf')

for chunk in pd.read_csv('sensor_large.csv', chunksize=chunk_size):
    temp_sum += chunk['temperature'].sum()
    temp_count += len(chunk)
    temp_min = min(temp_min, chunk['temperature'].min())
    temp_max = max(temp_max, chunk['temperature'].max())

# คำนวณ Overall Statistics
temp_mean = temp_sum / temp_count

print("=== Overall Statistics ===")
print(f"Total Records: {temp_count:,}")
print(f"Mean Temperature: {temp_mean:.2f}°C")
print(f"Min Temperature: {temp_min:.2f}°C")
print(f"Max Temperature: {temp_max:.2f}°C")
```

### 6.4 Chunking Strategy Best Practices

#### 6.4.1 การเลือก Chunk Size

```python
import os

def calculate_optimal_chunk_size(file_path, available_memory_mb=500):
    """
    คำนวณ Chunk Size ที่เหมาะสม

    Args:
        file_path: Path to CSV file
        available_memory_mb: Available memory in MB

    Returns:
        Optimal chunk size (rows)
    """
    # ประมาณ file size
    file_size_mb = os.path.getsize(file_path) / (1024 * 1024)

    # อ่าน sample เพื่อประมาณ memory per row
    sample = pd.read_csv(file_path, nrows=1000)
    memory_per_1000_rows = sample.memory_usage(deep=True).sum() / (1024 * 1024)
    memory_per_row = memory_per_1000_rows / 1000

    # คำนวณ chunk size
    chunk_size = int(available_memory_mb / memory_per_row)

    print(f"File Size: {file_size_mb:.2f} MB")
    print(f"Memory per Row: {memory_per_row:.4f} MB")
    print(f"Recommended Chunk Size: {chunk_size:,} rows")

    return chunk_size

# Example
# optimal_size = calculate_optimal_chunk_size('sensor_large.csv')
```

**แนวทาง:**
- 🔹 **Small Files (<100 MB):** อ่านทั้งหมดได้เลย
- 🔹 **Medium Files (100 MB - 1 GB):** Chunk size = 100,000 - 500,000 rows
- 🔹 **Large Files (>1 GB):** Chunk size = 50,000 - 100,000 rows

#### 6.4.2 Parallel Processing กับ Chunking

```python
from multiprocessing import Pool
import pandas as pd

def process_chunk(chunk_data):
    """
    Function สำหรับประมวลผล 1 chunk
    """
    chunk = chunk_data['chunk']
    chunk_id = chunk_data['chunk_id']

    # ประมวลผล
    chunk['timestamp'] = pd.to_datetime(chunk['timestamp'])
    chunk['temp_fahrenheit'] = (chunk['temperature'] * 9/5) + 32

    return chunk_id, len(chunk)

# อ่านและเตรียม chunks
chunks = []
for i, chunk in enumerate(pd.read_csv('sensor_large.csv', chunksize=100_000)):
    chunks.append({'chunk': chunk, 'chunk_id': i})

# Process แบบ Parallel
with Pool(processes=4) as pool:
    results = pool.map(process_chunk, chunks)

print(f"Processed {len(results)} chunks in parallel")
```

### 6.5 Chunking vs Full Load Comparison

```python
import time
import pandas as pd

# Test File
file_path = 'sensor_large.csv'

# Method 1: Full Load (ถ้าไฟล์ใหญ่จะ Memory Error)
print("=== Full Load ===")
start = time.time()
try:
    df = pd.read_csv(file_path)
    memory_usage = df.memory_usage(deep=True).sum() / (1024 * 1024)
    time_full = time.time() - start
    print(f"Time: {time_full:.2f} sec")
    print(f"Memory: {memory_usage:.2f} MB")
    print(f"Rows: {len(df):,}")
except MemoryError:
    print("MemoryError: File too large to load fully")

# Method 2: Chunking
print("\n=== Chunking ===")
start = time.time()
chunk_size = 100_000
total_rows = 0

for chunk in pd.read_csv(file_path, chunksize=chunk_size):
    total_rows += len(chunk)
    # Process chunk here

time_chunk = time.time() - start
print(f"Time: {time_chunk:.2f} sec")
print(f"Memory: ~{chunk_size * 0.01:.2f} MB (per chunk)")
print(f"Rows: {total_rows:,}")
```

---

## 7. Vectorized Operations

### 7.1 Vectorization คืออะไร?

**Vectorization** = การคำนวณทั้ง Array/Series ในคำสั่งเดียว แทนที่จะใช้ Loop

**ความแตกต่าง:**

```python
# ❌ Non-Vectorized (Loop)
result = []
for i in range(len(data)):
    result.append(data[i] * 2)

# ✅ Vectorized
result = data * 2
```

### 7.2 ทำไม Vectorization เร็วกว่า?

```
Python Loop:
┌────────┐
│ For i  │ → Check → Calculate → Append → Repeat (Slow)
└────────┘   (Python Overhead for each iteration)

Vectorized Operation:
┌────────────────┐
│ NumPy/Pandas   │ → Calculate ALL at once (Fast)
└────────────────┘   (C/Fortran Implementation)

Speedup: 10-100x faster
```

### 7.3 Vectorization Examples

#### 7.3.1 Basic Math Operations

```python
import numpy as np
import time

n = 1_000_000
data = np.random.uniform(0, 100, n)

# ❌ Loop (Slow)
start = time.time()
result_loop = []
for x in data:
    result_loop.append(x * 2 + 10)
time_loop = time.time() - start

# ✅ Vectorized (Fast)
start = time.time()
result_vectorized = data * 2 + 10
time_vectorized = time.time() - start

print(f"Loop:       {time_loop:.4f} sec")
print(f"Vectorized: {time_vectorized:.4f} sec")
print(f"Speedup:    {time_loop / time_vectorized:.1f}x")
```

#### 7.3.2 Conditional Operations

```python
# ❌ Loop
result_loop = []
for temp in temperatures:
    if temp > 25:
        result_loop.append('Hot')
    else:
        result_loop.append('Cool')

# ✅ Vectorized
result_vectorized = np.where(temperatures > 25, 'Hot', 'Cool')

# ✅ Pandas Vectorized
df['temp_label'] = df['temperature'].apply(lambda x: 'Hot' if x > 25 else 'Cool')  # Slower
df['temp_label'] = np.where(df['temperature'] > 25, 'Hot', 'Cool')  # Faster
```

#### 7.3.3 Multiple Conditions

```python
# Categorize temperature
# Cold: < 20, Cool: 20-25, Normal: 25-28, Hot: > 28

# ❌ Loop
categories = []
for temp in temperatures:
    if temp < 20:
        categories.append('Cold')
    elif temp < 25:
        categories.append('Cool')
    elif temp < 28:
        categories.append('Normal')
    else:
        categories.append('Hot')

# ✅ Vectorized (np.select)
conditions = [
    temperatures < 20,
    (temperatures >= 20) & (temperatures < 25),
    (temperatures >= 25) & (temperatures < 28),
    temperatures >= 28
]
choices = ['Cold', 'Cool', 'Normal', 'Hot']
categories_vectorized = np.select(conditions, choices)

# ✅ Pandas cut (สำหรับ binning)
df['temp_category'] = pd.cut(
    df['temperature'],
    bins=[-np.inf, 20, 25, 28, np.inf],
    labels=['Cold', 'Cool', 'Normal', 'Hot']
)
```

### 7.4 Vectorization Best Practices

**✅ DO:**
```python
# ใช้ NumPy/Pandas operations
result = df['temperature'] * 1.8 + 32

# ใช้ np.where สำหรับ conditions
df['status'] = np.where(df['temperature'] > 30, 'Alert', 'Normal')

# ใช้ Built-in functions
df['temp_mean'] = df['temperature'].mean()
```

**❌ DON'T:**
```python
# อย่าใช้ Loop
for i in range(len(df)):
    df.loc[i, 'fahrenheit'] = df.loc[i, 'temperature'] * 1.8 + 32

# อย่าใช้ apply เมื่อมี Vectorized alternative
df['fahrenheit'] = df['temperature'].apply(lambda x: x * 1.8 + 32)  # Slow
```

### 7.5 Performance Comparison Summary

```python
import pandas as pd
import numpy as np
import time

# Prepare Data
n = 500_000
df = pd.DataFrame({
    'temperature': np.random.uniform(20, 35, n)
})

print(f"Data Size: {n:,} records\n")

# Method 1: Loop with loc (ช้ามาก)
start = time.time()
df_loop = df.copy()
for i in range(len(df_loop)):
    df_loop.loc[i, 'fahrenheit'] = df_loop.loc[i, 'temperature'] * 1.8 + 32
time_loop = time.time() - start

# Method 2: Apply (ช้า)
start = time.time()
df_apply = df.copy()
df_apply['fahrenheit'] = df_apply['temperature'].apply(lambda x: x * 1.8 + 32)
time_apply = time.time() - start

# Method 3: Vectorized (เร็ว)
start = time.time()
df_vectorized = df.copy()
df_vectorized['fahrenheit'] = df_vectorized['temperature'] * 1.8 + 32
time_vectorized = time.time() - start

print("=== Performance Comparison ===")
print(f"Loop + loc:  {time_loop:.4f} sec (Baseline)")
print(f"Apply:       {time_apply:.4f} sec ({time_loop / time_apply:.1f}x faster than loop)")
print(f"Vectorized:  {time_vectorized:.4f} sec ({time_loop / time_vectorized:.1f}x faster than loop)")
```

**Expected Output:**
```
=== Performance Comparison ===
Loop + loc:  45.2341 sec (Baseline)
Apply:       0.8234 sec (54.9x faster than loop)
Vectorized:  0.0156 sec (2900.3x faster than loop)
```

---

## 8. สรุป Module 3

### 8.1 Key Takeaways

✅ **Data Transformation** เป็นขั้นตอนสำคัญใน ETL Pipeline
✅ **Feature Engineering** สร้าง Features ใหม่จากข้อมูลเดิมเพื่อเพิ่มคุณค่า
✅ **Rolling Window** ใช้สำหรับ Smoothing, Trend Detection, Anomaly Detection
✅ **Performance Optimization** ลดเวลาและ Memory ด้วย dtype, vectorization, chunking
✅ **NumPy เร็วกว่า Pandas** สำหรับการคำนวณบริสุทธิ์ (~2x)
✅ **Chunking** รองรับข้อมูลขนาดใหญ่ที่เกิน RAM
✅ **Vectorization เร็วกว่า Loop** มาก (10-1000x)

### 8.2 Skills ที่ได้จาก Module นี้

| Skill | Level |
|-------|-------|
| Feature Engineering | ⭐⭐⭐ |
| Rolling Window Operations | ⭐⭐⭐ |
| Performance Benchmarking | ⭐⭐⭐ |
| Pandas vs NumPy | ⭐⭐⭐ |
| Chunking Strategy | ⭐⭐ |
| Vectorization | ⭐⭐⭐ |
| Memory Optimization | ⭐⭐ |

### 8.3 Performance Optimization Checklist

เมื่อทำงานกับข้อมูลขนาดใหญ่ ตรวจสอบ:

- [ ] ใช้ dtype ที่เหมาะสม (float32, int8, category)
- [ ] ใช้ Vectorized Operations แทน Loop
- [ ] ใช้ NumPy สำหรับการคำนวณหนัก
- [ ] ใช้ Chunking สำหรับไฟล์ขนาดใหญ่
- [ ] หลีกเลี่ยง apply() เมื่อมี Vectorized alternative
- [ ] Monitor Memory Usage ด้วย `df.memory_usage()`

### 8.4 เตรียมพร้อมสำหรับ Module 4

ใน Module ถัดไป คุณจะได้เรียนรู้:
- Cloud Storage Concepts (Data Lake)
- การใช้ Cloud SDKs
- Data Partitioning Strategy สำหรับ Time-Series

---

## 📝 Challenge Questions

ก่อนไป Module 4 ลองตอบคำถามเหล่านี้:

1. **Rolling Window:** ถ้าข้อมูลมาทุก 5 นาที ต้องการ Moving Average 1 ชั่วโมง ควรใช้ window เท่าไร?
2. **Performance:** ทำไม Vectorized Operations เร็วกว่า Python Loop?
3. **Chunking:** ถ้าไฟล์ CSV 5 GB แต่มี RAM 8 GB ควรใช้ Chunk Size เท่าไร?
4. **NumPy vs Pandas:** ถ้าต้องการคำนวณ Mean ของ 10 ล้าน records ควรใช้อะไร?

---

## 🎯 ถัดไป: Labs & Practical Exercises

พร้อมทำ Lab แล้วหรือยัง?

**👉 [เริ่มทำ Lab Module 3](./labs/lab-module-3.md)**

ใน Lab คุณจะได้:
- Lab 3.1: Implement Rolling Average (5min window)
- Lab 3.2: Benchmark NumPy vs Pandas subtraction
- Lab 3.3: Chunking large CSV files
- Lab 3.4: Vectorization performance comparison

---

**[⬅️ กลับไปที่ Module 2](../module-2/module-2.md)** | **[Module 4: Cloud Integration & Storage ➡️](../module-4/module-4.md)** | **[🏠 กลับไปที่ Wiki หลัก](../wiki.md)**
