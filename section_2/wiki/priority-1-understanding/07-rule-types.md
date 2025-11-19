# ประเภทของ Rules (Decision Rules, Validation Rules, Calculation Rules)

## 1. Abstract (60 วินาที)

Business Rules แบ่งออกเป็น 3 ประเภทหลัก: **Decision Rules** (ตัดสินใจ เช่น อนุมัติ/ปฏิเสธ), **Validation Rules** (ตรวจสอบความถูกต้อง เช่น format ถูกไหม), และ **Calculation Rules** (คำนวณค่า เช่น ดอกเบี้ย, คะแนน) แต่ละประเภทมีวัตถุประสงค์และวิธีเขียนที่ต่างกัน การรู้ว่า rule ที่กำลังเขียนเป็นประเภทไหนช่วยให้ออกแบบได้ถูกต้อง

---

## 2. Core Principles (20/80)

หลัก 20% ที่ให้ผลลัพธ์ 80%:

1. **Know Your Rule Type** - ระบุประเภทก่อนเขียนเสมอ
2. **Different Output** - Decision → choice, Validation → pass/fail, Calculation → number
3. **Composability** - หลาย rules รวมกันได้
4. **Order Matters** - Validation → Calculation → Decision
5. **Single Purpose** - 1 rule ทำ 1 อย่าง

---

## 3. Mental Models

### Model 1: "กระบวนการพิจารณาคดี"
- **Validation** = ตรวจหลักฐาน (ถูกต้องครบไหม)
- **Calculation** = คำนวณค่าเสียหาย (เท่าไหร่)
- **Decision** = ตัดสินคดี (ผิด/ไม่ผิด)

### Model 2: "การสอบ"
- **Validation** = ตรวจว่าเขียนถูก format
- **Calculation** = คำนวณคะแนน
- **Decision** = ตัดเกรด (A/B/C/D/F)

### Model 3: "ร้านอาหาร"
- **Validation** = เช็ควัตถุดิบครบไหม
- **Calculation** = คำนวณราคา
- **Decision** = รับออเดอร์หรือไม่

---

## 4. Skill Tree

```
พื้นฐาน
├── แยกแยะ 3 ประเภทได้
├── เขียน rule แต่ละประเภทได้
└── รู้ว่าเมื่อไหร่ใช้ประเภทไหน

กลาง
├── ผสม rules หลายประเภท
├── สร้าง rule chains
├── จัดลำดับ rules
└── Handle dependencies

สูง
├── ออกแบบ complex rule flows
├── Optimize rule execution
├── สร้าง reusable rule templates
└── บริหาร rule portfolios
```

---

## 5. What NOT to Learn

- **อย่าเรียน** ทุก rule classification systems → แค่รู้ 3 ประเภทหลัก
- **อย่าเรียน** formal rule theory → เน้น practical
- **อย่าเรียน** complex calculation algorithms → เน้นการ implement
- **อย่าท่อง** ทุก use case → เข้าใจ pattern
- **อย่าเรียน** academic rule taxonomies → ไม่จำเป็น

---

## 6. Real-world Usage

### เมื่อไหร่ใช้ประเภทไหน:

| Situation | Rule Type | ตัวอย่าง |
|-----------|-----------|---------|
| ตัดสินใจ Yes/No | Decision | อนุมัติสินเชื่อ |
| เลือกจากหลายตัวเลือก | Decision | จัด risk level |
| ตรวจ format | Validation | เบอร์โทร ถูก format |
| ตรวจ completeness | Validation | เอกสารครบไหม |
| ตรวจ constraints | Validation | อายุอยู่ในเกณฑ์ไหม |
| คำนวณตัวเลข | Calculation | คำนวณดอกเบี้ย |
| รวมค่า | Calculation | คำนวณ total score |
| แปลงค่า | Calculation | แปลงสกุลเงิน |

---

## 7. Typical Pitfalls (10 ข้อ)

