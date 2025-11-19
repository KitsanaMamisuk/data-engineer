# Error Handling และ Validation

## 1. Abstract (60 วินาที)

Error Handling คือการจัดการกับข้อผิดพลาดที่อาจเกิดขึ้นใน code ส่วน Validation คือการตรวจสอบข้อมูลก่อนประมวลผล ในงาน BRMS ทั้งสองอย่างสำคัญมากเพราะ: 1) ข้อมูลจาก user อาจผิด 2) ต้อง log errors สำหรับ audit 3) ต้องบอก user ว่าผิดตรงไหน Python ใช้ try-except สำหรับ error handling และควรสร้าง custom exceptions สำหรับ business errors

---

## 2. Core Principles (20/80)

หลัก 20% ที่ให้ผลลัพธ์ 80%:

1. **Validate Early** - ตรวจสอบ input ก่อนทำอะไร
2. **Fail Fast** - พบ error ให้หยุดทันที
3. **Specific Exceptions** - catch exception ที่เฉพาะเจาะจง
4. **Informative Messages** - บอกว่าผิดอะไร ตรงไหน
5. **Log Everything** - บันทึก errors ทั้งหมด

---

## 3. Mental Models

### Model 1: "ด่านตรวจ"
- Validation = ด่านตรวจก่อนเข้าเมือง
- ไม่ผ่าน → ไม่ให้เข้า พร้อมบอกเหตุผล
- ผ่าน → ดำเนินการต่อ

### Model 2: "Try Before You Buy"
- Try = ลองทำ
- Except = ถ้าไม่ได้ ทำอย่างอื่นแทน
- Finally = ทำเสมอไม่ว่าจะสำเร็จหรือไม่

### Model 3: "Safety Net"
- Exception handling = ตาข่ายรองรับ
- ถ้าตก (error) จะไม่แตก

---

## 4. Skill Tree

```
พื้นฐาน
├── try-except-finally
├── Raise exceptions
└── Built-in exceptions

กลาง
├── Custom exceptions
├── Exception chaining
├── Context managers
└── Validation patterns

สูง
├── Exception hierarchies
├── Logging best practices
├── Recovery strategies
└── Monitoring & alerting
```

---

## 5. What NOT to Learn

- **อย่าท่อง** ทุก built-in exceptions → รู้ที่ใช้บ่อย
- **อย่าเรียน** advanced logging frameworks → ใช้ basic logging
- **อย่าเรียน** exception handling ของภาษาอื่น → focus Python
- **อย่าเรียน** distributed error handling → ไม่จำเป็นตอนนี้
- **อย่าเรียน** error recovery patterns ลึก → รู้พื้นฐานพอ

---

## 6. Real-world Usage

### ในงาน BRMS:

1. **Input validation** - ตรวจข้อมูลจาก user/API
2. **Business rule validation** - ตรวจตาม business logic
3. **Database errors** - handle connection/query errors
4. **External service errors** - API calls ล้มเหลว

---

## 7. Typical Pitfalls (10 ข้อ)

| # | ความผิดพลาด | วิธีเลี่ยง |
|---|-------------|-----------|
| 1 | Catch Exception กว้างเกินไป | Catch specific exceptions |
| 2 | Silent failures (pass) | Log หรือ re-raise |
| 3 | ไม่ validate input | Validate ทุก input |
| 4 | Error message กำกวม | บอกให้ชัดว่าผิดอะไร |
| 5 | ไม่ log errors | Log ทุก exception |
| 6 | Return None เมื่อ error | Raise exception แทน |
| 7 | Nested try-except | Flatten หรือแยก function |
| 8 | ไม่ cleanup resources | ใช้ finally หรือ context manager |
| 9 | ไม่มี custom exceptions | สร้าง business exceptions |
| 10 | Validation logic กระจาย | รวมไว้ที่เดียว |

---

## 8. Playbook (Step-by-step)

### Validation Flow:

