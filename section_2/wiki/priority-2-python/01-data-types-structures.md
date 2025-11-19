# Data Types และ Data Structures ที่ใช้บ่อยใน BRMS

## 1. Abstract (60 วินาที)

ในงาน BRMS ที่ใช้ Python จะใช้ data types และ structures หลักๆ คือ: **dict** (เก็บ rules และ data), **list** (เก็บหลาย items), **set** (เก็บ unique values), และ **tuple** (เก็บค่าคงที่) การเลือก data structure ที่ถูกต้องส่งผลต่อ performance และ readability ของ code เช่น ใช้ dict สำหรับ key-value lookup, ใช้ set สำหรับ membership check ที่เร็วกว่า list

---

## 2. Core Principles (20/80)

หลัก 20% ที่ให้ผลลัพธ์ 80%:

1. **dict เป็น king** - ใช้บ่อยที่สุดในงาน BRMS
2. **เลือก structure ตาม use case** - ไม่ใช่ใช้อันเดิมทุกที่
3. **Immutable vs Mutable** - รู้ว่าเมื่อไหร่ใช้อันไหน
4. **Performance matters** - O(1) vs O(n) lookup
5. **Type hints** - ระบุ types เสมอ

---

## 3. Mental Models

### Model 1: "ตู้เก็บของ"
- **dict** = ตู้ล็อคเกอร์ (มี key เปิด)
- **list** = ลิ้นชัก (เรียงลำดับ)
- **set** = กล่องใส่ของไม่ซ้ำ
- **tuple** = กล่องปิดผนึก (แก้ไม่ได้)

### Model 2: "สมุดต่างๆ"
- **dict** = พจนานุกรม (หาคำ → ความหมาย)
- **list** = สมุดบันทึก (เรียงตามลำดับ)
- **set** = สมุดเช็คชื่อ (ไม่ซ้ำ)

### Model 3: "ข้อมูลในธนาคาร"
- **dict** = ข้อมูลลูกค้า {id: data}
- **list** = รายการ transactions
- **set** = blacklist IDs

---

## 4. Skill Tree

```
พื้นฐาน
├── รู้จัก dict, list, set, tuple
├── สร้างและ access ข้อมูลได้
└── เข้าใจ mutable vs immutable

กลาง
├── เลือก structure ให้เหมาะกับงาน
├── ใช้ comprehensions
├── nested structures
└── เข้าใจ time complexity

สูง
├── Custom data classes
├── Performance optimization
├── Type hints และ generics
└── Dataclasses และ Pydantic
```

---

## 5. What NOT to Learn

- **อย่าท่อง** ทุก method → ดู reference เมื่อต้องใช้
- **อย่าเรียน** advanced data structures (heap, graph) → ไม่ค่อยใช้ในงาน BRMS
- **อย่าเรียน** memory management ลึก → Python จัดการให้
- **อย่าเรียน** C implementation → ไม่จำเป็น
- **อย่าเรียน** ทุก built-in functions → เรียนที่ใช้บ่อย

---

## 6. Real-world Usage

### ใช้อะไรเมื่อไหร่ในงาน BRMS:

| Data Structure | Use Case | ตัวอย่าง |
|----------------|----------|---------|
| **dict** | เก็บ rules, customer data | `{"salary": 50000, "age": 30}` |
| **list** | รายการ conditions, errors | `["error1", "error2"]` |
| **set** | blacklist, unique values | `{"blocked_id1", "blocked_id2"}` |
| **tuple** | constants, return multiple | `(min_age, max_age)` |

---

## 7. Typical Pitfalls (10 ข้อ)

