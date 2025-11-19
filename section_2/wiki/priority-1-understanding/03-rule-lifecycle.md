# วงจรชีวิตของ Business Rules (Rule Lifecycle)

## 1. Abstract (60 วินาที)

Rule Lifecycle คือขั้นตอนตั้งแต่กฎถูกคิดขึ้นมาจนถึงถูกปลดระวาง มี 6 ขั้นตอนหลัก: Design → Develop → Test → Deploy → Monitor → Retire การเข้าใจ lifecycle ช่วยให้คุณรู้ว่าแต่ละขั้นตอนต้องทำอะไร ใครรับผิดชอบ และต้อง document อะไรบ้าง ทุกกฎต้องผ่านทุกขั้นตอน ไม่มีทางลัด

---

## 2. Core Principles (20/80)

หลัก 20% ที่ให้ผลลัพธ์ 80%:

1. **Traceability** - ทุกกฎต้องติดตามได้ว่ามาจากไหน ใครสร้าง
2. **Version Control** - ทุกการเปลี่ยนแปลงต้องมี version
3. **Testing at Every Stage** - test ทุกขั้นตอน ไม่ใช่แค่ตอนท้าย
4. **Approval Workflow** - ต้องมีคนอนุมัติก่อน deploy
5. **Continuous Monitoring** - ติดตามผลหลัง deploy ตลอด

---

## 3. Mental Models

### Model 1: "ชีวิตของกฎหมาย"
- **ร่างกฎหมาย** = Design
- **พิจารณาในสภา** = Review & Test
- **ประกาศในราชกิจจา** = Deploy
- **บังคับใช้** = Active
- **ยกเลิก** = Retire

### Model 2: "Product Development"
- เหมือน product ทั่วไป: idea → prototype → test → launch → sunset
- กฎก็เป็น "product" ที่ต้องผ่านทุกขั้นตอน

### Model 3: "วงจรชีวิตซอฟต์แวร์ (SDLC)"
- Requirement → Design → Develop → Test → Deploy → Maintain
- Rule lifecycle คล้าย SDLC แต่ cycle เร็วกว่ามาก

---

## 4. Skill Tree

```
พื้นฐาน
├── รู้จัก 6 ขั้นตอนหลักของ lifecycle
├── เข้าใจว่าใครรับผิดชอบแต่ละขั้น
└── รู้ว่าต้อง document อะไรบ้าง

กลาง
├── สร้าง approval workflow
├── ทำ version control อย่างเป็นระบบ
├── สร้าง test cases ที่ครอบคลุม
└── ทำ impact analysis ก่อนเปลี่ยน rule

สูง
├── ออกแบบ governance framework
├── Automate lifecycle processes
├── วิเคราะห์ rule effectiveness
└── บริหาร rule portfolio
```

---

## 5. What NOT to Learn

- **อย่าเรียน** ทุก workflow tool → แค่เข้าใจแนวคิด
- **อย่าเรียน** complex approval hierarchies → เรียนแค่ที่ใช้ในองค์กร
- **อย่าเรียน** legal compliance frameworks ทั้งหมด → เรียนเฉพาะธนาคาร
- **อย่าจำ** ทุก document template → เข้าใจว่าต้อง document อะไร
- **อย่าเรียน** enterprise governance ขั้นสูง → เน้น practical workflow

---

## 6. Real-world Usage

### สถานการณ์ที่เจอบ่อย:

1. **กฎใหม่จาก regulator**
   - Design: BA รับ requirement จาก compliance
   - Develop: Developer เขียน rule
   - Test: QA + BA verify
   - Deploy: ต้องทันก่อน deadline
   - Monitor: ตรวจสอบว่าทำงานถูกต้อง

2. **แก้ไขกฎที่มีปัญหา**
   - รับ bug report
   - Analyze root cause
   - Create new version
   - Test regression
   - Deploy hotfix

3. **ยกเลิกโปรโมชั่น**
   - กำหนด expiry date
   - Retire rule เมื่อหมดเวลา
   - Archive สำหรับ audit

---

## 7. Typical Pitfalls (10 ข้อ)

