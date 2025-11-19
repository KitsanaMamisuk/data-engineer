# ความแตกต่างระหว่าง Hard-coded Logic กับ BRMS

## 1. Abstract (60 วินาที)

Hard-coded logic คือการเขียน business rules ลงไปใน source code โดยตรง เช่น `if salary > 50000: approve()` ส่วน BRMS คือการแยก rules ออกมาอยู่ภายนอก ให้แก้ไขได้โดยไม่ต้องแก้ code ข้อแตกต่างหลักคือ: hard-coded เปลี่ยนยาก ต้องรอ developer แต่ BRMS เปลี่ยนได้เร็ว BA ก็แก้ได้ เลือกใช้ให้ถูกสถานการณ์คือหัวใจสำคัญ

---

## 2. Core Principles (20/80)

หลัก 20% ที่ให้ผลลัพธ์ 80%:

1. **Flexibility vs Simplicity** - BRMS ยืดหยุ่นกว่าแต่ซับซ้อนกว่า
2. **Change Frequency** - ถ้าเปลี่ยนบ่อย → BRMS, เปลี่ยนน้อย → Hard-code
3. **Ownership** - Hard-code = IT owns, BRMS = Business owns
4. **Deployment Speed** - Hard-code ช้า, BRMS เร็ว
5. **Right Tool for Right Job** - ไม่มีอันไหนดีกว่าเสมอไป

---

## 3. Mental Models

### Model 1: "พิมพ์ vs เขียน"
- **Hard-coded** = พิมพ์หนังสือ (แก้ต้องพิมพ์ใหม่ทั้งเล่ม)
- **BRMS** = เขียนกระดาน whiteboard (ลบแก้ได้เลย)

### Model 2: "บ้านปูน vs บ้านน็อคดาวน์"
- **Hard-coded** = บ้านปูน (แข็งแรง แต่ต่อเติมยาก)
- **BRMS** = บ้านน็อคดาวน์ (ย้าย เปลี่ยน ต่อเติมง่าย)

### Model 3: "รอยสัก vs เสื้อผ้า"
- **Hard-coded** = รอยสัก (ถาวร ลบยาก)
- **BRMS** = เสื้อผ้า (เปลี่ยนได้ทุกวัน)

---

## 4. Skill Tree

```
พื้นฐาน
├── แยกแยะได้ว่าอะไรเป็น hard-coded อะไรเป็น BRMS
├── เข้าใจข้อดี-ข้อเสียของแต่ละแบบ
└── รู้ว่าเมื่อไหร่ควรใช้แบบไหน

กลาง
├── วิเคราะห์ existing code ว่าควร migrate ไป BRMS ไหม
├── ออกแบบ architecture ที่ผสม hard-coded + BRMS
├── Refactor hard-coded logic ไปเป็น BRMS
└── เขียน code ที่ integrate กับ BRMS

สูง
├── กำหนด strategy ว่าอะไรควรอยู่ที่ไหน
├── Optimize performance ของทั้งสองแบบ
├── สร้าง hybrid architecture
└── บริหาร technical debt
```

---

## 5. What NOT to Learn

- **อย่าเรียน** ทุก refactoring technique → เรียนแค่ที่เกี่ยวกับ rules
- **อย่าเรียน** การ migrate legacy system ทั้งหมด → เน้นแค่ rules migration
- **อย่าท่อง** ทุก design pattern → เรียนแค่ที่ใช้กับ rule engine
- **อย่าเรียน** architecture ขั้นสูง → เข้าใจแค่พื้นฐาน
- **อย่าเรียน** performance tuning ลึก → รู้พื้นฐานพอ

---

## 6. Real-world Usage

### สถานการณ์ที่ควรใช้ Hard-coded:

1. **Core system logic** - ไม่เปลี่ยนเลย (เช่น คำนวณ checksum)
2. **Security validation** - ต้องการความเร็วสูงสุด
3. **Simple, stable rules** - เช่น validation format

### สถานการณ์ที่ควรใช้ BRMS:

1. **Business policies** - เปลี่ยนตาม regulation
2. **Pricing/Discount** - เปลี่ยนตามการตลาด
3. **Approval workflows** - เปลี่ยนตามองค์กร
4. **Risk scoring** - ปรับตาม model ใหม่

---

## 7. Typical Pitfalls (10 ข้อ)