| # | ความผิดพลาด | วิธีเลี่ยง |
|---|-------------|-----------|
| 1 | ใช้ list แทน set สำหรับ lookup | ใช้ set ถ้าต้อง check membership |
| 2 | Modify dict ขณะ iterate | สร้าง copy ก่อน iterate |
| 3 | ใช้ mutable default argument | ใช้ None แล้ว assign ใน function |
| 4 | ไม่ handle KeyError | ใช้ .get() หรือ try-except |
| 5 | Nested dict ลึกเกินไป | ใช้ dataclass หรือ flatten |
| 6 | ไม่รู้ว่า list index เริ่มที่ 0 | ระวัง off-by-one errors |
| 7 | เปรียบเทียบ list ด้วย == ผิด | == เทียบ content, is เทียบ identity |
| 8 | ลืมว่า dict keys ต้อง hashable | ใช้ tuple แทน list เป็น key |
| 9 | String concatenation ใน loop | ใช้ list แล้ว join |
| 10 | ไม่ใช้ type hints | ใส่ type hints เสมอ |

---

## 8. Playbook (Step-by-step)

### เลือก Data Structure:

```
เริ่มต้น
    │
    ▼
ต้องการ key-value? ──Yes──→ dict
    │
    No
    │
    ▼
ต้องการลำดับ? ──Yes──→ ต้องแก้ไข? ──Yes──→ list
    │                      │
    No                     No
    │                      │
    ▼                      ▼
ต้อง unique? ──Yes──→ set   tuple
    │
    No
    │
    ▼
  list
```

---

## 9. Fast Practical Examples

### Example 1: dict สำหรับเก็บ Customer Data

```python
# เก็บข้อมูลลูกค้า
customer = {
    "id": "C001",
    "name": "สมชาย",
    "salary": 50000,
    "age": 35,
    "products": ["savings", "credit_card"],
    "is_pep": False
}

# Access ข้อมูล
name = customer["name"]
salary = customer.get("salary", 0)  # มี default

# Check key exists
if "age" in customer:
    print(f"อายุ: {customer['age']}")

# Update
customer["salary"] = 55000

# Add new key
customer["credit_score"] = 720

# Nested dict
customer_profile = {
    "personal": {
        "name": "สมชาย",
        "age": 35
    },
    "financial": {
        "salary": 50000,
        "debt": 10000
    }
}

# Access nested
debt = customer_profile["financial"]["debt"]
```

### Example 2: dict สำหรับ Rule Configuration

```python
# Rule configuration
loan_rules = {
    "eligibility": {
        "min_age": 20,
        "max_age": 60,
        "min_salary": 15000,
        "max_debt_ratio": 0.7
    },
    "scoring": {
        "weights": {
            "payment_history": 0.35,
            "credit_utilization": 0.30,
            "credit_age": 0.15,
            "credit_mix": 0.10,
            "new_credit": 0.10
        }
    },
    "pricing": {
        "base_rate": 7.0,
        "adjustments": {
            "GOLD": -1.0,
            "SILVER": -0.5,
            "NORMAL": 0
        }
    }
}

# Access rule values
min_salary = loan_rules["eligibility"]["min_salary"]
gold_adj = loan_rules["pricing"]["adjustments"]["GOLD"]
```

### Example 3: list สำหรับเก็บหลาย Items

```python
# รายการ errors
errors: list[str] = []

def validate_customer(customer: dict) -> list[str]:
    errors = []

    if customer.get("age", 0) < 20:
        errors.append("อายุต่ำกว่า 20 ปี")

    if customer.get("salary", 0) < 15000:
        errors.append("รายได้ต่ำกว่า 15,000 บาท")

    return errors

# รายการ transactions
transactions = [
    {"id": "T001", "amount": 1000, "type": "deposit"},
    {"id": "T002", "amount": 500, "type": "withdrawal"},
    {"id": "T003", "amount": 2000, "type": "transfer"}
]

# Filter transactions
deposits = [t for t in transactions if t["type"] == "deposit"]

# Sum amounts
total = sum(t["amount"] for t in transactions)

# Sort
sorted_transactions = sorted(transactions, key=lambda t: t["amount"], reverse=True)
```

### Example 4: set สำหรับ Unique Values และ Fast Lookup