| # | ความผิดพลาด | วิธีเลี่ยง |
|---|-------------|-----------|
| 1 | ผสม validation กับ decision | แยกเป็นคนละ rule |
| 2 | Calculate ก่อน validate | Validate ก่อนเสมอ |
| 3 | Decision rule ไม่ cover ทุก case | มี default case |
| 4 | Calculation precision error | ใช้ Decimal แทน float |
| 5 | Validation message กำกวม | บอกชัดว่าผิดตรงไหน |
| 6 | Chain rules ผิดลำดับ | กำหนด dependency ชัด |
| 7 | Calculation ซ้ำซ้อน | Cache ผลลัพธ์ |
| 8 | Validation ไม่ครบ | List ทุก constraint |
| 9 | Decision criteria ไม่ชัด | กำหนด threshold ชัดเจน |
| 10 | ไม่ test edge cases | Test boundary values |

---

## 8. Playbook (Step-by-step)

### กระบวนการทำงานทั่วไป:

```
Input Data
    ↓
[Validation Rules] → ถ้า fail → Return errors
    ↓ (ถ้า pass)
[Calculation Rules] → คำนวณค่าที่จำเป็น
    ↓
[Decision Rules] → ตัดสินใจ
    ↓
Output Result
```

### สร้าง Rule Flow สำหรับ Loan:

1. **Validation Phase**
   - ตรวจ format ข้อมูล
   - ตรวจเอกสารครบ
   - ตรวจ constraints

2. **Calculation Phase**
   - คำนวณ debt ratio
   - คำนวณ credit score
   - คำนวณ max amount

3. **Decision Phase**
   - ตัดสินใจ approve/reject
   - กำหนด interest rate
   - กำหนด conditions

---

## 9. Fast Practical Examples

### Example 1: Validation Rules

```python
class ValidationRules:
    """
    ตรวจสอบความถูกต้องของข้อมูล
    Output: valid (True/False) + errors
    """

    @staticmethod
    def validate_phone(phone):
        """ตรวจสอบ format เบอร์โทร"""
        import re
        pattern = r'^0[689]\d{8}$'
        if not re.match(pattern, phone):
            return {
                "valid": False,
                "error": "รูปแบบเบอร์โทรไม่ถูกต้อง (ต้องขึ้นต้นด้วย 06, 08, 09)"
            }
        return {"valid": True}

    @staticmethod
    def validate_age(age, min_age=20, max_age=60):
        """ตรวจสอบอายุอยู่ในเกณฑ์"""
        if age < min_age:
            return {
                "valid": False,
                "error": f"อายุต้องมากกว่าหรือเท่ากับ {min_age} ปี"
            }
        if age > max_age:
            return {
                "valid": False,
                "error": f"อายุต้องน้อยกว่าหรือเท่ากับ {max_age} ปี"
            }
        return {"valid": True}

    @staticmethod
    def validate_id_card(id_card):
        """ตรวจสอบรหัสบัตรประชาชน"""
        if len(id_card) != 13:
            return {
                "valid": False,
                "error": "รหัสบัตรประชาชนต้องมี 13 หลัก"
            }
        if not id_card.isdigit():
            return {
                "valid": False,
                "error": "รหัสบัตรประชาชนต้องเป็นตัวเลขเท่านั้น"
            }
        # Checksum validation
        if not validate_id_checksum(id_card):
            return {
                "valid": False,
                "error": "รหัสบัตรประชาชนไม่ถูกต้อง"
            }
        return {"valid": True}

    @staticmethod
    def validate_required_documents(submitted, required):
        """ตรวจสอบเอกสารครบถ้วน"""
        missing = [doc for doc in required if doc not in submitted]
        if missing:
            return {
                "valid": False,
                "error": f"ขาดเอกสาร: {', '.join(missing)}"
            }
        return {"valid": True}
```

### Example 2: Calculation Rules

