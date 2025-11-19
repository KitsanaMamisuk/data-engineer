# การทำงานร่วมกันระหว่าง Business Analyst กับ Developer

## 1. Abstract (60 วินาที)

ในงาน BRMS การทำงานร่วมกันระหว่าง BA (Business Analyst) และ Developer เป็นหัวใจสำคัญ BA เป็นคนที่เข้าใจ business requirements และแปลงเป็น rules ขณะที่ Developer เป็นคนที่ช่วยเรื่อง technical implementation และ complex logic ทั้งสองต้องสื่อสารกันอย่างมีประสิทธิภาพ โดย BA พูดภาษา business และ Developer พูดภาษา technical การมี shared understanding และ clear handoff process คือกุญแจสู่ความสำเร็จ

---

## 2. Core Principles (20/80)

หลัก 20% ที่ให้ผลลัพธ์ 80%:

1. **Shared Language** - สร้างภาษากลางที่ทั้งสองเข้าใจ
2. **Clear Boundaries** - กำหนดว่าใครทำอะไร
3. **Frequent Communication** - คุยกันบ่อยๆ
4. **Mutual Respect** - เคารพความเชี่ยวชาญของแต่ละฝ่าย
5. **Iterative Process** - ทำทีละขั้น review บ่อย

---

## 3. Mental Models

### Model 1: "สถาปนิก + ช่างก่อสร้าง"
- **BA** = สถาปนิก (ออกแบบตามที่ลูกค้าต้องการ)
- **Developer** = ช่างก่อสร้าง (สร้างจริงตามแบบ)
- ต้องคุยกันเพื่อให้แบบเป็นจริงได้

### Model 2: "หมอ + เภสัชกร"
- **BA** = หมอ (วินิจฉัยปัญหา กำหนดวิธีรักษา)
- **Developer** = เภสัชกร (จัดยาตาม prescription)
- ต้องสื่อสารให้ชัดเจน

### Model 3: "ทีมฟุตบอล"
- **BA** = โค้ช (วาง strategy)
- **Developer** = นักเตะ (ลงสนามจริง)
- ต้องเข้าใจแผนเดียวกัน

---

## 4. Skill Tree

```
พื้นฐาน
├── เข้าใจบทบาทของ BA และ Developer
├── สื่อสารกับคนต่าง background ได้
└── ใช้ tools สำหรับ collaboration

กลาง
├── แปล business → technical และกลับกัน
├── Review และให้ feedback ได้ดี
├── Resolve conflicts
└── Document สำหรับทั้งสองฝั่ง

สูง
├── Mentor ทีม BA/Dev
├── สร้าง collaboration framework
├── Optimize processes
└── สร้าง shared knowledge base
```

---

## 5. What NOT to Learn

- **อย่าเรียน** business domain ลึกเกินไป (ถ้าเป็น dev) → รู้พอสื่อสารได้
- **อย่าเรียน** technical details ลึก (ถ้าเป็น BA) → รู้พอเข้าใจข้อจำกัด
- **อย่าเรียน** ทุก collaboration tools → ใช้ที่ทีมใช้
- **อย่าเรียน** project management ลึก → รู้พื้นฐาน
- **อย่าเรียน** formal communication theory → ฝึกจากการทำจริง

---

## 6. Real-world Usage

### Workflow ทั่วไป:

1. **BA รับ requirement** จาก business
2. **BA วิเคราะห์** และสร้าง rule specifications
3. **BA + Dev review** specifications ด้วยกัน
4. **Dev implement** หรือ support BA ใน complex rules
5. **BA test** และ verify ว่าถูกต้องตาม requirement
6. **BA + Dev review** final rules
7. **Deploy** และ monitor

---

## 7. Typical Pitfalls (10 ข้อ)

| # | ความผิดพลาด | วิธีเลี่ยง |
|---|-------------|-----------|
| 1 | ไม่คุยกันจน deploy | คุยทุก stage |
| 2 | BA assume technical feasibility | ถาม dev ก่อน |
| 3 | Dev assume business intent | ถาม BA ให้ชัด |
| 4 | ใช้ศัพท์ที่อีกฝ่ายไม่เข้าใจ | ใช้ shared language |
| 5 | ไม่ document decisions | เขียนทุกการตัดสินใจ |
| 6 | Blame เมื่อเกิดปัญหา | Focus ที่การแก้ปัญหา |
| 7 | ทำงาน silo | มี regular sync |
| 8 | ไม่เปิดใจรับ feedback | สร้าง safe space |
| 9 | ข้าม review process | บังคับ review |
| 10 | ไม่แชร์ context | Share background info |