| # | ความผิดพลาด | วิธีเลี่ยง |
|---|-------------|-----------|
| 1 | Hard-code ทุกอย่าง | วิเคราะห์ change frequency ก่อน |
| 2 | ใช้ BRMS กับทุกอย่าง | บาง logic ควรอยู่ใน code |
| 3 | ไม่คิดเรื่อง performance | Test performance ทั้งสองแบบ |
| 4 | ผสมกันไม่ชัดเจน | กำหนด boundary ชัดเจน |
| 5 | Migrate ทุกอย่างทีเดียว | ค่อยๆ migrate ทีละส่วน |
| 6 | ไม่ train business users | ต้อง train ให้ใช้ BRMS เป็น |
| 7 | Hard-code magic numbers | ใช้ constants หรือ config |
| 8 | ไม่มี documentation | Document ว่าอะไรอยู่ที่ไหน |
| 9 | ไม่คิดเรื่อง testing | Test ทั้ง hard-coded และ BRMS |
| 10 | ลืม version control BRMS | ใช้ versioning เหมือน code |

---

## 8. Playbook (Step-by-step)

### ตัดสินใจว่าควรใช้แบบไหน:

1. **ถามคำถามเหล่านี้:**
   - กฎนี้เปลี่ยนบ่อยแค่ไหน? (เดือนละครั้ง/ปีละครั้ง)
   - ใครควรเป็นคนเปลี่ยน? (IT/Business)
   - ต้องการ traceability ไหม?
   - Performance สำคัญแค่ไหน?

2. **Decision Matrix:**
   - เปลี่ยนบ่อย + Business เปลี่ยน → **BRMS**
   - เปลี่ยนน้อย + IT เปลี่ยน → **Hard-code**
   - Performance critical + ง่าย → **Hard-code**
   - ต้อง audit + compliance → **BRMS**

3. **ถ้าต้อง migrate:**
   - ระบุ rules ที่ต้อง migrate
   - จัดลำดับความสำคัญ
   - Migrate ทีละ rule
   - Test regression ทุกครั้ง

---

## 9. Fast Practical Examples

### Example 1: Hard-coded (ไม่ดี)

```python
# Business logic ฝังใน code
def check_loan_eligibility(customer):
    # Magic numbers ฝังใน code
    if customer.salary < 30000:
        return False
    if customer.age < 20 or customer.age > 60:
        return False
    if customer.debt_ratio > 0.5:
        return False
    return True

# ปัญหา: ถ้าต้องเปลี่ยน 30000 เป็น 35000 ต้องแก้ code
```

### Example 2: Hard-coded (ดีขึ้น)

```python
# แยก constants ออกมา
LOAN_CONFIG = {
    "min_salary": 30000,
    "min_age": 20,
    "max_age": 60,
    "max_debt_ratio": 0.5
}

def check_loan_eligibility(customer):
    if customer.salary < LOAN_CONFIG["min_salary"]:
        return False
    # ... etc

# ดีขึ้น: แก้ config ได้ แต่ยังต้อง deploy ใหม่
```

### Example 3: BRMS Approach

```python
# Rule ถูกเก็บใน database/external system
# Python code แค่เรียกใช้
from brms_client import RuleEngine

def check_loan_eligibility(customer):
    engine = RuleEngine()
    result = engine.evaluate(
        rule_set="LOAN_ELIGIBILITY",
        data=customer.to_dict()
    )
    return result.decision

# ดีที่สุด: เปลี่ยน rule ได้โดยไม่ต้องแก้ code
```

### Example 4: การเปรียบเทียบ workflow

```python
# === Hard-coded workflow ===
# 1. BA ส่ง requirement
# 2. Developer แก้ code
# 3. Code review
# 4. QA test
# 5. Deploy to staging
# 6. UAT
# 7. Deploy to production
# รวม: 3-5 วัน

# === BRMS workflow ===
# 1. BA เปิด rule editor
# 2. แก้ไข rule
# 3. Test ใน editor
# 4. Submit for review
# 5. Approve และ activate
# รวม: 2-4 ชั่วโมง
```

### Example 5: Hybrid Approach

