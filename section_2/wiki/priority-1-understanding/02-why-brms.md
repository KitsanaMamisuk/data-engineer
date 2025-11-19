# ทำไมองค์กรต้องใช้ BRMS

## 1. Abstract (60 วินาที)

องค์กรใช้ BRMS เพราะ business rules เปลี่ยนบ่อยมาก โดยเฉพาะธนาคารที่ต้องปรับตามกฎหมาย การแข่งขัน และโปรโมชั่น ถ้าทุกครั้งที่เปลี่ยนกฎต้องรอ developer แก้ code → test → deploy อาจใช้เวลาหลายสัปดาห์ แต่ถ้าใช้ BRMS ฝั่ง business สามารถแก้ไขกฎได้เองภายในไม่กี่ชั่วโมง ลดค่าใช้จ่าย ลดเวลา และลดความผิดพลาด

---

## 2. Core Principles (20/80)

หลัก 20% ที่ให้ผลลัพธ์ 80%:

1. **Speed to Market** - เปลี่ยนกฎได้เร็ว ไม่ต้องรอ release cycle
2. **Reduce IT Dependency** - Business ไม่ต้องพึ่ง IT ทุกครั้ง
3. **Consistency** - กฎเดียวกันใช้ทั่วทั้งองค์กร
4. **Compliance** - ตรวจสอบได้ว่าทำตามกฎหมายหรือไม่
5. **Cost Reduction** - ลดค่าใช้จ่ายในการ maintain code

---

## 3. Mental Models

### Model 1: "ร้านอาหาร Fast Food vs Fine Dining"
- **Fast Food (BRMS)**: เปลี่ยนเมนูได้ทุกวัน แค่เปลี่ยนป้าย
- **Fine Dining (Hard-coded)**: เปลี่ยนเมนูต้อง retrain พ่อครัว ทำเมนูใหม่ใช้เวลานาน

### Model 2: "กฎกติกาบอร์ดเกม"
- เล่นเกมเดิม แต่เปลี่ยนกติกาได้
- ไม่ต้องซื้อเกมใหม่ทุกครั้งที่อยากเปลี่ยนวิธีเล่น

### Model 3: "รีโมททีวี"
- รีโมท (BRMS) ควบคุมทีวี (System) ได้ทันที
- ไม่ต้องเปิดฝาทีวีแก้วงจรทุกครั้งที่อยากเปลี่ยนช่อง

---

## 4. Skill Tree

```
พื้นฐาน
├── เข้าใจปัญหาของการ hard-code business logic
├── รู้ว่า business rules เปลี่ยนบ่อยแค่ไหน
└── เข้าใจ cost of change

กลาง
├── วิเคราะห์ ROI ของการใช้ BRMS
├── ระบุ use cases ที่เหมาะกับ BRMS
├── เข้าใจ compliance requirements
└── รู้ว่าเมื่อไหร่ไม่ควรใช้ BRMS

สูง
├── วางแผน BRMS implementation
├── บริหาร change management
├── วัด business value
└── Optimize rule performance at scale
```

---

## 5. What NOT to Learn

- **อย่าเรียน** รายละเอียด ROI calculation → แค่เข้าใจแนวคิด
- **อย่าเรียน** การขาย BRMS ให้ผู้บริหาร → ไม่ใช่หน้าที่ของ developer
- **อย่าเรียน** ทุก compliance framework → เรียนแค่ที่เกี่ยวข้องกับธนาคาร
- **อย่าเรียน** การเปรียบเทียบ vendor ทุกราย → งานนี้ใช้ internal platform
- **อย่าจำ** ตัวเลข statistics → เข้าใจแนวคิดว่าทำไมถึงดีกว่า

---

## 6. Real-world Usage

### สถานการณ์ที่เห็นประโยชน์ชัดเจน:

1. **กฎหมายเปลี่ยน**
   - ธปท. ออกกฎใหม่ → ธนาคารต้องปรับภายใน 30 วัน
   - ถ้า hard-code อาจไม่ทัน
   - ใช้ BRMS แก้ได้ภายในวัน

2. **โปรโมชั่นแข่งขัน**
   - คู่แข่งออกโปรใหม่ → ต้องตอบโต้เร็ว
   - แก้กฎคำนวณดอกเบี้ย/ส่วนลดได้ทันที

3. **Fraud Detection**
   - พบ pattern การโกงแบบใหม่
   - เพิ่มกฎตรวจจับได้ทันทีไม่ต้องรอ deploy