```python
class CalculationRules:
    """
    คำนวณค่าต่างๆ
    Output: ตัวเลขหรือ object ที่มีค่าคำนวณ
    """

    @staticmethod
    def calculate_debt_ratio(monthly_debt, monthly_income):
        """คำนวณอัตราส่วนหนี้ต่อรายได้"""
        if monthly_income <= 0:
            return {"error": "รายได้ต้องมากกว่า 0"}

        ratio = monthly_debt / monthly_income
        return {
            "debt_ratio": round(ratio, 4),
            "percentage": f"{ratio * 100:.2f}%"
        }

    @staticmethod
    def calculate_credit_score(factors):
        """คำนวณคะแนนเครดิต"""
        score = 0

        # Payment history (35%)
        score += factors.get("payment_history_score", 0) * 0.35

        # Credit utilization (30%)
        score += factors.get("utilization_score", 0) * 0.30

        # Credit history length (15%)
        score += factors.get("history_length_score", 0) * 0.15

        # Credit mix (10%)
        score += factors.get("credit_mix_score", 0) * 0.10

        # New credit (10%)
        score += factors.get("new_credit_score", 0) * 0.10

        return {
            "score": round(score),
            "max_score": 850,
            "breakdown": {
                "payment_history": factors.get("payment_history_score", 0) * 0.35,
                "utilization": factors.get("utilization_score", 0) * 0.30,
                "history_length": factors.get("history_length_score", 0) * 0.15,
                "credit_mix": factors.get("credit_mix_score", 0) * 0.10,
                "new_credit": factors.get("new_credit_score", 0) * 0.10
            }
        }

    @staticmethod
    def calculate_max_loan_amount(monthly_income, debt_ratio, multiplier=15):
        """คำนวณวงเงินสินเชื่อสูงสุด"""
        available_income = monthly_income * (1 - debt_ratio)
        max_amount = available_income * multiplier

        # Round down to nearest 10,000
        max_amount = (max_amount // 10000) * 10000

        return {
            "max_amount": max_amount,
            "monthly_payment_capacity": available_income
        }

    @staticmethod
    def calculate_interest_rate(base_rate, adjustments):
        """คำนวณอัตราดอกเบี้ย"""
        rate = base_rate

        for adj in adjustments:
            rate += adj.get("value", 0)

        # Ensure minimum rate
        rate = max(rate, 3.0)

        return {
            "final_rate": round(rate, 2),
            "base_rate": base_rate,
            "adjustments": adjustments
        }

    @staticmethod
    def calculate_monthly_payment(principal, annual_rate, months):
        """คำนวณค่างวดรายเดือน"""
        monthly_rate = annual_rate / 100 / 12

        if monthly_rate == 0:
            payment = principal / months
        else:
            payment = principal * (monthly_rate * (1 + monthly_rate)**months) / \
                     ((1 + monthly_rate)**months - 1)

        return {
            "monthly_payment": round(payment, 2),
            "total_payment": round(payment * months, 2),
            "total_interest": round(payment * months - principal, 2)
        }
```

### Example 3: Decision Rules

