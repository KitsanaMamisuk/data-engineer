# การแปลง Business Requirement เป็น Rule Logic

## 1. Abstract (60 วินาที)

การแปลง Business Requirement เป็น Rule Logic คือทักษะสำคัญที่สุดของ BRMS Developer เริ่มจากการรับ requirement ที่เขียนเป็นภาษาธรรมชาติ เช่น "ลูกค้าที่มีเงินเดือนมากกว่า 30,000 และอายุงานมากกว่า 6 เดือน สามารถยื่นกู้ได้" แล้วแปลงเป็น logic ที่คอมพิวเตอร์เข้าใจ เช่น `IF salary > 30000 AND work_months > 6 THEN eligible = true`

---

## 2. Core Principles (20/80)

หลัก 20% ที่ให้ผลลัพธ์ 80%:

1. **Clarify Ambiguity** - ถามให้ชัดเจนก่อนเขียน
2. **Identify Keywords** - หา "ถ้า", "และ", "หรือ", "ต้อง"
3. **Break Down** - แยกเป็นเงื่อนไขย่อยๆ
4. **Define Boundaries** - กำหนด edge cases
5. **Validate with BA** - ให้ BA ตรวจสอบ logic

---

## 3. Mental Models

### Model 1: "แปลภาษา"
- Requirement = ภาษาไทย
- Rule Logic = ภาษาอังกฤษ
- คุณคือล่าม ต้องแปลให้ถูกต้องครบถ้วน

### Model 2: "สูตรคณิตศาสตร์"
- "ผลรวมของ A กับ B มากกว่า C"
- → `A + B > C`

### Model 3: "กฎหมาย → ข้อบังคับ"
- กฎหมายเขียนเป็นภาษาคน
- ต้องแปลเป็นขั้นตอนปฏิบัติที่ชัดเจน

---

## 4. Skill Tree

```
พื้นฐาน
├── อ่าน requirement แล้ว identify keywords ได้
├── แปลง simple requirement เป็น IF-THEN
└── รู้จักถามคำถามที่ถูกต้อง

กลาง
├── จัดการ complex requirements
├── หา hidden requirements
├── สร้าง test cases จาก requirements
└── Document assumptions

สูง
├── ออกแบบ rule architecture จาก requirements
├── Negotiate requirements กับ BA
├── Identify conflicting requirements
└── Optimize rule structure
```

---

## 5. What NOT to Learn

- **อย่าเรียน** formal specification languages → ใช้ practical approach
- **อย่าเรียน** requirement engineering theory → เน้นการแปลง
- **อย่าเรียน** UML diagrams ทุกประเภท → แค่ที่ช่วยสื่อสาร
- **อย่าเรียน** business domain ลึกเกินไป → เข้าใจพอที่จะแปลงได้
- **อย่าท่อง** ทุก pattern → ฝึกจาก real requirements

---

## 6. Real-world Usage

### สถานการณ์ที่เจอบ่อย:

1. **Requirement ไม่ชัด**
   - "ลูกค้าที่ดี" → ต้องถามว่าหมายถึงอะไร
   - "จำนวนมาก" → มากคือเท่าไหร่

2. **Requirement ขัดแย้ง**
   - กฎ A บอก approve
   - กฎ B บอก reject
   - ต้องถามว่าใช้อันไหน

3. **Requirement ไม่ครบ**
   - ไม่บอก edge cases
   - ไม่บอก default behavior

---

## 7. Typical Pitfalls (10 ข้อ)

| # | ความผิดพลาด | วิธีเลี่ยง |
|---|-------------|-----------|
| 1 | Assume โดยไม่ถาม | ถามทุกครั้งที่ไม่ชัด |
| 2 | ลืม edge cases | ถามเรื่อง boundaries |
| 3 | แปลผิดเพราะ ambiguity | Clarify ก่อนเขียน |
| 4 | ไม่เข้าใจ business context | ถามเรื่อง "ทำไม" |
| 5 | Logic ซับซ้อนเกินไป | Break down เป็น rules ย่อย |
| 6 | AND/OR สับสน | เขียน truth table |
| 7 | ลืม default case | ถามว่าถ้าไม่เข้าเงื่อนไขทำอะไร |
| 8 | ไม่ validate กับ BA | ให้ BA review ทุกครั้ง |
| 9 | Data type ไม่ตรง | Confirm data type กับ source |
| 10 | ไม่ document assumptions | เขียน assumptions ทุกข้อ |