| # | ความผิดพลาด | วิธีเลี่ยง |
|---|-------------|-----------|
| 1 | ข้าม testing phase | บังคับให้ test ทุก rule |
| 2 | ไม่ทำ version control | ใช้ระบบ versioning อัตโนมัติ |
| 3 | Deploy โดยไม่ได้ approve | มี approval gate บังคับ |
| 4 | ไม่มี rollback plan | เตรียม rollback ทุกครั้ง |
| 5 | ลืม retire rule เก่า | ตั้ง expiry date หรือ review schedule |
| 6 | ไม่ document การเปลี่ยนแปลง | บังคับ change log |
| 7 | ไม่ทำ impact analysis | วิเคราะห์ dependency ก่อนแก้ |
| 8 | ไม่ monitor หลัง deploy | ตั้ง alert และ dashboard |
| 9 | ให้คนเดียว control ทุกอย่าง | Separation of duties |
| 10 | ไม่มี staging environment | ต้องมี Dev → Test → Prod |

---

## 8. Playbook (Step-by-step)

### Rule Lifecycle แบบละเอียด:

#### Phase 1: Design (1-2 วัน)
1. รับ business requirement
2. วิเคราะห์ว่าเป็น rule ประเภทไหน
3. ออกแบบ rule logic (pseudocode/flowchart)
4. Review กับ stakeholders
5. สร้าง test cases

#### Phase 2: Develop (1-2 วัน)
1. สร้าง rule ใน BRMS platform
2. เขียน unit tests
3. Self-review code
4. Commit to version control
5. Request peer review

#### Phase 3: Test (1-2 วัน)
1. Run unit tests
2. Integration testing
3. UAT กับ BA
4. Performance testing (ถ้าจำเป็น)
5. Sign-off จาก QA

#### Phase 4: Deploy (ไม่กี่ชั่วโมง)
1. Request deployment approval
2. Deploy to staging
3. Smoke test
4. Deploy to production
5. Verify production deployment

#### Phase 5: Monitor (ตลอดเวลา)
1. ตรวจสอบ logs
2. Monitor performance metrics
3. รับ feedback จาก users
4. Track business KPIs
5. Report issues

#### Phase 6: Retire (เมื่อจำเป็น)
1. ระบุ rule ที่ต้อง retire
2. แจ้ง stakeholders
3. Archive rule และ documentation
4. Deactivate rule
5. Verify ไม่มี dependency ค้าง

---

## 9. Fast Practical Examples

### Example 1: Rule Version History

```python
rule_history = {
    "rule_id": "LOAN_APPROVAL_001",
    "versions": [
        {
            "version": "1.0",
            "created_by": "john_ba",
            "created_date": "2024-01-15",
            "status": "RETIRED",
            "description": "Initial version"
        },
        {
            "version": "1.1",
            "created_by": "jane_ba",
            "created_date": "2024-03-20",
            "status": "RETIRED",
            "description": "Updated salary threshold"
        },
        {
            "version": "2.0",
            "created_by": "john_ba",
            "created_date": "2024-06-01",
            "status": "ACTIVE",
            "description": "Added credit score criteria"
        }
    ]
}
```

### Example 2: Approval Workflow

```python
class RuleApprovalWorkflow:
    STATES = ["DRAFT", "REVIEW", "APPROVED", "DEPLOYED", "ACTIVE", "RETIRED"]

    def submit_for_review(self, rule):
        if rule.status == "DRAFT":
            rule.status = "REVIEW"
            self.notify_reviewers(rule)

    def approve(self, rule, approver):
        if rule.status == "REVIEW" and approver.can_approve:
            rule.status = "APPROVED"
            rule.approved_by = approver
            rule.approved_date = datetime.now()

    def deploy(self, rule, environment):
        if rule.status == "APPROVED":
            # Deploy logic
            rule.status = "DEPLOYED"
            rule.deployed_to = environment
```

### Example 3: Test Case Structure

```python
test_cases = [
    {
        "test_id": "TC001",
        "rule_id": "LOAN_APPROVAL_001",
        "description": "High income customer should be approved",
        "input": {
            "salary": 100000,
            "debt_ratio": 0.2,
            "credit_score": 750
        },
        "expected_output": "APPROVED",
        "actual_output": None,
        "status": "PENDING"
    },
    {
        "test_id": "TC002",
        "rule_id": "LOAN_APPROVAL_001",
        "description": "Low credit score should be rejected",
        "input": {
            "salary": 50000,
            "debt_ratio": 0.3,
            "credit_score": 500
        },
        "expected_output": "REJECTED",
        "actual_output": None,
        "status": "PENDING"
    }
]
```

### Example 4: Deployment Record