---

## 8. Playbook (Step-by-step)

### Collaboration Workflow:

```
┌─────────────────────────────────────────────────┐
│         BA-Developer Collaboration Flow         │
└─────────────────────────────────────────────────┘

Phase 1: Requirement Gathering
┌─────────┐
│   BA    │ → รับ requirement จาก business
└────┬────┘
     │
     ▼
Phase 2: Initial Analysis
┌─────────┐
│   BA    │ → วิเคราะห์และสร้าง draft specification
└────┬────┘
     │
     ▼
Phase 3: Technical Feasibility Review
┌─────────────┐
│  BA + Dev   │ → Review ร่วมกัน
└──────┬──────┘    - Technical feasibility
       │           - Performance concerns
       │           - Integration points
       ▼
Phase 4: Specification Refinement
┌─────────┐
│   BA    │ → ปรับ spec ตาม feedback
└────┬────┘
     │
     ▼
Phase 5: Implementation
┌─────────┐     ┌─────────┐
│   BA    │ or  │   Dev   │ → สร้าง rules
└────┬────┘     └────┬────┘
     │               │
     └───────┬───────┘
             ▼
Phase 6: Review
┌─────────────┐
│  BA + Dev   │ → Review rules ด้วยกัน
└──────┬──────┘
       │
       ▼
Phase 7: Testing
┌─────────┐
│   BA    │ → Test ตาม business scenarios
└────┬────┘
     │
     ▼
Phase 8: Deployment
┌─────────────┐
│  BA + Dev   │ → Deploy และ monitor
└─────────────┘
```

---

## 9. Fast Practical Examples

### Example 1: Communication Template

```markdown
# Rule Specification Document

## Rule Overview
- **Rule ID**: LOAN_APPROVAL_001
- **Rule Name**: Basic Loan Eligibility
- **Owner**: John (BA)
- **Developer**: Jane (Dev)
- **Status**: Draft

## Business Requirement
ลูกค้าที่มีเงินเดือนตั้งแต่ 30,000 บาทขึ้นไป และอายุระหว่าง 20-60 ปี
สามารถยื่นขอสินเชื่อส่วนบุคคลได้

## Rule Logic (Business Language)
IF:
- เงินเดือน >= 30,000 บาท
- อายุ >= 20 ปี
- อายุ <= 60 ปี
THEN:
- ผ่านเกณฑ์เบื้องต้น
ELSE:
- ไม่ผ่านเกณฑ์

## Technical Notes (Dev fills in)
- **Data Source**: customer_profile table
- **Fields**: salary (decimal), age (int)
- **Performance Notes**: Index on salary and age
- **Edge Cases**: null values → reject

## Test Cases
| Scenario | salary | age | Expected |
|----------|--------|-----|----------|
| Pass | 30000 | 25 | Eligible |
| Fail salary | 25000 | 30 | Not eligible |
| Fail age low | 40000 | 18 | Not eligible |
| Fail age high | 50000 | 65 | Not eligible |

## Sign-offs
- [ ] BA approved: ___________
- [ ] Dev approved: ___________
- [ ] Test completed: ___________
```

### Example 2: Handoff Checklist

```python
handoff_checklist = {
    "ba_to_dev": {
        "before_handoff": [
            "Requirements document complete",
            "Test cases defined",
            "Edge cases identified",
            "Business rules clear and unambiguous",
            "Expected data format specified",
            "Performance requirements stated"
        ],
        "during_handoff": [
            "Walk through requirements together",
            "Discuss technical concerns",
            "Clarify any ambiguities",
            "Agree on timeline",
            "Define communication plan"
        ]
    },

    "dev_to_ba": {
        "before_handoff": [
            "Implementation complete",
            "Unit tests passing",
            "Code/rules documented",
            "Technical limitations documented",
            "Test data prepared"
        ],
        "during_handoff": [
            "Demo the implementation",
            "Explain any deviations from spec",
            "Walk through test scenarios",
            "Discuss deployment plan"
        ]
    }
}
```

### Example 3: Daily Standup Format

```python
# Daily Standup สำหรับ BA-Dev Team

standup_template = {
    "format": "Round-robin, 2 min per person",
    "questions": [
        "What did you complete yesterday?",
        "What are you working on today?",
        "Any blockers or need help?",
        "Any updates others should know?"
    ],

    "example_ba": {
        "yesterday": "Completed spec for KYC validation rules",
        "today": "Review session with Dev at 2pm, start credit scoring spec",
        "blockers": "Waiting for clarification from compliance team",
        "updates": "New requirement coming for fraud detection"
    },

    "example_dev": {
        "yesterday": "Implemented loan eligibility rules",
        "today": "Write unit tests, prepare for review with BA",
        "blockers": "Need test data for edge cases",
        "updates": "Found potential performance issue, will discuss in review"
    }
}
```