```
Input Data
    │
    ▼
┌─────────────────┐
│ Type Validation │ → ไม่ผ่าน → ValidationError
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Format Check    │ → ไม่ผ่าน → ValidationError
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Business Rules  │ → ไม่ผ่าน → BusinessError
└────────┬────────┘
         │
         ▼
    Process Data
```

---

## 9. Fast Practical Examples

### Example 1: Basic Try-Except

```python
def get_customer_age(customer: dict) -> int:
    """ดึงอายุลูกค้าอย่างปลอดภัย"""
    try:
        age = customer["age"]
        return int(age)
    except KeyError:
        raise ValueError("ไม่พบข้อมูลอายุ")
    except (TypeError, ValueError):
        raise ValueError("รูปแบบอายุไม่ถูกต้อง")

# Usage
try:
    age = get_customer_age({"name": "John"})
except ValueError as e:
    print(f"Error: {e}")
```

### Example 2: Custom Exceptions

```python
# สร้าง Exception hierarchy สำหรับ BRMS

class BRMSError(Exception):
    """Base exception for BRMS"""
    pass

class ValidationError(BRMSError):
    """Data validation errors"""
    def __init__(self, field: str, message: str):
        self.field = field
        self.message = message
        super().__init__(f"{field}: {message}")

class BusinessRuleError(BRMSError):
    """Business rule violations"""
    def __init__(self, rule_id: str, message: str):
        self.rule_id = rule_id
        self.message = message
        super().__init__(f"Rule {rule_id}: {message}")

class EligibilityError(BusinessRuleError):
    """Customer not eligible"""
    pass

class CalculationError(BRMSError):
    """Calculation errors"""
    pass

# Usage
def validate_salary(salary: float) -> None:
    if salary <= 0:
        raise ValidationError("salary", "ต้องมากกว่า 0")
    if salary < 15000:
        raise EligibilityError("MIN_SALARY", "รายได้ต่ำกว่าเกณฑ์ขั้นต่ำ")

try:
    validate_salary(10000)
except ValidationError as e:
    print(f"Validation failed: {e.field} - {e.message}")
except EligibilityError as e:
    print(f"Not eligible: {e.rule_id} - {e.message}")
```

### Example 3: Comprehensive Validation

```python
from typing import Optional
from dataclasses import dataclass

@dataclass
class ValidationResult:
    is_valid: bool
    errors: list[str]

def validate_loan_application(application: dict) -> ValidationResult:
    """
    Validate loan application data

    Returns:
        ValidationResult with validation status and errors
    """
    errors = []

    # Required fields
    required_fields = ["customer_id", "salary", "age", "loan_amount"]
    for field in required_fields:
        if field not in application:
            errors.append(f"Missing required field: {field}")

    if errors:
        return ValidationResult(False, errors)

    # Type validation
    try:
        salary = float(application["salary"])
        if salary <= 0:
            errors.append("salary must be positive")
    except (TypeError, ValueError):
        errors.append("salary must be a number")

    try:
        age = int(application["age"])
        if age < 0:
            errors.append("age must be positive")
    except (TypeError, ValueError):
        errors.append("age must be an integer")

    try:
        loan_amount = float(application["loan_amount"])
        if loan_amount <= 0:
            errors.append("loan_amount must be positive")
    except (TypeError, ValueError):
        errors.append("loan_amount must be a number")

    # Business rules
    if not errors:
        if age < 20 or age > 60:
            errors.append("age must be between 20 and 60")
        if salary < 15000:
            errors.append("salary must be at least 15,000")
        if loan_amount > salary * 15:
            errors.append("loan_amount exceeds maximum allowed")

    return ValidationResult(len(errors) == 0, errors)

# Usage
application = {
    "customer_id": "C001",
    "salary": 10000,
    "age": 18,
    "loan_amount": 500000
}

result = validate_loan_application(application)
if not result.is_valid:
    for error in result.errors:
        print(f"- {error}")
```

### Example 4: Error Handling with Logging

