# Lab Module 1: Python Fundamentals & Environment

**🎯 วัตถุประสงค์:**
- ติดตั้งและทดสอบ Virtual Environment
- ฝึกใช้งาน Pandas และ NumPy พื้นฐาน
- โหลดและวิเคราะห์ข้อมูล Time-Series

**⏱️ เวลาที่ใช้:** 1 ชั่วโมง

---

## 📋 Pre-requisites

ก่อนเริ่ม Lab ตรวจสอบว่าคุณมี:
- ✅ Python 3.8+ ติดตั้งแล้ว
- ✅ Text Editor หรือ IDE (VS Code, PyCharm, etc.)
- ✅ Terminal/Command Prompt

---

## Lab 1.1: Virtual Environment Setup (15 นาที)

### วัตถุประสงค์
เรียนรู้การสร้างและจัดการ Virtual Environment

### ขั้นตอน

#### Step 1: สร้าง Virtual Environment

```bash
# ตรวจสอบ Python version
python --version

# สร้าง Virtual Environment ชื่อ iot_env
python -m venv iot_env
```

#### Step 2: Activate Environment

```bash
# บน macOS/Linux
source iot_env/bin/activate

# บน Windows
iot_env\Scripts\activate

# ตรวจสอบว่า Activate สำเร็จ
# ควรเห็น (iot_env) หน้า Command Prompt
```

#### Step 3: ติดตั้ง Required Libraries

```bash
# อัปเดต pip
pip install --upgrade pip

# ติดตั้ง Libraries
pip install pandas numpy jupyter

# ตรวจสอบ Libraries ที่ติดตั้ง
pip list
```

**Expected Output:**
```
Package         Version
--------------- -------
pandas          1.3.5
numpy           1.21.6
jupyter         1.0.0
...
```

#### Step 4: สร้างไฟล์ requirements.txt

```bash
# สร้างไฟล์บันทึก Dependencies
pip freeze > requirements.txt

# ดูเนื้อหาในไฟล์
cat requirements.txt  # macOS/Linux
type requirements.txt  # Windows
```

### ✅ Checkpoint 1.1

ตรวจสอบว่า:
- [ ] Virtual Environment สร้างสำเร็จ
- [ ] Activate ได้และเห็น `(iot_env)` หน้า Prompt
- [ ] ติดตั้ง pandas, numpy, jupyter สำเร็จ
- [ ] สร้างไฟล์ `requirements.txt` ได้

---

## Lab 1.2: NumPy Basics (15 นาที)

### วัตถุประสงค์
ทดสอบการใช้งาน NumPy Arrays และ Vectorized Operations

### ขั้นตอน

#### Step 1: สร้างไฟล์ Python Script

สร้างไฟล์ `numpy_test.py` และใส่โค้ดด้านล่าง:

```python
import numpy as np

print("=== Lab 1.2: NumPy Basics ===\n")

# 1. สร้าง NumPy Array
temperatures = np.array([25.5, 26.0, 24.8, 25.2, 26.5, 27.0, 26.8])
print(f"Temperatures (°C): {temperatures}")
print(f"Data Type: {temperatures.dtype}")
print(f"Array Shape: {temperatures.shape}")
print(f"Number of elements: {temperatures.size}\n")

# 2. Memory Optimization
print("=== Memory Comparison ===")
temp_float64 = np.array([25, 26, 24, 25, 26, 27, 26], dtype=np.float64)
temp_float32 = np.array([25, 26, 24, 25, 26, 27, 26], dtype=np.float32)
temp_int8 = np.array([25, 26, 24, 25, 26, 27, 26], dtype=np.int8)

print(f"float64 memory: {temp_float64.nbytes} bytes")
print(f"float32 memory: {temp_float32.nbytes} bytes")
print(f"int8 memory: {temp_int8.nbytes} bytes")

memory_saved = temp_float64.nbytes - temp_int8.nbytes
saving_percent = (memory_saved / temp_float64.nbytes) * 100
print(f"Memory saved (float64 → int8): {memory_saved} bytes ({saving_percent:.1f}%)\n")

# 3. Vectorized Operations
print("=== Vectorized Operations ===")
celsius = np.array([0, 10, 20, 30, 40])
fahrenheit = (celsius * 9/5) + 32

print(f"Celsius:    {celsius}")
print(f"Fahrenheit: {fahrenheit}\n")

# 4. Statistical Operations
print("=== Statistical Operations ===")
print(f"Mean Temperature: {temperatures.mean():.2f}°C")
print(f"Max Temperature: {temperatures.max():.2f}°C")
print(f"Min Temperature: {temperatures.min():.2f}°C")
print(f"Std Deviation: {temperatures.std():.2f}°C")
```

