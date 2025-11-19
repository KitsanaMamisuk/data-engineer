# Functions และ Modular Programming

## 1. Abstract (60 วินาที)

Functions คือการแบ่ง code ออกเป็นส่วนย่อยที่ทำงานเฉพาะอย่าง ทำให้ reuse ได้ test ง่าย และอ่านเข้าใจง่าย Modular Programming คือการออกแบบ code ให้เป็น modules ที่เป็นอิสระต่อกัน ในงาน BRMS ทุก rule ควรเป็น function แยก เช่น `check_eligibility()`, `calculate_score()`, `make_decision()` ช่วยให้ maintain และ test ได้ง่าย

---

## 2. Core Principles (20/80)

หลัก 20% ที่ให้ผลลัพธ์ 80%:

1. **Single Responsibility** - 1 function ทำ 1 อย่าง
2. **Pure Functions** - ผลลัพธ์ขึ้นกับ input เท่านั้น
3. **Meaningful Names** - ชื่อบอกว่าทำอะไร
4. **Small Functions** - ไม่เกิน 20-30 บรรทัด
5. **Type Hints** - ระบุ input/output types

---

## 3. Mental Models

### Model 1: "เครื่องจักร"
- Function = เครื่องจักร
- Input = วัตถุดิบ
- Output = ผลผลิต
- เครื่องแต่ละตัวทำงานเฉพาะอย่าง

### Model 2: "สูตรอาหาร"
- Function = 1 สูตร
- Parameters = วัตถุดิบ
- Return = อาหารที่ปรุงเสร็จ

### Model 3: "กล่องดำ"
- ไม่ต้องรู้ว่าทำงานอย่างไรข้างใน
- รู้แค่ input อะไร output อะไร

---

## 4. Skill Tree

```
พื้นฐาน
├── เขียน function พื้นฐาน
├── ใช้ parameters และ return
└── เข้าใจ scope

กลาง
├── Default parameters
├── *args, **kwargs
├── Lambda functions
└── Decorators

สูง
├── Higher-order functions
├── Closures
├── Function composition
└── Functional programming patterns
```

---

## 5. What NOT to Learn

- **อย่าเรียน** functional programming ลึก → เน้น practical
- **อย่าเรียน** advanced decorators → รู้พื้นฐานพอ
- **อย่าเรียน** metaclasses → ไม่ค่อยใช้
- **อย่าท่อง** ทุก built-in functions → เรียนที่ใช้บ่อย
- **อย่าเรียน** async functions ตอนนี้ → ค่อยเรียนทีหลัง

---

## 6. Real-world Usage

### ในงาน BRMS:

1. **Validation functions** - ตรวจสอบข้อมูล
2. **Calculation functions** - คำนวณค่าต่างๆ
3. **Decision functions** - ตัดสินใจ
4. **Helper functions** - ช่วยเหลือทั่วไป

---

## 7. Typical Pitfalls (10 ข้อ)

| # | ความผิดพลาด | วิธีเลี่ยง |
|---|-------------|-----------|
| 1 | Function ใหญ่เกินไป | แยกเป็น function ย่อย |
| 2 | ชื่อไม่สื่อความหมาย | ใช้ชื่อที่บอกว่าทำอะไร |
| 3 | Side effects มากเกินไป | ใช้ pure functions |
| 4 | Mutable default argument | ใช้ None แทน |
| 5 | ไม่ handle errors | เพิ่ม error handling |
| 6 | ไม่มี docstring | เขียน docstring ทุก function |
| 7 | Parameters มากเกินไป | Group เป็น object |
| 8 | Nested functions ลึก | Flatten ออกมา |
| 9 | ไม่ใช้ type hints | ใส่ type hints เสมอ |
| 10 | Return หลาย types | Return type เดียว |

---

## 8. Playbook (Step-by-step)

### เขียน Function ที่ดี:

