# ข้อดี-ข้อเสียของ Low-code

## 1. Abstract (60 วินาที)

Low-code มีข้อดีหลักคือ เร็ว ง่าย และให้ Business Users สร้างได้เอง แต่มีข้อเสียคือ ยืดหยุ่นน้อยกว่า traditional code, performance อาจช้ากว่า และอาจ lock-in กับ vendor การเลือกใช้ต้องชั่งน้ำหนักระหว่าง speed กับ flexibility สำหรับงาน BRMS ในธนาคาร low-code เหมาะมากเพราะ rules เปลี่ยนบ่อยและต้องการ business ownership

---

## 2. Core Principles (20/80)

หลัก 20% ที่ให้ผลลัพธ์ 80%:

1. **Speed vs Flexibility Trade-off** - เร็วแต่ยืดหยุ่นน้อยกว่า
2. **Right Tool for Right Job** - ไม่ใช่ทุกงานที่เหมาะกับ low-code
3. **Total Cost of Ownership** - คิดค่าใช้จ่ายทั้งหมด
4. **Vendor Lock-in Risk** - ต้องพิจารณาความเสี่ยง
5. **Hybrid Approach** - ผสม low-code กับ traditional

---

## 3. Mental Models

### Model 1: "รถยนต์ Auto vs Manual"
- **Auto (Low-code)** = ขับง่าย แต่ควบคุมน้อยกว่า
- **Manual (Traditional)** = ยากกว่า แต่ควบคุมได้มากกว่า

### Model 2: "บ้านสำเร็จรูป vs สร้างเอง"
- **สำเร็จรูป** = เร็ว ถูก แต่เปลี่ยนแปลงยาก
- **สร้างเอง** = ออกแบบได้ทุกอย่าง แต่นาน แพง

### Model 3: "อาหารพร้อมทาน vs ทำเอง"
- **พร้อมทาน** = สะดวก เร็ว แต่ปรับรสไม่ได้
- **ทำเอง** = ปรับได้ตามใจ แต่เสียเวลา

---

## 4. Skill Tree

```
พื้นฐาน
├── รู้ข้อดีหลักของ low-code
├── รู้ข้อเสียหลักของ low-code
└── ตัดสินใจได้ว่าเมื่อไหร่ควรใช้

กลาง
├── วิเคราะห์ trade-offs สำหรับ specific use case
├── ประเมิน total cost of ownership
├── Mitigate vendor lock-in risk
└── วางแผน hybrid approach

สูง
├── กำหนด low-code strategy ขององค์กร
├── Evaluate และ select platforms
├── สร้าง governance framework
└── Measure ROI
```

---

## 5. What NOT to Learn

- **อย่าจำ** ทุก pros/cons → เข้าใจ principles
- **อย่าเรียน** TCO calculation ละเอียด → เข้าใจแนวคิด
- **อย่าเรียน** ทุก vendor comparison → focus ที่ใช้งาน
- **อย่าท่อง** statistics → เข้าใจ trends
- **อย่าเรียน** market analysis → ไม่ใช่หน้าที่ developer

---

## 6. Real-world Usage

### เมื่อไหร่ Low-code ดี vs ไม่ดี:

| Situation | Low-code? | เหตุผล |
|-----------|-----------|-------|
| Business rules เปลี่ยนบ่อย | ✅ ใช้ | เปลี่ยนได้เร็ว |
| Complex algorithm | ❌ ไม่ใช้ | ต้อง optimize เอง |
| Prototype/MVP | ✅ ใช้ | ลองไอเดียเร็ว |
| High-performance system | ❌ ไม่ใช้ | ต้อง tune เอง |
| Standard CRUD app | ✅ ใช้ | ตรงไปตรงมา |
| Unique UI/UX | ❌ ไม่ใช้ | ต้อง custom |

---

## 7. Typical Pitfalls (10 ข้อ)

| # | ความผิดพลาด | วิธีเลี่ยง |
|---|-------------|-----------|
| 1 | เลือก low-code โดยไม่ดู requirements | วิเคราะห์ requirements ก่อน |
| 2 | ไม่คิด vendor lock-in | มี exit strategy |
| 3 | ประเมิน cost ต่ำไป | คิด TCO ทั้งหมด |
| 4 | คิดว่าไม่ต้อง maintain | ต้อง maintain เหมือนกัน |
| 5 | ไม่ test performance | Load test ก่อน production |
| 6 | คิดว่า citizen dev ทำทุกอย่างได้ | ยังต้องมี developer |
| 7 | ไม่มี governance | กำหนด policies |
| 8 | ทำทุกอย่างด้วย low-code | ใช้ hybrid |
| 9 | ไม่ train users | Training สำคัญ |
| 10 | เลือกเพราะ hype | เลือกตาม fit |

