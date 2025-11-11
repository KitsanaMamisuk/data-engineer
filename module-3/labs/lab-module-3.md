# Lab Module 3: ETL - Transformation & Optimization

**<¯ '18#0*L:**
- 6*#I2 Features I'" Rolling Window Operations
- '1A%0@#5"@5" Performance #0+'H2 Pandas A%0 NumPy
- C
I Chunking Strategy 12#I-!9%2C+H
- @4H!#0*44 2I'" Vectorized Operations

**ñ @'%25HC
I:** 2 
1H'B!

---

## =Ë Pre-requisites

H-@#4H! Lab #'*-'H28!5:
-  Virtual Environment activated
-  Pandas, NumPy 41IA%I'
-  3 Module 1 A%0 2 !2A%I'

---

## Lab 3.1: Rolling Window Operations (30 25)

### '18#0*L
@#5"#9I2#*#I2 Rolling Average A%0 Rolling Statistics *3+#1 Time-Series Data

### 1I-

#### Step 1: *#I2 Sample Time-Series Data

*#I2D%L `lab3_1_rolling.py`:

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

print("=== Lab 3.1: Rolling Window Operations ===\n")

# *#I2 Time-Series Data (1 '1, 8 5 25)
np.random.seed(42)
date_range = pd.date_range(start='2025-01-01',
                           end='2025-01-02',
                           freq='5min')

# *#I2I-!9%-8+ 9!45H!5 pattern A%0 noise
n_records = len(date_range)
hour_of_day = date_range.hour
daily_pattern = 3 * np.sin((hour_of_day - 6) * np.pi / 12)  # Peak -H2"
noise = np.random.normal(0, 1, n_records)  # Random noise
temperature = 25 + daily_pattern + noise

df = pd.DataFrame({
    'timestamp': date_range,
    'sensor_id': 'S001',
    'temperature': temperature
})

df.set_index('timestamp', inplace=True)

print(f"Total records: {len(df)}")
print(f"Date range: {df.index.min()} to {df.index.max()}")
print(f"\nFirst 10 records:")
print(df.head(10))
```

#### Step 2: *#I2 Rolling Average (5-minute window)

```python
print("\n=== Rolling Average (5 periods = 25 minutes) ===")

# Rolling Average - +I2H22 5 periods
df['temp_rolling_5'] = df['temperature'].rolling(window=5).mean()

# Rolling Average - +I2H22 12 periods (1 
1H'B!)
df['temp_rolling_12'] = df['temperature'].rolling(window=12).mean()

print(df[['temperature', 'temp_rolling_5', 'temp_rolling_12']].head(15))

# 3''2!AH2
df['temp_diff'] = df['temperature'] - df['temp_rolling_5']

print(f"\nOriginal Std: {df['temperature'].std():.2f}")
print(f"Rolling 5 Std: {df['temp_rolling_5'].std():.2f}")
print(f"Rolling 12 Std: {df['temp_rolling_12'].std():.2f}")
```

#### Step 3: *#I2 Rolling Statistics -7HF

```python
print("\n=== Rolling Statistics ===")

# Rolling Min, Max, Std
window = 12  # 1 hour window

df['temp_rolling_min'] = df['temperature'].rolling(window=window).min()
df['temp_rolling_max'] = df['temperature'].rolling(window=window).max()
df['temp_rolling_std'] = df['temperature'].rolling(window=window).std()

# Range (Max - Min)
df['temp_rolling_range'] = df['temp_rolling_max'] - df['temp_rolling_min']

print("Rolling Statistics (1-hour window):")
print(df[['temperature', 'temp_rolling_min', 'temp_rolling_max',
          'temp_rolling_std', 'temp_rolling_range']].tail(10))
```

#### Step 4: Visualization (Optional)

```python
# *#I2#2@#5"@5"
plt.figure(figsize=(14, 6))