```python
import logging
from datetime import datetime

# Setup logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger("brms")

def process_rule(rule_id: str, data: dict) -> dict:
    """Process a business rule with proper error handling"""

    logger.info(f"Processing rule {rule_id}")

    try:
        # Validate input
        if not rule_id:
            raise ValidationError("rule_id", "Rule ID is required")

        if not data:
            raise ValidationError("data", "Data is required")

        # Process
        result = execute_rule(rule_id, data)

        logger.info(f"Rule {rule_id} completed successfully")
        return result

    except ValidationError as e:
        logger.warning(f"Validation error in rule {rule_id}: {e}")
        raise

    except BusinessRuleError as e:
        logger.warning(f"Business rule error in {rule_id}: {e}")
        raise

    except Exception as e:
        logger.error(f"Unexpected error in rule {rule_id}: {e}", exc_info=True)
        raise BRMSError(f"Failed to process rule {rule_id}") from e

def execute_rule(rule_id: str, data: dict) -> dict:
    """Execute the actual rule logic"""
    # Rule execution logic here
    return {"status": "completed", "rule_id": rule_id}
```

### Example 5: Context Manager for Cleanup

```python
from contextlib import contextmanager

@contextmanager
def rule_execution_context(rule_id: str):
    """
    Context manager สำหรับ rule execution
    จัดการ setup และ cleanup
    """
    logger.info(f"Starting rule execution: {rule_id}")
    start_time = datetime.now()

    try:
        yield
    except Exception as e:
        logger.error(f"Rule {rule_id} failed: {e}")
        raise
    finally:
        elapsed = (datetime.now() - start_time).total_seconds()
        logger.info(f"Rule {rule_id} finished in {elapsed:.3f}s")

# Usage
def process_loan_application(application: dict) -> dict:
    with rule_execution_context("LOAN_APPROVAL"):
        # Validation
        result = validate_loan_application(application)
        if not result.is_valid:
            raise ValidationError("application", str(result.errors))

        # Process
        decision = make_decision(application)
        return decision
```

### Example 6: Validation Decorator

```python
from functools import wraps
from typing import Callable

def validate_input(schema: dict):
    """
    Decorator สำหรับ validate function input

    Args:
        schema: dict of {field_name: validator_function}
    """
    def decorator(func: Callable) -> Callable:
        @wraps(func)
        def wrapper(*args, **kwargs):
            # Get first argument (assuming it's the data dict)
            if args:
                data = args[0]
            else:
                raise ValidationError("input", "No input data provided")

            # Validate each field
            errors = []
            for field, validator in schema.items():
                if field not in data:
                    errors.append(f"Missing field: {field}")
                    continue

                try:
                    if not validator(data[field]):
                        errors.append(f"Invalid value for {field}")
                except Exception as e:
                    errors.append(f"Validation error for {field}: {e}")

            if errors:
                raise ValidationError("input", ", ".join(errors))

            return func(*args, **kwargs)
        return wrapper
    return decorator

# Validators
def is_positive(value) -> bool:
    return float(value) > 0

def is_valid_age(value) -> bool:
    return 20 <= int(value) <= 60

def is_valid_id(value) -> bool:
    return isinstance(value, str) and len(value) > 0

# Usage
@validate_input({
    "customer_id": is_valid_id,
    "salary": is_positive,
    "age": is_valid_age
})
def calculate_loan_limit(data: dict) -> float:
    return data["salary"] * 15

# Test
try:
    limit = calculate_loan_limit({
        "customer_id": "C001",
        "salary": 50000,
        "age": 18  # Invalid age
    })
except ValidationError as e:
    print(f"Error: {e}")
```

### Example 7: Result Pattern (Alternative to Exceptions)