---

## 8. Playbook (Step-by-step)

### ตัดสินใจว่าควรใช้ Low-code หรือไม่:

#### Step 1: วิเคราะห์ Requirements
- Business rules เปลี่ยนบ่อยไหม?
- ใครต้องการ access แก้ไข?
- Performance requirement เป็นอย่างไร?

#### Step 2: ประเมิน Pros/Cons
สร้างตารางเปรียบเทียบสำหรับ specific use case

#### Step 3: คำนวณ TCO
- Initial cost (license, setup)
- Ongoing cost (maintenance, training)
- Hidden cost (integration, migration)

#### Step 4: พิจารณา Risks
- Vendor lock-in
- Security
- Scalability

#### Step 5: ตัดสินใจ
- Full low-code
- Hybrid
- Traditional code

---

## 9. Fast Practical Examples

### Example 1: Pros และ Cons Table

```markdown
## ข้อดีของ Low-code

| ข้อดี | รายละเอียด | Impact ในงาน BRMS |
|------|-----------|------------------|
| **Speed** | สร้างได้เร็วกว่า 5-10 เท่า | Rules deploy ได้ภายในชั่วโมง |
| **Accessibility** | Business users ใช้งานได้ | BA สร้าง rules เองได้ |
| **Maintenance** | แก้ไขง่าย | ลด developer workload |
| **Visualization** | เห็นภาพ logic | ง่ายต่อการ review |
| **Testing** | Test ใน platform | ทดสอบได้ทันที |
| **Documentation** | Auto-generated | Audit trail อัตโนมัติ |
| **Consistency** | Standard patterns | ลด errors |
| **Collaboration** | BA + Dev ทำงานร่วมกัน | Communication ดีขึ้น |

## ข้อเสียของ Low-code

| ข้อเสีย | รายละเอียด | Mitigation |
|--------|-----------|-----------|
| **Limited flexibility** | ทำได้แค่ที่ platform support | ใช้ custom code เสริม |
| **Performance** | อาจช้ากว่า native code | Optimize critical paths |
| **Vendor lock-in** | ยากที่จะย้าย platform | วางแผน exit strategy |
| **Security concerns** | อาจมี vulnerabilities | Regular security audits |
| **Scalability limits** | อาจ scale ได้จำกัด | Design for scale |
| **Learning curve** | Platform-specific | Training program |
| **Hidden costs** | License, training, integration | Full TCO analysis |
| **Over-simplification** | อาจทำให้ logic ง่ายเกินไป | Code review process |
```

### Example 2: TCO Comparison

```python
# Total Cost of Ownership Analysis

def calculate_tco_lowcode():
    """TCO สำหรับ Low-code BRMS"""
    return {
        # Initial costs
        "platform_license": 500000,  # ต่อปี
        "setup_consulting": 300000,
        "training": 100000,

        # Ongoing costs (ต่อปี)
        "maintenance": 100000,
        "support": 50000,

        # Productivity gains
        "time_to_deploy": "2-4 hours",  # vs 2-4 weeks
        "ba_productivity": "+50%",

        # 3-year TCO
        "total_3_year": 2050000
    }

def calculate_tco_traditional():
    """TCO สำหรับ Traditional Development"""
    return {
        # Initial costs
        "development": 800000,  # Custom BRMS
        "infrastructure": 200000,

        # Ongoing costs (ต่อปี)
        "developer_maintenance": 400000,  # 1 FTE
        "infrastructure": 100000,

        # Productivity
        "time_to_deploy": "2-4 weeks",
        "ba_dependency": "100% on developers",

        # 3-year TCO
        "total_3_year": 2500000
    }

# Low-code ถูกกว่าในระยะยาว
# และ time-to-market เร็วกว่ามาก
```

### Example 3: Decision Matrix