### Example 4: Feedback Session Template

```python
# Template สำหรับ Review/Feedback Session

feedback_session = {
    "setup": {
        "duration": "30-60 minutes",
        "attendees": ["BA", "Developer", "Optional: Tech Lead"],
        "materials": ["Specifications", "Implemented rules", "Test results"]
    },

    "agenda": [
        {
            "item": "Overview",
            "duration": "5 min",
            "owner": "BA",
            "activities": ["Recap requirements", "State objectives"]
        },
        {
            "item": "Demo",
            "duration": "10 min",
            "owner": "Dev",
            "activities": ["Walk through implementation", "Show test results"]
        },
        {
            "item": "Review",
            "duration": "20 min",
            "owner": "Both",
            "activities": [
                "Check against requirements",
                "Identify gaps",
                "Discuss edge cases"
            ]
        },
        {
            "item": "Feedback",
            "duration": "10 min",
            "owner": "Both",
            "activities": [
                "What went well",
                "What could improve",
                "Action items"
            ]
        },
        {
            "item": "Next Steps",
            "duration": "5 min",
            "owner": "Both",
            "activities": ["Define actions", "Set deadlines", "Schedule follow-up"]
        }
    ],

    "output": {
        "meeting_notes": "Document decisions and action items",
        "updated_spec": "If changes needed",
        "action_items": "Assigned with owners and deadlines"
    }
}
```

### Example 5: Shared Glossary

```python
# Shared Glossary - ภาษากลางระหว่าง BA และ Dev

shared_glossary = {
    # Business Terms → Technical Mapping
    "เงินเดือน": {
        "technical_term": "salary",
        "data_type": "decimal(12,2)",
        "source": "customer_profile.monthly_salary",
        "notes": "Base salary only, excluding OT and bonuses"
    },

    "อายุ": {
        "technical_term": "age",
        "data_type": "int",
        "source": "calculated from customer_profile.birth_date",
        "notes": "Age as of application date"
    },

    "อัตราส่วนหนี้ต่อรายได้": {
        "technical_term": "debt_to_income_ratio / DTI",
        "data_type": "decimal(5,4)",
        "formula": "total_monthly_debt / monthly_income",
        "notes": "0.5 = 50%"
    },

    "ผู้มีอำนาจทางการเมือง": {
        "technical_term": "PEP (Politically Exposed Person)",
        "data_type": "boolean",
        "source": "kyc_profile.is_pep",
        "notes": "From KYC check"
    },

    "คะแนนเครดิต": {
        "technical_term": "credit_score",
        "data_type": "int",
        "source": "credit_bureau.score",
        "notes": "Range 300-850"
    }
}
```

### Example 6: Conflict Resolution Process

```python
# Process สำหรับ resolve conflicts

conflict_resolution = {
    "common_conflicts": [
        {
            "type": "Technical Feasibility",
            "example": "BA wants feature ที่ทำได้ยากหรือช้า",
            "resolution": [
                "Dev อธิบาย technical constraints",
                "BA ระบุ business priority",
                "หาทาง compromise",
                "Escalate ถ้าไม่ได้"
            ]
        },
        {
            "type": "Scope Creep",
            "example": "Requirements เพิ่มขึ้นเรื่อยๆ",
            "resolution": [
                "Review original scope",
                "Prioritize additions",
                "Negotiate timeline",
                "Document changes"
            ]
        },
        {
            "type": "Interpretation Differences",
            "example": "เข้าใจ requirement ไม่ตรงกัน",
            "resolution": [
                "กลับไปที่ source (business)",
                "สร้าง examples ให้เห็นชัด",
                "Write down และ confirm",
                "Update documentation"
            ]
        }
    ],

    "escalation_path": [
        "Try to resolve between BA and Dev",
        "Involve Tech Lead or BA Lead",
        "Involve Project Manager",
        "Involve Business Stakeholder"
    ],

    "principles": [
        "Focus on problem, not person",
        "Data-driven decisions",
        "Respect each other's expertise",
        "Document resolutions"
    ]
}
```

### Example 7: RACI Matrix

