# Use Cases ของ BRMS ในธนาคาร

## 1. Abstract (60 วินาที)

ธนาคารใช้ BRMS ในงานที่ต้องตัดสินใจตามกฎเกณฑ์ที่เปลี่ยนบ่อย เช่น อนุมัติสินเชื่อ ตรวจสอบ KYC ประเมินความเสี่ยง และคำนวณดอกเบี้ย ทุกงานเหล่านี้มีกฎมากมายที่ต้องปรับตามกฎหมาย การแข่งขัน และนโยบายธนาคาร การใช้ BRMS ทำให้เปลี่ยนกฎได้เร็ว ตรวจสอบได้ (audit) และลดความผิดพลาดจากคน

---

## 2. Core Principles (20/80)

หลัก 20% ที่ให้ผลลัพธ์ 80%:

1. **Decision Automation** - ให้ระบบตัดสินใจแทนคน
2. **Regulatory Compliance** - ทำตามกฎหมาย ธปท. และ regulator
3. **Risk Management** - ลดความเสี่ยงทางการเงิน
4. **Customer Experience** - ตอบลูกค้าได้เร็วขึ้น
5. **Operational Efficiency** - ลดงาน manual และความผิดพลาด

---

## 3. Mental Models

### Model 1: "กรรมการตัดสิน"
- BRMS = กรรมการที่ตัดสินตามกติกา
- กติกาเปลี่ยนได้ แต่กรรมการตัดสินอย่างยุติธรรมเสมอ

### Model 2: "Traffic Light System"
- เขียว (อนุมัติ) / เหลือง (ต้อง review) / แดง (ปฏิเสธ)
- กฎกำหนดว่าเมื่อไหร่เปิดไฟอะไร

### Model 3: "Recipe System"
- Input = วัตถุดิบ (ข้อมูลลูกค้า)
- Recipe = กฎ (เงื่อนไขการอนุมัติ)
- Output = ผลลัพธ์ (อนุมัติ/ปฏิเสธ)

---

## 4. Skill Tree

```
พื้นฐาน
├── รู้จัก use cases หลักในธนาคาร
├── เข้าใจ flow การอนุมัติสินเชื่อ
└── รู้จัก KYC/AML พื้นฐาน

กลาง
├── เขียน rules สำหรับ loan approval
├── สร้าง risk scoring model
├── ทำ compliance checking rules
└── ออกแบบ workflow automation

สูง
├── Optimize decision accuracy
├── Integrate multiple rule sets
├── สร้าง monitoring dashboards
└── บริหาร rule portfolio ทั้งองค์กร
```

---

## 5. What NOT to Learn

- **อย่าเรียน** ทุก banking regulations → เรียนแค่ที่เกี่ยวข้อง
- **อย่าเรียน** financial modeling ขั้นสูง → เน้น rule implementation
- **อย่าเรียน** credit scoring algorithms → เน้นการนำ model มาใช้
- **อย่าจำ** ทุก compliance requirement → เข้าใจแนวคิด
- **อย่าเรียน** banking operations ทั้งหมด → เรียนเฉพาะที่ใช้ BRMS

---

## 6. Real-world Usage

### 5 Use Cases หลักที่เจอในงาน:

#### 1. Loan Origination (อนุมัติสินเชื่อ)
- ตรวจสอบคุณสมบัติ
- คำนวณวงเงิน
- กำหนดดอกเบี้ย

#### 2. KYC/AML (ตรวจสอบตัวตน)
- ยืนยันเอกสาร
- ตรวจ blacklist
- กำหนด risk level

#### 3. Credit Scoring
- คำนวณคะแนนเครดิต
- ปรับตาม risk factors

#### 4. Fraud Detection
- ตรวจจับ pattern ผิดปกติ
- Alert เมื่อพบ suspicious activity

#### 5. Pricing/Promotion
- คำนวณดอกเบี้ย/ค่าธรรมเนียม
- Apply โปรโมชั่น

---

## 7. Typical Pitfalls (10 ข้อ)