plt.plot(df.index, df['temperature'], label='Original', alpha=0.5, linewidth=0.8)
plt.plot(df.index, df['temp_rolling_5'], label='Rolling 5 (25 min)', linewidth=1.5)
plt.plot(df.index, df['temp_rolling_12'], label='Rolling 12 (1 hour)', linewidth=1.5)

plt.xlabel('Timestamp')
plt.ylabel('Temperature (°C)')
plt.title('Temperature with Rolling Averages')
plt.legend()
plt.grid(True, alpha=0.3)
plt.tight_layout()

# 16#9
plt.savefig('rolling_average_comparison.png', dpi=150)
print("\n Chart saved to: rolling_average_comparison.png")
```

#### Step 5: Advanced - Expanding Window

```python
print("\n=== Expanding Window (Cumulative) ===")

# Expanding mean - 3'2@#4H!I6181
df['temp_expanding_mean'] = df['temperature'].expanding().mean()

# Expanding std
df['temp_expanding_std'] = df['temperature'].expanding().std()

print("Expanding Statistics:")
print(df[['temperature', 'temp_expanding_mean', 'temp_expanding_std']].iloc[::50])
```

###  Checkpoint 3.1

#'*-'H2:
- [ ] *#I2 Time-Series Data DI*3@#G
- [ ] 3' Rolling Average (5 A%0 12 periods) DI
- [ ] 3' Rolling Min, Max, Std DI
- [ ] @I2C'2!AH2#0+'H2 Rolling A%0 Expanding
- [ ] *#I2#2@#5"@5"DI (optional)

**Expected Output:**
```
Original Std: 1.83
Rolling 5 Std: 1.65  (%% - @#209 smooth)
Rolling 12 Std: 1.48 (%%!2'H2)
```

---

## Lab 3.2: Performance Benchmarking - Pandas vs NumPy (30 25)

### '18#0*L
@#5"@5"'2!@#G'C2##0!'%%#0+'H2 Pandas A%0 NumPy

### 1I-

#### Step 1: Setup Benchmark Environment

*#I2D%L `lab3_2_benchmark.py`:

```python
import pandas as pd
import numpy as np
import time

print("=== Lab 3.2: Performance Benchmarking ===\n")

# *#I2I-!9%2C+H
n_records = 1_000_000
print(f"Creating {n_records:,} records for benchmarking...")

np.random.seed(42)
data = {
    'temperature': np.random.uniform(20, 30, n_records),
    'humidity': np.random.uniform(40, 80, n_records),
    'pressure': np.random.uniform(990, 1020, n_records)
}

df = pd.DataFrame(data)
print(f"DataFrame created: {df.shape[0]:,} rows × {df.shape[1]} columns\n")
```

#### Step 2: Benchmark - Simple Arithmetic

```python
print("=== Benchmark 1: Simple Arithmetic (Temperature - 20) ===")

# Method 1: Pandas Series Operation
start = time.time()
result_pandas = df['temperature'] - 20
time_pandas = time.time() - start

# Method 2: NumPy Array Operation
start = time.time()
result_numpy = df['temperature'].values - 20
time_numpy = time.time() - start

# Method 3: Python Loop (
I2!2 - D!HA03)
start = time.time()
result_loop = [temp - 20 for temp in df['temperature']]
time_loop = time.time() - start

print(f"Pandas Series:  {time_pandas*1000:.2f} ms")
print(f"NumPy Array:    {time_numpy*1000:.2f} ms")
print(f"Python Loop:    {time_loop*1000:.2f} ms")

# 3' Speedup
speedup_pandas_vs_loop = time_loop / time_pandas
speedup_numpy_vs_loop = time_loop / time_numpy
speedup_numpy_vs_pandas = time_pandas / time_numpy

print(f"\nSpeedup (Pandas vs Loop): {speedup_pandas_vs_loop:.1f}x faster")
print(f"Speedup (NumPy vs Loop):  {speedup_numpy_vs_loop:.1f}x faster")
print(f"Speedup (NumPy vs Pandas): {speedup_numpy_vs_pandas:.1f}x faster")
```

#### Step 3: Benchmark - Complex Calculations

```python
print("\n=== Benchmark 2: Complex Calculation (Celsius to Fahrenheit) ===")

# Formula: F = (C * 9/5) + 32

# Method 1: Pandas
start = time.time()
result_pandas = (df['temperature'] * 9/5) + 32
time_pandas = time.time() - start

# Method 2: NumPy
start = time.time()
result_numpy = (df['temperature'].values * 9/5) + 32
time_numpy = time.time() - start

# Method 3: Python Loop
start = time.time()
result_loop = [(temp * 9/5) + 32 for temp in df['temperature']]
time_loop = time.time() - start

print(f"Pandas Series:  {time_pandas*1000:.2f} ms")
print(f"NumPy Array:    {time_numpy*1000:.2f} ms")
print(f"Python Loop:    {time_loop*1000:.2f} ms")

print(f"\nNumPy is {time_pandas/time_numpy:.1f}x faster than Pandas")
print(f"NumPy is {time_loop/time_numpy:.1f}x faster than Loop")
```

#### Step 4: Benchmark - Aggregation

```python
print("\n=== Benchmark 3: Aggregation (Mean, Std) ===")

# Method 1: Pandas
start = time.time()
mean_pandas = df['temperature'].mean()
std_pandas = df['temperature'].std()
time_pandas = time.time() - start

# Method 2: NumPy
start = time.time()
mean_numpy = np.mean(df['temperature'].values)
std_numpy = np.std(df['temperature'].values)
time_numpy = time.time() - start

print(f"Pandas: {time_pandas*1000:.2f} ms (Mean: {mean_pandas:.2f}, Std: {std_pandas:.2f})")
print(f"NumPy:  {time_numpy*1000:.2f} ms (Mean: {mean_numpy:.2f}, Std: {std_numpy:.2f})")
print(f"\nSpeedup: {time_pandas/time_numpy:.1f}x")
```

#### Step 5: Benchmark - Multiple Columns

```python
print("\n=== Benchmark 4: Multiple Column Operations ===")

# 3': (temperature - humidity) / pressure

# Method 1: Pandas
start = time.time()
result_pandas = (df['temperature'] - df['humidity']) / df['pressure']
time_pandas = time.time() - start

# Method 2: NumPy
start = time.time()
temp_arr = df['temperature'].values
hum_arr = df['humidity'].values
press_arr = df['pressure'].values
result_numpy = (temp_arr - hum_arr) / press_arr
time_numpy = time.time() - start

print(f"Pandas: {time_pandas*1000:.2f} ms")
print(f"NumPy:  {time_numpy*1000:.2f} ms")
print(f"Speedup: {time_pandas/time_numpy:.1f}x")
```

#### Step 6: Summary A%0 Best Practices

```python
print("\n" + "="*60)
print("BENCHMARK SUMMARY")
print("="*60)

summary = """
Performance Ranking (2@#G'*8D
I2*8):
1. ¡ NumPy Array Operations    (Fastest)
2. ¡ Pandas Vectorized Ops     (Fast)
3. = Python Loops              (Slowest - -"H2C
I!)

Best Practices:
 C
I NumPy *3+#1 numerical calculations
 C
I Pandas *3+#1 data manipulation
L +%5@%5H" Python loops 1I-!9%2C+H
 Convert Pandas to NumPy H-3'1I-
"""

print(summary)
```

###  Checkpoint 3.2

#'*-'H2:
- [ ] @I2C'2!AH2'2!@#G'#0+'H2 NumPy, Pandas, Loop
- [ ] NumPy @#G''H2 Pandas #0!2 1.5-3x
- [ ] Loop 
I2'H2 Vectorized operations !2 (10-100x)
- [ ] #9I'H2@!7H-D+#H'#C
I NumPy vs Pandas

**Expected Results:**
```
NumPy:  ~5-10ms
Pandas: ~10-20ms
Loop:   ~500-1000ms

NumPy @#G''H2 Loop #0!2 50-100x
```

---

## Lab 3.3: Chunking Strategy *3+#1 Big Data (40 25)

### '18#0*L
@#5"#9I2#C
I Chunking 12#D%L2C+H5HD!H*2!2#B+%1I+!% Memory DI

### 1I-

#### Step 1: *#I2D%L CSV 2C+H

*#I2D%L `lab3_3_chunking.py`:

```python
import pandas as pd
import numpy as np
import time

print("=== Lab 3.3: Chunking Strategy ===\n")

# Step 1: *#I2D%L CSV 2C+H*3+#1*-
print("Step 1: Creating large CSV file...")

n_records = 5_000_000  # 5 million records
np.random.seed(42)

# *#I2I-!9%5%0 chunk @7H-D!HC+IC
I Memory !2
chunk_size = 500_000
output_file = 'sensor_large.csv'

# *#I2D%L CSV
for i in range(0, n_records, chunk_size):
    size = min(chunk_size, n_records - i)

    chunk_data = {
        'timestamp': pd.date_range(start='2025-01-01', periods=size, freq='5s'),
        'sensor_id': np.random.choice(['S001', 'S002', 'S003', 'S004', 'S005'], size),
        'temperature': np.random.uniform(20, 30, size),
        'humidity': np.random.uniform(40, 80, size),
        'pressure': np.random.uniform(990, 1020, size)
    }

    df_chunk = pd.DataFrame(chunk_data)

    # Write to CSV (append mode after first chunk)
    if i == 0:
        df_chunk.to_csv(output_file, index=False, mode='w')
    else:
        df_chunk.to_csv(output_file, index=False, mode='a', header=False)

    print(f"  Written: {i+size:,} / {n_records:,} rows", end='\r')

print(f"\n Created: {output_file} ({n_records:,} rows)")

# #'*-2D%L
import os
file_size_mb = os.path.getsize(output_file) / (1024 * 1024)
print(f"File size: {file_size_mb:.1f} MB\n")
```

#### Step 2: -H2D%LAD!HC
I Chunking (D!HA03)

```python
print("=== Method 1: Read All at Once (Not Recommended) ===")

try:
    start = time.time()
    df_all = pd.read_csv(output_file)
    time_load = time.time() - start

    memory_mb = df_all.memory_usage(deep=True).sum() / (1024 * 1024)

    print(f" Loaded {len(df_all):,} rows in {time_load:.2f} seconds")
    print(f"Memory usage: {memory_mb:.1f} MB")

    # 3'H2@	%5H"
    start = time.time()
    mean_temp = df_all['temperature'].mean()
    time_calc = time.time() - start

    print(f"Mean temperature: {mean_temp:.2f}°C (calculated in {time_calc:.3f}s)")

except MemoryError:
    print("L MemoryError: File too large to load all at once!")
```

#### Step 3: -H2D%LAC
I Chunking (A03)

```python
print("\n=== Method 2: Chunked Reading (Recommended) ===")

chunk_size = 100_000  # -H25%0 100k rows
chunks_processed = 0
total_rows = 0

# Accumulators *3+#13'H2*44
sum_temp = 0
sum_humidity = 0
count = 0

start = time.time()

# -H25%0 chunk
for chunk in pd.read_csv(output_file, chunksize=chunk_size):
    # Process each chunk
    sum_temp += chunk['temperature'].sum()
    sum_humidity += chunk['humidity'].sum()
    count += len(chunk)

    chunks_processed += 1
    total_rows += len(chunk)

    print(f"  Processed: {total_rows:,} rows ({chunks_processed} chunks)", end='\r')

time_load = time.time() - start

# 3'H2@	%5H"
mean_temp = sum_temp / count
mean_humidity = sum_humidity / count

print(f"\n Processed {total_rows:,} rows in {time_load:.2f} seconds")
print(f"Mean temperature: {mean_temp:.2f}°C")
print(f"Mean humidity: {mean_humidity:.2f}%")
print(f"Chunks processed: {chunks_processed}")
```

#### Step 4: Chunking with Transformation

```python
print("\n=== Method 3: Chunking with Transformation ===")

chunk_size = 100_000
output_processed = 'sensor_processed.csv'

chunks_written = 0
start = time.time()

for chunk in pd.read_csv(output_file, chunksize=chunk_size):
    # Transform each chunk
    chunk['temp_fahrenheit'] = (chunk['temperature'] * 9/5) + 32
    chunk['temp_category'] = pd.cut(chunk['temperature'],
                                    bins=[0, 23, 27, 100],
                                    labels=['Cool', 'Normal', 'Warm'])

    # Filter outliers
    chunk = chunk[(chunk['temperature'] >= 15) & (chunk['temperature'] <= 35)]

    # Write processed chunk
    if chunks_written == 0:
        chunk.to_csv(output_processed, index=False, mode='w')
    else:
        chunk.to_csv(output_processed, index=False, mode='a', header=False)

    chunks_written += 1
    print(f"  Transformed: {chunks_written} chunks", end='\r')

time_transform = time.time() - start

print(f"\n Transformed and saved to: {output_processed}")
print(f"Time taken: {time_transform:.2f} seconds")

# #'*-D%L5HDI
df_sample = pd.read_csv(output_processed, nrows=5)
print(f"\nSample of processed data:")
print(df_sample)
```

#### Step 5: Chunking with Aggregation

```python
print("\n=== Method 4: Chunking with Aggregation ===")

chunk_size = 100_000

# Accumulators *3+#1 aggregation
sensor_stats = {}

for chunk in pd.read_csv(output_file, chunksize=chunk_size):
    # Group by sensor_id A%03'*44
    chunk_stats = chunk.groupby('sensor_id').agg({
        'temperature': ['sum', 'count', 'min', 'max']
    })

    # Accumulate statistics
    for sensor_id in chunk_stats.index:
        if sensor_id not in sensor_stats:
            sensor_stats[sensor_id] = {
                'sum': 0,
                'count': 0,
                'min': float('inf'),
                'max': float('-inf')
            }

        stats = chunk_stats.loc[sensor_id, 'temperature']
        sensor_stats[sensor_id]['sum'] += stats['sum']
        sensor_stats[sensor_id]['count'] += stats['count']
        sensor_stats[sensor_id]['min'] = min(sensor_stats[sensor_id]['min'], stats['min'])
        sensor_stats[sensor_id]['max'] = max(sensor_stats[sensor_id]['max'], stats['max'])

# 3'H2@	%5H"*8I2"
print("\nAggregated Statistics by Sensor:")
print("-" * 70)
for sensor_id, stats in sensor_stats.items():
    mean = stats['sum'] / stats['count']
    print(f"{sensor_id}: Mean={mean:.2f}°C, Min={stats['min']:.2f}°C, Max={stats['max']:.2f}°C, Count={stats['count']:,}")
```

#### Step 6: Best Practices Summary

```python
print("\n" + "="*70)
print("CHUNKING BEST PRACTICES")
print("="*70)

practices = """
 When to use Chunking:
   - D%L2C+H (> Available RAM)
   - I-2##0!'%%5%0*H'
   - I-2# Transform A%0@5"%1

 Chunk Size Selection:
   - @%G@4D (< 10k): 
I2, overhead !2
   - C+H@4D (> 1M): C
I Memory !2
   - A03: 50k-500k rows (6I1 RAM)

 Optimization Tips:
   - C
I dtype specification
   - C
I usecols @%7-@	20 columns 5HI-2#
   - C
I iterator pattern A list - chunks
   - @5"%%1L5%0 chunk (append mode)

  Limitations:
   - D!H@+!201 operations 5HI-9I-!9%1I+! (@
H sorting 1ID%L)
   - 1I-'H22#-H21I+!
"""

print(practices)
```

###  Checkpoint 3.3

#'*-'H2:
- [ ] *#I2D%L CSV 2C+HDI (5M rows)
- [ ] -H2D%LI'" Chunking DI*3@#G
- [ ] 3'H2*442 Chunks DI
- [ ] Transform data 5%0 Chunk A%0@5"%1DI
- [ ] 3 Aggregation I2! Chunks DI
- [ ] @I2C'H2@!7H-D+#H'#C
I Chunking

**Expected Performance:**
```
File: 5,000,000 rows (~300 MB)
Chunk size: 100,000 rows
Chunks: 50 chunks
Processing time: 10-30 seconds
Memory usage: < 50 MB (vs ~1 GB I2B+%1I+!)
```

---

## Lab 3.4: Vectorized Operations Practice (20 25)

### '18#0*L
6@5"BIA Vectorized A2#C
I Loops

### Challenge 1: A% Loop @G Vectorized

```python
import pandas as pd
import numpy as np
import time

print("=== Lab 3.4: Vectorized Operations ===\n")

# *#I2I-!9%
n = 1_000_000
df = pd.DataFrame({
    'temperature': np.random.uniform(20, 30, n),
    'humidity': np.random.uniform(40, 80, n)
})

print("Challenge 1: Calculate Heat Index")
print("-" * 50)

# L A Loop (
I2)
def calculate_heat_index_loop(temp, humidity):
    """Heat Index = T + 0.5 * (T + humidity/10)"""
    result = []
    for t, h in zip(temp, humidity):
        hi = t + 0.5 * (t + h/10)
        result.append(hi)
    return result