#### Step 2: รัน Script

```bash
python numpy_test.py
```

**Expected Output:**
```
=== Lab 1.2: NumPy Basics ===

Temperatures (°C): [25.5 26.  24.8 25.2 26.5 27.  26.8]
Data Type: float64
Array Shape: (7,)
Number of elements: 7

=== Memory Comparison ===
float64 memory: 56 bytes
float32 memory: 28 bytes
int8 memory: 7 bytes
Memory saved (float64 → int8): 49 bytes (87.5%)

=== Vectorized Operations ===
Celsius:    [ 0 10 20 30 40]
Fahrenheit: [ 32.  50.  68.  86. 104.]

=== Statistical Operations ===
Mean Temperature: 26.00°C
Max Temperature: 27.00°C
Min Temperature: 24.80°C
Std Deviation: 0.73°C
```

### ✅ Checkpoint 1.2

ตรวจสอบว่า:
- [ ] สร้าง NumPy Array ได้สำเร็จ
- [ ] เข้าใจ Memory Optimization (dtype)
- [ ] ใช้ Vectorized Operations ได้
- [ ] คำนวณค่าสถิติ (mean, max, min, std) ได้

---

## Lab 1.3: Pandas DataFrame Basics (15 นาที)

### วัตถุประสงค์
ฝึกสร้างและจัดการ DataFrame

### ขั้นตอน

#### Step 1: สร้างไฟล์ `pandas_test.py`

```python
import pandas as pd
import numpy as np

print("=== Lab 1.3: Pandas DataFrame Basics ===\n")

# 1. สร้าง DataFrame จาก Dictionary
sensor_data = {
    'timestamp': [
        '2025-01-01 00:00', '2025-01-01 00:05', '2025-01-01 00:10',
        '2025-01-01 00:15', '2025-01-01 00:20', '2025-01-01 00:25'
    ],
    'sensor_id': ['S001', 'S001', 'S001', 'S002', 'S002', 'S002'],
    'temperature': [25.5, 26.0, 24.8, 22.3, 23.1, 22.8],
    'humidity': [60, 62, 58, 55, 57, 56]
}

df = pd.DataFrame(sensor_data)
print("Original DataFrame:")
print(df)
print(f"\nDataFrame Shape: {df.shape}")
print(f"Columns: {list(df.columns)}")
print(f"\nData Types:\n{df.dtypes}\n")

# 2. แปลง timestamp เป็น datetime
df['timestamp'] = pd.to_datetime(df['timestamp'])
print("After datetime conversion:")
print(df.dtypes)
print()

# 3. Data Selection
print("=== Data Selection ===")
print("Temperature column:")
print(df['temperature'])
print()

print("Rows where temperature > 24:")
print(df[df['temperature'] > 24])
print()

# 4. Aggregation
print("=== Aggregation by Sensor ===")
sensor_stats = df.groupby('sensor_id').agg({
    'temperature': ['mean', 'min', 'max'],
    'humidity': ['mean', 'min', 'max']
})
print(sensor_stats)
print()

# 5. Feature Engineering
print("=== Feature Engineering ===")
df['temp_fahrenheit'] = (df['temperature'] * 9/5) + 32
df['temp_category'] = pd.cut(df['temperature'],
                              bins=[0, 23, 25, 30],
                              labels=['Cool', 'Normal', 'Warm'])
print("DataFrame with new features:")
print(df[['sensor_id', 'temperature', 'temp_fahrenheit', 'temp_category']])
```