| # | ความผิดพลาด | วิธีเลี่ยง |
|---|-------------|-----------|
| 1 | กฎไม่ cover edge cases | ทำ comprehensive testing |
| 2 | ไม่ update ตาม regulation | มี compliance review process |
| 3 | Approve ที่ควร reject | ทำ regular accuracy audit |
| 4 | กฎซับซ้อนเกินไป | Break down เป็น rule ย่อยๆ |
| 5 | ไม่มี manual override option | เตรียม exception handling |
| 6 | Performance ช้า | Optimize และ cache |
| 7 | ไม่ log decision reason | บันทึก reason ทุกครั้ง |
| 8 | Bias ใน rules | ทำ fairness testing |
| 9 | ไม่ train staff | Train ให้เข้าใจ system |
| 10 | ไม่มี disaster recovery | Prepare backup และ failover |

---

## 8. Playbook (Step-by-step)

### สร้าง Loan Approval Rule Set:

1. **Gather Requirements**
   - เก็บเงื่อนไขจาก business
   - เก็บ regulatory requirements
   - ระบุ edge cases

2. **Design Rule Structure**
   - แบ่งเป็น pre-qualification, scoring, decision
   - กำหนด input/output ของแต่ละ step

3. **Implement Rules**
   - เขียน eligibility rules
   - เขียน scoring rules
   - เขียน decision rules

4. **Test Thoroughly**
   - Test positive cases (should approve)
   - Test negative cases (should reject)
   - Test edge cases
   - Test performance

5. **Deploy และ Monitor**
   - Deploy ทีละ environment
   - Monitor approval rates
   - Track accuracy

---

## 9. Fast Practical Examples

### Example 1: Loan Eligibility Check

```python
def check_loan_eligibility(applicant):
    """
    ตรวจสอบคุณสมบัติเบื้องต้นสำหรับสินเชื่อ
    """
    rules = {
        "min_age": 20,
        "max_age": 60,
        "min_salary": 15000,
        "min_work_months": 6,
        "max_debt_ratio": 0.7
    }

    # Check age
    if applicant.age < rules["min_age"]:
        return {"eligible": False, "reason": "อายุต่ำกว่าเกณฑ์"}
    if applicant.age > rules["max_age"]:
        return {"eligible": False, "reason": "อายุเกินเกณฑ์"}

    # Check income
    if applicant.salary < rules["min_salary"]:
        return {"eligible": False, "reason": "รายได้ต่ำกว่าเกณฑ์"}

    # Check employment
    if applicant.work_months < rules["min_work_months"]:
        return {"eligible": False, "reason": "อายุงานน้อยกว่าเกณฑ์"}

    # Check debt ratio
    if applicant.debt_ratio > rules["max_debt_ratio"]:
        return {"eligible": False, "reason": "ภาระหนี้สูงเกินไป"}

    return {"eligible": True, "reason": "ผ่านเกณฑ์เบื้องต้น"}
```

### Example 2: Credit Scoring

```python
def calculate_credit_score(applicant):
    """
    คำนวณ credit score จาก factors ต่างๆ
    """
    score = 0
    max_score = 1000

    # Income score (30%)
    if applicant.salary >= 100000:
        score += 300
    elif applicant.salary >= 50000:
        score += 200
    elif applicant.salary >= 30000:
        score += 100

    # Employment score (20%)
    if applicant.work_months >= 60:
        score += 200
    elif applicant.work_months >= 24:
        score += 150
    elif applicant.work_months >= 12:
        score += 100

    # Debt ratio score (25%)
    if applicant.debt_ratio <= 0.3:
        score += 250
    elif applicant.debt_ratio <= 0.5:
        score += 150
    elif applicant.debt_ratio <= 0.7:
        score += 50

    # Payment history score (25%)
    if applicant.late_payments == 0:
        score += 250
    elif applicant.late_payments <= 2:
        score += 150
    elif applicant.late_payments <= 5:
        score += 50

    return {
        "score": score,
        "max_score": max_score,
        "grade": get_grade(score)
    }

def get_grade(score):
    if score >= 800:
        return "A"
    elif score >= 600:
        return "B"
    elif score >= 400:
        return "C"
    else:
        return "D"
```

### Example 3: KYC Document Check