```python
deployment_record = {
    "deployment_id": "DEP-2024-0615",
    "rule_id": "LOAN_APPROVAL_001",
    "version": "2.0",
    "environment": "PRODUCTION",
    "deployed_by": "deploy_bot",
    "deployed_date": "2024-06-15T10:30:00",
    "approval": {
        "approved_by": "manager_jane",
        "approved_date": "2024-06-15T09:00:00"
    },
    "rollback_version": "1.1",
    "status": "SUCCESS"
}
```

### Example 5: Monitoring Dashboard Data

```python
rule_monitoring = {
    "rule_id": "LOAN_APPROVAL_001",
    "period": "2024-06-01 to 2024-06-30",
    "metrics": {
        "total_executions": 15420,
        "avg_execution_time_ms": 45,
        "outcomes": {
            "APPROVED": 8500,
            "REJECTED": 4200,
            "REVIEW_REQUIRED": 2720
        },
        "errors": 0,
        "exceptions": 3
    }
}
```

---

## 10. Diagnostic Questions

ตอบคำถามเหล่านี้เพื่อทดสอบความเข้าใจ:

1. Rule lifecycle มีกี่ขั้นตอนหลัก อะไรบ้าง?
2. ทำไมต้องมี approval workflow?
3. Version control ของ rules ต่างจาก code อย่างไร?
4. เมื่อไหร่ควร retire rule?
5. Impact analysis ทำเพื่ออะไร?
6. ถ้า deploy แล้วเจอ bug ควรทำอย่างไร?
7. ใครควรเป็นคน approve rule changes?
8. Test cases ต้องครอบคลุมอะไรบ้าง?
9. Monitor metrics อะไรบ้างหลัง deploy?
10. Archive rule ต้องเก็บข้อมูลอะไรบ้าง?

---

## 11. Explainer for 5-year-old

ลองนึกถึง **กติกาเล่นเกมในห้องเรียน** นะ

1. **คิดกติกา** (Design) - ครูคิดว่าเล่นยังไงดี
2. **เขียนลงกระดาษ** (Develop) - เขียนกติกาให้ชัดเจน
3. **ลองเล่น** (Test) - ลองเล่นดูก่อนว่าสนุกไหม ยุติธรรมไหม
4. **ประกาศใช้** (Deploy) - บอกนักเรียนทุกคนว่ากติกาคืออะไร
5. **ดูว่าโอเคไหม** (Monitor) - ดูว่าทุกคนเข้าใจไหม มีปัญหาไหม
6. **เลิกใช้** (Retire) - ถ้ากติกาไม่ดี ก็เปลี่ยนกติกาใหม่

ทุกกติกาต้องผ่านทุกขั้นตอน ไม่งั้นเล่นแล้วจะมีปัญหา!

---

## 12. Comparison Table

| ขั้นตอน | ใครทำ | ทำอะไร | Output |
|---------|-------|--------|--------|
| **Design** | BA + Business | กำหนด logic | Requirement doc, Test cases |
| **Develop** | Developer | เขียน rule | Rule code, Unit tests |
| **Test** | QA + BA | ทดสอบ | Test results, Sign-off |
| **Deploy** | DevOps/Developer | ติดตั้ง | Deployment record |
| **Monitor** | Operations | ติดตาม | Metrics, Alerts |
| **Retire** | BA + Developer | ปลดระวาง | Archive, Audit trail |

---

## 13. 7-Day Rapid Learning Roadmap

### สำหรับหัวข้อนี้โดยเฉพาะ (ใช้เวลา 1.5-2 ชั่วโมง)

| เวลา | กิจกรรม |
|------|---------|
| 0:00-0:20 | อ่าน Abstract และ Core Principles |
| 0:20-0:40 | ศึกษา Playbook ทั้ง 6 phases |
| 0:40-1:00 | ดู Practical Examples |
| 1:00-1:20 | อ่าน Pitfalls และจำไว้ |
| 1:20-1:40 | ตอบ Diagnostic Questions |
| 1:40-2:00 | วาด diagram lifecycle ด้วยตัวเอง |

### เป้าหมายหลังจบ:
- [ ] อธิบาย 6 ขั้นตอนของ lifecycle ได้
- [ ] รู้ว่าใครรับผิดชอบแต่ละขั้น
- [ ] เข้าใจความสำคัญของ version control
- [ ] บอกได้ว่าต้อง document อะไรบ้าง