```python
# Blacklist - ใช้ set เพราะ lookup เร็ว O(1)
blacklisted_ids: set[str] = {"ID001", "ID002", "ID003"}

def is_blacklisted(customer_id: str) -> bool:
    return customer_id in blacklisted_ids  # O(1)

# เปรียบเทียบกับ list
blacklist_list = ["ID001", "ID002", "ID003"]
# customer_id in blacklist_list  # O(n) - ช้ากว่า

# High risk countries
high_risk_countries: set[str] = {"XX", "YY", "ZZ"}

def check_country_risk(country: str) -> str:
    if country in high_risk_countries:
        return "HIGH"
    return "LOW"

# หา unique values
customer_ids = ["C001", "C002", "C001", "C003", "C002"]
unique_ids = set(customer_ids)  # {"C001", "C002", "C003"}

# Set operations
vip_customers = {"C001", "C002", "C003"}
active_customers = {"C002", "C003", "C004"}

# Intersection - ลูกค้า VIP ที่ active
vip_active = vip_customers & active_customers  # {"C002", "C003"}

# Union - ลูกค้าทั้งหมด
all_customers = vip_customers | active_customers

# Difference - VIP ที่ไม่ active
vip_inactive = vip_customers - active_customers  # {"C001"}
```

### Example 5: tuple สำหรับ Immutable Data

```python
# Constants
AGE_RANGE = (20, 60)  # min, max
SALARY_TIERS = (15000, 30000, 50000, 100000)

def check_age(age: int) -> bool:
    min_age, max_age = AGE_RANGE
    return min_age <= age <= max_age

# Return multiple values
def calculate_loan_terms(customer: dict) -> tuple[float, int]:
    """
    Returns:
        tuple: (interest_rate, max_term_months)
    """
    if customer["credit_score"] >= 750:
        return (5.0, 60)
    elif customer["credit_score"] >= 650:
        return (6.5, 48)
    else:
        return (8.0, 36)

# Unpack
rate, term = calculate_loan_terms({"credit_score": 700})

# Named tuple for clarity
from collections import namedtuple

LoanTerms = namedtuple("LoanTerms", ["rate", "max_term", "max_amount"])

def get_loan_terms(grade: str) -> LoanTerms:
    terms = {
        "A": LoanTerms(5.0, 60, 2000000),
        "B": LoanTerms(6.5, 48, 1000000),
        "C": LoanTerms(8.0, 36, 500000)
    }
    return terms.get(grade, LoanTerms(10.0, 24, 200000))

terms = get_loan_terms("A")
print(f"Rate: {terms.rate}, Term: {terms.max_term}")
```

### Example 6: Comprehensions

```python
# List comprehension
salaries = [30000, 45000, 60000, 25000, 80000]
high_salaries = [s for s in salaries if s >= 50000]

# Dict comprehension
customers = [
    {"id": "C001", "name": "A", "score": 750},
    {"id": "C002", "name": "B", "score": 650},
    {"id": "C003", "name": "C", "score": 700}
]
score_map = {c["id"]: c["score"] for c in customers}
# {"C001": 750, "C002": 650, "C003": 700}

# Set comprehension
unique_scores = {c["score"] for c in customers}

# Nested comprehension
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flattened = [num for row in matrix for num in row]

# Conditional comprehension
grades = {
    c["id"]: "A" if c["score"] >= 750 else "B" if c["score"] >= 650 else "C"
    for c in customers
}
```

### Example 7: Type Hints with Data Structures

```python
from typing import Optional

# Basic type hints
def process_customer(customer: dict[str, any]) -> dict[str, any]:
    pass

# More specific
CustomerData = dict[str, str | int | float | bool]

def get_customer(customer_id: str) -> Optional[CustomerData]:
    pass

# List of dicts
def get_transactions(customer_id: str) -> list[dict[str, any]]:
    pass

# Using TypedDict for structured dicts
from typing import TypedDict

class Customer(TypedDict):
    id: str
    name: str
    salary: int
    age: int

def validate_customer(customer: Customer) -> bool:
    return customer["age"] >= 20 and customer["salary"] >= 15000

# Dataclass (preferred for complex structures)
from dataclasses import dataclass

@dataclass
class CustomerProfile:
    id: str
    name: str
    salary: int
    age: int
    credit_score: int = 0

    def is_eligible(self) -> bool:
        return self.age >= 20 and self.salary >= 15000

customer = CustomerProfile("C001", "สมชาย", 50000, 35, 720)
print(customer.is_eligible())  # True
```