---

## 8. Playbook (Step-by-step)

### กระบวนการแปลง Requirement:

#### Step 1: รับ Requirement
```
Requirement: "ลูกค้าที่มีเงินเดือนตั้งแต่ 30,000 บาทขึ้นไป
และมีอายุงานมากกว่า 6 เดือน สามารถยื่นกู้สินเชื่อส่วนบุคคลได้"
```

#### Step 2: Identify Keywords
```
- "ตั้งแต่...ขึ้นไป" → >=
- "และ" → AND
- "มากกว่า" → >
- "สามารถ...ได้" → eligible = true
```

#### Step 3: Extract Conditions
```
Condition 1: เงินเดือน >= 30,000
Condition 2: อายุงาน > 6 เดือน
```

#### Step 4: Ask Clarifying Questions
```
Q1: เงินเดือน 30,000 พอดี ผ่านไหม? → ได้ (>=)
Q2: อายุงาน 6 เดือนพอดี ผ่านไหม? → ไม่ได้ (>)
Q3: ถ้าไม่ผ่านแล้วทำอะไร? → eligible = false
Q4: เงินเดือนนี้รวม OT ไหม? → ไม่รวม (base salary เท่านั้น)
```

#### Step 5: Write Rule Logic
```python
def check_eligibility(customer):
    if customer.salary >= 30000 and customer.work_months > 6:
        return {"eligible": True, "reason": "ผ่านเกณฑ์"}
    else:
        return {"eligible": False, "reason": "ไม่ผ่านเกณฑ์"}
```

#### Step 6: Create Test Cases
```python
test_cases = [
    {"salary": 30000, "work_months": 7, "expected": True},   # Boundary pass
    {"salary": 29999, "work_months": 7, "expected": False},  # Fail salary
    {"salary": 30000, "work_months": 6, "expected": False},  # Fail months
    {"salary": 50000, "work_months": 12, "expected": True},  # Clear pass
]
```

#### Step 7: Validate with BA
- แสดง logic และ test cases ให้ BA
- ให้ BA confirm ว่าถูกต้อง

---

## 9. Fast Practical Examples

### Example 1: Simple Requirement

**Requirement:**
"ลูกค้าที่มีอายุ 20-60 ปี สามารถสมัครบัตรเครดิตได้"

**Clarifying Questions:**
- อายุ 20 พอดี ได้ไหม? → ได้
- อายุ 60 พอดี ได้ไหม? → ได้
- ถ้าอายุไม่ผ่าน return อะไร? → "ไม่ผ่านเกณฑ์อายุ"

**Rule Logic:**
```python
def check_credit_card_eligibility(age):
    if 20 <= age <= 60:
        return {"eligible": True}
    elif age < 20:
        return {"eligible": False, "reason": "อายุต่ำกว่า 20 ปี"}
    else:
        return {"eligible": False, "reason": "อายุมากกว่า 60 ปี"}
```

### Example 2: OR Condition

**Requirement:**
"ลูกค้าที่เป็น Gold Member หรือมียอดเงินฝากมากกว่า 1 ล้านบาท
จะได้รับดอกเบี้ยพิเศษ"

**Identify Keywords:**
- "หรือ" → OR

**Rule Logic:**
```python
def check_special_rate_eligibility(customer):
    if customer.member_type == "GOLD" or customer.deposit > 1000000:
        return {"special_rate": True}
    return {"special_rate": False}
```

### Example 3: Nested Conditions

**Requirement:**
"สินเชื่อจะได้รับอนุมัติถ้า:
1. Credit score มากกว่าหรือเท่ากับ 700 หรือ
2. Credit score มากกว่าหรือเท่ากับ 600 และมีผู้ค้ำประกัน"

**Analysis:**
- เงื่อนไข 1: score >= 700
- เงื่อนไข 2: score >= 600 AND has_guarantor
- รวม: (score >= 700) OR (score >= 600 AND has_guarantor)

**Rule Logic:**
```python
def check_loan_approval(applicant):
    # Condition 1: High credit score
    if applicant.credit_score >= 700:
        return {"approved": True, "reason": "Credit score สูง"}

    # Condition 2: Medium score with guarantor
    if applicant.credit_score >= 600 and applicant.has_guarantor:
        return {"approved": True, "reason": "มีผู้ค้ำประกัน"}

    return {"approved": False, "reason": "ไม่ผ่านเกณฑ์"}
```

