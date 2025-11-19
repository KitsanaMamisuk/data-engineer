# แนวคิดและหลักการของ BRMS (Business Rules Management System)

## 1. Abstract (60 วินาที)

BRMS คือระบบที่แยก "กฎเกณฑ์ทางธุรกิจ" ออกจาก code หลัก ทำให้คนที่ไม่ใช่ programmer สามารถแก้ไขกฎได้เอง เช่น "ถ้าลูกค้ามีเงินเดือน > 50,000 และไม่มีหนี้เสีย → อนุมัติสินเชื่อได้" แทนที่จะต้องรอ developer แก้ code ทุกครั้งที่กฎเปลี่ยน ฝั่ง business สามารถเข้าไปแก้เงื่อนไขได้เองผ่านหน้าจอที่ออกแบบมาให้ใช้งานง่าย

---

## 2. Core Principles (20/80)

หลัก 20% ที่ให้ผลลัพธ์ 80%:

1. **Separation of Concerns** - แยก business logic ออกจาก application code
2. **Externalization** - กฎอยู่ภายนอก code ในรูปแบบที่แก้ไขได้ง่าย
3. **Centralization** - กฎทั้งหมดอยู่ที่เดียว ไม่กระจาย
4. **Version Control** - ทุกการเปลี่ยนแปลงถูกบันทึกและย้อนกลับได้
5. **Business Ownership** - ฝั่ง business เป็นเจ้าของกฎ ไม่ใช่ IT

---

## 3. Mental Models

### Model 1: "สูตรอาหาร vs พ่อครัว"
- **พ่อครัว** = Application Code (ทำตามสูตร)
- **สูตรอาหาร** = Business Rules (เปลี่ยนได้โดยไม่ต้องเปลี่ยนพ่อครัว)
- เมื่อต้องการเปลี่ยนรสชาติ แค่แก้สูตร ไม่ต้อง retrain พ่อครัว

### Model 2: "กฎจราจร vs รถยนต์"
- **รถยนต์** = System
- **กฎจราจร** = Business Rules
- รถวิ่งตามกฎ แต่กฎเปลี่ยนได้โดยไม่ต้องแก้รถ

### Model 3: "ตู้ ATM"
- ตู้ ATM (system) ทำงานตามกฎที่ธนาคารกำหนด
- ธนาคารเปลี่ยนวงเงินถอน/ค่าธรรมเนียมได้ โดยไม่ต้องเปลี่ยนตู้

---

## 4. Skill Tree

```
พื้นฐาน
├── เข้าใจว่า Business Rule คืออะไร
├── แยกแยะได้ว่าอะไรควรเป็น rule อะไรควรเป็น code
└── อ่าน rule แบบ if-then ได้

กลาง
├── เขียน rule ด้วย syntax ของ BRMS
├── จัดกลุ่ม rules (Rule Sets)
├── กำหนดลำดับการ execute
└── ทำ versioning และ rollback

สูง
├── ออกแบบ Rule Architecture
├── Optimize performance
├── ทำ Rule Testing Automation
└── บริหาร Rule Lifecycle
```

---

## 5. What NOT to Learn

- **อย่าเรียน** ทุก BRMS tools ในตลาด (Drools, IBM ODM, etc.) → เรียนแค่แนวคิด
- **อย่าเรียน** การ setup BRMS server ตั้งแต่ศูนย์ → งานนี้ใช้ platform สำเร็จรูป
- **อย่าเรียน** Rule Language ทุกภาษา → เรียนแค่ที่ใช้ในงาน (Python-based)
- **อย่าเรียน** Advanced AI/ML integration กับ BRMS → ยังไม่จำเป็นตอนนี้
- **อย่าท่อง** syntax → เข้าใจแนวคิด แล้วค่อยดู reference ตอนเขียน

---

## 6. Real-world Usage

### สถานการณ์ที่ใช้บ่อยที่สุดในธนาคาร:

1. **อนุมัติสินเชื่อ**
   - Input: ข้อมูลลูกค้า (เงินเดือน, หนี้, อายุ)
   - Rule: ตรวจสอบเงื่อนไข 10-20 ข้อ
   - Output: อนุมัติ/ปฏิเสธ/ส่งต่อ

2. **KYC (Know Your Customer)**
   - ตรวจสอบเอกสาร
   - เปรียบเทียบกับ blacklist
   - กำหนด risk level

3. **คำนวณดอกเบี้ย/ค่าธรรมเนียม**
   - เงื่อนไขต่างกันตามประเภทลูกค้า
   - โปรโมชั่นที่เปลี่ยนบ่อย

---

## 7. Typical Pitfalls (10 ข้อ)

| # | ความผิดพลาด | วิธีเลี่ยง |
|---|-------------|-----------|
| 1 | เขียน rule ซับซ้อนเกินไป | แตกเป็น rule ย่อยๆ |
| 2 | ไม่ทำ version control | ใช้ระบบ versioning ทุกครั้ง |
| 3 | ไม่ test rule ก่อน deploy | สร้าง test cases ครบทุก scenario |
| 4 | Rule ขัดแย้งกัน | กำหนด priority และ conflict resolution |
| 5 | ไม่มี documentation | เขียน description ทุก rule |
| 6 | Hard-code ค่าคงที่ใน rule | ใช้ parameters/variables |
| 7 | ไม่คิดเรื่อง performance | ทำ profiling และ optimize |
| 8 | ให้ business แก้ rule โดยไม่ review | มี approval workflow |
| 9 | ไม่ทำ audit trail | log ทุกการเปลี่ยนแปลง |
| 10 | ลืม edge cases | ทำ boundary testing |

---

## 8. Playbook (Step-by-step)

### เมื่อได้รับ requirement ใหม่:

1. **รับ requirement** จาก Business Analyst
2. **วิเคราะห์** ว่าเป็น rule ประเภทไหน (Decision/Validation/Calculation)
3. **ออกแบบ** rule structure (if-then, decision table, etc.)
4. **เขียน rule** ใน BRMS platform
5. **Test** ด้วย test cases จาก BA
6. **Review** กับ BA และ peer developer
7. **Deploy** ไป test environment
8. **UAT** กับ business user
9. **Deploy** ไป production
10. **Monitor** และ maintain

---

## 9. Fast Practical Examples

### Example 1: Rule อนุมัติสินเชื่อ
```python
# Pseudocode
if customer.salary >= 30000 and customer.debt_ratio < 0.5:
    if customer.credit_score >= 700:
        return "APPROVED"
    else:
        return "REVIEW_REQUIRED"
else:
    return "REJECTED"
```

### Example 2: Rule คำนวณดอกเบี้ย
```python
# Decision Table Style
def get_interest_rate(customer_type, loan_amount):
    rules = {
        ("GOLD", "< 1M"): 5.5,
        ("GOLD", ">= 1M"): 4.5,
        ("SILVER", "< 1M"): 7.0,
        ("SILVER", ">= 1M"): 6.0,
    }
    return rules.get((customer_type, loan_amount), 8.0)
```

### Example 3: Validation Rule
```python
def validate_age(age):
    if age < 20:
        return {"valid": False, "message": "อายุต้องมากกว่า 20 ปี"}
    if age > 60:
        return {"valid": False, "message": "อายุต้องไม่เกิน 60 ปี"}
    return {"valid": True}
```

### Example 4: Rule ตรวจสอบเอกสาร KYC
```python
required_docs = ["ID_CARD", "INCOME_PROOF", "ADDRESS_PROOF"]

def check_kyc_documents(submitted_docs):
    missing = [doc for doc in required_docs if doc not in submitted_docs]
    if missing:
        return {"status": "INCOMPLETE", "missing": missing}
    return {"status": "COMPLETE"}
```

### Example 5: Rule กำหนด Risk Level
```python
def assess_risk(transaction_amount, country, is_pep):
    risk_score = 0

    if transaction_amount > 1000000:
        risk_score += 30
    if country in HIGH_RISK_COUNTRIES:
        risk_score += 40
    if is_pep:  # Politically Exposed Person
        risk_score += 30

    if risk_score >= 70:
        return "HIGH"
    elif risk_score >= 40:
        return "MEDIUM"
    return "LOW"
```