4. **Product Launch**
   - ออกผลิตภัณฑ์ใหม่ → กำหนดกฎได้เลย
   - ไม่ต้องเขียน code ใหม่ทั้งหมด

---

## 7. Typical Pitfalls (10 ข้อ)

| # | ความผิดพลาด | วิธีเลี่ยง |
|---|-------------|-----------|
| 1 | คิดว่า BRMS แก้ทุกปัญหา | ใช้กับ business logic ที่เปลี่ยนบ่อยเท่านั้น |
| 2 | ไม่คำนวณ total cost | รวมค่า training, maintenance, license |
| 3 | ไม่เตรียม business users | ต้อง train ให้ใช้งานเป็น |
| 4 | ใช้ BRMS กับทุกอย่าง | บาง logic ควรอยู่ใน code |
| 5 | ไม่มี governance | ต้องมีคนดูแลภาพรวมของ rules |
| 6 | คิดว่าไม่ต้องใช้ developer | ยังต้องมี developer ดูแล complex rules |
| 7 | ไม่ทำ change management | ต้องเปลี่ยน mindset คนในองค์กร |
| 8 | ไม่วัด performance | ต้องติดตามว่าเร็วขึ้นจริงไหม |
| 9 | ละเลย security | กฎบางอย่างต้อง restrict access |
| 10 | ไม่มี backup plan | ต้องมีแผนรองรับถ้า BRMS ล่ม |

---

## 8. Playbook (Step-by-step)

### เมื่อต้องอธิบายว่าทำไมต้องใช้ BRMS:

1. **ระบุปัญหาปัจจุบัน**
   - ใช้เวลานานแค่ไหนในการเปลี่ยน business rule
   - มีกี่ครั้งที่พลาด deadline

2. **คำนวณ cost of delay**
   - รายได้ที่เสียไปจากการเปลี่ยนช้า
   - ค่าปรับจากการไม่ comply

3. **เปรียบเทียบ before/after**
   - ก่อน: แก้ code → test → deploy = 2 สัปดาห์
   - หลัง: แก้ rule → test → activate = 2 ชั่วโมง

4. **แสดง use cases**
   - ยกตัวอย่างจริงในองค์กร
   - ถ้าใช้ BRMS จะเปลี่ยนอะไรได้บ้าง

5. **สรุป benefits**
   - Speed, Cost, Compliance, Consistency

---

## 9. Fast Practical Examples

### Example 1: ค่าใช้จ่ายในการเปลี่ยนกฎ

```
# แบบ Hard-coded
Developer time: 8 hours × 2,000 บาท = 16,000 บาท
QA time: 4 hours × 1,500 บาท = 6,000 บาท
Deployment: 2 hours × 2,000 บาท = 4,000 บาท
Total: 26,000 บาท + 3-5 วันรอ

# แบบ BRMS
BA time: 1 hour × 1,500 บาท = 1,500 บาท
Testing: 1 hour × 1,500 บาท = 1,500 บาท
Total: 3,000 บาท + ใช้งานได้ทันที
```

### Example 2: Compliance Scenario

```python
# เมื่อ ธปท. ประกาศกฎใหม่
# "ห้ามเรียกเก็บค่าธรรมเนียม SMS สำหรับลูกค้าอายุเกิน 65 ปี"

# Hard-coded approach
# - หา code ที่คิดค่า SMS
# - แก้ไข, test, deploy
# - ใช้เวลา 1-2 สัปดาห์

# BRMS approach
# - เปิด rule editor
# - เพิ่มเงื่อนไข: if customer.age >= 65: sms_fee = 0
# - Test และ activate
# - ใช้เวลา 2-3 ชั่วโมง
```

### Example 3: Marketing Campaign

```python
# โปรโมชั่น: ลูกค้าใหม่ได้ดอกเบี้ยพิเศษ 3 เดือน
# เริ่ม 1 ม.ค. หมด 31 ม.ค.

# ใน BRMS สร้าง rule:
rule = {
    "name": "New Year Promo 2024",
    "condition": {
        "customer_type": "NEW",
        "application_date": {"between": ["2024-01-01", "2024-01-31"]}
    },
    "action": {
        "special_rate": 2.5,
        "duration_months": 3
    },
    "effective_date": "2024-01-01",
    "expiry_date": "2024-01-31"
}
```

### Example 4: Risk Management

```python
# Fraud pattern ใหม่: ถอนเงินหลายครั้งใน 1 ชั่วโมง

# เพิ่ม rule ใน BRMS ได้ทันที:
def detect_rapid_withdrawal(transactions):
    recent = [t for t in transactions
              if t.type == "WITHDRAWAL"
              and t.time > now() - timedelta(hours=1)]

    if len(recent) >= 5:
        return {"alert": "RAPID_WITHDRAWAL", "action": "BLOCK"}
    return {"alert": None}
```