1. **กำหนดวัตถุประสงค์** - function นี้ทำอะไร?
2. **ตั้งชื่อ** - ใช้ verb + noun (e.g., `calculate_score`)
3. **กำหนด parameters** - input อะไรบ้าง?
4. **กำหนด return type** - output เป็นอะไร?
5. **เขียน docstring** - อธิบาย function
6. **Implement** - เขียน logic
7. **Add type hints** - ระบุ types
8. **Test** - ทดสอบ function

---

## 9. Fast Practical Examples

### Example 1: Basic Function Structure

```python
def check_age_eligibility(age: int, min_age: int = 20, max_age: int = 60) -> bool:
    """
    ตรวจสอบว่าอายุอยู่ในเกณฑ์หรือไม่

    Args:
        age: อายุของผู้สมัคร
        min_age: อายุขั้นต่ำ (default: 20)
        max_age: อายุสูงสุด (default: 60)

    Returns:
        bool: True ถ้าอายุอยู่ในเกณฑ์
    """
    return min_age <= age <= max_age

# Usage
is_eligible = check_age_eligibility(25)  # True
is_eligible = check_age_eligibility(18)  # False
is_eligible = check_age_eligibility(25, min_age=18)  # True
```

### Example 2: Function with Multiple Returns

```python
from typing import Optional

def validate_salary(
    salary: float,
    min_salary: float = 15000
) -> tuple[bool, Optional[str]]:
    """
    ตรวจสอบรายได้

    Returns:
        tuple: (is_valid, error_message)
    """
    if salary <= 0:
        return False, "รายได้ต้องมากกว่า 0"

    if salary < min_salary:
        return False, f"รายได้ต่ำกว่าเกณฑ์ขั้นต่ำ {min_salary:,.0f} บาท"

    return True, None

# Usage
is_valid, error = validate_salary(10000)
if not is_valid:
    print(error)  # "รายได้ต่ำกว่าเกณฑ์ขั้นต่ำ 15,000 บาท"
```

### Example 3: Functions with Dict Input/Output

```python
def calculate_debt_ratio(customer: dict) -> dict:
    """
    คำนวณอัตราส่วนหนี้ต่อรายได้

    Args:
        customer: dict containing monthly_debt and monthly_income

    Returns:
        dict: {ratio: float, percentage: str, is_acceptable: bool}
    """
    monthly_debt = customer.get("monthly_debt", 0)
    monthly_income = customer.get("monthly_income", 0)

    if monthly_income <= 0:
        return {
            "ratio": float("inf"),
            "percentage": "N/A",
            "is_acceptable": False,
            "error": "รายได้ต้องมากกว่า 0"
        }

    ratio = monthly_debt / monthly_income

    return {
        "ratio": round(ratio, 4),
        "percentage": f"{ratio * 100:.2f}%",
        "is_acceptable": ratio <= 0.5
    }

# Usage
customer = {"monthly_debt": 15000, "monthly_income": 50000}
result = calculate_debt_ratio(customer)
print(result)
# {'ratio': 0.3, 'percentage': '30.00%', 'is_acceptable': True}
```

### Example 4: Modular Functions for Rules