---

## 10. Diagnostic Questions

ตอบคำถามเหล่านี้เพื่อทดสอบความเข้าใจ:

1. ทำไมต้องแยก business rules ออกจาก code?
2. ใครควรเป็นคนแก้ไข business rules?
3. ถ้ากฎ 2 ข้อขัดแย้งกัน จะจัดการอย่างไร?
4. BRMS ต่างจากการเขียน if-else ใน code อย่างไร?
5. เมื่อไหร่ควรใช้ BRMS เมื่อไหร่ไม่ควร?
6. Version control ของ rules สำคัญอย่างไร?
7. จะ test business rules อย่างไรให้ครบถ้วน?
8. ข้อดีของการให้ business user จัดการ rules เองคืออะไร?
9. Performance ของ rule engine ขึ้นกับอะไรบ้าง?
10. Audit trail ของ rules ช่วยอะไรในงานธนาคาร?

---

## 11. Explainer for 5-year-old

ลองนึกถึง **เกมตัวต่อ LEGO** นะ

- **LEGO blocks** = กฎต่างๆ (เช่น "ถ้าสีแดงต้องต่อกับสีแดง")
- **คนเล่น** = ระบบคอมพิวเตอร์
- **คู่มือ** = BRMS

เวลาเราอยากเปลี่ยนวิธีต่อ LEGO เราแค่เปลี่ยน "คู่มือ" ไม่ต้องเปลี่ยน "ตัว LEGO" หรือ "คนเล่น"

เหมือนกัน ถ้าธนาคารอยากเปลี่ยนกฎว่า "ให้กู้เงินได้เท่าไหร่" ก็แค่เปลี่ยนกฎใน BRMS ไม่ต้องเปลี่ยนระบบคอมพิวเตอร์ทั้งหมด

---

## 12. Comparison Table

| หัวข้อ | สิ่งที่คุณอาจรู้มาก่อน | BRMS |
|--------|----------------------|------|
| **การแก้ไขกฎ** | แก้ code → test → deploy | แก้ผ่าน UI → test → activate |
| **ใครแก้** | Developer เท่านั้น | Business Analyst ก็ได้ |
| **เวลาที่ใช้** | หลายวัน-สัปดาห์ | หลักชั่วโมง-วัน |
| **ความเสี่ยง** | แก้ code ผิดพลาดได้ง่าย | มี validation ก่อน save |
| **Traceability** | ดูจาก git history | มี audit log ในตัว |
| **Testing** | เขียน unit test เอง | มี test tool ในตัว |
| **Reusability** | Copy-paste code | สร้าง rule template ใช้ซ้ำ |

---

## 13. 7-Day Rapid Learning Roadmap

### สำหรับหัวข้อนี้โดยเฉพาะ (ใช้เวลา 2-3 ชั่วโมง)

| ชั่วโมง | กิจกรรม |
|---------|---------|
| 0:00-0:30 | อ่าน Abstract และ Core Principles ให้เข้าใจ |
| 0:30-1:00 | ศึกษา Mental Models ทั้ง 3 แบบ |
| 1:00-1:30 | ดู Practical Examples และลองเขียนตาม |
| 1:30-2:00 | ตอบ Diagnostic Questions (เขียนคำตอบสั้นๆ) |
| 2:00-2:30 | อ่าน Pitfalls และจำไว้ |
| 2:30-3:00 | ทบทวนโดยอธิบายให้คนอื่นฟัง (หรือเขียนสรุป) |

### เป้าหมายหลังจบ:
- [ ] อธิบายได้ว่า BRMS คืออะไรใน 1 นาที
- [ ] บอกได้ว่าทำไมธนาคารถึงใช้ BRMS
- [ ] แยกแยะได้ว่าอะไรควรเป็น rule อะไรควรเป็น code
- [ ] เขียน rule แบบง่ายๆ ได้
