# Lab Module 2: ETL - Data Loading & Cleansing

**🎯 วัตถุประสงค์:**
- ฝึกโหลดข้อมูล CSV ขนาดใหญ่แบบ Optimized
- คำนวณ Memory Saving จาก dtype optimization
- ฝึกจัดการ Missing Values ด้วยเทคนิคต่างๆ
- ตรวจสอบ Data Integrity และสร้าง Quality Report

**⏱️ เวลาที่ใช้:** 1.5 ชั่วโมง

**📋 เตรียมความพร้อม:**
- Python 3.8+
- Pandas, NumPy ติดตั้งแล้ว
- Jupyter Notebook หรือ Python IDE

---

## 📚 สารบัญ

1. [Setup & Data Preparation](#setup--data-preparation)
2. [Lab 2.1: Optimized CSV Loading](#lab-21-optimized-csv-loading)
3. [Lab 2.2: Missing Value Handling](#lab-22-missing-value-handling)
4. [Lab 2.3: Data Validation & Integrity](#lab-23-data-validation--integrity)
5. [Challenge: Complete ETL Pipeline](#challenge-complete-etl-pipeline)
6. [Checkpoint & Submission](#checkpoint--submission)

---

## Setup & Data Preparation

### Step 1: สร้าง Lab Directory

```bash
# สร้าง directory สำหรับ Lab
mkdir -p ~/data-engineer-labs/module-2
cd ~/data-engineer-labs/module-2

# สร้าง data directory
mkdir data
```

### Step 2: สร้างข้อมูลตัวอย่าง

สร้างไฟล์ `generate_sensor_data.py`:

```python
"""
Generate sensor_large.csv for Lab 2
"""
import pandas as pd
import numpy as np
from datetime import datetime, timedelta

def generate_sensor_data(num_records=100000):
    """
    สร้างข้อมูล IoT Sensor แบบจำลอง

    Parameters:
        num_records: จำนวน records (default: 100,000)

    Output:
        data/sensor_large.csv
    """
    print(f"Generating {num_records:,} sensor records...")

    # ตั้งค่า random seed สำหรับ reproducibility
    np.random.seed(42)

    # สร้าง timestamps (ทุก 5 นาที)
    start_date = datetime(2025, 1, 1, 0, 0, 0)
    timestamps = [start_date + timedelta(minutes=5*i) for i in range(num_records)]

    # สร้าง sensor IDs (20 sensors)
    sensor_ids = [f"S{i:03d}" for i in range(1, 21)]
    sensor_data = np.random.choice(sensor_ids, num_records)

    # สร้างข้อมูล temperature (15-35°C with pattern)
    base_temp = 25.0
    temp_variation = np.sin(np.linspace(0, 10*np.pi, num_records)) * 5  # Daily pattern
    temp_noise = np.random.normal(0, 1, num_records)
    temperature = base_temp + temp_variation + temp_noise

    # สร้างข้อมูล humidity (40-80% with correlation to temp)
    base_humidity = 60.0
    humidity_variation = -temp_variation * 0.8  # Inverse correlation
    humidity_noise = np.random.normal(0, 2, num_records)
    humidity = base_humidity + humidity_variation + humidity_noise
    humidity = np.clip(humidity, 0, 100).astype(int)

    # สร้างข้อมูล pressure (1000-1020 hPa)
    pressure = np.random.normal(1013, 5, num_records)

    # สร้าง status (OK, WARNING, ERROR)
    status_probs = [0.85, 0.10, 0.05]  # 85% OK, 10% WARNING, 5% ERROR
    status = np.random.choice(['OK', 'WARNING', 'ERROR'], num_records, p=status_probs)

    # สร้าง DataFrame
    df = pd.DataFrame({
        'timestamp': timestamps,
        'sensor_id': sensor_data,
        'temperature': temperature,
        'humidity': humidity,
        'pressure': pressure,
        'status': status
    })

    # แทรก Missing Values (~5%)
    missing_indices = np.random.choice(num_records, size=int(num_records * 0.05), replace=False)
    df.loc[missing_indices[:len(missing_indices)//3], 'temperature'] = np.nan
    df.loc[missing_indices[len(missing_indices)//3:2*len(missing_indices)//3], 'humidity'] = np.nan
    df.loc[missing_indices[2*len(missing_indices)//3:], 'pressure'] = np.nan

    # แทรก Outliers (~1%)
    outlier_indices = np.random.choice(num_records, size=int(num_records * 0.01), replace=False)
    df.loc[outlier_indices, 'temperature'] = np.random.choice([150, -100, 999])

    # แทรก Duplicates (~0.5%)
    dup_indices = np.random.choice(num_records-100, size=int(num_records * 0.005), replace=False)
    duplicates = df.iloc[dup_indices].copy()
    df = pd.concat([df, duplicates], ignore_index=True)
    df = df.sample(frac=1).reset_index(drop=True)  # Shuffle

    # บันทึกไฟล์
    output_path = 'data/sensor_large.csv'
    df.to_csv(output_path, index=False)

    file_size = os.path.getsize(output_path) / 1024**2  # MB
    print(f"✅ Generated {len(df):,} records")
    print(f"✅ File size: {file_size:.2f} MB")
    print(f"✅ Saved to: {output_path}")

    return df

if __name__ == "__main__":
    import os
    df = generate_sensor_data(100000)

    # แสดง Preview
    print("\nData Preview:")
    print(df.head(10))

    print("\nData Info:")
    print(df.info(memory_usage='deep'))

    print("\nMissing Values:")
    print(df.isnull().sum())
```

### Step 3: รันสคริปต์สร้างข้อมูล

```bash
python generate_sensor_data.py
```

**Expected Output:**
```
Generating 100,000 sensor records...
✅ Generated 100,500 records
✅ File size: 8.23 MB
✅ Saved to: data/sensor_large.csv
```

---

## Lab 2.1: Optimized CSV Loading

### 🎯 เป้าหมาย
โหลด `sensor_large.csv` แบบ Optimized และคำนวณ Memory Saving

### Step 1: โหลดแบบปกติ (Baseline)

สร้างไฟล์ `lab_2_1_loading.py`:

```python
"""
Lab 2.1: Optimized CSV Loading
"""
import pandas as pd
import numpy as np
import time

# ===========================
# Part 1: Normal Loading
# ===========================

print("=" * 60)
print("Part 1: Normal Loading (Baseline)")
print("=" * 60)

start_time = time.time()

# โหลดแบบปกติ - ไม่กำหนด dtype
df_normal = pd.read_csv('data/sensor_large.csv')

load_time_normal = time.time() - start_time

# วัด Memory Usage
memory_normal = df_normal.memory_usage(deep=True).sum() / 1024**2  # MB

print(f"\n📊 Loading Results (Normal):")
print(f"   Rows:        {len(df_normal):,}")
print(f"   Columns:     {len(df_normal.columns)}")
print(f"   Load Time:   {load_time_normal:.3f} seconds")
print(f"   Memory:      {memory_normal:.2f} MB")

print(f"\n📋 Data Types:")
print(df_normal.dtypes)

print(f"\n📈 Memory Usage per Column:")
print(df_normal.memory_usage(deep=True) / 1024**2)  # MB
```

**Expected Output:**
```
============================================================
Part 1: Normal Loading (Baseline)
============================================================

📊 Loading Results (Normal):
   Rows:        100,500
   Columns:     6
   Load Time:   0.245 seconds
   Memory:      8.12 MB

📋 Data Types:
timestamp      object
sensor_id      object
temperature    float64
humidity       int64
pressure       float64
status         object
dtype: object

📈 Memory Usage per Column:
Index          0.77
timestamp      6.14
sensor_id      6.14
temperature    0.77
humidity       0.77
pressure       0.77
status         6.14
dtype: float64
```

### Step 2: โหลดแบบ Optimized

เพิ่ม code ต่อใน `lab_2_1_loading.py`:

```python
# ===========================
# Part 2: Optimized Loading
# ===========================

print("\n" + "=" * 60)
print("Part 2: Optimized Loading")
print("=" * 60)

start_time = time.time()

# กำหนด dtype specification
dtype_spec = {
    'sensor_id': 'category',      # 20 unique values → category
    'temperature': 'float32',     # ไม่ต้องการความละเอียดสูง
    'humidity': 'int16',          # 0-100 → int16
    'pressure': 'float32',        # ไม่ต้องการความละเอียดสูง
    'status': 'category'          # 3 unique values → category
}

# โหลดแบบ Optimized
df_optimized = pd.read_csv(
    'data/sensor_large.csv',
    dtype=dtype_spec,
    parse_dates=['timestamp']     # แปลง timestamp เป็น datetime
)

load_time_optimized = time.time() - start_time

# วัด Memory Usage
memory_optimized = df_optimized.memory_usage(deep=True).sum() / 1024**2  # MB

print(f"\n📊 Loading Results (Optimized):")
print(f"   Rows:        {len(df_optimized):,}")
print(f"   Columns:     {len(df_optimized.columns)}")
print(f"   Load Time:   {load_time_optimized:.3f} seconds")
print(f"   Memory:      {memory_optimized:.2f} MB")

print(f"\n📋 Data Types:")
print(df_optimized.dtypes)

print(f"\n📈 Memory Usage per Column:")
print(df_optimized.memory_usage(deep=True) / 1024**2)  # MB
```

### Step 3: คำนวณ Savings

เพิ่ม code ต่อ:

```python
# ===========================
# Part 3: Comparison
# ===========================

print("\n" + "=" * 60)
print("Part 3: Comparison & Savings")
print("=" * 60)

# คำนวณ Savings
memory_saving = memory_normal - memory_optimized
memory_saving_percent = (memory_saving / memory_normal) * 100

time_diff = load_time_normal - load_time_optimized
time_diff_percent = (time_diff / load_time_normal) * 100

print(f"\n💾 Memory Comparison:")
print(f"   Normal:        {memory_normal:.2f} MB")
print(f"   Optimized:     {memory_optimized:.2f} MB")
print(f"   Saving:        {memory_saving:.2f} MB ({memory_saving_percent:.1f}%)")

print(f"\n⏱️  Time Comparison:")
print(f"   Normal:        {load_time_normal:.3f} seconds")
print(f"   Optimized:     {load_time_optimized:.3f} seconds")
print(f"   Difference:    {time_diff:.3f} seconds ({time_diff_percent:.1f}%)")

# สร้าง Summary Table
summary = pd.DataFrame({
    'Metric': ['Rows', 'Memory (MB)', 'Load Time (s)'],
    'Normal': [len(df_normal), f"{memory_normal:.2f}", f"{load_time_normal:.3f}"],
    'Optimized': [len(df_optimized), f"{memory_optimized:.2f}", f"{load_time_optimized:.3f}"],
    'Saving (%)': [
        '0%',
        f"{memory_saving_percent:.1f}%",
        f"{time_diff_percent:.1f}%" if time_diff > 0 else '0%'
    ]
})

print(f"\n📊 Summary Table:")
print(summary.to_string(index=False))

# Visualization (สำหรับ Jupyter)
print(f"\n💡 Optimization Tips:")
print("   ✅ Use 'category' for repeated strings")
print("   ✅ Use float32 instead of float64 (save 50%)")
print("   ✅ Use int8/16 for small ranges")
print("   ✅ Parse dates with parse_dates parameter")
```

**Expected Output:**
```
============================================================
Part 3: Comparison & Savings
============================================================

💾 Memory Comparison:
   Normal:        8.12 MB
   Optimized:     1.89 MB
   Saving:        6.23 MB (76.7%)

⏱️  Time Comparison:
   Normal:        0.245 seconds
   Optimized:     0.198 seconds
   Difference:    0.047 seconds (19.2%)

📊 Summary Table:
   Metric  Normal Optimized Saving (%)
     Rows  100500   100500         0%
Memory (MB)   8.12     1.89      76.7%
Load Time (s)  0.245    0.198      19.2%
```

### 📝 Checkpoint 2.1

**คำถาม:**
1. Memory Saving ของคุณคือเท่าไหร่? (ควรได้ >70%)
2. Column ไหนประหยัด Memory มากที่สุด? ทำไม?
3. ถ้าเปลี่ยน `sensor_id` จาก category เป็น object จะเกิดอะไรขึ้น?

**ทดสอบ:**
```python
# ทดสอบเปลี่ยน dtype
df_test = df_optimized.copy()
df_test['sensor_id'] = df_test['sensor_id'].astype('object')
memory_test = df_test.memory_usage(deep=True).sum() / 1024**2
print(f"Memory with object: {memory_test:.2f} MB")
print(f"Memory increase: {((memory_test - memory_optimized) / memory_optimized * 100):.1f}%")
```

---

## Lab 2.2: Missing Value Handling

### 🎯 เป้าหมาย
ฝึกจัดการ Missing Values ด้วยเทคนิคต่างๆ และเปรียบเทียบผลลัพธ์

### Step 1: ตรวจสอบ Missing Values

สร้างไฟล์ `lab_2_2_missing_values.py`:

```python
"""
Lab 2.2: Missing Value Handling
"""
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

# โหลดข้อมูล (แบบ Optimized)
dtype_spec = {
    'sensor_id': 'category',
    'temperature': 'float32',
    'humidity': 'int16',
    'pressure': 'float32',
    'status': 'category'
}

df = pd.read_csv(
    'data/sensor_large.csv',
    dtype=dtype_spec,
    parse_dates=['timestamp']
)

# ตั้ง timestamp เป็น index
df.set_index('timestamp', inplace=True)
df.sort_index(inplace=True)

print("=" * 60)
print("Lab 2.2: Missing Value Analysis")
print("=" * 60)

# ===========================
# Part 1: Detect Missing Values
# ===========================

print("\n📊 Part 1: Missing Value Detection")
print("-" * 60)

# นับจำนวน Missing
missing_count = df.isnull().sum()
missing_percent = (df.isnull().sum() / len(df) * 100).round(2)

missing_report = pd.DataFrame({
    'Column': df.columns,
    'Missing Count': missing_count.values,
    'Missing %': missing_percent.values
})

print("\n📋 Missing Values Report:")
print(missing_report.to_string(index=False))

# แสดงแถวที่มี Missing
print(f"\n🔍 Total rows with missing values: {df.isnull().any(axis=1).sum():,}")

# แสดงตัวอย่างแถวที่มี Missing
print("\n📄 Sample rows with missing values:")
print(df[df.isnull().any(axis=1)].head(10))
```

### Step 2: ทดลองเทคนิคต่างๆ

เพิ่ม code:

```python
# ===========================
# Part 2: Handling Strategies
# ===========================

print("\n" + "=" * 60)
print("Part 2: Missing Value Handling Strategies")
print("=" * 60)

# เลือกเฉพาะ temperature column สำหรับทดลอง
temp_original = df['temperature'].copy()
temp_with_missing = temp_original.isnull().sum()

print(f"\n🌡️  Temperature Column:")
print(f"   Total values:    {len(temp_original):,}")
print(f"   Missing values:  {temp_with_missing:,} ({temp_with_missing/len(temp_original)*100:.2f}%)")

# Strategy 1: Drop
print("\n" + "-" * 60)
print("Strategy 1: DROP")
print("-" * 60)

temp_dropped = temp_original.dropna()
print(f"   Remaining values: {len(temp_dropped):,}")
print(f"   Lost values:      {len(temp_original) - len(temp_dropped):,}")
print(f"   Mean (before):    {temp_original.mean():.2f}°C")
print(f"   Mean (after):     {temp_dropped.mean():.2f}°C")

# Strategy 2: Forward Fill
print("\n" + "-" * 60)
print("Strategy 2: FORWARD FILL (Ffill)")
print("-" * 60)

temp_ffill = temp_original.fillna(method='ffill')
print(f"   Remaining NaN:    {temp_ffill.isnull().sum()}")
print(f"   Mean:             {temp_ffill.mean():.2f}°C")
print(f"   Median:           {temp_ffill.median():.2f}°C")

# แสดงตัวอย่างการเติม
sample_idx = temp_original[temp_original.isnull()].index[0]
window = 5
print(f"\n   Example (forward fill):")
comparison = pd.DataFrame({
    'Original': temp_original.loc[sample_idx-pd.Timedelta(minutes=window):sample_idx+pd.Timedelta(minutes=window)],
    'Ffill': temp_ffill.loc[sample_idx-pd.Timedelta(minutes=window):sample_idx+pd.Timedelta(minutes=window)]
})
print(comparison)

# Strategy 3: Mean Fill
print("\n" + "-" * 60)
print("Strategy 3: MEAN FILL")
print("-" * 60)

temp_mean = temp_original.fillna(temp_original.mean())
print(f"   Remaining NaN:    {temp_mean.isnull().sum()}")
print(f"   Mean:             {temp_mean.mean():.2f}°C")
print(f"   Fill value used:  {temp_original.mean():.2f}°C")

# Strategy 4: Median Fill
print("\n" + "-" * 60)
print("Strategy 4: MEDIAN FILL")
print("-" * 60)

temp_median = temp_original.fillna(temp_original.median())
print(f"   Remaining NaN:    {temp_median.isnull().sum()}")
print(f"   Median:           {temp_median.median():.2f}°C")
print(f"   Fill value used:  {temp_original.median():.2f}°C")

# Strategy 5: Interpolation
print("\n" + "-" * 60)
print("Strategy 5: INTERPOLATION (Linear)")
print("-" * 60)

temp_interpolated = temp_original.interpolate(method='linear')
print(f"   Remaining NaN:    {temp_interpolated.isnull().sum()}")
print(f"   Mean:             {temp_interpolated.mean():.2f}°C")

# แสดงตัวอย่างการ interpolate
print(f"\n   Example (interpolation):")
comparison = pd.DataFrame({
    'Original': temp_original.loc[sample_idx-pd.Timedelta(minutes=window):sample_idx+pd.Timedelta(minutes=window)],
    'Interpolated': temp_interpolated.loc[sample_idx-pd.Timedelta(minutes=window):sample_idx+pd.Timedelta(minutes=window)]
})
print(comparison)
```

### Step 3: เปรียบเทียบผลลัพธ์

```python
# ===========================
# Part 3: Comparison
# ===========================

print("\n" + "=" * 60)
print("Part 3: Strategy Comparison")
print("=" * 60)

# สร้าง comparison table
strategies = {
    'Original': temp_original,
    'Drop': temp_dropped,
    'Ffill': temp_ffill,
    'Mean': temp_mean,
    'Median': temp_median,
    'Interpolate': temp_interpolated
}

comparison_df = pd.DataFrame({
    'Strategy': strategies.keys(),
    'Count': [len(s) for s in strategies.values()],
    'Missing': [s.isnull().sum() for s in strategies.values()],
    'Mean': [f"{s.mean():.2f}" for s in strategies.values()],
    'Median': [f"{s.median():.2f}" for s in strategies.values()],
    'Std': [f"{s.std():.2f}" for s in strategies.values()],
    'Min': [f"{s.min():.2f}" for s in strategies.values()],
    'Max': [f"{s.max():.2f}" for s in strategies.values()]
})

print("\n📊 Comparison Table:")
print(comparison_df.to_string(index=False))

# Recommendations
print("\n" + "=" * 60)
print("💡 Recommendations")
print("=" * 60)

print("""
1. DROP:
   ✅ Use when: Missing < 5%
   ❌ Avoid when: Losing too much data

2. FORWARD FILL (Ffill):
   ✅ Use when: Time-series with short gaps
   ❌ Avoid when: Long gaps (propagates old values)

3. MEAN FILL:
   ✅ Use when: No time dependency
   ❌ Avoid when: Time-series (ignores trends)

4. MEDIAN FILL:
   ✅ Use when: Data has outliers
   ❌ Avoid when: Time-series (ignores trends)

5. INTERPOLATION:
   ✅ Use when: Time-series with smooth trends
   ❌ Avoid when: Irregular patterns

Recommendation for IoT Sensor Data:
→ Use INTERPOLATION or FFILL for temperature/humidity
→ Consider time gaps: if gap > 30 min, use interpolation
""")
```

### Step 4: ประยุกต์กับข้อมูลทั้งหมด

```python
# ===========================
# Part 4: Apply to Full Dataset
# ===========================

print("\n" + "=" * 60)
print("Part 4: Apply Strategy to Full Dataset")
print("=" * 60)

def clean_sensor_data(df, strategy='auto'):
    """
    ทำความสะอาดข้อมูล sensor แบบอัตโนมัติ
    """
    df_clean = df.copy()

    print(f"\n🧹 Cleaning with strategy: {strategy}")
    print("-" * 60)

    # Temperature & Pressure: Interpolation (time-series)
    print("   Temperature: Interpolation")
    df_clean['temperature'] = df_clean['temperature'].interpolate(method='linear')

    print("   Pressure: Interpolation")
    df_clean['pressure'] = df_clean['pressure'].interpolate(method='linear')

    # Humidity: Forward Fill then Interpolate
    print("   Humidity: Ffill + Interpolation")
    df_clean['humidity'] = df_clean['humidity'].fillna(method='ffill')
    df_clean['humidity'] = df_clean['humidity'].interpolate(method='linear')

    # ตรวจสอบ remaining NaN
    remaining_nan = df_clean.isnull().sum().sum()
    print(f"\n   ✅ Remaining NaN: {remaining_nan}")

    return df_clean

# ทำความสะอาด
df_cleaned = clean_sensor_data(df, strategy='auto')

print("\n📊 Before vs After:")
print("\nBefore:")
print(df.isnull().sum())

print("\nAfter:")
print(df_cleaned.isnull().sum())

# บันทึกผลลัพธ์
df_cleaned.to_csv('data/sensor_cleaned.csv')
print(f"\n💾 Saved cleaned data to: data/sensor_cleaned.csv")
```

### 📝 Checkpoint 2.2

**คำถาม:**
1. Strategy ไหนเหมาะกับ Temperature มากที่สุด? ทำไม?
2. Mean vs Median - แตกต่างกันอย่างไร? เมื่อไหร่ควรใช้ Median?
3. Ffill vs Interpolation - อะไรคือข้อดีข้อเสียของแต่ละวิธี?

**ทดสอบ:**
```python
# ทดสอบกับข้อมูลที่มี gap ยาว
# สร้าง missing gap 2 ชั่วโมง (24 records)
test_series = temp_original.copy()
gap_start = 1000
test_series.iloc[gap_start:gap_start+24] = np.nan

# ทดลอง Ffill vs Interpolate
ffill_gap = test_series.fillna(method='ffill')
interp_gap = test_series.interpolate(method='linear')

print(f"Ffill (gap): {ffill_gap.iloc[gap_start:gap_start+24].values[:5]}")
print(f"Interpolate (gap): {interp_gap.iloc[gap_start:gap_start+24].values[:5]}")
```

---

## Lab 2.3: Data Validation & Integrity

### 🎯 เป้าหมาย
สร้างระบบตรวจสอบ Data Quality และ Integrity อัตโนมัติ

### Step 1: Basic Validation Checks

สร้างไฟล์ `lab_2_3_validation.py`:

```python
"""
Lab 2.3: Data Validation & Integrity Checks
"""
import pandas as pd
import numpy as np
import json

# โหลดข้อมูลที่ cleaned แล้ว
df = pd.read_csv(
    'data/sensor_cleaned.csv',
    parse_dates=['timestamp'],
    index_col='timestamp'
)

print("=" * 60)
print("Lab 2.3: Data Validation & Integrity")
print("=" * 60)

# ===========================
# Part 1: Data Type Validation
# ===========================

print("\n📋 Part 1: Data Type Validation")
print("-" * 60)

# กำหนด expected types
expected_types = {
    'sensor_id': 'object',
    'temperature': 'float',
    'humidity': 'int',
    'pressure': 'float',
    'status': 'object'
}

print("\n🔍 Type Check Results:")
type_check_passed = True

for col, expected in expected_types.items():
    actual = str(df[col].dtype)
    is_match = False

    if expected == 'float' and 'float' in actual:
        is_match = True
    elif expected == 'int' and 'int' in actual:
        is_match = True
    elif expected == 'object' and 'object' in actual:
        is_match = True
    elif expected == 'category' and 'category' in actual:
        is_match = True

    status = "✅ PASS" if is_match else "❌ FAIL"
    print(f"   {col:15s} Expected: {expected:10s} | Actual: {actual:10s} | {status}")

    if not is_match:
        type_check_passed = False

print(f"\n{'✅ All type checks passed' if type_check_passed else '❌ Some type checks failed'}")
```

### Step 2: Range Validation

```python
# ===========================
# Part 2: Range Validation
# ===========================

print("\n" + "=" * 60)
print("Part 2: Value Range Validation")
print("=" * 60)

# กำหนด valid ranges
range_rules = {
    'temperature': (-50, 100),   # °C
    'humidity': (0, 100),        # %
    'pressure': (900, 1100)      # hPa
}

print("\n🔍 Range Check Results:")
range_violations = {}

for col, (min_val, max_val) in range_rules.items():
    # หาค่าที่อยู่นอกช่วง
    out_of_range = df[
        (df[col] < min_val) | (df[col] > max_val)
    ]

    if len(out_of_range) > 0:
        range_violations[col] = out_of_range
        print(f"   ❌ {col:15s} [{min_val:6.1f}, {max_val:6.1f}] → {len(out_of_range):,} violations")
        print(f"      Actual range: [{df[col].min():.2f}, {df[col].max():.2f}]")

        # แสดงตัวอย่าง
        print(f"      Sample violations:")
        print(out_of_range[[col]].head(3).to_string())
    else:
        print(f"   ✅ {col:15s} [{min_val:6.1f}, {max_val:6.1f}] → All values valid")

# จัดการ Outliers
if range_violations:
    print("\n💡 Outlier Handling:")
    print("   Option 1: Remove outliers")
    print("   Option 2: Clip to range")
    print("   Option 3: Replace with NaN then interpolate")

    # ตัวอย่าง: Clip to range
    df_cleaned = df.copy()
    for col, (min_val, max_val) in range_rules.items():
        df_cleaned[col] = df_cleaned[col].clip(min_val, max_val)

    print("\n   ✅ Applied clipping to outliers")
else:
    df_cleaned = df.copy()
```

### Step 3: Duplicate Detection

```python
# ===========================
# Part 3: Duplicate Detection
# ===========================

print("\n" + "=" * 60)
print("Part 3: Duplicate Detection")
print("=" * 60)

# ตรวจสอบ duplicates (timestamp + sensor_id ต้อง unique)
duplicates = df_cleaned[df_cleaned.duplicated(subset=['sensor_id'], keep=False)]
dup_count = df_cleaned.duplicated(subset=['sensor_id']).sum()

print(f"\n🔍 Duplicate Check:")
print(f"   Checking: (timestamp, sensor_id) combination")
print(f"   Duplicate records found: {dup_count:,}")

if dup_count > 0:
    print(f"\n   Sample duplicates:")
    print(duplicates[['sensor_id', 'temperature', 'humidity']].head(6))

    # ลบ duplicates
    df_cleaned = df_cleaned.drop_duplicates(subset=['sensor_id'], keep='first')
    print(f"\n   ✅ Removed {dup_count:,} duplicate records")
    print(f"   Remaining records: {len(df_cleaned):,}")
else:
    print("   ✅ No duplicates found")
```

### Step 4: Consistency Checks

```python
# ===========================
# Part 4: Consistency Checks
# ===========================

print("\n" + "=" * 60)
print("Part 4: Data Consistency Checks")
print("=" * 60)

# Check 1: Timestamp Sequence
print("\n🔍 Check 1: Timestamp Sequence")
time_gaps = df_cleaned.index.to_series().diff()
expected_gap = pd.Timedelta(minutes=5)
irregular_gaps = time_gaps[time_gaps != expected_gap].dropna()

if len(irregular_gaps) > 0:
    print(f"   ⚠️  Found {len(irregular_gaps):,} irregular time gaps")
    print(f"   Expected: 5 minutes")
    print(f"   Sample irregular gaps:")
    print(irregular_gaps.head(5))
else:
    print("   ✅ All timestamps are sequential (5-minute intervals)")

# Check 2: Sensor ID Validity
print("\n🔍 Check 2: Sensor ID Format")
valid_sensor_pattern = r'^S\d{3}$'  # S001, S002, ...
invalid_sensors = df_cleaned[~df_cleaned['sensor_id'].str.match(valid_sensor_pattern)]

if len(invalid_sensors) > 0:
    print(f"   ❌ Found {len(invalid_sensors):,} invalid sensor IDs")
    print(f"   Valid pattern: S001, S002, ...")
    print(invalid_sensors['sensor_id'].unique())
else:
    print("   ✅ All sensor IDs are valid")

# Check 3: Status Values
print("\n🔍 Check 3: Status Values")
valid_statuses = ['OK', 'WARNING', 'ERROR']
invalid_status = df_cleaned[~df_cleaned['status'].isin(valid_statuses)]

if len(invalid_status) > 0:
    print(f"   ❌ Found {len(invalid_status):,} invalid status values")
    print(f"   Valid values: {valid_statuses}")
    print(f"   Found: {invalid_status['status'].unique()}")
else:
    print("   ✅ All status values are valid")

# Check 4: Null Values (should be 0 after cleaning)
print("\n🔍 Check 4: Null Values")
null_count = df_cleaned.isnull().sum()
if null_count.sum() > 0:
    print("   ❌ Found NULL values:")
    print(null_count[null_count > 0])
else:
    print("   ✅ No NULL values")
```

### Step 5: Generate Data Quality Report

```python
# ===========================
# Part 5: Data Quality Report
# ===========================

print("\n" + "=" * 60)
print("Part 5: Data Quality Report")
print("=" * 60)

def generate_quality_report(df):
    """
    สร้าง comprehensive data quality report
    """
    report = {
        'summary': {
            'total_records': len(df),
            'total_columns': len(df.columns),
            'memory_usage_mb': round(df.memory_usage(deep=True).sum() / 1024**2, 2),
            'date_range': {
                'start': str(df.index.min()),
                'end': str(df.index.max()),
                'duration_days': (df.index.max() - df.index.min()).days
            }
        },
        'data_quality': {
            'completeness': {},
            'validity': {},
            'uniqueness': {},
            'consistency': {}
        },
        'statistics': {}
    }

    # Completeness (Missing Values)
    for col in df.columns:
        missing = df[col].isnull().sum()
        report['data_quality']['completeness'][col] = {
            'missing_count': int(missing),
            'missing_percent': round(missing / len(df) * 100, 2),
            'complete': missing == 0
        }

    # Validity (Ranges)
    numeric_cols = df.select_dtypes(include=['number']).columns
    for col in numeric_cols:
        report['data_quality']['validity'][col] = {
            'min': float(df[col].min()),
            'max': float(df[col].max()),
            'mean': float(df[col].mean()),
            'median': float(df[col].median()),
            'std': float(df[col].std())
        }

    # Uniqueness
    for col in df.columns:
        unique_count = df[col].nunique()
        report['data_quality']['uniqueness'][col] = {
            'unique_values': int(unique_count),
            'unique_percent': round(unique_count / len(df) * 100, 2)
        }

    # Statistics Summary
    report['statistics'] = df.describe().to_dict()

    return report

# สร้าง report
quality_report = generate_quality_report(df_cleaned)

# แสดง report
print("\n📊 Data Quality Report Summary:")
print(f"   Total Records:  {quality_report['summary']['total_records']:,}")
print(f"   Total Columns:  {quality_report['summary']['total_columns']}")
print(f"   Memory Usage:   {quality_report['summary']['memory_usage_mb']} MB")
print(f"   Date Range:     {quality_report['summary']['date_range']['start']} to")
print(f"                   {quality_report['summary']['date_range']['end']}")

print("\n📋 Completeness (Missing Values):")
for col, metrics in quality_report['data_quality']['completeness'].items():
    status = "✅" if metrics['complete'] else "❌"
    print(f"   {status} {col:15s} Missing: {metrics['missing_percent']}%")

print("\n🔍 Uniqueness:")
for col, metrics in quality_report['data_quality']['uniqueness'].items():
    print(f"   {col:15s} Unique: {metrics['unique_values']:,} ({metrics['unique_percent']}%)")

# บันทึก report เป็น JSON
with open('data/quality_report.json', 'w') as f:
    json.dump(quality_report, f, indent=2)

print(f"\n💾 Saved quality report to: data/quality_report.json")

# บันทึกข้อมูลที่ validated แล้ว
df_cleaned.to_csv('data/sensor_validated.csv')
print(f"💾 Saved validated data to: data/sensor_validated.csv")
```

### 📝 Checkpoint 2.3

**คำถาม:**
1. อะไรคือความแตกต่างระหว่าง Validation และ Cleaning?
2. ทำไมต้องตรวจสอบ (timestamp, sensor_id) combination ว่า unique?
3. Range Check vs Outlier Detection - แตกต่างกันอย่างไร?

**ทดสอบ:**
```python
# สร้าง validation pipeline ที่สามารถใช้ซ้ำได้
class DataValidator:
    def __init__(self, df):
        self.df = df
        self.report = {'passed': [], 'failed': [], 'warnings': []}

    def validate_all(self):
        self.check_types()
        self.check_ranges()
        self.check_nulls()
        self.check_duplicates()
        return self.report

    # เพิ่ม methods อื่นๆ...

# ใช้งาน
validator = DataValidator(df)
validation_results = validator.validate_all()
```

---

## Challenge: Complete ETL Pipeline

### 🎯 เป้าหมาย
สร้าง Complete ETL Pipeline ที่รวมทุกอย่างจาก Lab 2.1-2.3

### Challenge Requirements

สร้าง `etl_pipeline.py` ที่มี:

1. **Extract:** โหลด CSV แบบ Optimized
2. **Transform:** Clean & Validate
3. **Load:** บันทึกผลลัพธ์
4. **Report:** สร้าง Quality Report

```python
"""
Complete ETL Pipeline for IoT Sensor Data
"""
import pandas as pd
import numpy as np
import json
from datetime import datetime

class SensorDataETL:
    """
    ETL Pipeline สำหรับ IoT Sensor Data
    """

    def __init__(self, input_file):
        self.input_file = input_file
        self.df_raw = None
        self.df_cleaned = None
        self.metrics = {
            'extract': {},
            'transform': {},
            'load': {},
            'quality': {}
        }

    def extract(self):
        """
        Extract: โหลดข้อมูลแบบ Optimized
        """
        print("=" * 60)
        print("STEP 1: EXTRACT")
        print("=" * 60)

        # TODO: โหลดข้อมูลด้วย dtype optimization
        # TODO: คำนวณ memory usage
        # TODO: บันทึก metrics

        pass

    def transform(self):
        """
        Transform: ทำความสะอาดและ Validate
        """
        print("\n" + "=" * 60)
        print("STEP 2: TRANSFORM")
        print("=" * 60)

        # TODO: จัดการ Missing Values
        # TODO: ตรวจสอบ Ranges และลบ Outliers
        # TODO: ลบ Duplicates
        # TODO: Validate Data Quality

        pass

    def load(self, output_dir='data/output'):
        """
        Load: บันทึกข้อมูลและ Report
        """
        print("\n" + "=" * 60)
        print("STEP 3: LOAD")
        print("=" * 60)

        # TODO: บันทึก cleaned data
        # TODO: บันทึก quality report
        # TODO: บันทึก metrics

        pass

    def run(self):
        """
        รัน ETL Pipeline ทั้งหมด
        """
        print("\n" + "=" * 70)
        print("🚀 IoT Sensor Data ETL Pipeline")
        print("=" * 70)
        print(f"Input:  {self.input_file}")
        print(f"Start:  {datetime.now()}")

        # Execute ETL
        self.extract()
        self.transform()
        self.load()

        print("\n" + "=" * 70)
        print("✅ ETL Pipeline Completed")
        print("=" * 70)

        return self.df_cleaned, self.metrics

# ใช้งาน
if __name__ == "__main__":
    etl = SensorDataETL('data/sensor_large.csv')
    df_final, metrics = etl.run()

    # แสดง Summary
    print("\n📊 Pipeline Summary:")
    print(json.dumps(metrics, indent=2))
```

### Challenge Tasks

1. **เติม code ใน `extract()` method:**
   - โหลดข้อมูลด้วย dtype optimization
   - วัด memory usage และ load time
   - บันทึก metrics

2. **เติม code ใน `transform()` method:**
   - จัดการ Missing Values (เลือก strategy ที่เหมาะสม)
   - ตรวจสอบและจัดการ Outliers
   - ลบ Duplicates
   - Validate ข้อมูลทั้งหมด

3. **เติม code ใน `load()` method:**
   - บันทึก cleaned data เป็น CSV
   - สร้างและบันทึก quality report (JSON)
   - บันทึก pipeline metrics

4. **เพิ่ม features:**
   - Logging system
   - Error handling
   - Progress indicators
   - Performance profiling

### Expected Output

```
======================================================================
🚀 IoT Sensor Data ETL Pipeline
======================================================================
Input:  data/sensor_large.csv
Start:  2025-01-15 10:30:00

============================================================
STEP 1: EXTRACT
============================================================
✅ Loaded 100,500 records
✅ Memory optimization: 76.7% saving
✅ Load time: 0.198 seconds

============================================================
STEP 2: TRANSFORM
============================================================
🧹 Handling missing values...
   ✅ Temperature: Interpolated 5,025 values
   ✅ Humidity: Interpolated 5,025 values
   ✅ Pressure: Interpolated 5,025 values

🔍 Validating ranges...
   ❌ Temperature: 1,005 outliers found
   ✅ Removed outliers

🔄 Checking duplicates...
   ✅ Removed 503 duplicates

✅ Final records: 99,000

============================================================
STEP 3: LOAD
============================================================
💾 Saved: data/output/sensor_cleaned_20250115.csv
💾 Saved: data/output/quality_report_20250115.json
💾 Saved: data/output/pipeline_metrics_20250115.json

======================================================================
✅ ETL Pipeline Completed
======================================================================
```

---

## Checkpoint & Submission

### ✅ Checklist

ตรวจสอบว่าคุณทำครบทุก Lab:

- [ ] **Lab 2.1:** โหลด CSV แบบ Optimized และคำนวณ Memory Saving
  - [ ] Memory Saving > 70%
  - [ ] เข้าใจการเลือก dtype ที่เหมาะสม
  - [ ] รู้จัก category dtype

- [ ] **Lab 2.2:** จัดการ Missing Values
  - [ ] ทดลอง 5 strategies (Drop, Ffill, Mean, Median, Interpolate)
  - [ ] เปรียบเทียบผลลัพธ์
  - [ ] เลือก strategy ที่เหมาะสมกับ Time-Series

- [ ] **Lab 2.3:** Data Validation
  - [ ] ตรวจสอบ Data Types
  - [ ] ตรวจสอบ Value Ranges
  - [ ] ตรวจสอบ Duplicates
  - [ ] สร้าง Quality Report

- [ ] **Challenge:** Complete ETL Pipeline
  - [ ] รวม Extract, Transform, Load
  - [ ] มี Metrics และ Reporting
  - [ ] สามารถใช้ซ้ำได้

### 📤 Submission

บันทึกไฟล์เหล่านี้:

```
module-2/
├── lab_2_1_loading.py
├── lab_2_2_missing_values.py
├── lab_2_3_validation.py
├── etl_pipeline.py
└── data/
    ├── sensor_large.csv
    ├── sensor_cleaned.csv
    ├── sensor_validated.csv
    └── quality_report.json
```

### 🎓 Learning Outcomes

หลังจากทำ Lab นี้ คุณควรจะ:

✅ โหลดข้อมูล CSV แบบ Optimized ได้
✅ ประหยัด Memory ด้วย dtype optimization
✅ จัดการ Missing Values ด้วยเทคนิคที่เหมาะสม
✅ ตรวจสอบ Data Quality และ Integrity
✅ สร้าง ETL Pipeline แบบอัตโนมัติ

---

## 💡 Additional Resources

### เอกสารอ้างอิง

- [Pandas dtype optimization](https://pandas.pydata.org/docs/user_guide/scale.html)
- [Handling Missing Data](https://pandas.pydata.org/docs/user_guide/missing_data.html)
- [Data Validation Best Practices](https://docs.python.org/3/library/dataclasses.html)

### ขั้นตอนต่อไป

ใน Module 3 คุณจะได้เรียนรู้:
- Feature Engineering (Rolling Average)
- Performance Benchmarking (Pandas vs NumPy)
- Chunking สำหรับไฟล์ขนาดใหญ่มาก

---

**[⬅️ กลับไป Module 2](../module-2.md)** | **[ไปที่ Module 3 Labs ➡️](../../module-3/labs/lab-module-3.md)**
