# แนวคิดของ Low-code Development

## 1. Abstract (60 วินาที)

Low-code Development คือการสร้างซอฟต์แวร์โดยใช้ visual interface แทนการเขียน code ทั้งหมด ผู้ใช้สามารถลาก-วาง components, กำหนด logic ผ่าน flowchart และกรอก parameters แทนการเขียน code ในงาน BRMS ที่ TTB นั้น Low-code ช่วยให้ Business Analysts สร้างและแก้ไข rules ได้เองโดยไม่ต้องพึ่ง developer ตลอดเวลา ลดเวลาในการ deploy changes จากหลายสัปดาห์เหลือไม่กี่ชั่วโมง

---

## 2. Core Principles (20/80)

หลัก 20% ที่ให้ผลลัพธ์ 80%:

1. **Visual Development** - สร้าง logic ผ่าน UI แทน code
2. **Abstraction** - ซ่อนความซับซ้อนของ code
3. **Citizen Developer** - คนที่ไม่ใช่ programmer ก็สร้างได้
4. **Speed to Market** - เร็วกว่าเขียน code ปกติ
5. **Maintain by Business** - Business ดูแลเองได้

---

## 3. Mental Models

### Model 1: "เลโก้ vs ปั้นดิน"
- **Traditional code** = ปั้นดิน (สร้างได้ทุกรูป แต่ต้องมีฝีมือ)
- **Low-code** = เลโก้ (ต่อ blocks สำเร็จรูป ง่าย เร็ว)

### Model 2: "เครื่องชงกาแฟ"
- กาแฟสด (traditional) = ต้องรู้วิธีชง
- เครื่องแคปซูล (low-code) = กดปุ่มเดียว ได้กาแฟ

### Model 3: "Canva vs Photoshop"
- Photoshop = powerful แต่เรียนรู้ยาก
- Canva = ง่าย เร็ว ทำได้ส่วนใหญ่

---

## 4. Skill Tree

```
พื้นฐาน
├── เข้าใจแนวคิด low-code
├── ใช้งาน visual interface ได้
└── รู้ข้อจำกัดของ low-code

กลาง
├── ออกแบบ rules ใน low-code platform
├── ผสม low-code กับ custom code
├── Test และ debug ใน platform
└── Train คนอื่นให้ใช้งาน

สูง
├── พัฒนา low-code platform
├── สร้าง custom components
├── Optimize platform performance
└── กำหนด governance policies
```

---

## 5. What NOT to Learn

- **อย่าเรียน** ทุก low-code platforms → เรียนแค่ที่ใช้ในงาน
- **อย่าเรียน** การสร้าง low-code platform → เน้นการใช้งาน
- **อย่าเรียน** UI/UX design ลึก → เข้าใจพื้นฐานพอ
- **อย่าเปรียบเทียบ** ทุก platform → focus ที่งาน
- **อย่าท่อง** ทุก feature → เรียนรู้เมื่อต้องใช้

---

## 6. Real-world Usage

### ในงาน BRMS ที่ TTB:

1. **Business Analyst สร้าง rules**
   - ลาก-วาง conditions
   - กำหนด actions
   - Test ด้วย sample data

2. **Developer ช่วย complex cases**
   - เขียน custom functions
   - Optimize performance
   - Integrate กับระบบอื่น

3. **Maintenance**
   - BA แก้ไข rules ง่ายๆ ได้เอง
   - Developer ดู complex changes

---

## 7. Typical Pitfalls (10 ข้อ)

| # | ความผิดพลาด | วิธีเลี่ยง |
|---|-------------|-----------|
| 1 | คิดว่า low-code ทำได้ทุกอย่าง | รู้ข้อจำกัด |
| 2 | ไม่ train users | จัด training ให้ครบ |
| 3 | ไม่มี governance | กำหนด rules และ process |
| 4 | สร้าง spaghetti logic | ออกแบบให้ modular |
| 5 | ไม่ version control | ใช้ platform versioning |
| 6 | ไม่ test | Test ทุก change |
| 7 | ไม่มี documentation | Document rules |
| 8 | ให้ทุกคนแก้ได้ | Control access |
| 9 | คิดว่าไม่ต้องมี developer | ยังต้องมี developer support |
| 10 | ไม่ monitor usage | Track และ optimize |

---

## 8. Playbook (Step-by-step)

### การใช้งาน Low-code BRMS Platform:

#### สำหรับ Business Analyst:

1. **Login เข้า platform**
2. **เลือก rule set ที่ต้องการแก้**
3. **สร้าง/แก้ไข rule**
   - ลาก condition blocks
   - กำหนด operators และ values
   - ระบุ actions
4. **Test rule**
   - ใส่ test data
   - ดู result
   - แก้ไขถ้าไม่ถูก
5. **Submit for review**
6. **รอ approval**
7. **Deploy**

#### สำหรับ Developer:

1. **Review rules ที่ BA สร้าง**
2. **ช่วย complex logic**
   - เขียน custom functions
   - Optimize queries
3. **Support deployment**
4. **Monitor performance**

---

## 9. Fast Practical Examples

### Example 1: Visual Rule Builder Interface

```
┌─────────────────────────────────────────────────┐
│  Rule Builder: Loan Eligibility                 │
├─────────────────────────────────────────────────┤
│                                                 │
│  IF:                                            │
│  ┌─────────────────────────────────────┐        │
│  │ [Age] [>=] [20]                AND │        │
│  │ [Age] [<=] [60]                AND │        │
│  │ [Salary] [>=] [30,000]         AND │        │
│  │ [Debt Ratio] [<=] [0.5]            │        │
│  └─────────────────────────────────────┘        │
│                                                 │
│  THEN:                                          │
│  ┌─────────────────────────────────────┐        │
│  │ Decision: [Approved]                │        │
│  │ Message: [ผ่านเกณฑ์เบื้องต้น]            │        │
│  └─────────────────────────────────────┘        │
│                                                 │
│  ELSE:                                          │
│  ┌─────────────────────────────────────┐        │
│  │ Decision: [Rejected]                │        │
│  │ Message: [ไม่ผ่านเกณฑ์]                │        │
│  └─────────────────────────────────────┘        │
│                                                 │
│  [Save] [Test] [Submit for Review]              │
└─────────────────────────────────────────────────┘
```

### Example 2: Low-code vs Traditional Code

**Low-code (Visual):**
```yaml
Rule: Loan Eligibility Check
Conditions:
  - Field: customer.age
    Operator: between
    Values: [20, 60]
  - Field: customer.salary
    Operator: >=
    Value: 30000
  - Field: customer.debt_ratio
    Operator: <=
    Value: 0.5
Actions:
  - Set: decision = "APPROVED"
  - Set: message = "ผ่านเกณฑ์"
Else:
  - Set: decision = "REJECTED"
  - Set: message = "ไม่ผ่านเกณฑ์"
```

**Traditional Code:**
```python
def check_loan_eligibility(customer):
    if (20 <= customer.age <= 60 and
        customer.salary >= 30000 and
        customer.debt_ratio <= 0.5):
        return {
            "decision": "APPROVED",
            "message": "ผ่านเกณฑ์"
        }
    else:
        return {
            "decision": "REJECTED",
            "message": "ไม่ผ่านเกณฑ์"
        }
```

### Example 3: Decision Table in Low-code

```
┌──────────────────────────────────────────────────────┐
│  Decision Table: Interest Rate Calculation           │
├─────────────┬──────────────┬──────────────┬──────────┤
│ Credit Grade│ Customer Type│ Interest Rate│ Max Loan │
├─────────────┼──────────────┼──────────────┼──────────┤
│ A           │ GOLD         │ 4.5%         │ 3,000,000│
│ A           │ SILVER       │ 5.0%         │ 2,000,000│
│ A           │ NORMAL       │ 5.5%         │ 1,000,000│
│ B           │ GOLD         │ 5.5%         │ 2,000,000│
│ B           │ SILVER       │ 6.0%         │ 1,500,000│
│ B           │ NORMAL       │ 6.5%         │ 1,000,000│
│ C           │ ANY          │ 7.5%         │   500,000│
├─────────────┴──────────────┴──────────────┴──────────┤
│ [Add Row] [Delete Row] [Save] [Test]                 │
└──────────────────────────────────────────────────────┘
```

### Example 4: Flow-based Rule Design

```
┌─────────────────────────────────────────────────┐
│  Flow: Loan Application Processing              │
├─────────────────────────────────────────────────┤
│                                                 │
│  ┌─────────┐                                    │
│  │  Start  │                                    │
│  └────┬────┘                                    │
│       │                                         │
│       ▼                                         │
│  ┌─────────────┐                                │
│  │  Validate   │                                │
│  │  Documents  │                                │
│  └────┬────────┘                                │
│       │                                         │
│   ┌───┴───┐                                     │
│   │ Pass? │                                     │
│   └───┬───┘                                     │
│  Yes  │   No                                    │
│   ┌───┴───┐   ┌──────────┐                      │
│   │       │   │  Return  │                      │
│   │       │   │  Error   │                      │
│   │       │   └──────────┘                      │
│   ▼                                             │
│  ┌─────────────┐                                │
│  │  Calculate  │                                │
│  │  Score      │                                │
│  └────┬────────┘                                │
│       │                                         │
│       ▼                                         │
│  ┌─────────────┐                                │
│  │   Make      │                                │
│  │  Decision   │                                │
│  └────┬────────┘                                │
│       │                                         │
│       ▼                                         │
│  ┌─────────┐                                    │
│  │   End   │                                    │
│  └─────────┘                                    │
│                                                 │
└─────────────────────────────────────────────────┘
```