start = time.time()
heat_index_loop = calculate_heat_index_loop(df['temperature'], df['humidity'])
time_loop = time.time() - start

print(f"Loop method: {time_loop*1000:.2f} ms")

#  A Vectorized (@#G')
start = time.time()
heat_index_vec = df['temperature'] + 0.5 * (df['temperature'] + df['humidity']/10)
time_vec = time.time() - start

print(f"Vectorized:  {time_vec*1000:.2f} ms")
print(f"Speedup: {time_loop/time_vec:.1f}x faster\n")
```

### Challenge 2: Conditional Operations

```python
print("Challenge 2: Temperature Category (Conditional)")
print("-" * 50)

# L A Loop
def categorize_temp_loop(temps):
    result = []
    for temp in temps:
        if temp < 20:
            result.append('Cold')
        elif temp < 25:
            result.append('Cool')
        elif temp < 30:
            result.append('Warm')
        else:
            result.append('Hot')
    return result

start = time.time()
category_loop = categorize_temp_loop(df['temperature'])
time_loop = time.time() - start

#  A Vectorized (np.select)
start = time.time()
conditions = [
    df['temperature'] < 20,
    (df['temperature'] >= 20) & (df['temperature'] < 25),
    (df['temperature'] >= 25) & (df['temperature'] < 30),
    df['temperature'] >= 30
]
choices = ['Cold', 'Cool', 'Warm', 'Hot']
category_vec = np.select(conditions, choices)
time_vec = time.time() - start