#### Step 2: รัน Script

```bash
python pandas_test.py
```

### ✅ Checkpoint 1.3

ตรวจสอบว่า:
- [ ] สร้าง DataFrame จาก Dictionary ได้
- [ ] แปลง timestamp เป็น datetime type ได้
- [ ] Filter และเลือกข้อมูลได้
- [ ] ทำ Aggregation (groupby) ได้
- [ ] เพิ่ม Feature ใหม่ได้

---

## Lab 1.4: Time-Series Data Analysis (15 นาที)

### วัตถุประสงค์
ทำงานกับ Time-Series Data จริง

### ขั้นตอน

#### Step 1: สร้าง Sample Time-Series Data

สร้างไฟล์ `timeseries_test.py`:

```python
import pandas as pd
import numpy as np

print("=== Lab 1.4: Time-Series Data Analysis ===\n")

# 1. สร้าง Time-Series Data (1 วัน, ทุก 5 นาที)
date_range = pd.date_range(start='2025-01-01',
                           end='2025-01-02',
                           freq='5min')

# สร้างข้อมูลสุ่มแบบ realistic (มี pattern)
np.random.seed(42)
n_records = len(date_range)

# Base temperature + random noise + daily pattern
hour_of_day = date_range.hour
daily_pattern = 3 * np.sin((hour_of_day - 6) * np.pi / 12)  # Peak ตอนบ่าย
temperature = 25 + daily_pattern + np.random.normal(0, 0.5, n_records)

df = pd.DataFrame({
    'timestamp': date_range,
    'sensor_id': 'S001',
    'temperature': temperature
})

print(f"Total records: {len(df)}")
print(f"Date range: {df['timestamp'].min()} to {df['timestamp'].max()}")
print(f"\nFirst 10 records:")
print(df.head(10))
print()

# 2. Set timestamp as index
df.set_index('timestamp', inplace=True)

# 3. Resampling - คำนวณค่าเฉลี่ยทุก 1 ชั่วโมง
print("=== Resampling to 1 Hour ===")
df_hourly = df[['temperature']].resample('1h').mean()
print(df_hourly.head(10))
print()

# 4. Rolling Window - Moving Average 12 periods (1 ชั่วโมง)
print("=== Rolling Average (12 periods = 1 hour) ===")
df['temp_ma_1h'] = df['temperature'].rolling(window=12).mean()
print(df[['temperature', 'temp_ma_1h']].head(15))
print()

# 5. Time-based Statistics
print("=== Daily Statistics ===")
print(f"Daily Mean: {df['temperature'].mean():.2f}°C")
print(f"Daily Max: {df['temperature'].max():.2f}°C")
print(f"Daily Min: {df['temperature'].min():.2f}°C")
print(f"Temperature Range: {df['temperature'].max() - df['temperature'].min():.2f}°C")
```

#### Step 2: รัน Script

```bash
python timeseries_test.py
```

### ✅ Checkpoint 1.4

ตรวจสอบว่า:
- [ ] สร้าง Time-Series Data ด้วย `pd.date_range()` ได้
- [ ] ตั้ง timestamp เป็น Index ได้
- [ ] ทำ Resampling (เปลี่ยน Frequency) ได้
- [ ] คำนวณ Rolling Average ได้
- [ ] วิเคราะห์ข้อมูล Time-Series ได้

---

## 🏆 Challenge Exercise (เพิ่มเติม)

### Challenge 1: Memory Optimization Calculator

สร้าง script คำนวณว่าถ้ามีข้อมูล Temperature 1 ล้าน records จะประหยัด Memory ได้เท่าไรระหว่าง `float64` กับ `float32`

**Expected Output:**
```
Records: 1,000,000
float64 memory: 8.0 MB
float32 memory: 4.0 MB
Memory saved: 4.0 MB (50.0%)
```

<details>
<summary>💡 Solution (คลิกเพื่อดู)</summary>