---

## 10. Diagnostic Questions

ตอบคำถามเหล่านี้เพื่อทดสอบความเข้าใจ:

1. เมื่อไหร่ควรใช้ dict แทน list?
2. set ต่างจาก list อย่างไร?
3. ทำไม tuple ถึงเป็น immutable?
4. .get() ต่างจาก [] access อย่างไร?
5. Time complexity ของ `in` operator ใน set vs list?
6. Mutable default argument เป็นปัญหาอย่างไร?
7. เมื่อไหร่ควรใช้ dataclass?
8. Dict comprehension ใช้ทำอะไร?
9. ทำไมต้องใช้ type hints?
10. Nested dict ลึกเกินไปมีปัญหาอย่างไร?

---

## 11. Explainer for 5-year-old

ลองนึกถึง **กล่องเก็บของเล่น** นะ

**dict = กล่องมีป้าย**
- ป้าย "รถ" → รถของเล่น
- ป้าย "ตุ๊กตา" → ตุ๊กตา
- อยากได้รถ? ดูป้าย "รถ" เจอเลย!

**list = กล่องเรียงลำดับ**
- ช่องที่ 1 = รถ
- ช่องที่ 2 = ตุ๊กตา
- ช่องที่ 3 = บอล

**set = กล่องของไม่ซ้ำ**
- ถ้าใส่รถ 2 คัน มันเก็บแค่คันเดียว

**tuple = กล่องล็อค**
- ใส่แล้วเปลี่ยนไม่ได้

---

## 12. Comparison Table

| หัวข้อ | dict | list | set | tuple |
|--------|------|------|-----|-------|
| **Ordered** | Yes (3.7+) | Yes | No | Yes |
| **Mutable** | Yes | Yes | Yes | No |
| **Duplicates** | Keys: No | Yes | No | Yes |
| **Access** | By key O(1) | By index O(1) | N/A | By index O(1) |
| **Lookup** | O(1) | O(n) | O(1) | O(n) |
| **Use case** | Key-value | Sequence | Unique | Constants |

### Common Operations

| Operation | dict | list | set |
|-----------|------|------|-----|
| Create | `{}` or `dict()` | `[]` or `list()` | `set()` |
| Add | `d[k] = v` | `l.append(x)` | `s.add(x)` |
| Remove | `del d[k]` | `l.remove(x)` | `s.remove(x)` |
| Check | `k in d` | `x in l` | `x in s` |
| Length | `len(d)` | `len(l)` | `len(s)` |

---

## 13. 7-Day Rapid Learning Roadmap

### สำหรับหัวข้อนี้โดยเฉพาะ (ใช้เวลา 2-3 ชั่วโมง)

| เวลา | กิจกรรม |
|------|---------|
| 0:00-0:20 | อ่าน Abstract และ Core Principles |
| 0:20-0:40 | ศึกษา Example 1-2 (dict) |
| 0:40-1:00 | ศึกษา Example 3-4 (list, set) |
| 1:00-1:20 | ศึกษา Example 5-6 (tuple, comprehensions) |
| 1:20-1:40 | ศึกษา Example 7 (type hints) |
| 1:40-2:00 | ลองเขียน code ด้วยตัวเอง |
| 2:00-2:30 | อ่าน Pitfalls และตอบ Diagnostic Questions |
| 2:30-3:00 | สรุปและทบทวน |

### เป้าหมายหลังจบ:
- [ ] เลือก data structure ที่เหมาะกับงานได้
- [ ] ใช้ dict operations ได้คล่อง
- [ ] เข้าใจ time complexity พื้นฐาน
- [ ] เขียน type hints ได้
