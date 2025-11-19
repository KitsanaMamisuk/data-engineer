# องค์ประกอบของ Business Rule

## 1. Abstract (60 วินาที)

Business Rule ประกอบด้วย 4 ส่วนหลัก: **Condition** (เงื่อนไข), **Action** (สิ่งที่ทำ), **Data** (ข้อมูลที่ใช้), และ **Metadata** (ข้อมูลเกี่ยวกับ rule) เขียนในรูปแบบ "ถ้า...แล้ว..." (IF-THEN) เช่น "ถ้าลูกค้ามีรายได้มากกว่า 30,000 บาท แล้วอนุมัติสินเชื่อ" การเข้าใจองค์ประกอบเหล่านี้ช่วยให้เขียน rule ได้ถูกต้อง ครบถ้วน และ maintain ง่าย

---

## 2. Core Principles (20/80)

หลัก 20% ที่ให้ผลลัพธ์ 80%:

1. **IF-THEN Structure** - ทุก rule มี condition และ action
2. **Clear Data Definition** - ต้องกำหนดชัดว่าใช้ data อะไร
3. **Single Responsibility** - 1 rule ทำ 1 เรื่อง
4. **Self-documenting** - rule ควรอ่านเข้าใจได้เอง
5. **Testable** - ต้อง test ได้ง่าย

---

## 3. Mental Models

### Model 1: "ประโยคเงื่อนไข"
- **Condition** = ประธาน + กริยา (ถ้าลูกค้า มีรายได้...)
- **Action** = ผลลัพธ์ (แล้ว อนุมัติ)

### Model 2: "Recipe (สูตรอาหาร)"
- **Ingredients** = Data (วัตถุดิบ)
- **Instructions** = Condition + Action (วิธีทำ)
- **Dish name** = Metadata (ชื่อเมนู)

### Model 3: "กฎจราจร"
- "ถ้าไฟแดง (condition) แล้วหยุด (action)"
- ใช้กับรถทุกคัน (data)
- กฎข้อที่ 1 (metadata)

---

## 4. Skill Tree

```
พื้นฐาน
├── รู้จัก 4 องค์ประกอบหลัก
├── เขียน IF-THEN statement ได้
└── ระบุ input/output ของ rule ได้

กลาง
├── เขียน compound conditions (AND/OR)
├── จัดการ multiple actions
├── สร้าง rule templates
└── เขียน metadata ที่ดี

สูง
├── ออกแบบ complex rule structures
├── Optimize rule performance
├── สร้าง reusable rule components
└── จัดการ rule dependencies
```

---

## 5. What NOT to Learn

- **อย่าเรียน** ทุก rule syntax → เรียนแค่ที่ใช้ในงาน
- **อย่าเรียน** advanced logic programming → เน้น practical
- **อย่าท่อง** ทุก operator → ดู reference เมื่อต้องใช้
- **อย่าเรียน** formal specification languages → เน้น readable
- **อย่าเรียน** mathematical proof of rules → ไม่จำเป็น

---

## 6. Real-world Usage

### สถานการณ์ที่ใช้บ่อย:

1. **เขียน rule ใหม่**
   - กำหนด condition ให้ครบ
   - ระบุ action ที่ชัดเจน
   - เตรียม test data

2. **Debug rule ที่มีปัญหา**
   - ตรวจ condition ว่าครบไหม
   - ดู data type ตรงไหม
   - check logic errors

3. **Review rule ของคนอื่น**
   - ดูว่า readable ไหม
   - check edge cases
   - verify metadata

---

## 7. Typical Pitfalls (10 ข้อ)

| # | ความผิดพลาด | วิธีเลี่ยง |
|---|-------------|-----------|
| 1 | Condition ไม่ครบ | List ทุก condition ก่อนเขียน |
| 2 | Action กำกวม | ระบุ output ที่ชัดเจน |
| 3 | Data type ไม่ตรง | Validate data ก่อน process |
| 4 | ไม่มี default case | เตรียม else clause เสมอ |
| 5 | Rule ซับซ้อนเกินไป | แยกเป็น rule ย่อยๆ |
| 6 | ไม่มี description | เขียน metadata ทุกครั้ง |
| 7 | Hard-code values | ใช้ parameters/constants |
| 8 | ลืม null handling | Check null/empty ก่อน |
| 9 | Logic errors (AND/OR) | Test ทุก combination |
| 10 | ไม่ test edge cases | เตรียม boundary test cases |