### Example 4: Decision Table

**Requirement:**
"อัตราดอกเบี้ยกำหนดตาม Credit Grade และประเภทลูกค้า:
- Grade A + Gold = 4.5%
- Grade A + Silver = 5.0%
- Grade A + Normal = 5.5%
- Grade B + Gold = 5.5%
- Grade B + Silver = 6.0%
- Grade B + Normal = 6.5%
- อื่นๆ = 7.0%"

**Rule Logic (Decision Table):**
```python
def get_interest_rate(credit_grade, customer_type):
    rate_table = {
        ("A", "GOLD"): 4.5,
        ("A", "SILVER"): 5.0,
        ("A", "NORMAL"): 5.5,
        ("B", "GOLD"): 5.5,
        ("B", "SILVER"): 6.0,
        ("B", "NORMAL"): 6.5,
    }

    return rate_table.get((credit_grade, customer_type), 7.0)
```

### Example 5: Range-based Rules

**Requirement:**
"วงเงินสินเชื่อกำหนดตามรายได้:
- รายได้ 15,000 - 29,999: วงเงิน 50,000
- รายได้ 30,000 - 49,999: วงเงิน 150,000
- รายได้ 50,000 - 99,999: วงเงิน 300,000
- รายได้ 100,000 ขึ้นไป: วงเงิน 500,000"

**Clarifying Questions:**
- รายได้ต่ำกว่า 15,000 ได้ไหม? → ไม่ได้
- ขอบเขตรวม upper bound ไหม? → ไม่รวม (< 30,000)

**Rule Logic:**
```python
def calculate_credit_limit(monthly_income):
    if monthly_income < 15000:
        return {"limit": 0, "reason": "รายได้ไม่ผ่านเกณฑ์ขั้นต่ำ"}

    if monthly_income < 30000:
        return {"limit": 50000}
    elif monthly_income < 50000:
        return {"limit": 150000}
    elif monthly_income < 100000:
        return {"limit": 300000}
    else:
        return {"limit": 500000}
```

### Example 6: Complex Business Requirement

**Requirement:**
"การอนุมัติสินเชื่อบ้าน:
- ต้องมีอายุ 20-65 ปี
- รายได้ขั้นต่ำ 30,000 บาท
- อัตราส่วนหนี้ต่อรายได้ไม่เกิน 60%
- ต้องมีเอกสารครบ: บัตรประชาชน, ทะเบียนบ้าน, สลิปเงินเดือน
- ถ้าผ่านทุกข้อ → อนุมัติ
- ถ้าไม่ผ่านข้อใดข้อหนึ่ง → ปฏิเสธพร้อมระบุเหตุผล"

**Rule Logic:**
```python
def check_home_loan_eligibility(applicant):
    errors = []

    # Check age
    if applicant.age < 20:
        errors.append("อายุต่ำกว่า 20 ปี")
    elif applicant.age > 65:
        errors.append("อายุเกิน 65 ปี")

    # Check income
    if applicant.salary < 30000:
        errors.append("รายได้ต่ำกว่า 30,000 บาท")

    # Check debt ratio
    debt_ratio = applicant.monthly_debt / applicant.salary
    if debt_ratio > 0.6:
        errors.append(f"อัตราส่วนหนี้ต่อรายได้ ({debt_ratio*100:.0f}%) เกิน 60%")

    # Check documents
    required_docs = ["ID_CARD", "HOUSE_REG", "SALARY_SLIP"]
    missing_docs = [doc for doc in required_docs
                   if doc not in applicant.documents]
    if missing_docs:
        errors.append(f"เอกสารไม่ครบ: {', '.join(missing_docs)}")

    # Decision
    if errors:
        return {
            "approved": False,
            "reasons": errors
        }
    else:
        return {
            "approved": True,
            "reason": "ผ่านเกณฑ์ทั้งหมด"
        }
```

### Example 7: Handling Ambiguous Requirement

**Original Requirement:**
"ลูกค้าที่ดีจะได้ส่วนลด"

**Clarifying Questions:**
1. "ลูกค้าที่ดี" คืออะไร?
   - อยู่กับธนาคารมากกว่า 1 ปี
   - ไม่เคยผิดนัดชำระ
   - มีผลิตภัณฑ์มากกว่า 2 ชนิด

2. ส่วนลดเท่าไหร่?
   - 10% จากค่าธรรมเนียม