```python
def should_use_lowcode(requirements):
    """
    ตัดสินใจว่าควรใช้ low-code หรือไม่

    Returns:
        "LOW_CODE" | "TRADITIONAL" | "HYBRID"
    """
    score = 0

    # Factors favoring low-code (+)
    if requirements.get("change_frequency") == "HIGH":
        score += 3
    if requirements.get("business_ownership") == True:
        score += 3
    if requirements.get("time_to_market") == "URGENT":
        score += 2
    if requirements.get("standard_patterns") == True:
        score += 2

    # Factors against low-code (-)
    if requirements.get("custom_ui") == True:
        score -= 2
    if requirements.get("high_performance") == True:
        score -= 3
    if requirements.get("complex_algorithms") == True:
        score -= 3
    if requirements.get("unique_integration") == True:
        score -= 2

    # Decision
    if score >= 4:
        return "LOW_CODE"
    elif score <= -2:
        return "TRADITIONAL"
    else:
        return "HYBRID"

# Example usage
requirements = {
    "change_frequency": "HIGH",
    "business_ownership": True,
    "time_to_market": "URGENT",
    "standard_patterns": True,
    "custom_ui": False,
    "high_performance": False,
    "complex_algorithms": False,
    "unique_integration": False
}

decision = should_use_lowcode(requirements)
print(f"Recommendation: {decision}")  # "LOW_CODE"
```

### Example 4: Hybrid Approach

```python
"""
ตัวอย่าง Hybrid Approach:
- Low-code: Rule management, UI
- Traditional: Complex calculations, Integrations
"""

# Low-code Platform - Rule Editor
"""
┌─────────────────────────────────┐
│  Rule: Calculate Interest Rate  │
│  ─────────────────────────────  │
│  IF credit_grade == 'A'         │
│  THEN call: calculate_rate_a()  │
│                                 │
│  IF credit_grade == 'B'         │
│  THEN call: calculate_rate_b()  │
│                                 │
│  DEFAULT: call: default_rate()  │
└─────────────────────────────────┘
"""

# Traditional Code - Complex Calculations
def calculate_rate_a(customer, market_data):
    """
    Complex calculation ที่ต้องเขียน code
    - ใช้ market data realtime
    - Complex math formulas
    - Performance optimization
    """
    base_rate = market_data.mlr
    spread = calculate_spread(customer)
    adjustments = get_adjustments(customer)

    return optimize_rate(base_rate - spread + adjustments)

def calculate_spread(customer):
    """Complex spread calculation"""
    # Monte Carlo simulation
    # Risk modeling
    # etc.
    pass

# Integration - Custom Code
class ExternalCreditBureau:
    """Integration ที่ต้องเขียน code เอง"""

    def get_credit_score(self, customer_id):
        # API call to external service
        # Error handling
        # Retry logic
        pass
```

### Example 5: Risk Assessment

```python
def assess_lowcode_risks():
    """ประเมินความเสี่ยงของการใช้ Low-code"""

    risks = {
        "vendor_lock_in": {
            "probability": "HIGH",
            "impact": "HIGH",
            "mitigation": [
                "Document rules in standard format",
                "Keep export capability",
                "Negotiate contract terms",
                "Design for portability"
            ]
        },

        "performance_issues": {
            "probability": "MEDIUM",
            "impact": "MEDIUM",
            "mitigation": [
                "Load testing",
                "Performance monitoring",
                "Optimize critical paths",
                "Use custom code where needed"
            ]
        },

        "security_vulnerabilities": {
            "probability": "MEDIUM",
            "impact": "HIGH",
            "mitigation": [
                "Regular security audits",
                "Access controls",
                "Encryption",
                "Compliance checks"
            ]
        },

        "skill_dependency": {
            "probability": "MEDIUM",
            "impact": "MEDIUM",
            "mitigation": [
                "Cross-training",
                "Documentation",
                "Standard practices",
                "Knowledge sharing"
            ]
        },

        "scalability_limits": {
            "probability": "LOW",
            "impact": "HIGH",
            "mitigation": [
                "Capacity planning",
                "Architecture review",
                "Hybrid approach",
                "Scale testing"
            ]
        }
    }

    return risks
```

### Example 6: Real-world Comparison