print(f"Loop method: {time_loop*1000:.2f} ms")
print(f"Vectorized:  {time_vec*1000:.2f} ms")
print(f"Speedup: {time_loop/time_vec:.1f}x faster\n")
```

### Challenge 3: Apply vs Vectorized

```python
print("Challenge 3: Complex Function - Apply vs Vectorized")
print("-" * 50)

# Function 1I-
def complex_formula(temp, humidity):
    """Complex calculation"""
    return (temp ** 2) + np.sqrt(humidity) - (temp * humidity / 100)

# L Apply (
I2'H2 Vectorized)
start = time.time()
result_apply = df.apply(lambda row: complex_formula(row['temperature'], row['humidity']), axis=1)
time_apply = time.time() - start

#  Vectorized
start = time.time()
result_vec = (df['temperature'] ** 2) + np.sqrt(df['humidity']) - (df['temperature'] * df['humidity'] / 100)
time_vec = time.time() - start

print(f"Apply method: {time_apply*1000:.2f} ms")
print(f"Vectorized:   {time_vec*1000:.2f} ms")
print(f"Speedup: {time_apply/time_vec:.1f}x faster\n")
```

###  Checkpoint 3.4

#'*-'H2:
- [ ] @I2C'2!AH2#0+'H2 Loop A%0 Vectorized
- [ ] C
I `np.select()` *3+#1 conditional operations DI
- [ ] Vectorized @#G''H2 Loop A%0 Apply !2
- [ ] *2!2#@5"BIA Vectorized DI

---

## <Æ Challenge Exercise

### Final Challenge: Complete Pipeline Optimization

*#I2 script 5H:

1. B+%D%L CSV 2C+HI'" Chunking
2. 3 Rolling Average (1 hour window) CAH%0 Chunk
3. @#5"@5" Performance (Pandas vs NumPy)
4. 16%%1L

**Requirements:**
- C
I Chunking (chunk_size = 100k)
- C
I Vectorized operations
- '1@'%2AH%01I-
- Memory usage < 100 MB

<details>
<summary>=¡ Hint (%4@7H-9)</summary>

```python
import pandas as pd
import numpy as np
import time