```python
class DecisionRules:
    """
    ตัดสินใจ
    Output: decision + reason/conditions
    """

    @staticmethod
    def decide_loan_approval(applicant, calculated_values):
        """ตัดสินใจอนุมัติสินเชื่อ"""

        # Check eligibility criteria
        if calculated_values["debt_ratio"] > 0.5:
            return {
                "decision": "REJECTED",
                "reason": "อัตราส่วนหนี้ต่อรายได้เกิน 50%"
            }

        credit_score = calculated_values["credit_score"]

        if credit_score >= 750:
            return {
                "decision": "APPROVED",
                "reason": "Credit score ดีเยี่ยม",
                "conditions": []
            }
        elif credit_score >= 650:
            return {
                "decision": "APPROVED",
                "reason": "Credit score ปานกลาง-ดี",
                "conditions": ["ต้องมีผู้ค้ำประกัน"]
            }
        elif credit_score >= 550:
            return {
                "decision": "REVIEW_REQUIRED",
                "reason": "Credit score ต่ำ ต้องพิจารณาเพิ่มเติม",
                "conditions": ["ต้องผ่านการอนุมัติจากผู้จัดการ"]
            }
        else:
            return {
                "decision": "REJECTED",
                "reason": "Credit score ต่ำเกินไป"
            }

    @staticmethod
    def decide_risk_level(factors):
        """ตัดสินใจระดับความเสี่ยง"""
        risk_score = sum(factors.values())

        if risk_score >= 70:
            return {
                "decision": "HIGH",
                "action": "ENHANCED_DUE_DILIGENCE",
                "score": risk_score
            }
        elif risk_score >= 40:
            return {
                "decision": "MEDIUM",
                "action": "STANDARD_REVIEW",
                "score": risk_score
            }
        else:
            return {
                "decision": "LOW",
                "action": "AUTO_APPROVE",
                "score": risk_score
            }

    @staticmethod
    def decide_customer_tier(customer):
        """ตัดสินใจระดับลูกค้า"""
        total_balance = customer.get("total_balance", 0)
        tenure_months = customer.get("tenure_months", 0)
        products = customer.get("num_products", 0)

        # Gold tier
        if total_balance >= 5000000 or \
           (total_balance >= 1000000 and products >= 3):
            return {
                "decision": "GOLD",
                "benefits": ["ดอกเบี้ยพิเศษ", "ค่าธรรมเนียมฟรี", "Priority service"]
            }

        # Silver tier
        if total_balance >= 500000 or \
           (tenure_months >= 24 and products >= 2):
            return {
                "decision": "SILVER",
                "benefits": ["ส่วนลดค่าธรรมเนียม", "Online priority"]
            }

        # Normal tier
        return {
            "decision": "NORMAL",
            "benefits": ["สิทธิประโยชน์พื้นฐาน"]
        }

    @staticmethod
    def decide_interest_rate_tier(credit_grade, customer_type):
        """ตัดสินใจระดับอัตราดอกเบี้ย"""
        rate_matrix = {
            ("A", "GOLD"): 4.5,
            ("A", "SILVER"): 5.0,
            ("A", "NORMAL"): 5.5,
            ("B", "GOLD"): 5.5,
            ("B", "SILVER"): 6.0,
            ("B", "NORMAL"): 6.5,
            ("C", "GOLD"): 6.5,
            ("C", "SILVER"): 7.0,
            ("C", "NORMAL"): 7.5,
        }

        key = (credit_grade, customer_type)
        rate = rate_matrix.get(key, 8.0)  # Default rate

        return {
            "decision": rate,
            "credit_grade": credit_grade,
            "customer_type": customer_type
        }
```

### Example 4: Combined Rule Flow

```python
class LoanProcessingRules:
    """
    รวม rules ทุกประเภทเข้าด้วยกัน
    """

    def __init__(self):
        self.validation = ValidationRules()
        self.calculation = CalculationRules()
        self.decision = DecisionRules()

    def process_loan_application(self, application):
        """
        Flow: Validation → Calculation → Decision
        """

        # Step 1: Validation
        validations = [
            self.validation.validate_age(application.age),
            self.validation.validate_phone(application.phone),
            self.validation.validate_id_card(application.id_card),
        ]

        errors = [v["error"] for v in validations if not v["valid"]]
        if errors:
            return {
                "status": "VALIDATION_FAILED",
                "errors": errors
            }

        # Step 2: Calculation
        debt_ratio = self.calculation.calculate_debt_ratio(
            application.monthly_debt,
            application.monthly_income
        )

        credit_score = self.calculation.calculate_credit_score(
            application.credit_factors
        )

        max_amount = self.calculation.calculate_max_loan_amount(
            application.monthly_income,
            debt_ratio["debt_ratio"]
        )

        calculated = {
            "debt_ratio": debt_ratio["debt_ratio"],
            "credit_score": credit_score["score"],
            "max_amount": max_amount["max_amount"]
        }

        # Step 3: Decision
        decision = self.decision.decide_loan_approval(
            application,
            calculated
        )

        # Calculate final terms if approved
        if decision["decision"] == "APPROVED":
            interest_rate = self.decision.decide_interest_rate_tier(
                self.get_credit_grade(credit_score["score"]),
                application.customer_type
            )

            approved_amount = min(application.requested_amount, max_amount["max_amount"])

            monthly_payment = self.calculation.calculate_monthly_payment(
                approved_amount,
                interest_rate["decision"],
                application.loan_term_months
            )

            return {
                "status": "APPROVED",
                "decision": decision,
                "terms": {
                    "approved_amount": approved_amount,
                    "interest_rate": interest_rate["decision"],
                    "monthly_payment": monthly_payment["monthly_payment"],
                    "total_payment": monthly_payment["total_payment"]
                },
                "calculations": calculated
            }

        return {
            "status": decision["decision"],
            "decision": decision,
            "calculations": calculated
        }

    @staticmethod
    def get_credit_grade(score):
        if score >= 750:
            return "A"
        elif score >= 650:
            return "B"
        else:
            return "C"
```

