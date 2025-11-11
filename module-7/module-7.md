# Module 7: Data Quality & Testing

**🎯 วัตถุประสงค์การเรียนรู้:**
- สร้าง Data Validation Framework สำหรับ IoT Data
- เขียน Unit Tests สำหรับ ETL Functions
- ทำความเข้าใจ Pipeline Idempotency
- ใช้งาน Data Quality Metrics และ Monitoring
- ออกแบบ Testing Strategy สำหรับ Production Pipeline

**⏱️ เวลาที่ใช้:** 4.5 ชั่วโมง (ทฤษฎี 2.5 ชม. + Lab 2 ชม.)

---

## 📚 สารบัญ

1. [Data Quality Overview](#1-data-quality-overview)
2. [Data Validation Framework](#2-data-validation-framework)
3. [Unit Testing for ETL](#3-unit-testing-for-etl)
4. [Pipeline Idempotency](#4-pipeline-idempotency)
5. [Data Quality Metrics](#5-data-quality-metrics)
6. [Testing Strategies](#6-testing-strategies)
7. [Labs & Practical Exercises](./labs/lab-module-7.md)

---

## 1. Data Quality Overview

### 1.1 Data Quality คืออะไร?

**Data Quality** = คุณภาพของข้อมูล ที่วัดจากความถูกต้อง สมบูรณ์ และเชื่อถือได้

```
Data Quality Dimensions:

┌─────────────────────────────────────────────┐
│         6 Dimensions of Data Quality        │
├─────────────────────────────────────────────┤
│                                             │
│  1. ✅ Accuracy (ความถูกต้อง)               │
│     ข้อมูลสะท้อนความเป็นจริง               │
│                                             │
│  2. ✅ Completeness (ความสมบูรณ์)           │
│     ไม่มี Missing Values                   │
│                                             │
│  3. ✅ Consistency (ความสอดคล้อง)           │
│     ข้อมูลไม่ขัดแย้งกัน                     │
│                                             │
│  4. ✅ Timeliness (ความทันเวลา)             │
│     ข้อมูลปัจจุบัน ไม่ล้าสมัย               │
│                                             │
│  5. ✅ Validity (ความถูกต้องตามกฎ)          │
│     ข้อมูลอยู่ในช่วงที่กำหนด                │
│                                             │
│  6. ✅ Uniqueness (ความไม่ซ้ำ)              │
│     ไม่มี Duplicates                       │
│                                             │
└─────────────────────────────────────────────┘
```

### 1.2 ทำไม Data Quality สำคัญ?

**ผลกระทบของ Poor Data Quality:**

❌ **ต่อการตัดสินใจ:**
- Analytics ผิดพลาด
- Business Insights ไม่ถูกต้อง
- ตัดสินใจผิด → สูญเสียรายได้

❌ **ต่อ ML Models:**
- Model ทำนายผิด
- Bias ในข้อมูล
- Model ไม่ Generalize ได้

❌ **ต่อ Operations:**
- Pipeline ล้มเหลวบ่อย
- เสียเวลา Debug
- ต้นทุนเพิ่มขึ้น

### 1.3 Data Quality Framework

```
Data Quality Framework Architecture:

┌─────────────────────────────────────────────┐
│           Data Quality Framework            │
└─────────────────────────────────────────────┘
              ↓
    ┌─────────┴─────────┐
    ↓                   ↓
┌─────────┐      ┌─────────────┐
│Validation│      │  Monitoring │
│  Rules   │      │  & Metrics  │
└────┬────┘      └──────┬──────┘
     │                  │
     ↓                  ↓
┌─────────────────────────┐
│   Automated Testing     │
│   • Unit Tests          │
│   • Integration Tests   │
│   • Data Tests          │
└─────────────────────────┘
```

---

## 2. Data Validation Framework

### 2.1 Validation Rules สำหรับ IoT Data

```python
import pandas as pd
import numpy as np
from typing import Dict, List, Any

class DataValidator:
    """
    Data Validation Framework for IoT Sensor Data
    """

    def __init__(self, df: pd.DataFrame):
        self.df = df
        self.errors = []
        self.warnings = []

    def validate_schema(self, expected_schema: Dict[str, str]) -> bool:
        """
        ตรวจสอบ Schema (columns และ data types)

        Args:
            expected_schema: {'column_name': 'expected_type'}

        Returns:
            bool: True if schema is valid
        """
        # Check required columns
        missing_cols = set(expected_schema.keys()) - set(self.df.columns)
        if missing_cols:
            self.errors.append(f"Missing columns: {missing_cols}")
            return False

        # Check data types
        for col, expected_type in expected_schema.items():
            actual_type = str(self.df[col].dtype)

            if expected_type == 'datetime':
                if not pd.api.types.is_datetime64_any_dtype(self.df[col]):
                    self.errors.append(f"{col}: expected datetime, got {actual_type}")
            elif expected_type == 'numeric':
                if not pd.api.types.is_numeric_dtype(self.df[col]):
                    self.errors.append(f"{col}: expected numeric, got {actual_type}")
            elif expected_type not in actual_type:
                self.warnings.append(f"{col}: expected {expected_type}, got {actual_type}")

        return len(self.errors) == 0

    def validate_completeness(self, required_columns: List[str],
                             max_missing_pct: float = 5.0) -> bool:
        """
        ตรวจสอบความสมบูรณ์ของข้อมูล

        Args:
            required_columns: Columns ที่ต้องไม่มี NULL
            max_missing_pct: % Missing ที่ยอมรับได้

        Returns:
            bool: True if completeness check passes
        """
        for col in required_columns:
            if col not in self.df.columns:
                continue

            missing_count = self.df[col].isnull().sum()
            missing_pct = (missing_count / len(self.df)) * 100

            if missing_pct > 0:
                if col in ['timestamp', 'sensor_id']:
                    # Critical columns ต้องไม่มี NULL เลย
                    self.errors.append(f"{col}: has {missing_count} NULL values")
                elif missing_pct > max_missing_pct:
                    self.errors.append(
                        f"{col}: {missing_pct:.1f}% missing (> {max_missing_pct}%)"
                    )
                else:
                    self.warnings.append(
                        f"{col}: {missing_pct:.1f}% missing"
                    )

        return len(self.errors) == 0

    def validate_ranges(self, range_rules: Dict[str, tuple]) -> bool:
        """
        ตรวจสอบช่วงค่า

        Args:
            range_rules: {'column': (min, max)}

        Returns:
            bool: True if range validation passes
        """
        for col, (min_val, max_val) in range_rules.items():
            if col not in self.df.columns:
                continue

            out_of_range = self.df[
                (self.df[col] < min_val) | (self.df[col] > max_val)
            ]

            if len(out_of_range) > 0:
                out_of_range_pct = (len(out_of_range) / len(self.df)) * 100
                self.errors.append(
                    f"{col}: {len(out_of_range)} values out of range "
                    f"[{min_val}, {max_val}] ({out_of_range_pct:.1f}%)"
                )

        return len(self.errors) == 0

    def validate_uniqueness(self, unique_columns: List[str]) -> bool:
        """
        ตรวจสอบความไม่ซ้ำ

        Args:
            unique_columns: Columns ที่ต้องไม่ซ้ำ

        Returns:
            bool: True if uniqueness check passes
        """
        duplicates = self.df[self.df.duplicated(subset=unique_columns, keep=False)]

        if len(duplicates) > 0:
            dup_pct = (len(duplicates) / len(self.df)) * 100
            self.errors.append(
                f"Found {len(duplicates)} duplicate records ({dup_pct:.1f}%)"
            )
            return False

        return True

    def validate_consistency(self) -> bool:
        """
        ตรวจสอบความสอดคล้องของข้อมูล

        Returns:
            bool: True if consistency checks pass
        """
        # Example: timestamp ต้องเรียงตามลำดับ
        if 'timestamp' in self.df.columns:
            if not self.df['timestamp'].is_monotonic_increasing:
                self.warnings.append("Timestamps are not in chronological order")

        # Example: temperature และ humidity ควรมีความสัมพันธ์
        if 'temperature' in self.df.columns and 'humidity' in self.df.columns:
            # High temp ควรมี low humidity (โดยทั่วไป)
            # นี่เป็นตัวอย่าง - ขึ้นกับ domain knowledge
            pass

        return True

    def get_report(self) -> Dict[str, Any]:
        """
        สร้างรายงานการ Validate

        Returns:
            dict: Validation report
        """
        return {
            'total_records': len(self.df),
            'errors': self.errors,
            'warnings': self.warnings,
            'is_valid': len(self.errors) == 0,
            'quality_score': self._calculate_quality_score()
        }

    def _calculate_quality_score(self) -> float:
        """
        คำนวณคะแนนคุณภาพข้อมูล (0-100)

        Returns:
            float: Quality score
        """
        deductions = 0

        # ลดคะแนนตามจำนวน errors และ warnings
        deductions += len(self.errors) * 10  # Error = -10 points
        deductions += len(self.warnings) * 2  # Warning = -2 points

        score = max(0, 100 - deductions)
        return score


# ===========================
# การใช้งาน Validator
# ===========================

def validate_iot_data(df: pd.DataFrame) -> Dict[str, Any]:
    """
    Validate IoT sensor data

    Args:
        df: DataFrame to validate

    Returns:
        dict: Validation report
    """
    validator = DataValidator(df)

    # 1. Schema Validation
    expected_schema = {
        'timestamp': 'datetime',
        'sensor_id': 'object',
        'temperature': 'float',
        'humidity': 'int'
    }
    validator.validate_schema(expected_schema)

    # 2. Completeness Validation
    required_columns = ['timestamp', 'sensor_id', 'temperature', 'humidity']
    validator.validate_completeness(required_columns, max_missing_pct=5.0)

    # 3. Range Validation
    range_rules = {
        'temperature': (-50, 100),  # °C
        'humidity': (0, 100),       # %
    }
    validator.validate_ranges(range_rules)

    # 4. Uniqueness Validation
    validator.validate_uniqueness(['timestamp', 'sensor_id'])

    # 5. Consistency Validation
    validator.validate_consistency()

    return validator.get_report()


# ===========================
# ตัวอย่างการใช้งาน
# ===========================

# สร้างข้อมูลทดสอบ
df_test = pd.DataFrame({
    'timestamp': pd.date_range('2025-01-01', periods=100, freq='5min'),
    'sensor_id': ['S001'] * 100,
    'temperature': np.random.uniform(20, 30, 100),
    'humidity': np.random.randint(40, 80, 100)
})

# Add some issues
df_test.loc[10, 'temperature'] = 150  # Out of range
df_test.loc[20, 'humidity'] = None     # Missing value

# Validate
report = validate_iot_data(df_test)

print("=== Validation Report ===")
print(f"Total Records: {report['total_records']}")
print(f"Valid: {report['is_valid']}")
print(f"Quality Score: {report['quality_score']}/100")

if report['errors']:
    print("\nErrors:")
    for error in report['errors']:
        print(f"  ❌ {error}")

if report['warnings']:
    print("\nWarnings:")
    for warning in report['warnings']:
        print(f"  ⚠️  {warning}")
```

### 2.2 Custom Validation Rules

```python
from typing import Callable

class CustomValidator(DataValidator):
    """
    Extended validator with custom rules
    """

    def add_custom_rule(self, rule_name: str,
                       rule_func: Callable,
                       error_message: str):
        """
        เพิ่ม Custom Validation Rule

        Args:
            rule_name: ชื่อ rule
            rule_func: Function ที่ return True/False
            error_message: ข้อความแจ้ง error
        """
        if not rule_func(self.df):
            self.errors.append(f"{rule_name}: {error_message}")


# ตัวอย่าง Custom Rules
def rule_temperature_humidity_correlation(df):
    """Check if temp and humidity have inverse correlation"""
    if 'temperature' in df.columns and 'humidity' in df.columns:
        corr = df['temperature'].corr(df['humidity'])
        return corr < 0  # ควรมี negative correlation
    return True

def rule_minimum_records(df, min_records=10):
    """Check minimum number of records"""
    return len(df) >= min_records

def rule_sensor_distribution(df):
    """Check if sensors are evenly distributed"""
    if 'sensor_id' in df.columns:
        sensor_counts = df['sensor_id'].value_counts()
        # Check if any sensor has < 10% of total
        min_pct = (sensor_counts.min() / len(df)) * 100
        return min_pct >= 10
    return True


# ใช้งาน
validator = CustomValidator(df_test)
validator.add_custom_rule(
    'temp_humidity_correlation',
    rule_temperature_humidity_correlation,
    'Temperature and humidity should have negative correlation'
)
validator.add_custom_rule(
    'minimum_records',
    lambda df: rule_minimum_records(df, min_records=50),
    'Must have at least 50 records'
)
```

---

## 3. Unit Testing for ETL

### 3.1 Why Unit Testing?

**ประโยชน์:**
- ✅ ตรวจจับ bugs ก่อนส่ง production
- ✅ ป้องกัน regression (code ใหม่ทำให้เดิมพัง)
- ✅ เป็น documentation ของ code
- ✅ Refactor ได้อย่างมั่นใจ

### 3.2 Unit Tests สำหรับ ETL Functions

```python
import unittest
import pandas as pd
import numpy as np
from etl_functions import (
    extract_sensor_data,
    transform_sensor_data,
    validate_sensor_data,
    load_sensor_data
)
import tempfile
import os


class TestETLFunctions(unittest.TestCase):
    """
    Unit tests for ETL functions
    """

    def setUp(self):
        """Setup test data before each test"""
        # สร้าง test DataFrame
        self.test_df = pd.DataFrame({
            'timestamp': pd.date_range('2025-01-01', periods=100, freq='5min'),
            'sensor_id': ['S001'] * 100,
            'temperature': np.random.uniform(20, 30, 100),
            'humidity': np.random.randint(40, 80, 100),
            'pressure': np.random.uniform(990, 1020, 100)
        })

        # สร้าง temporary CSV file
        self.temp_file = tempfile.NamedTemporaryFile(mode='w', delete=False, suffix='.csv')
        self.test_df.to_csv(self.temp_file.name, index=False)
        self.temp_file.close()

    def tearDown(self):
        """Cleanup after each test"""
        # ลบ temporary file
        if os.path.exists(self.temp_file.name):
            os.remove(self.temp_file.name)

    # =============================
    # Tests for Extract
    # =============================

    def test_extract_returns_dataframe(self):
        """Test extract_sensor_data returns DataFrame"""
        result = extract_sensor_data(self.temp_file.name)
        self.assertIsInstance(result, pd.DataFrame)

    def test_extract_preserves_row_count(self):
        """Test extract doesn't lose rows"""
        result = extract_sensor_data(self.temp_file.name)
        self.assertEqual(len(result), len(self.test_df))

    def test_extract_has_required_columns(self):
        """Test extract returns all required columns"""
        result = extract_sensor_data(self.temp_file.name)
        required_cols = ['timestamp', 'sensor_id', 'temperature', 'humidity']
        for col in required_cols:
            self.assertIn(col, result.columns)

    def test_extract_invalid_file_raises_error(self):
        """Test extract raises error for invalid file"""
        with self.assertRaises(Exception):
            extract_sensor_data('/nonexistent/file.csv')

    # =============================
    # Tests for Transform
    # =============================

    def test_transform_adds_features(self):
        """Test transform adds new feature columns"""
        result = transform_sensor_data(self.test_df.copy())

        expected_features = ['temp_rolling_1h', 'hour', 'day_of_week']
        for feature in expected_features:
            self.assertIn(feature, result.columns)

    def test_transform_handles_missing_values(self):
        """Test transform handles missing values"""
        # Add missing values
        df_with_nulls = self.test_df.copy()
        df_with_nulls.loc[10, 'temperature'] = np.nan

        result = transform_sensor_data(df_with_nulls)

        # Check no nulls in temperature
        self.assertEqual(result['temperature'].isnull().sum(), 0)

    def test_transform_removes_outliers(self):
        """Test transform removes outliers"""
        # Add outlier
        df_with_outliers = self.test_df.copy()
        df_with_outliers.loc[10, 'temperature'] = 200  # Extreme value

        result = transform_sensor_data(df_with_outliers)

        # Check outlier removed
        self.assertTrue((result['temperature'] >= -50).all())
        self.assertTrue((result['temperature'] <= 100).all())

    def test_transform_preserves_data_types(self):
        """Test transform preserves correct data types"""
        result = transform_sensor_data(self.test_df.copy())

        self.assertTrue(pd.api.types.is_datetime64_any_dtype(result['timestamp']))
        self.assertTrue(pd.api.types.is_numeric_dtype(result['temperature']))

    # =============================
    # Tests for Validate
    # =============================

    def test_validate_accepts_valid_data(self):
        """Test validate passes for valid data"""
        # Should not raise error
        try:
            validate_sensor_data(self.test_df)
        except Exception as e:
            self.fail(f"Validate raised unexpected exception: {e}")

    def test_validate_rejects_missing_columns(self):
        """Test validate rejects data with missing columns"""
        df_incomplete = self.test_df.drop('temperature', axis=1)

        with self.assertRaises(ValueError):
            validate_sensor_data(df_incomplete)

    def test_validate_rejects_null_timestamps(self):
        """Test validate rejects NULL timestamps"""
        df_with_null = self.test_df.copy()
        df_with_null.loc[10, 'timestamp'] = pd.NaT

        with self.assertRaises(ValueError):
            validate_sensor_data(df_with_null)

    def test_validate_rejects_out_of_range(self):
        """Test validate rejects out of range values"""
        df_invalid = self.test_df.copy()
        df_invalid.loc[10, 'temperature'] = 200  # Out of range

        with self.assertRaises(ValueError):
            validate_sensor_data(df_invalid)

    # =============================
    # Tests for Load
    # =============================

    def test_load_creates_output_file(self):
        """Test load creates output file"""
        with tempfile.TemporaryDirectory() as tmpdir:
            load_sensor_data(self.test_df, tmpdir, partition_by_date=False)

            # Check file created
            files = os.listdir(tmpdir)
            self.assertTrue(len(files) > 0)

    def test_load_partitioned_creates_structure(self):
        """Test load with partitioning creates correct structure"""
        with tempfile.TemporaryDirectory() as tmpdir:
            load_sensor_data(self.test_df, tmpdir, partition_by_date=True)

            # Check partition folders created
            year_folders = [d for d in os.listdir(tmpdir) if d.startswith('year=')]
            self.assertTrue(len(year_folders) > 0)


# =============================
# Integration Tests
# =============================

class TestETLPipeline(unittest.TestCase):
    """
    Integration tests for complete ETL pipeline
    """

    def test_full_pipeline(self):
        """Test complete ETL pipeline end-to-end"""
        # Create test data
        test_df = pd.DataFrame({
            'timestamp': pd.date_range('2025-01-01', periods=50, freq='5min'),
            'sensor_id': ['S001'] * 50,
            'temperature': np.random.uniform(20, 30, 50),
            'humidity': np.random.randint(40, 80, 50),
            'pressure': np.random.uniform(990, 1020, 50)
        })

        # Save to temp file
        with tempfile.NamedTemporaryFile(mode='w', delete=False, suffix='.csv') as f:
            test_df.to_csv(f.name, index=False)
            temp_csv = f.name

        try:
            # Extract
            df_extracted = extract_sensor_data(temp_csv)
            self.assertEqual(len(df_extracted), 50)

            # Transform
            df_transformed = transform_sensor_data(df_extracted)
            self.assertIn('temp_rolling_1h', df_transformed.columns)

            # Validate
            validate_sensor_data(df_transformed)  # Should not raise

            # Load
            with tempfile.TemporaryDirectory() as tmpdir:
                output_path = load_sensor_data(df_transformed, tmpdir)
                self.assertTrue(os.path.exists(tmpdir))

        finally:
            # Cleanup
            if os.path.exists(temp_csv):
                os.remove(temp_csv)


# =============================
# Run Tests
# =============================

if __name__ == '__main__':
    # Run tests
    unittest.main(verbosity=2)
```

### 3.3 Running Tests

```bash
# รัน tests ทั้งหมด
python -m unittest test_etl_functions.py

# รัน test class เดียว
python -m unittest test_etl_functions.TestETLFunctions

# รัน test method เดียว
python -m unittest test_etl_functions.TestETLFunctions.test_extract_returns_dataframe

# รันด้วย pytest (ถ้าติดตั้ง)
pip install pytest
pytest test_etl_functions.py -v

# Coverage report
pip install pytest-cov
pytest test_etl_functions.py --cov=etl_functions --cov-report=html
```

---

## 4. Pipeline Idempotency

### 4.1 Idempotency คืออะไร?

**Idempotent Pipeline** = Pipeline ที่รันซ้ำกี่ครั้งก็ได้ผลลัพธ์เดิม

```
Idempotent vs Non-Idempotent:

┌──────────────────────────────────┐
│    Idempotent (ดี)               │
├──────────────────────────────────┤
│  Run 1: Process 100 records      │
│  Run 2: Process 100 records      │
│  Run 3: Process 100 records      │
│  Result: 100 records (same)      │
└──────────────────────────────────┘

┌──────────────────────────────────┐
│    Non-Idempotent (แย่)          │
├──────────────────────────────────┤
│  Run 1: Insert 100 records       │
│  Run 2: Insert 100 records       │
│  Run 3: Insert 100 records       │
│  Result: 300 records (duplicate!)│
└──────────────────────────────────┘
```

### 4.2 ทำให้ Pipeline เป็น Idempotent

**เทคนิค:**

1. **Use UPSERT instead of INSERT**
```python
# ❌ Non-Idempotent
df.to_sql('sensor_data', con=engine, if_exists='append')

# ✅ Idempotent
# ลบข้อมูลวันเดิมก่อน แล้วค่อย insert
engine.execute(f"DELETE FROM sensor_data WHERE date = '{target_date}'")
df.to_sql('sensor_data', con=engine, if_exists='append')
```

2. **Check if already processed**
```python
def is_already_processed(file_path):
    """Check if file was already processed"""
    processed_files = load_processed_files_log()
    return file_path in processed_files

def process_file(file_path):
    if is_already_processed(file_path):
        print(f"File {file_path} already processed, skipping...")
        return

    # Process file
    df = extract_sensor_data(file_path)
    # ... ETL ...

    # Mark as processed
    mark_as_processed(file_path)
```

3. **Use deterministic output paths**
```python
# ✅ Idempotent - same input always writes to same output
output_path = f'output/year={year}/month={month}/day={day}/data.parquet'

# ❌ Non-Idempotent - uses timestamp
output_path = f'output/data_{datetime.now().timestamp()}.parquet'
```

4. **Delete before write**
```python
# ✅ Idempotent
if os.path.exists(output_file):
    os.remove(output_file)
df.to_parquet(output_file)
```

---

## 5. Data Quality Metrics

### 5.1 Key Metrics

```python
def calculate_quality_metrics(df: pd.DataFrame) -> Dict[str, float]:
    """
    คำนวณ Data Quality Metrics

    Returns:
        dict: Quality metrics
    """
    total_cells = len(df) * len(df.columns)

    metrics = {
        # 1. Completeness
        'completeness': (1 - df.isnull().sum().sum() / total_cells) * 100,

        # 2. Uniqueness
        'uniqueness': (1 - df.duplicated().sum() / len(df)) * 100,

        # 3. Validity (example: temperature range)
        'validity': 0.0,

        # 4. Consistency
        'consistency': 0.0,

        # 5. Timeliness
        'timeliness': 0.0
    }

    # Validity: % records with valid values
    if 'temperature' in df.columns:
        valid_temp = len(df[(df['temperature'] >= -50) & (df['temperature'] <= 100)])
        metrics['validity'] = (valid_temp / len(df)) * 100

    # Consistency: timestamp ordering
    if 'timestamp' in df.columns:
        metrics['consistency'] = 100.0 if df['timestamp'].is_monotonic_increasing else 90.0

    # Timeliness: data freshness (example)
    if 'timestamp' in df.columns:
        latest_time = df['timestamp'].max()
        age_hours = (pd.Timestamp.now() - latest_time).total_seconds() / 3600
        metrics['timeliness'] = max(0, 100 - age_hours)  # Decrease with age

    return metrics
```

### 5.2 Quality Dashboard

```python
import matplotlib.pyplot as plt

def plot_quality_metrics(metrics: Dict[str, float]):
    """
    สร้างกราฟแสดง Quality Metrics

    Args:
        metrics: Dictionary ของ metrics
    """
    fig, ax = plt.subplots(figsize=(10, 6))

    metric_names = list(metrics.keys())
    metric_values = list(metrics.values())

    colors = ['green' if v >= 95 else 'orange' if v >= 80 else 'red'
              for v in metric_values]

    bars = ax.barh(metric_names, metric_values, color=colors)

    # Add value labels
    for bar in bars:
        width = bar.get_width()
        ax.text(width, bar.get_y() + bar.get_height()/2,
                f'{width:.1f}%',
                ha='left', va='center', fontweight='bold')

    ax.set_xlabel('Score (%)')
    ax.set_title('Data Quality Metrics')
    ax.set_xlim(0, 100)
    ax.axvline(x=95, color='green', linestyle='--', alpha=0.3, label='Excellent (≥95%)')
    ax.axvline(x=80, color='orange', linestyle='--', alpha=0.3, label='Good (≥80%)')
    ax.legend()

    plt.tight_layout()
    plt.savefig('quality_metrics.png', dpi=150)
    print("✅ Quality metrics chart saved")
```

---

## 6. Testing Strategies

### 6.1 Testing Pyramid

```
Testing Pyramid:

         ╱╲
        ╱  ╲
       ╱ E2E ╲           ← น้อยที่สุด (ช้า, แพง)
      ╱──────╲
     ╱ Integ. ╲          ← ปานกลาง
    ╱──────────╲
   ╱ Unit Tests ╲        ← มากที่สุด (เร็ว, ถูก)
  ╱──────────────╲

Level 1 (70%): Unit Tests
  - Test individual functions
  - Fast, isolated
  - Mock external dependencies

Level 2 (20%): Integration Tests
  - Test components working together
  - Test database interactions
  - Test file I/O

Level 3 (10%): End-to-End Tests
  - Test complete pipeline
  - Test with real data
  - Slow but comprehensive
```

### 6.2 Test Coverage

```bash
# ติดตั้ง coverage tool
pip install coverage

# รัน tests พร้อม coverage
coverage run -m unittest test_etl_functions.py

# ดู report
coverage report -m

# สร้าง HTML report
coverage html
# เปิด htmlcov/index.html
```

**เป้าหมาย Coverage:**
- ✅ Critical functions: 100%
- ✅ Overall: ≥ 80%

---

## 7. สรุป Module 7

### 7.1 Key Takeaways

✅ **Data Quality** มี 6 มิติ: Accuracy, Completeness, Consistency, Timeliness, Validity, Uniqueness
✅ **Validation Framework** ตรวจสอบ Schema, Ranges, Nulls, Duplicates
✅ **Unit Testing** ป้องกัน bugs และ regression
✅ **Idempotency** รันซ้ำได้ผลลัพธ์เดิม
✅ **Quality Metrics** วัดและ Monitor คุณภาพข้อมูล
✅ **Testing Pyramid** Unit (70%), Integration (20%), E2E (10%)

### 7.2 Testing Best Practices

| Practice | Description |
|----------|-------------|
| ✅ **Test First** | เขียน tests ก่อน code (TDD) |
| ✅ **Isolated Tests** | แต่ละ test ไม่ขึ้นกัน |
| ✅ **Fast Tests** | Tests รันเร็ว (< 1 วินาที) |
| ✅ **Readable Tests** | ชื่อ test บอกความตั้งใจชัดเจน |
| ✅ **High Coverage** | ≥ 80% code coverage |
| ✅ **Automated** | รัน tests อัตโนมัติใน CI/CD |

### 7.3 Skills ที่ได้จาก Module นี้

| Skill | Level |
|-------|-------|
| Data Validation | ⭐⭐⭐⭐ |
| Unit Testing | ⭐⭐⭐⭐ |
| Idempotency Design | ⭐⭐⭐ |
| Quality Metrics | ⭐⭐⭐⭐ |
| Testing Strategies | ⭐⭐⭐ |

### 7.4 เตรียมพร้อมสำหรับ Module 8

ใน Module สุดท้าย คุณจะได้:
- สรุปทั้งหมดที่เรียนมา
- Map Skills กับ Career Path
- มุมมอง Real-Time Processing และ MLOps
- Final Project Exercise

---

## 📝 Challenge Questions

1. **Data Quality:** อะไรคือความแตกต่างระหว่าง Accuracy และ Validity?
2. **Testing:** ทำไม Unit Tests ต้องเยอะกว่า Integration Tests?
3. **Idempotency:** Pipeline แบบไหนไม่เป็น Idempotent? ยกตัวอย่าง
4. **Coverage:** Coverage 100% รับประกันว่าไม่มี bugs ไหม? ทำไม?

---

## 🎯 ถัดไป: Labs & Practical Exercises

**👉 [เริ่มทำ Lab Module 7](./labs/lab-module-7.md)**

ใน Lab คุณจะได้:
- สร้าง Data Validation Framework
- เขียน Unit Tests สำหรับ ETL Functions
- ทดสอบ Pipeline Idempotency
- สร้าง Quality Metrics Dashboard

---

**[⬅️ Module 6: Airflow IoT Pipeline](../module-6/module-6.md)** | **[🏠 Wiki](../wiki.md)** | **[Module 8: Summary & Future ➡️](../module-8/module-8.md)**