```python
# ผสมกัน: Hard-code สิ่งที่ไม่เปลี่ยน, BRMS สิ่งที่เปลี่ยนบ่อย

def process_loan_application(application):
    # Hard-coded: Basic validation (ไม่เปลี่ยน)
    if not is_valid_id(application.customer_id):
        return {"status": "ERROR", "message": "Invalid ID"}

    # Hard-coded: Security check (ไม่เปลี่ยน)
    if is_blacklisted(application.customer_id):
        return {"status": "REJECTED", "reason": "BLACKLISTED"}

    # BRMS: Eligibility rules (เปลี่ยนบ่อย)
    eligibility = rule_engine.evaluate("ELIGIBILITY", application)
    if not eligibility.passed:
        return {"status": "REJECTED", "reason": eligibility.reason}

    # BRMS: Pricing rules (เปลี่ยนบ่อย)
    pricing = rule_engine.evaluate("PRICING", application)

    return {
        "status": "APPROVED",
        "interest_rate": pricing.rate,
        "max_amount": pricing.amount
    }
```

---

## 10. Diagnostic Questions

ตอบคำถามเหล่านี้เพื่อทดสอบความเข้าใจ:

1. ข้อดีหลักของ hard-coded logic คืออะไร?
2. เมื่อไหร่ควรใช้ BRMS แทน hard-coded?
3. ทำไมบางกฎควรอยู่ใน code แม้มี BRMS?
4. Hybrid approach คืออะไร?
5. ถ้ากฎเปลี่ยนปีละครั้ง ควรใช้แบบไหน?
6. Performance ต่างกันอย่างไร?
7. ค่าใช้จ่ายต่างกันอย่างไร?
8. ใครควรเป็นคน maintain แต่ละแบบ?
9. Testing ต่างกันอย่างไร?
10. ตัดสินใจอย่างไรว่าควร migrate rule ไป BRMS?

---

## 11. Explainer for 5-year-old

ลองนึกถึง **การตกแต่งห้อง** นะ

**Hard-coded = ทาสีผนัง**
- สวยดี แต่ถ้าอยากเปลี่ยนสี ต้องทาใหม่ทั้งห้อง
- ใช้เวลา ใช้เงิน ยุ่งยาก

**BRMS = ติดวอลเปเปอร์**
- ถ้าเบื่อ ลอกออกแปะใหม่ได้เลย
- เร็ว ง่าย ไม่ยุ่งยาก

แต่บางที่ก็ควรทาสี (เช่น ห้องน้ำ กันน้ำดี)
บางที่ก็ควรติดวอลเปเปอร์ (เช่น ห้องนอน เปลี่ยนบ่อย)

เลือกให้ถูกที่!

---

## 12. Comparison Table

| หัวข้อ | Hard-coded | BRMS |
|--------|------------|------|
| **เปลี่ยนแปลง** | ยาก (แก้ code + deploy) | ง่าย (แก้ rule + activate) |
| **เวลา deploy** | หลายวัน | หลักชั่วโมง |
| **ใครแก้ได้** | Developer เท่านั้น | BA ก็ได้ |
| **Performance** | เร็วกว่า | ช้ากว่าเล็กน้อย |
| **Initial cost** | ต่ำ | สูง (setup + training) |
| **Maintenance cost** | สูง (developer time) | ต่ำ |
| **Traceability** | ดูจาก git | มี audit log ในตัว |
| **Testing** | Unit test | Rule test + Unit test |
| **Complexity** | ต่ำ | สูงกว่า |
| **Scalability** | จำกัด | ดีกว่า |
| **Suitable for** | Stable logic | Changing logic |

---

## 13. 7-Day Rapid Learning Roadmap

### สำหรับหัวข้อนี้โดยเฉพาะ (ใช้เวลา 1-1.5 ชั่วโมง)

| เวลา | กิจกรรม |
|------|---------|
| 0:00-0:15 | อ่าน Abstract และ Core Principles |
| 0:15-0:30 | ศึกษา Practical Examples ทั้ง 5 ตัวอย่าง |
| 0:30-0:45 | อ่าน Comparison Table และ Pitfalls |
| 0:45-1:00 | ลองตอบ Diagnostic Questions |
| 1:00-1:15 | ฝึกตัดสินใจ: ยก 5 scenarios แล้วเลือกว่าใช้แบบไหน |
| 1:15-1:30 | สรุปเป็น decision tree ของตัวเอง |

### เป้าหมายหลังจบ:
- [ ] แยกแยะได้ทันทีว่าอะไรควร hard-code อะไรควรเป็น BRMS
- [ ] อธิบายข้อดี-ข้อเสียของแต่ละแบบได้
- [ ] เข้าใจ hybrid approach
- [ ] ตัดสินใจได้ว่าเมื่อไหร่ควรใช้แบบไหน