---

## 10. Diagnostic Questions

ตอบคำถามเหล่านี้เพื่อทดสอบความเข้าใจ:

1. Business Rule มีกี่ประเภทหลัก อะไรบ้าง?
2. Validation Rule ต่างจาก Decision Rule อย่างไร?
3. ทำไมต้อง validate ก่อน calculate?
4. Output ของ Calculation Rule เป็นอะไร?
5. Decision Rule ที่ดีต้องมีอะไรบ้าง?
6. เมื่อไหร่ควรแยก rule ออกเป็นหลายตัว?
7. Rule chain คืออะไร?
8. Default case สำคัญอย่างไรใน Decision Rule?
9. Validation error message ที่ดีเป็นอย่างไร?
10. ทำไม calculation precision ถึงสำคัญ?

---

## 11. Explainer for 5-year-old

ลองนึกถึง **การซื้อไอศกรีม** นะ

1. **Validation** = ตรวจเงิน
   - "เงินเป็นเงินจริงไหม?" ✓
   - "เงินมีพอไหม?" ✓

2. **Calculation** = คำนวณราคา
   - "ไอศกรีม 1 ถ้วย = 20 บาท"
   - "ท็อปปิ้งเพิ่ม = 5 บาท"
   - "รวม = 25 บาท"

3. **Decision** = ตัดสินใจ
   - "ขายให้ได้ไหม?" → **ได้!**
   - "ทอนเท่าไหร่?" → **75 บาท**

ทุกครั้งที่ซื้อของ ก็ทำ 3 อย่างนี้เสมอ!

---

## 12. Comparison Table

| หัวข้อ | Validation | Calculation | Decision |
|--------|------------|-------------|----------|
| **วัตถุประสงค์** | ตรวจสอบความถูกต้อง | คำนวณค่า | ตัดสินใจ |
| **Output** | True/False + error | Number/Value | Choice + reason |
| **ตัวอย่าง** | Format ถูกไหม | คำนวณดอกเบี้ย | อนุมัติ/ปฏิเสธ |
| **เมื่อใช้** | เริ่มต้น process | หลัง validate | หลัง calculate |
| **Fail behavior** | หยุด process | ส่ง error | เลือก default |

### Flow Diagram

```
┌─────────────┐
│   Input     │
└──────┬──────┘
       ↓
┌─────────────┐     ┌─────────┐
│ Validation  │────→│  Error  │
│   Rules     │ fail└─────────┘
└──────┬──────┘
       ↓ pass
┌─────────────┐
│ Calculation │
│   Rules     │
└──────┬──────┘
       ↓
┌─────────────┐
│  Decision   │
│   Rules     │
└──────┬──────┘
       ↓
┌─────────────┐
│   Output    │
└─────────────┘
```

---

## 13. 7-Day Rapid Learning Roadmap

### สำหรับหัวข้อนี้โดยเฉพาะ (ใช้เวลา 2-2.5 ชั่วโมง)

| เวลา | กิจกรรม |
|------|---------|
| 0:00-0:20 | อ่าน Abstract และ Core Principles |
| 0:20-0:45 | ศึกษา Example 1: Validation Rules |
| 0:45-1:10 | ศึกษา Example 2: Calculation Rules |
| 1:10-1:35 | ศึกษา Example 3: Decision Rules |
| 1:35-2:00 | ศึกษา Example 4: Combined Flow |
| 2:00-2:20 | อ่าน Pitfalls และ Comparison |
| 2:20-2:30 | ตอบ Diagnostic Questions |

### เป้าหมายหลังจบ:
- [ ] แยกแยะ 3 ประเภทของ rules ได้ทันที
- [ ] เขียน rule แต่ละประเภทได้
- [ ] รู้ว่าควรใช้ประเภทไหนเมื่อไหร่
- [ ] ต่อ rules เข้าด้วยกันเป็น flow ได้