### Example 5: Custom Function in Low-code

```python
# Developer สร้าง custom function
# BA เรียกใช้ผ่าน visual interface

@register_function("calculate_dti")
def calculate_debt_to_income(monthly_debt, monthly_income):
    """
    Calculate Debt-to-Income ratio
    Available as: calculate_dti(debt, income)
    """
    if monthly_income <= 0:
        return 999  # Invalid
    return round(monthly_debt / monthly_income, 4)

@register_function("check_blacklist")
def check_customer_blacklist(customer_id):
    """
    Check if customer is in blacklist
    Available as: check_blacklist(customer_id)
    """
    # Query blacklist database
    return blacklist_service.is_blacklisted(customer_id)
```

**BA เรียกใช้ใน Visual Builder:**
```yaml
Conditions:
  - Function: calculate_dti
    Parameters:
      - Field: monthly_debt
      - Field: monthly_income
    Operator: <=
    Value: 0.5

  - Function: check_blacklist
    Parameters:
      - Field: customer_id
    Operator: ==
    Value: false
```

---

## 10. Diagnostic Questions

ตอบคำถามเหล่านี้เพื่อทดสอบความเข้าใจ:

1. Low-code development คืออะไร?
2. ใครได้ประโยชน์จาก low-code มากที่สุด?
3. Low-code ต่างจาก no-code อย่างไร?
4. ข้อจำกัดหลักของ low-code คืออะไร?
5. Developer ยังจำเป็นไหมในงาน low-code?
6. เมื่อไหร่ควรใช้ low-code เมื่อไหร่ควรเขียน code?
7. Governance ใน low-code คืออะไร?
8. Training สำคัญอย่างไรใน low-code?
9. Version control ใน low-code ทำอย่างไร?
10. ข้อดีหลักของ low-code ในงาน BRMS คืออะไร?

---

## 11. Explainer for 5-year-old

ลองนึกถึง **การวาดรูป** นะ

**วาดเอง (Traditional code)**
- ต้องมีฝีมือ
- วาดได้ทุกอย่าง
- ใช้เวลานาน

**ใช้แสตมป์ (Low-code)**
- แค่กด stamp ลงกระดาษ
- เลือกรูปที่มีให้
- เร็ว ง่าย

ถ้าอยากได้รูปดาว ก็หยิบ stamp รูปดาวมากดเลย ไม่ต้องวาดเอง!

Low-code ก็เหมือนกัน แทนที่จะเขียน code เอง ก็ใช้ของที่มีให้ต่อกันเป็นโปรแกรม

---

## 12. Comparison Table

| หัวข้อ | Traditional Coding | Low-code |
|--------|-------------------|----------|
| **ใครใช้** | Developer | BA + Developer |
| **เรียนรู้** | ยาก (เดือน-ปี) | ง่าย (วัน-สัปดาห์) |
| **ความเร็ว** | ช้า | เร็ว |
| **Flexibility** | สูงมาก | ปานกลาง |
| **Maintenance** | ต้องมี developer | Business ทำเองได้บางส่วน |
| **Cost** | สูง (developer time) | ต่ำกว่า (หลัง setup) |
| **Complexity** | ทำได้ทุกอย่าง | มีข้อจำกัด |
| **Testing** | Unit tests | Visual testing + Unit tests |
| **Debugging** | IDE + debugger | Platform tools |
| **Version control** | Git | Platform versioning |

### Low-code vs No-code

| หัวข้อ | Low-code | No-code |
|--------|----------|---------|
| **Code** | บางส่วนเขียน code ได้ | ไม่เขียน code เลย |
| **กลุ่มผู้ใช้** | Developer + Business | Business only |
| **Flexibility** | ปานกลาง-สูง | ต่ำ |
| **Use cases** | Enterprise apps | Simple apps |

---

## 13. 7-Day Rapid Learning Roadmap

### สำหรับหัวข้อนี้โดยเฉพาะ (ใช้เวลา 1.5-2 ชั่วโมง)

| เวลา | กิจกรรม |
|------|---------|
| 0:00-0:20 | อ่าน Abstract และ Core Principles |
| 0:20-0:40 | ศึกษา Examples 1-3 (Visual interfaces) |
| 0:40-1:00 | ศึกษา Examples 4-5 (Flow + Custom functions) |
| 1:00-1:20 | อ่าน Playbook และ Pitfalls |
| 1:20-1:40 | ตอบ Diagnostic Questions |
| 1:40-2:00 | สรุป: เมื่อไหร่ใช้ low-code เมื่อไหร่เขียน code |

### เป้าหมายหลังจบ:
- [ ] อธิบาย low-code concept ได้
- [ ] รู้ข้อดี-ข้อเสียของ low-code
- [ ] เข้าใจบทบาทของ BA vs Developer ใน low-code
- [ ] รู้ว่าเมื่อไหร่ควรใช้ low-code