---

## 8. Playbook (Step-by-step)

### เขียน Business Rule:

1. **เข้าใจ requirement**
   - อ่าน business requirement
   - ถามให้ชัดเจนถ้าไม่เข้าใจ

2. **ระบุ Data**
   - list input ทั้งหมด
   - กำหนด data type
   - ระบุ source ของ data

3. **เขียน Condition**
   - เขียนเป็นภาษาธรรมชาติก่อน
   - แปลงเป็น logical expression
   - ตรวจสอบ AND/OR logic

4. **กำหนด Action**
   - ระบุ output
   - กำหนด side effects (ถ้ามี)
   - เตรียม error handling

5. **เพิ่ม Metadata**
   - ตั้งชื่อที่สื่อความหมาย
   - เขียน description
   - กำหนด version

6. **Test**
   - Test positive cases
   - Test negative cases
   - Test edge cases

---

## 9. Fast Practical Examples

### Example 1: โครงสร้างพื้นฐานของ Rule

```python
rule = {
    # Metadata - ข้อมูลเกี่ยวกับ rule
    "metadata": {
        "id": "LOAN_APPROVAL_001",
        "name": "Basic Loan Approval",
        "description": "ตรวจสอบคุณสมบัติเบื้องต้นสำหรับสินเชื่อส่วนบุคคล",
        "version": "1.0",
        "created_by": "john_ba",
        "created_date": "2024-01-15",
        "category": "LOAN"
    },

    # Data - ข้อมูลที่ใช้
    "data": {
        "inputs": [
            {"name": "salary", "type": "number", "required": True},
            {"name": "age", "type": "number", "required": True},
            {"name": "debt_ratio", "type": "number", "required": True}
        ],
        "outputs": [
            {"name": "decision", "type": "string"},
            {"name": "reason", "type": "string"}
        ]
    },

    # Condition - เงื่อนไข
    "condition": {
        "all": [  # AND
            {"field": "salary", "operator": ">=", "value": 30000},
            {"field": "age", "operator": ">=", "value": 20},
            {"field": "age", "operator": "<=", "value": 60},
            {"field": "debt_ratio", "operator": "<=", "value": 0.5}
        ]
    },

    # Action - สิ่งที่ทำ
    "action": {
        "then": {
            "decision": "APPROVED",
            "reason": "ผ่านเกณฑ์ทั้งหมด"
        },
        "else": {
            "decision": "REJECTED",
            "reason": "ไม่ผ่านเกณฑ์"
        }
    }
}
```

### Example 2: Condition Types

```python
# Simple condition
condition_simple = {
    "field": "salary",
    "operator": ">=",
    "value": 30000
}

# Compound condition (AND)
condition_and = {
    "all": [
        {"field": "salary", "operator": ">=", "value": 30000},
        {"field": "age", "operator": ">=", "value": 20}
    ]
}

# Compound condition (OR)
condition_or = {
    "any": [
        {"field": "customer_type", "operator": "==", "value": "GOLD"},
        {"field": "salary", "operator": ">=", "value": 100000}
    ]
}

# Nested condition
condition_nested = {
    "all": [
        {"field": "age", "operator": ">=", "value": 20},
        {
            "any": [
                {"field": "salary", "operator": ">=", "value": 50000},
                {"field": "guarantor", "operator": "==", "value": True}
            ]
        }
    ]
}
```

### Example 3: Action Types