```python
def verify_kyc_documents(customer, documents):
    """
    ตรวจสอบเอกสาร KYC
    """
    required_docs = {
        "THAI": ["ID_CARD", "HOUSE_REGISTRATION"],
        "FOREIGNER": ["PASSPORT", "WORK_PERMIT", "VISA"]
    }

    customer_type = "THAI" if customer.nationality == "TH" else "FOREIGNER"
    required = required_docs[customer_type]

    # Check if all required documents are submitted
    submitted = [doc.doc_type for doc in documents]
    missing = [doc for doc in required if doc not in submitted]

    if missing:
        return {
            "status": "INCOMPLETE",
            "missing_docs": missing
        }

    # Verify each document
    issues = []
    for doc in documents:
        if doc.is_expired():
            issues.append(f"{doc.doc_type} หมดอายุ")
        if not doc.is_readable():
            issues.append(f"{doc.doc_type} ไม่ชัดเจน")

    if issues:
        return {
            "status": "NEEDS_REVIEW",
            "issues": issues
        }

    return {"status": "VERIFIED"}
```

### Example 4: AML Risk Assessment

```python
def assess_aml_risk(customer, transaction):
    """
    ประเมินความเสี่ยง AML
    """
    HIGH_RISK_COUNTRIES = ["XX", "YY", "ZZ"]
    PEP_THRESHOLD = 1000000

    risk_score = 0

    # Country risk
    if customer.country in HIGH_RISK_COUNTRIES:
        risk_score += 40

    # Transaction amount risk
    if transaction.amount > 2000000:
        risk_score += 30
    elif transaction.amount > 1000000:
        risk_score += 20

    # PEP (Politically Exposed Person)
    if customer.is_pep:
        risk_score += 30

    # Cash transaction
    if transaction.is_cash:
        risk_score += 20

    # Unusual pattern
    if is_unusual_pattern(customer, transaction):
        risk_score += 30

    # Determine risk level
    if risk_score >= 70:
        return {"risk_level": "HIGH", "action": "BLOCK_AND_REPORT"}
    elif risk_score >= 40:
        return {"risk_level": "MEDIUM", "action": "ENHANCED_DUE_DILIGENCE"}
    else:
        return {"risk_level": "LOW", "action": "PROCEED"}
```

### Example 5: Interest Rate Calculation

```python
def calculate_interest_rate(loan_application):
    """
    คำนวณอัตราดอกเบี้ย
    """
    # Base rate
    base_rate = 7.0

    # Adjust by credit score
    credit_adjustments = {
        "A": -2.0,
        "B": -1.0,
        "C": 0,
        "D": +1.0
    }

    # Adjust by loan amount
    amount_adjustments = {
        (0, 500000): 0,
        (500000, 2000000): -0.25,
        (2000000, float('inf')): -0.5
    }

    # Adjust by customer type
    customer_adjustments = {
        "GOLD": -0.5,
        "SILVER": -0.25,
        "NORMAL": 0
    }

    # Calculate final rate
    rate = base_rate
    rate += credit_adjustments.get(loan_application.credit_grade, 0)

    for range_tuple, adj in amount_adjustments.items():
        if range_tuple[0] <= loan_application.amount < range_tuple[1]:
            rate += adj
            break

    rate += customer_adjustments.get(loan_application.customer_type, 0)

    # Apply promotion if any
    if loan_application.promo_code:
        promo_discount = get_promo_discount(loan_application.promo_code)
        rate -= promo_discount

    return {
        "base_rate": base_rate,
        "final_rate": max(rate, 3.0),  # Minimum rate
        "breakdown": {
            "credit_adj": credit_adjustments.get(loan_application.credit_grade, 0),
            "customer_adj": customer_adjustments.get(loan_application.customer_type, 0)
        }
    }
```

### Example 6: Fraud Detection

```python
def detect_fraud(transaction, customer_history):
    """
    ตรวจจับ fraud
    """
    alerts = []

    # Unusual amount
    avg_amount = calculate_average(customer_history)
    if transaction.amount > avg_amount * 5:
        alerts.append({
            "type": "UNUSUAL_AMOUNT",
            "severity": "HIGH",
            "detail": f"จำนวนเงินสูงกว่าปกติ 5 เท่า"
        })

    # Unusual time
    if transaction.time.hour < 6 or transaction.time.hour > 23:
        alerts.append({
            "type": "UNUSUAL_TIME",
            "severity": "MEDIUM",
            "detail": "ทำรายการนอกเวลาปกติ"
        })

    # Unusual location
    if transaction.location != customer_history.usual_location:
        alerts.append({
            "type": "UNUSUAL_LOCATION",
            "severity": "MEDIUM",
            "detail": "ทำรายการจากสถานที่ใหม่"
        })

    # Rapid succession
    recent_count = count_recent_transactions(customer_history, minutes=30)
    if recent_count > 5:
        alerts.append({
            "type": "RAPID_TRANSACTIONS",
            "severity": "HIGH",
            "detail": f"ทำรายการ {recent_count} ครั้งใน 30 นาที"
        })

    if alerts:
        max_severity = max(a["severity"] for a in alerts)
        return {
            "fraud_suspected": True,
            "alerts": alerts,
            "action": "BLOCK" if max_severity == "HIGH" else "VERIFY"
        }

    return {"fraud_suspected": False, "action": "ALLOW"}
```