chunk_size = 100_000
output_file = 'sensor_optimized.csv'

start_total = time.time()

for i, chunk in enumerate(pd.read_csv('sensor_large.csv', chunksize=chunk_size)):
    # 1. Optimize dtypes
    chunk['sensor_id'] = chunk['sensor_id'].astype('category')
    chunk['temperature'] = chunk['temperature'].astype(np.float32)

    # 2. Rolling average (NumPy for speed)
    temps = chunk['temperature'].values
    # C
I np.convolve *3+#1 rolling average

    # 3. Vectorized transformation
    chunk['temp_normalized'] = (chunk['temperature'] - chunk['temperature'].mean()) / chunk['temperature'].std()

    # 4. Write result
    mode = 'w' if i == 0 else 'a'
    header = True if i == 0 else False
    chunk.to_csv(output_file, mode=mode, header=header, index=False)

print(f"Total time: {time.time() - start_total:.2f}s")
```
</details>

---

## =Ê Lab Summary

### *4H5HDI@#5"#9I

 **Lab 3.1:** Rolling Window Operations *3+#1 Time-Series
 **Lab 3.2:** Performance Benchmarking (NumPy vs Pandas vs Loop)
 **Lab 3.3:** Chunking Strategy *3+#1I-!9%2C+H
 **Lab 3.4:** Vectorized Operations Best Practices

### Performance Summary

| Operation | Loop | Pandas | NumPy |
|-----------|------|--------|-------|
| Simple Math | 1000ms | 10ms | 5ms |
| Complex Calc | 2000ms | 20ms | 10ms |
| Aggregation | - | 15ms | 8ms |
| **Speedup** | **Baseline** | **~100x** | **~200x** |

### Skills Acquired

| Skill | Level |
|-------|-------|
| Rolling Windows | PPPP |
| Performance Benchmarking | PPPP |
| Chunking Strategy | PPPP |
| Vectorized Operations | PPPP |
| NumPy Optimization | PPP |

---

## <¯ 1D

8#I-!*3+#1 Module 4 A%I'!

**=I [Module 4: Cloud Integration & Storage](../../module-4/module-4.md)**

---

**[ %1D Module 3](../module-3.md)** | **[<à %1D+I2+%1](../../wiki.md)**