```markdown
## Case Study: Loan Approval System

### Scenario A: 100% Traditional Code

Timeline: 3 เดือน
Team: 3 developers
Costs:
- Development: 900,000 บาท
- Maintenance: 300,000 บาท/ปี
- Rule changes: 2-4 สัปดาห์

Pros:
- Full control
- Best performance
- No vendor dependency

Cons:
- Slow to change
- Requires developers for everything
- Higher maintenance cost


### Scenario B: 100% Low-code

Timeline: 1 เดือน
Team: 1 developer + 2 BAs
Costs:
- License: 500,000 บาท/ปี
- Setup: 300,000 บาท
- Rule changes: 2-4 ชั่วโมง

Pros:
- Very fast deployment
- BA can maintain
- Visual audit trail

Cons:
- Vendor lock-in
- Limited customization
- Platform constraints


### Scenario C: Hybrid (Recommended)

Timeline: 2 เดือน
Team: 2 developers + 2 BAs
Costs:
- License: 500,000 บาท/ปี
- Development: 400,000 บาท
- Maintenance: 200,000 บาท/ปี
- Rule changes: ชั่วโมง-วัน (แล้วแต่ complexity)

Pros:
- Balanced approach
- Best of both worlds
- Flexibility where needed

Cons:
- More complex architecture
- Needs clear boundaries
```

---

## 10. Diagnostic Questions

ตอบคำถามเหล่านี้เพื่อทดสอบความเข้าใจ:

1. ข้อดีหลักของ low-code คืออะไร?
2. ข้อเสียหลักของ low-code คืออะไร?
3. Vendor lock-in คืออะไร และป้องกันอย่างไร?
4. เมื่อไหร่ควรใช้ low-code? เมื่อไหร่ไม่ควร?
5. TCO ต้องคิดอะไรบ้าง?
6. Hybrid approach คืออะไร?
7. ทำไม low-code ถึงเหมาะกับ BRMS?
8. Performance ของ low-code ต่างจาก traditional อย่างไร?
9. Training สำคัญอย่างไรใน low-code?
10. ความเสี่ยงหลักของ low-code คืออะไร?

---

## 11. Explainer for 5-year-old

ลองนึกถึง **การสร้างบ้านเลโก้** นะ

**ข้อดี:**
- ต่อง่าย เร็ว
- ไม่ต้องมีฝีมือมาก
- เปลี่ยนได้เลย

**ข้อเสีย:**
- ต้องใช้ชิ้นส่วนที่มีให้
- อาจไม่แข็งแรงเท่าบ้านจริง
- ถ้าเลโก้หมด ก็ต้องซื้อเพิ่ม

เหมือน low-code เลย: ง่าย เร็ว แต่ทำได้แค่ที่เครื่องมือให้มา!

---

## 12. Comparison Table

### ข้อดี vs ข้อเสีย Summary

| ด้าน | ข้อดี | ข้อเสีย |
|-----|------|--------|
| **Speed** | 5-10x เร็วกว่า | - |
| **Cost** | ถูกกว่าในระยะยาว | Initial cost สูง |
| **Users** | Business users ใช้ได้ | ต้อง train |
| **Flexibility** | - | จำกัดกว่า code |
| **Performance** | - | อาจช้ากว่า |
| **Vendor** | Support ดี | Lock-in risk |
| **Maintenance** | ง่ายกว่า | ต้องพึ่ง vendor |
| **Scalability** | Platform handles | มีขีดจำกัด |

### เมื่อไหร่ใช้อะไร

| Use Case | Recommendation |
|----------|---------------|
| BRMS / Business Rules | Low-code |
| Simple CRUD apps | Low-code |
| Prototype / MVP | Low-code |
| Complex algorithms | Traditional |
| High-performance | Traditional |
| Custom integrations | Hybrid |
| Enterprise apps | Hybrid |

---

## 13. 7-Day Rapid Learning Roadmap

### สำหรับหัวข้อนี้โดยเฉพาะ (ใช้เวลา 1-1.5 ชั่วโมง)

| เวลา | กิจกรรม |
|------|---------|
| 0:00-0:15 | อ่าน Abstract และ Core Principles |
| 0:15-0:30 | ศึกษา Example 1 (Pros/Cons Table) |
| 0:30-0:45 | ศึกษา Example 2-3 (TCO, Decision) |
| 0:45-1:00 | ศึกษา Example 4-5 (Hybrid, Risks) |
| 1:00-1:15 | อ่าน Comparison Tables |
| 1:15-1:30 | ตอบ Diagnostic Questions |

### เป้าหมายหลังจบ:
- [ ] บอกข้อดี-ข้อเสียหลักของ low-code ได้
- [ ] ตัดสินใจได้ว่าเมื่อไหร่ควรใช้
- [ ] เข้าใจ vendor lock-in และ mitigation
- [ ] รู้จัก hybrid approach