```python
# แบ่ง code เป็น modules ย่อย

# --- Validation Module ---
def validate_age(age: int) -> tuple[bool, str]:
    if age < 20:
        return False, "อายุต่ำกว่า 20 ปี"
    if age > 60:
        return False, "อายุเกิน 60 ปี"
    return True, ""

def validate_salary(salary: float) -> tuple[bool, str]:
    if salary < 15000:
        return False, "รายได้ต่ำกว่า 15,000 บาท"
    return True, ""

def validate_documents(docs: list[str]) -> tuple[bool, str]:
    required = ["ID_CARD", "INCOME_PROOF"]
    missing = [d for d in required if d not in docs]
    if missing:
        return False, f"ขาดเอกสาร: {', '.join(missing)}"
    return True, ""

# --- Calculation Module ---
def calculate_credit_score(factors: dict) -> int:
    score = 0
    score += factors.get("payment_history", 0) * 350
    score += factors.get("credit_utilization", 0) * 300
    score += factors.get("credit_age", 0) * 150
    score += factors.get("credit_mix", 0) * 100
    score += factors.get("new_credit", 0) * 100
    return int(score)

def calculate_max_loan(salary: float, debt_ratio: float) -> float:
    available = salary * (1 - debt_ratio)
    return available * 15

# --- Decision Module ---
def make_decision(score: int, validations: list[tuple[bool, str]]) -> dict:
    # Check validations
    errors = [msg for valid, msg in validations if not valid]
    if errors:
        return {"decision": "REJECTED", "reasons": errors}

    # Check score
    if score >= 700:
        return {"decision": "APPROVED", "reasons": ["คะแนนเครดิตดี"]}
    elif score >= 600:
        return {"decision": "REVIEW", "reasons": ["ต้องพิจารณาเพิ่มเติม"]}
    else:
        return {"decision": "REJECTED", "reasons": ["คะแนนเครดิตต่ำ"]}

# --- Main Process ---
def process_loan_application(application: dict) -> dict:
    """Main function ที่ใช้ modular functions"""

    # Validate
    validations = [
        validate_age(application["age"]),
        validate_salary(application["salary"]),
        validate_documents(application["documents"])
    ]

    # Calculate
    score = calculate_credit_score(application["credit_factors"])
    max_loan = calculate_max_loan(
        application["salary"],
        application.get("debt_ratio", 0)
    )

    # Decide
    decision = make_decision(score, validations)
    decision["max_loan"] = max_loan
    decision["credit_score"] = score

    return decision
```

### Example 5: Higher-Order Functions

```python
from typing import Callable

# Function ที่รับ function เป็น parameter
def apply_rules(
    data: dict,
    rules: list[Callable[[dict], tuple[bool, str]]]
) -> list[str]:
    """
    Apply หลาย rules กับ data

    Args:
        data: ข้อมูลที่จะตรวจสอบ
        rules: list ของ rule functions

    Returns:
        list: รายการ error messages
    """
    errors = []
    for rule in rules:
        is_valid, message = rule(data)
        if not is_valid:
            errors.append(message)
    return errors

# Rule functions
def rule_min_age(data: dict) -> tuple[bool, str]:
    return data.get("age", 0) >= 20, "อายุต่ำกว่า 20 ปี"

def rule_min_salary(data: dict) -> tuple[bool, str]:
    return data.get("salary", 0) >= 15000, "รายได้ต่ำกว่า 15,000 บาท"

def rule_max_debt_ratio(data: dict) -> tuple[bool, str]:
    return data.get("debt_ratio", 0) <= 0.5, "หนี้สินมากกว่า 50%"

# Usage
customer = {"age": 18, "salary": 30000, "debt_ratio": 0.3}
rules = [rule_min_age, rule_min_salary, rule_max_debt_ratio]
errors = apply_rules(customer, rules)
print(errors)  # ["อายุต่ำกว่า 20 ปี"]
```

### Example 6: Function Factory (Closure)

```python
def create_threshold_checker(
    field: str,
    threshold: float,
    operator: str = ">="
) -> Callable[[dict], bool]:
    """
    สร้าง function สำหรับ check threshold

    Args:
        field: ชื่อ field ที่จะ check
        threshold: ค่า threshold
        operator: ">=" หรือ "<="

    Returns:
        function ที่ check threshold
    """
    def checker(data: dict) -> bool:
        value = data.get(field, 0)
        if operator == ">=":
            return value >= threshold
        elif operator == "<=":
            return value <= threshold
        elif operator == ">":
            return value > threshold
        elif operator == "<":
            return value < threshold
        return False

    return checker

# สร้าง checkers
check_min_salary = create_threshold_checker("salary", 15000)
check_max_debt = create_threshold_checker("debt_ratio", 0.5, "<=")
check_min_age = create_threshold_checker("age", 20)

# Usage
customer = {"salary": 20000, "debt_ratio": 0.3, "age": 25}
print(check_min_salary(customer))  # True
print(check_max_debt(customer))    # True
print(check_min_age(customer))     # True
```

### Example 7: Decorator for Logging