### Example 5: Product Pricing

```python
# เปลี่ยนดอกเบี้ยตาม market rate

# BRMS ทำให้เปลี่ยนได้ทันที:
pricing_rules = {
    "home_loan": {
        "base_rate": 6.5,  # เปลี่ยนได้ทันที
        "spread": {
            "excellent_credit": -1.0,
            "good_credit": -0.5,
            "fair_credit": 0,
            "poor_credit": +1.0
        }
    }
}
```

---

## 10. Diagnostic Questions

ตอบคำถามเหล่านี้เพื่อทดสอบความเข้าใจ:

1. องค์กรประเภทไหนได้ประโยชน์จาก BRMS มากที่สุด?
2. ถ้าไม่ใช้ BRMS จะเกิดปัญหาอะไรบ้าง?
3. BRMS ช่วยเรื่อง compliance อย่างไร?
4. ทำไม speed to market ถึงสำคัญในธุรกิจธนาคาร?
5. การลด IT dependency มีข้อดีอะไรบ้าง?
6. เมื่อไหร่ที่ไม่ควรใช้ BRMS?
7. BRMS ช่วยลด cost อย่างไร?
8. ทำไมต้องมี governance ในการใช้ BRMS?
9. ความแตกต่างระหว่าง one-time cost กับ ongoing cost ของ BRMS?
10. BRMS ช่วยเรื่อง audit อย่างไร?

---

## 11. Explainer for 5-year-old

ลองนึกถึง **กติกาในห้องเรียน** นะ

ถ้าครูอยากเปลี่ยนกติกาจาก "ยกมือก่อนพูด" เป็น "พูดได้เลยถ้าเป็นคำถาม"

**แบบไม่มี BRMS**:
- ครูต้องไปขอให้อาจารย์ใหญ่พิมพ์กติกาใหม่
- รอ 1 สัปดาห์
- ค่อยเอามาติดในห้อง

**แบบมี BRMS**:
- ครูเขียนกติกาใหม่บนกระดาน
- เสร็จเลย ใช้ได้เลย

ธนาคารก็เหมือนกัน อยากเปลี่ยนกฎได้เร็วๆ ไม่ต้องรอนาน

---

## 12. Comparison Table

| หัวข้อ | ไม่ใช้ BRMS | ใช้ BRMS |
|--------|------------|----------|
| **เวลาเปลี่ยนกฎ** | 1-4 สัปดาห์ | 1-4 ชั่วโมง |
| **ค่าใช้จ่ายต่อครั้ง** | สูง (developer + QA + deploy) | ต่ำ (BA + test) |
| **ความเสี่ยง** | สูง (แก้ code อาจพลาด) | ต่ำ (มี validation) |
| **Compliance** | ยากที่จะ prove | มี audit trail ชัดเจน |
| **Consistency** | กฎอาจกระจายหลายที่ | กฎอยู่ที่เดียว |
| **Business ownership** | IT เป็นเจ้าของ | Business เป็นเจ้าของ |
| **Agility** | ต่ำ | สูง |
| **Initial cost** | ต่ำ | สูง (setup + training) |
| **Long-term cost** | สูง (maintenance) | ต่ำ |
| **Scalability** | ยาก | ง่าย |

---

## 13. 7-Day Rapid Learning Roadmap

### สำหรับหัวข้อนี้โดยเฉพาะ (ใช้เวลา 1-2 ชั่วโมง)

| เวลา | กิจกรรม |
|------|---------|
| 0:00-0:20 | อ่าน Abstract และ Core Principles |
| 0:20-0:40 | ศึกษา Practical Examples |
| 0:40-1:00 | อ่าน Pitfalls และ Comparison Table |
| 1:00-1:20 | ตอบ Diagnostic Questions 5 ข้อแรก |
| 1:20-1:40 | ลองอธิบายให้คนอื่นฟังว่า "ทำไมต้องใช้ BRMS" |
| 1:40-2:00 | สรุปเป็น bullet points สั้นๆ เก็บไว้ทบทวน |

### เป้าหมายหลังจบ:
- [ ] อธิบายได้ 5 เหตุผลที่องค์กรใช้ BRMS
- [ ] บอกได้ว่าเมื่อไหร่ไม่ควรใช้ BRMS
- [ ] เข้าใจ cost-benefit ของ BRMS
- [ ] ยกตัวอย่าง use case ในธนาคารได้