---

## 10. Diagnostic Questions

ตอบคำถามเหล่านี้เพื่อทดสอบความเข้าใจ:

1. BRMS ช่วย loan origination อย่างไร?
2. KYC และ AML ต่างกันอย่างไร?
3. Credit scoring ใช้ factors อะไรบ้าง?
4. ทำไมต้อง log decision reason?
5. Fraud detection ดูจากอะไร?
6. การปรับ interest rate ขึ้นกับอะไร?
7. PEP คืออะไร ทำไมถึง high risk?
8. Manual override จำเป็นเมื่อไหร่?
9. ทำไม compliance สำคัญในธนาคาร?
10. BRMS ช่วยลด operational risk อย่างไร?

---

## 11. Explainer for 5-year-old

ลองนึกถึง **เกมเศรษฐี** นะ

- **ธนาคาร** = คนเป็นแบงก์ในเกม
- **กฎของธนาคาร** = กติกาว่าให้กู้เงินได้เท่าไหร่

ถ้าเราอยากกู้เงินจากแบงก์ แบงก์ก็ต้องดูว่า:
- เรามีเงินพอจ่ายคืนไหม? (ดูเงินเดือน)
- เราเป็นคนดีไหม? (ดูประวัติ)
- เราขอมากไปไหม? (ดูวงเงิน)

ถ้าผ่านทุกข้อ → **ได้กู้!**
ถ้าไม่ผ่าน → **ไม่ได้กู้**

BRMS ก็เหมือนกติกาในเกม ที่บอกว่าเมื่อไหร่ให้กู้ เมื่อไหร่ไม่ให้กู้

---

## 12. Comparison Table

| Use Case | Input | Output | เปลี่ยนบ่อยแค่ไหน |
|----------|-------|--------|-----------------|
| **Loan Approval** | ข้อมูลลูกค้า, เอกสาร | อนุมัติ/ปฏิเสธ/วงเงิน | เดือนละครั้ง |
| **KYC** | เอกสาร, ข้อมูล | Verified/Rejected | ตาม regulation |
| **Credit Scoring** | ประวัติการเงิน | คะแนน/Grade | ไตรมาสละครั้ง |
| **Fraud Detection** | Transaction | Alert/Block/Allow | สัปดาห์ละครั้ง |
| **Pricing** | ข้อมูลลูกค้า, Product | Interest rate | ตามตลาด |
| **AML** | Transaction, Customer | Risk level | ตาม regulation |

---

## 13. 7-Day Rapid Learning Roadmap

### สำหรับหัวข้อนี้โดยเฉพาะ (ใช้เวลา 2-3 ชั่วโมง)

| เวลา | กิจกรรม |
|------|---------|
| 0:00-0:30 | อ่าน Abstract และ Core Principles |
| 0:30-1:00 | ศึกษา Practical Examples (Loan + KYC) |
| 1:00-1:30 | ศึกษา Practical Examples (Credit Score + Fraud) |
| 1:30-2:00 | ศึกษา Practical Examples (Pricing + AML) |
| 2:00-2:30 | อ่าน Pitfalls และ Playbook |
| 2:30-3:00 | ตอบ Diagnostic Questions + สรุป |

### เป้าหมายหลังจบ:
- [ ] อธิบาย 5 use cases หลักในธนาคารได้
- [ ] เข้าใจ flow การอนุมัติสินเชื่อ
- [ ] รู้จัก KYC/AML คืออะไร
- [ ] เขียน rule แบบง่ายๆ สำหรับแต่ละ use case ได้