```python
import functools
import logging
from datetime import datetime

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def log_execution(func: Callable) -> Callable:
    """Decorator สำหรับ log function execution"""

    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start_time = datetime.now()
        logger.info(f"Starting {func.__name__}")

        try:
            result = func(*args, **kwargs)
            elapsed = (datetime.now() - start_time).total_seconds()
            logger.info(f"Completed {func.__name__} in {elapsed:.3f}s")
            return result
        except Exception as e:
            logger.error(f"Error in {func.__name__}: {e}")
            raise

    return wrapper

@log_execution
def calculate_credit_score(customer: dict) -> int:
    """Calculate credit score"""
    # ... calculation logic
    return 750

# Usage
score = calculate_credit_score({"id": "C001"})
# Logs:
# INFO:__main__:Starting calculate_credit_score
# INFO:__main__:Completed calculate_credit_score in 0.001s
```

---

## 10. Diagnostic Questions

ตอบคำถามเหล่านี้เพื่อทดสอบความเข้าใจ:

1. ทำไม function ควรทำแค่ 1 อย่าง?
2. Pure function คืออะไร?
3. Side effect คืออะไร และมีปัญหาอย่างไร?
4. Default argument ควรหลีกเลี่ยงอะไร?
5. ทำไมควรใช้ type hints?
6. Docstring ควรมีอะไรบ้าง?
7. Higher-order function คืออะไร?
8. Closure คืออะไร?
9. Decorator ใช้ทำอะไร?
10. เมื่อไหร่ควรแยก function ออกมา?

---

## 11. Explainer for 5-year-old

ลองนึกถึง **เครื่องทำไอศกรีม** นะ

**Function = เครื่องทำไอศกรีม**
- ใส่นม + น้ำตาล + รส (input/parameters)
- เครื่องทำงาน (function body)
- ได้ไอศกรีม (return/output)

ถ้าอยากได้ไอศกรีมช็อคโกแลต ก็ใส่รสช็อคโกแลตเข้าไป
ถ้าอยากได้สตรอเบอร์รี่ ก็เปลี่ยนรส

เครื่องทำงานเหมือนเดิม แค่เปลี่ยน input ก็ได้ output ต่างกัน!

---

## 12. Comparison Table

### Function Types

| Type | คำอธิบาย | ตัวอย่าง |
|------|---------|---------|
| **Pure** | ผลลัพธ์ขึ้นกับ input เท่านั้น | `def add(a, b): return a + b` |
| **Impure** | มี side effects | `def save(data): db.write(data)` |
| **Higher-order** | รับ/return function | `def apply(f, x): return f(x)` |
| **Lambda** | Anonymous function | `lambda x: x * 2` |

### Best Practices

| ควรทำ | ไม่ควรทำ |
|------|---------|
| ชื่อบอกการทำงาน | ชื่อกำกวม (process, handle) |
| 1 function 1 งาน | Function ทำหลายอย่าง |
| Type hints | ไม่ระบุ types |
| Docstring | ไม่มี documentation |
| Return early | Nested if ลึก |
| < 20 lines | Function ยาวมาก |

---

## 13. 7-Day Rapid Learning Roadmap

### สำหรับหัวข้อนี้โดยเฉพาะ (ใช้เวลา 2-3 ชั่วโมง)

| เวลา | กิจกรรม |
|------|---------|
| 0:00-0:20 | อ่าน Abstract และ Core Principles |
| 0:20-0:40 | ศึกษา Example 1-3 (Basic functions) |
| 0:40-1:10 | ศึกษา Example 4 (Modular) |
| 1:10-1:40 | ศึกษา Example 5-7 (Advanced) |
| 1:40-2:10 | ลองเขียน modular functions ด้วยตัวเอง |
| 2:10-2:40 | อ่าน Pitfalls และ Best Practices |
| 2:40-3:00 | ตอบ Diagnostic Questions |

### เป้าหมายหลังจบ:
- [ ] เขียน function ที่มี single responsibility
- [ ] ใช้ type hints และ docstring ได้
- [ ] แบ่ง code เป็น modules ได้
- [ ] เข้าใจ higher-order functions