```python
from dataclasses import dataclass
from typing import TypeVar, Generic, Optional

T = TypeVar('T')

@dataclass
class Result(Generic[T]):
    """Result type that can be success or failure"""
    success: bool
    value: Optional[T] = None
    error: Optional[str] = None

    @classmethod
    def ok(cls, value: T) -> 'Result[T]':
        return cls(success=True, value=value)

    @classmethod
    def fail(cls, error: str) -> 'Result[T]':
        return cls(success=False, error=error)

def validate_and_calculate(data: dict) -> Result[dict]:
    """
    Validate and calculate loan terms
    Returns Result instead of raising exceptions
    """
    # Validate
    if "salary" not in data:
        return Result.fail("Missing salary")

    salary = data.get("salary", 0)
    if salary < 15000:
        return Result.fail("Salary below minimum")

    # Calculate
    max_loan = salary * 15
    rate = 7.0 if salary < 50000 else 6.0

    return Result.ok({
        "max_loan": max_loan,
        "interest_rate": rate
    })

# Usage
result = validate_and_calculate({"salary": 30000})

if result.success:
    print(f"Approved: {result.value}")
else:
    print(f"Failed: {result.error}")
```

---

## 10. Diagnostic Questions

ตอบคำถามเหล่านี้เพื่อทดสอบความเข้าใจ:

1. ทำไมไม่ควร catch Exception กว้างๆ?
2. Custom exception ช่วยอะไร?
3. Validation กับ Error Handling ต่างกันอย่างไร?
4. finally block ใช้ทำอะไร?
5. ทำไมต้อง log errors?
6. Exception chaining คืออะไร?
7. ควร raise exception หรือ return error code?
8. Context manager ช่วยอะไร?
9. Result pattern คืออะไร?
10. เมื่อไหร่ควรใช้ validation decorator?

---

## 11. Explainer for 5-year-old

ลองนึกถึง **การเล่นเกม** นะ

**Validation = กฎก่อนเล่น**
- ต้องมีลูกเต๋า ✓
- ต้องมีตัวหมาก ✓
- ถ้าไม่มี → บอกว่าขาดอะไร

**Error Handling = ถ้าเล่นผิด**
- ถ้าทอยเต๋าหลุด → หยิบมาทอยใหม่
- ถ้าตัวหมากล้ม → ตั้งใหม่
- ไม่ต้องเริ่มเกมใหม่ทั้งหมด!

---

## 12. Comparison Table

### Exception Types ที่ใช้บ่อย

| Exception | เมื่อไหร่ |
|-----------|---------|
| `ValueError` | ค่าไม่ถูกต้อง |
| `TypeError` | Type ผิด |
| `KeyError` | ไม่มี key ใน dict |
| `AttributeError` | ไม่มี attribute |
| `FileNotFoundError` | ไม่พบไฟล์ |
| `ConnectionError` | เชื่อมต่อไม่ได้ |

### Validation vs Exception

| หัวข้อ | Validation | Exception |
|--------|------------|-----------|
| **เมื่อไหร่** | ก่อน process | ระหว่าง process |
| **ใคร error** | User input | System/Logic |
| **Response** | Return errors | Raise exception |
| **Recovery** | ให้ user แก้ไข | Log และ handle |

---

## 13. 7-Day Rapid Learning Roadmap

### สำหรับหัวข้อนี้โดยเฉพาะ (ใช้เวลา 2-2.5 ชั่วโมง)

| เวลา | กิจกรรม |
|------|---------|
| 0:00-0:20 | อ่าน Abstract และ Core Principles |
| 0:20-0:40 | ศึกษา Example 1-2 (Basic, Custom) |
| 0:40-1:00 | ศึกษา Example 3-4 (Comprehensive, Logging) |
| 1:00-1:20 | ศึกษา Example 5-7 (Context, Decorator, Result) |
| 1:20-1:50 | ลองเขียน validation ด้วยตัวเอง |
| 1:50-2:20 | อ่าน Pitfalls |
| 2:20-2:30 | ตอบ Diagnostic Questions |

### เป้าหมายหลังจบ:
- [ ] ใช้ try-except ได้ถูกต้อง
- [ ] สร้าง custom exceptions ได้
- [ ] เขียน validation ที่ครบถ้วน
- [ ] Log errors อย่างเหมาะสม