```python
# Simple action - return value
action_simple = {
    "return": {
        "decision": "APPROVED",
        "amount": 100000
    }
}

# Conditional action
action_conditional = {
    "then": {
        "decision": "APPROVED",
        "rate": 5.5
    },
    "else": {
        "decision": "REJECTED",
        "rate": None
    }
}

# Multiple actions
action_multiple = {
    "actions": [
        {"set": {"status": "APPROVED"}},
        {"calculate": {"amount": "salary * 15"}},
        {"notify": {"channel": "email", "template": "approval"}}
    ]
}

# Branching actions
action_branching = {
    "switch": {
        "field": "credit_grade",
        "cases": {
            "A": {"rate": 5.0, "max_amount": 2000000},
            "B": {"rate": 6.0, "max_amount": 1000000},
            "C": {"rate": 7.0, "max_amount": 500000},
            "default": {"rate": 8.0, "max_amount": 200000}
        }
    }
}
```

### Example 4: Complete Rule in Python

```python
class LoanApprovalRule:
    """
    Business Rule สำหรับอนุมัติสินเชื่อ
    """

    # Metadata
    RULE_ID = "LOAN_APPROVAL_001"
    RULE_NAME = "Basic Loan Approval"
    VERSION = "1.0"

    # Parameters (configurable)
    MIN_SALARY = 30000
    MIN_AGE = 20
    MAX_AGE = 60
    MAX_DEBT_RATIO = 0.5

    def evaluate(self, applicant):
        """
        Evaluate the rule against applicant data

        Data (Input):
        - salary: number
        - age: number
        - debt_ratio: number

        Action (Output):
        - decision: APPROVED/REJECTED
        - reason: string
        """

        # Condition 1: Check salary
        if applicant.salary < self.MIN_SALARY:
            return {
                "decision": "REJECTED",
                "reason": f"รายได้ต่ำกว่า {self.MIN_SALARY:,} บาท"
            }

        # Condition 2: Check age
        if applicant.age < self.MIN_AGE:
            return {
                "decision": "REJECTED",
                "reason": f"อายุต่ำกว่า {self.MIN_AGE} ปี"
            }

        if applicant.age > self.MAX_AGE:
            return {
                "decision": "REJECTED",
                "reason": f"อายุมากกว่า {self.MAX_AGE} ปี"
            }

        # Condition 3: Check debt ratio
        if applicant.debt_ratio > self.MAX_DEBT_RATIO:
            return {
                "decision": "REJECTED",
                "reason": f"อัตราส่วนหนี้ต่อรายได้เกิน {self.MAX_DEBT_RATIO*100}%"
            }

        # All conditions passed
        return {
            "decision": "APPROVED",
            "reason": "ผ่านเกณฑ์ทั้งหมด"
        }
```

### Example 5: Metadata Best Practices

```python
metadata_good = {
    "id": "KYC_DOC_VERIFY_001",
    "name": "KYC Document Verification",
    "description": "ตรวจสอบความถูกต้องและครบถ้วนของเอกสาร KYC สำหรับลูกค้าบุคคลธรรมดา",
    "version": "2.1",
    "category": "KYC",
    "tags": ["document", "verification", "retail"],

    # Audit info
    "created_by": "jane_ba",
    "created_date": "2024-01-10",
    "modified_by": "john_ba",
    "modified_date": "2024-03-15",
    "approved_by": "manager_lisa",
    "approved_date": "2024-03-16",

    # Lifecycle info
    "status": "ACTIVE",
    "effective_date": "2024-03-20",
    "expiry_date": None,

    # Traceability
    "requirement_id": "REQ-2024-0123",
    "change_request": "CR-2024-0456"
}
```

### Example 6: Data Definition

```python
data_definition = {
    "inputs": [
        {
            "name": "customer",
            "type": "object",
            "required": True,
            "properties": {
                "id": {"type": "string", "required": True},
                "name": {"type": "string", "required": True},
                "age": {"type": "integer", "required": True, "min": 0, "max": 120},
                "salary": {"type": "number", "required": True, "min": 0},
                "customer_type": {
                    "type": "string",
                    "required": True,
                    "enum": ["GOLD", "SILVER", "NORMAL"]
                }
            }
        },
        {
            "name": "loan_amount",
            "type": "number",
            "required": True,
            "min": 10000,
            "max": 10000000
        }
    ],

    "outputs": [
        {
            "name": "decision",
            "type": "string",
            "enum": ["APPROVED", "REJECTED", "REVIEW_REQUIRED"]
        },
        {
            "name": "approved_amount",
            "type": "number",
            "nullable": True
        },
        {
            "name": "interest_rate",
            "type": "number",
            "nullable": True
        },
        {
            "name": "reasons",
            "type": "array",
            "items": {"type": "string"}
        }
    ]
}
```