```python
import numpy as np

n_records = 1_000_000

# สร้าง dummy data
data_float64 = np.random.rand(n_records).astype(np.float64)
data_float32 = np.random.rand(n_records).astype(np.float32)

mem_float64 = data_float64.nbytes / (1024 * 1024)  # Convert to MB
mem_float32 = data_float32.nbytes / (1024 * 1024)

saved = mem_float64 - mem_float32
percent = (saved / mem_float64) * 100

print(f"Records: {n_records:,}")
print(f"float64 memory: {mem_float64:.1f} MB")
print(f"float32 memory: {mem_float32:.1f} MB")
print(f"Memory saved: {saved:.1f} MB ({percent:.1f}%)")
```
</details>

### Challenge 2: IoT Data Generator

สร้าง function สำหรับ generate IoT sensor data สำหรับหลาย sensors

**Requirements:**
- รองรับหลาย sensor_ids
- สร้างข้อมูล 1 วัน (ทุก 5 นาที)
- แต่ละ sensor มี temperature pattern ต่างกัน
- บันทึกลง CSV file

<details>
<summary>💡 Solution (คลิกเพื่อดู)</summary>

```python
import pandas as pd
import numpy as np

def generate_iot_data(sensor_ids, start_date='2025-01-01', freq='5min'):
    """
    Generate IoT sensor data for multiple sensors

    Args:
        sensor_ids: List of sensor IDs
        start_date: Start date
        freq: Data frequency

    Returns:
        DataFrame with IoT data
    """
    date_range = pd.date_range(start=start_date,
                               end=pd.to_datetime(start_date) + pd.Timedelta(days=1),
                               freq=freq)

    all_data = []

    for sensor_id in sensor_ids:
        np.random.seed(hash(sensor_id) % (2**32))  # Different seed per sensor
        n_records = len(date_range)

        hour_of_day = date_range.hour
        daily_pattern = 3 * np.sin((hour_of_day - 6) * np.pi / 12)
        base_temp = 25 + (hash(sensor_id) % 5)  # Different base temp

        temperature = base_temp + daily_pattern + np.random.normal(0, 0.5, n_records)
        humidity = 60 + np.random.normal(0, 5, n_records)

        sensor_df = pd.DataFrame({
            'timestamp': date_range,
            'sensor_id': sensor_id,
            'temperature': temperature,
            'humidity': humidity
        })

        all_data.append(sensor_df)

    df = pd.concat(all_data, ignore_index=True)
    df = df.sort_values('timestamp').reset_index(drop=True)

    return df

# Test
sensors = ['S001', 'S002', 'S003']
df = generate_iot_data(sensors)

print(f"Total records: {len(df)}")
print(f"Sensors: {df['sensor_id'].unique()}")
print(f"\nSample data:")
print(df.head(10))

# Save to CSV
df.to_csv('iot_sample_data.csv', index=False)
print("\n✅ Data saved to iot_sample_data.csv")
```
</details>

---

## 📊 Lab Summary

### สิ่งที่คุณได้เรียนรู้

✅ **Lab 1.1:** สร้างและจัดการ Virtual Environment
✅ **Lab 1.2:** ใช้งาน NumPy Arrays และ Vectorized Operations
✅ **Lab 1.3:** สร้างและจัดการ Pandas DataFrame
✅ **Lab 1.4:** วิเคราะห์ Time-Series Data

### Skills Acquired

| Skill | Confidence Level |
|-------|------------------|
| Virtual Environment | ⭐⭐⭐ |
| NumPy Operations | ⭐⭐⭐ |
| Pandas DataFrame | ⭐⭐⭐ |
| Time-Series Analysis | ⭐⭐ |

---

## 🎯 ถัดไป

คุณพร้อมสำหรับ Module 2 แล้ว!

**👉 [Module 2: ETL - Data Loading & Cleansing](../../module-2/module-2.md)**

---

## 📞 Need Help?

หากติดปัญหา:
1. ตรวจสอบ Python version (`python --version`)
2. ตรวจสอบว่า Virtual Environment activate แล้ว
3. ตรวจสอบ Libraries ติดตั้งครบ (`pip list`)

---

**[⬅️ กลับไป Module 1](../module-1.md)** | **[🏠 กลับไปหน้าหลัก](../../wiki.md)**