**Refined Requirement:**
"ลูกค้าที่อยู่กับธนาคารมากกว่า 1 ปี และไม่เคยผิดนัดชำระ
และมีผลิตภัณฑ์มากกว่า 2 ชนิด จะได้รับส่วนลด 10% จากค่าธรรมเนียม"

**Rule Logic:**
```python
def check_discount_eligibility(customer):
    is_good_customer = (
        customer.tenure_months > 12 and
        customer.missed_payments == 0 and
        customer.num_products > 2
    )

    if is_good_customer:
        return {"discount_percent": 10}
    return {"discount_percent": 0}
```

---

## 10. Diagnostic Questions

ตอบคำถามเหล่านี้เพื่อทดสอบความเข้าใจ:

1. ทำไมต้องถามคำถาม clarify ก่อนเขียน rule?
2. Keywords อะไรบ้างที่ต้องระวังเป็นพิเศษ?
3. Edge case คืออะไร?
4. ถ้า requirement ขัดแย้งกัน ทำอย่างไร?
5. AND กับ OR ต่างกันอย่างไรใน logic?
6. ทำไมต้อง document assumptions?
7. Test case สร้างจากอะไร?
8. Decision table ใช้เมื่อไหร่?
9. Range-based rule ต้องระวังอะไร?
10. ใครควรเป็นคน validate rule logic?

---

## 11. Explainer for 5-year-old

ลองนึกถึง **การเล่นเกมกับเพื่อน** นะ

เพื่อนบอกว่า: "ถ้าทอยได้มากกว่า 4 จะชนะ"

เราต้องถามว่า:
- "ถ้าได้ 4 พอดี ชนะไหม?" → ไม่ (มากกว่า 4)
- "ถ้าได้ 5 ล่ะ?" → ชนะ (5 > 4)
- "ถ้าได้ 0 ล่ะ?" → แพ้

แล้วเราก็เขียนกฎว่า:
```
ถ้า ตัวเลข > 4:
    ชนะ
ไม่งั้น:
    แพ้
```

นี่แหละคือการ "แปลงกติกาเป็นกฎ"!

---

## 12. Comparison Table

### Keywords และการแปลง

| Keyword (ภาษาคน) | Logic Operator |
|------------------|----------------|
| และ, กับ, พร้อมกับ | AND |
| หรือ, ไม่ก็ | OR |
| ไม่, ไม่ใช่ | NOT |
| มากกว่า | > |
| น้อยกว่า | < |
| มากกว่าหรือเท่ากับ, ตั้งแต่...ขึ้นไป | >= |
| น้อยกว่าหรือเท่ากับ, ไม่เกิน | <= |
| เท่ากับ, คือ | == |
| ไม่เท่ากับ | != |
| อยู่ระหว่าง | >= AND <= |
| อย่างใดอย่างหนึ่ง | IN |

### Common Ambiguities

| Ambiguous | ต้องถามว่า |
|-----------|-----------|
| "มาก" | มากกว่าเท่าไหร่? |
| "น้อย" | น้อยกว่าเท่าไหร่? |
| "ดี" | เกณฑ์คืออะไร? |
| "เร็วๆ นี้" | ภายในกี่วัน? |
| "บางครั้ง" | frequency เท่าไหร่? |
| "อาจจะ" | เงื่อนไขคืออะไร? |

---

## 13. 7-Day Rapid Learning Roadmap

### สำหรับหัวข้อนี้โดยเฉพาะ (ใช้เวลา 2-3 ชั่วโมง)

| เวลา | กิจกรรม |
|------|---------|
| 0:00-0:20 | อ่าน Abstract และ Core Principles |
| 0:20-0:40 | ศึกษา Playbook (7 steps) |
| 0:40-1:00 | ศึกษา Example 1-3 (Simple to Nested) |
| 1:00-1:30 | ศึกษา Example 4-6 (Decision Table, Range, Complex) |
| 1:30-2:00 | ศึกษา Example 7 (Ambiguous) |
| 2:00-2:30 | ฝึกแปลง 3 requirements ด้วยตัวเอง |
| 2:30-3:00 | อ่าน Pitfalls และตอบ Diagnostic Questions |

### เป้าหมายหลังจบ:
- [ ] แปลง simple requirement เป็น rule ได้ถูกต้อง
- [ ] รู้จักถามคำถามที่ถูกต้อง
- [ ] Handle AND/OR/NOT ได้
- [ ] สร้าง test cases จาก requirement ได้