---

## 10. Diagnostic Questions

ตอบคำถามเหล่านี้เพื่อทดสอบความเข้าใจ:

1. Business Rule มีกี่องค์ประกอบหลัก อะไรบ้าง?
2. Condition กับ Action ต่างกันอย่างไร?
3. Metadata สำคัญอย่างไร?
4. Compound condition คืออะไร?
5. เมื่อไหร่ควรใช้ AND เมื่อไหร่ควรใช้ OR?
6. Default case สำคัญอย่างไร?
7. Data type validation ทำทำไม?
8. Rule ควรมี description อย่างไร?
9. ถ้า rule ซับซ้อนเกินไป ควรทำอย่างไร?
10. Version control ของ rule สำคัญอย่างไร?

---

## 11. Explainer for 5-year-old

ลองนึกถึง **กฎของเกม** นะ

**เกม:** ถ้าทอยลูกเต๋าได้ 6 แล้วเดินได้ 2 รอบ

- **ถ้า** (Condition) = ทอยได้ 6
- **แล้ว** (Action) = เดินได้ 2 รอบ
- **ลูกเต๋า** (Data) = ตัวเลขที่ทอยได้
- **กฎข้อที่ 1** (Metadata) = บอกว่าเป็นกฎอะไร

ทุกกฎต้องบอกว่า:
1. **ถ้าอะไร** - ต้องเกิดอะไรก่อน
2. **แล้วทำอะไร** - ถ้าเกิดแล้วทำอะไรต่อ
3. **ใช้อะไร** - ดูจากอะไร
4. **กฎอะไร** - เรียกว่าอะไร

---

## 12. Comparison Table

| องค์ประกอบ | คำอธิบาย | ตัวอย่าง |
|------------|---------|---------|
| **Condition** | เงื่อนไขที่ต้อง check | salary >= 30000 |
| **Action** | สิ่งที่ทำเมื่อ condition เป็นจริง | return "APPROVED" |
| **Data** | ข้อมูลที่ใช้ใน rule | salary, age, debt_ratio |
| **Metadata** | ข้อมูลเกี่ยวกับ rule | id, name, version |

### Operators ที่ใช้บ่อย

| Operator | ความหมาย | ตัวอย่าง |
|----------|---------|---------|
| == | เท่ากับ | status == "ACTIVE" |
| != | ไม่เท่ากับ | type != "BLOCKED" |
| > | มากกว่า | age > 20 |
| >= | มากกว่าหรือเท่ากับ | salary >= 30000 |
| < | น้อยกว่า | debt_ratio < 0.5 |
| <= | น้อยกว่าหรือเท่ากับ | age <= 60 |
| in | อยู่ใน list | type in ["A", "B"] |
| not in | ไม่อยู่ใน list | status not in ["BLOCKED"] |
| AND | และ | age >= 20 AND salary >= 30000 |
| OR | หรือ | type == "GOLD" OR salary >= 100000 |

---

## 13. 7-Day Rapid Learning Roadmap

### สำหรับหัวข้อนี้โดยเฉพาะ (ใช้เวลา 1.5-2 ชั่วโมง)

| เวลา | กิจกรรม |
|------|---------|
| 0:00-0:20 | อ่าน Abstract และ Core Principles |
| 0:20-0:40 | ศึกษา Example 1-3 (โครงสร้างหลัก) |
| 0:40-1:00 | ศึกษา Example 4-6 (Complete rule) |
| 1:00-1:20 | ลองเขียน rule ของตัวเองตาม template |
| 1:20-1:40 | อ่าน Pitfalls และ Comparison Table |
| 1:40-2:00 | ตอบ Diagnostic Questions |

### เป้าหมายหลังจบ:
- [ ] บอก 4 องค์ประกอบของ rule ได้
- [ ] เขียน IF-THEN statement ได้ถูกต้อง
- [ ] ใช้ AND/OR ได้ถูกต้อง
- [ ] เขียน metadata ที่ดีได้