```python
# RACI Matrix สำหรับ BRMS Development

raci_matrix = {
    "activities": {
        "Gather Requirements": {
            "BA": "R",  # Responsible
            "Dev": "C",  # Consulted
            "Business": "A",  # Accountable
            "QA": "I"  # Informed
        },
        "Write Rule Specification": {
            "BA": "R",
            "Dev": "C",
            "Business": "A",
            "QA": "I"
        },
        "Technical Review": {
            "BA": "C",
            "Dev": "R",
            "Tech Lead": "A",
            "QA": "I"
        },
        "Implement Rules": {
            "BA": "C",
            "Dev": "R",
            "Tech Lead": "A",
            "QA": "I"
        },
        "Create Test Cases": {
            "BA": "R",
            "Dev": "C",
            "Business": "C",
            "QA": "A"
        },
        "Execute Testing": {
            "BA": "A",
            "Dev": "C",
            "QA": "R"
        },
        "Deploy to Production": {
            "BA": "C",
            "Dev": "R",
            "Tech Lead": "A",
            "Ops": "R"
        }
    },

    "legend": {
        "R": "Responsible - Does the work",
        "A": "Accountable - Final authority",
        "C": "Consulted - Input required",
        "I": "Informed - Kept updated"
    }
}
```

---

## 10. Diagnostic Questions

ตอบคำถามเหล่านี้เพื่อทดสอบความเข้าใจ:

1. บทบาทหลักของ BA และ Developer ต่างกันอย่างไร?
2. ทำไม shared language ถึงสำคัญ?
3. ขั้นตอนไหนที่ BA และ Dev ต้องทำงานร่วมกัน?
4. ถ้าเกิด conflict ระหว่าง BA และ Dev ควรทำอย่างไร?
5. Handoff checklist ช่วยอะไร?
6. RACI คืออะไร และใช้ทำไม?
7. ทำไมต้อง document decisions?
8. Daily standup ช่วย collaboration อย่างไร?
9. เมื่อไหร่ควร escalate issue?
10. ทำไม Developer ต้องเข้าใจ business context?

---

## 11. Explainer for 5-year-old

ลองนึกถึง **การสร้างบ้านตุ๊กตา** นะ

- **BA** = คนที่ถามว่า "อยากได้บ้านแบบไหน?"
  - ห้องนอนกี่ห้อง?
  - สีอะไร?
  - มีสวนไหม?

- **Developer** = คนที่สร้างบ้านจริง
  - ตัดกระดาษ
  - ประกอบ
  - ทาสี

ทั้งสองคนต้อง **คุยกัน** ถ้า BA อยากได้บ้าน 10 ชั้น แต่มีแค่กระดาษนิดเดียว Developer ก็ต้องบอกว่า "ทำไม่ได้นะ" แล้วหาทางออกด้วยกัน

---

## 12. Comparison Table

### บทบาทของ BA vs Developer

| หัวข้อ | BA | Developer |
|--------|-------|-----------|
| **Focus** | Business needs | Technical implementation |
| **Language** | Business terms | Technical terms |
| **Output** | Specifications | Working rules |
| **Skills** | Analysis, Communication | Coding, Problem-solving |
| **Questions** | "What?" "Why?" | "How?" |
| **Metrics** | Business value | Performance, Quality |

### Collaboration Tools

| Tool | Purpose | When to Use |
|------|---------|-------------|
| **Jira/Trello** | Task tracking | Throughout project |
| **Confluence/Wiki** | Documentation | Specs, decisions |
| **Slack/Teams** | Quick communication | Daily |
| **Zoom/Meet** | Meetings | Reviews, discussions |
| **Git** | Version control | Rule changes |
| **Shared drive** | File sharing | Documents, test data |

---

## 13. 7-Day Rapid Learning Roadmap

### สำหรับหัวข้อนี้โดยเฉพาะ (ใช้เวลา 1.5-2 ชั่วโมง)

| เวลา | กิจกรรม |
|------|---------|
| 0:00-0:20 | อ่าน Abstract และ Core Principles |
| 0:20-0:40 | ศึกษา Playbook (Collaboration Flow) |
| 0:40-1:00 | ศึกษา Example 1-3 (Templates) |
| 1:00-1:20 | ศึกษา Example 4-7 (Processes) |
| 1:20-1:40 | อ่าน Pitfalls และ Comparison Tables |
| 1:40-2:00 | ตอบ Diagnostic Questions |

### เป้าหมายหลังจบ:
- [ ] เข้าใจบทบาทของ BA และ Developer
- [ ] รู้ว่าต้องสื่อสารกันอย่างไร
- [ ] มี template สำหรับ documentation
- [ ] รู้วิธี resolve conflicts
