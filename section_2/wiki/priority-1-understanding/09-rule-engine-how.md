# Rule Engine ทำงานอย่างไร

## 1. Abstract (60 วินาที)

Rule Engine คือซอฟต์แวร์ที่รับ input data แล้ว evaluate กับ rules ที่กำหนดไว้ เพื่อให้ได้ output/decision การทำงานมี 3 ขั้นตอน: 1) Load rules 2) Match rules กับ data 3) Execute actions ของ rules ที่ match Rule Engine ช่วยให้แยก business logic ออกจาก application code ทำให้เปลี่ยนกฎได้โดยไม่ต้องแก้ code หลัก

---

## 2. Core Principles (20/80)

หลัก 20% ที่ให้ผลลัพธ์ 80%:

1. **Pattern Matching** - จับคู่ data กับ conditions ใน rules
2. **Conflict Resolution** - ตัดสินว่าใช้ rule ไหนเมื่อหลาย rules match
3. **Forward Chaining** - evaluate rules ไปเรื่อยๆ จนไม่มี rule match
4. **Working Memory** - เก็บ data ที่กำลัง evaluate
5. **Inference** - สรุปผลลัพธ์จาก rules

---

## 3. Mental Models

### Model 1: "ระบบกรองน้ำ"
- น้ำ (data) ไหลผ่านตัวกรอง (rules)
- ตัวกรองแต่ละตัวจับสิ่งที่ตรงกับเกณฑ์
- น้ำที่ออกมา = ผลลัพธ์

### Model 2: "ด่านตรวจ"
- รถ (data) ผ่านด่าน (rules)
- แต่ละด่านตรวจสอบคนละอย่าง
- รถที่ผ่านทุกด่าน = อนุมัติ

### Model 3: "ต้นไม้ตัดสินใจ"
- เริ่มจากราก ถามคำถาม
- คำตอบนำไปสู่กิ่งต่างๆ
- ปลายกิ่ง = decision

---

## 4. Skill Tree

```
พื้นฐาน
├── เข้าใจ basic flow ของ rule engine
├── รู้จัก pattern matching
└── เข้าใจ rule execution order

กลาง
├── จัดการ conflict resolution
├── เข้าใจ forward vs backward chaining
├── Optimize rule performance
└── Debug rule execution

สูง
├── ออกแบบ custom rule engine
├── Implement complex algorithms
├── Scale rule engine
└── Integrate กับ external systems
```

---

## 5. What NOT to Learn

- **อย่าเรียน** ทุก rule engine algorithms → เรียนแค่ที่ใช้
- **อย่าเรียน** Rete algorithm ลึก → เข้าใจแนวคิดพอ
- **อย่าเรียน** การสร้าง rule engine ตั้งแต่ศูนย์ → ใช้ที่มีอยู่
- **อย่าเรียน** backward chaining ถ้าไม่ได้ใช้
- **อย่าท่อง** internal implementation → เข้าใจ behavior

---

## 6. Real-world Usage

### สถานการณ์ที่เจอ:

1. **Evaluate loan application**
   - Input: ข้อมูลผู้สมัคร
   - Rules: เงื่อนไขการอนุมัติ
   - Output: approve/reject + เหตุผล

2. **Calculate pricing**
   - Input: ข้อมูลลูกค้า + product
   - Rules: pricing rules
   - Output: ราคา + ส่วนลด

3. **Detect fraud**
   - Input: transaction
   - Rules: fraud patterns
   - Output: alert/allow

---

## 7. Typical Pitfalls (10 ข้อ)

| # | ความผิดพลาด | วิธีเลี่ยง |
|---|-------------|-----------|
| 1 | Rules ขัดแย้งกัน | กำหนด priority ชัดเจน |
| 2 | Rule execution order ผิด | กำหนด salience/priority |
| 3 | Infinite loop | ระวัง rules ที่ trigger กันเอง |
| 4 | Performance ช้า | Optimize rules, ใช้ caching |
| 5 | Memory leak | จัดการ working memory |
| 6 | ไม่ test ทุก scenarios | สร้าง comprehensive test cases |
| 7 | ไม่ log rule execution | Enable logging สำหรับ debug |
| 8 | Rules ซับซ้อนเกินไป | Break down เป็น rules ย่อย |
| 9 | ไม่ handle null/missing data | เขียน null checks |
| 10 | ไม่มี default case | เตรียม fallback behavior |

---

## 8. Playbook (Step-by-step)

### การทำงานของ Rule Engine:

```
┌─────────────────────────────────────────┐
│           Rule Engine Flow              │
└─────────────────────────────────────────┘

Step 1: Load Rules
┌─────────────┐
│ Rule Store  │ → Load rules into memory
│ (Database)  │
└─────────────┘

Step 2: Insert Data
┌─────────────┐
│ Input Data  │ → Insert into Working Memory
└─────────────┘

Step 3: Pattern Matching
┌─────────────┐
│   Match     │ → Find rules where conditions match data
│   Rules     │
└─────────────┘

Step 4: Conflict Resolution
┌─────────────┐
│  Resolve    │ → If multiple rules match, decide which to fire
│  Conflicts  │
└─────────────┘

Step 5: Execute Actions
┌─────────────┐
│  Execute    │ → Run actions of selected rules
│  Actions    │
└─────────────┘

Step 6: Return Results
┌─────────────┐
│  Results    │ → Return output to caller
└─────────────┘
```

---

## 9. Fast Practical Examples

### Example 1: Simple Rule Engine

```python
class SimpleRuleEngine:
    def __init__(self):
        self.rules = []

    def add_rule(self, rule):
        """เพิ่ม rule เข้า engine"""
        self.rules.append(rule)

    def evaluate(self, data):
        """Evaluate data กับทุก rules"""
        results = []

        for rule in self.rules:
            if self.match(rule['condition'], data):
                result = self.execute(rule['action'], data)
                results.append({
                    'rule_id': rule['id'],
                    'result': result
                })

        return results

    def match(self, condition, data):
        """Check if condition matches data"""
        field = condition['field']
        operator = condition['operator']
        value = condition['value']

        data_value = data.get(field)

        if operator == '>=':
            return data_value >= value
        elif operator == '<=':
            return data_value <= value
        elif operator == '==':
            return data_value == value
        elif operator == '>':
            return data_value > value
        elif operator == '<':
            return data_value < value

        return False

    def execute(self, action, data):
        """Execute rule action"""
        return action

# Usage
engine = SimpleRuleEngine()

# Add rules
engine.add_rule({
    'id': 'RULE_001',
    'condition': {'field': 'salary', 'operator': '>=', 'value': 30000},
    'action': {'eligible': True}
})

engine.add_rule({
    'id': 'RULE_002',
    'condition': {'field': 'salary', 'operator': '<', 'value': 30000},
    'action': {'eligible': False, 'reason': 'รายได้ไม่ผ่านเกณฑ์'}
})

# Evaluate
data = {'salary': 35000, 'age': 25}
results = engine.evaluate(data)
print(results)
```

### Example 2: Rule Engine with Priority

```python
class PriorityRuleEngine:
    def __init__(self):
        self.rules = []

    def add_rule(self, rule, priority=0):
        """เพิ่ม rule พร้อม priority"""
        rule['priority'] = priority
        self.rules.append(rule)
        # Sort by priority (สูง = execute ก่อน)
        self.rules.sort(key=lambda x: x['priority'], reverse=True)

    def evaluate(self, data, stop_on_first_match=False):
        """
        Evaluate data กับ rules
        stop_on_first_match: หยุดเมื่อ match rule แรก
        """
        results = []

        for rule in self.rules:
            if self.match(rule, data):
                result = self.execute(rule, data)
                results.append(result)

                if stop_on_first_match:
                    break

        return results

    def match(self, rule, data):
        """Check all conditions"""
        conditions = rule.get('conditions', [])

        for condition in conditions:
            if not self.check_condition(condition, data):
                return False

        return True

    def check_condition(self, condition, data):
        """Check single condition"""
        field = condition['field']
        operator = condition['operator']
        value = condition['value']
        data_value = data.get(field)

        operators = {
            '>=': lambda a, b: a >= b,
            '<=': lambda a, b: a <= b,
            '>': lambda a, b: a > b,
            '<': lambda a, b: a < b,
            '==': lambda a, b: a == b,
            '!=': lambda a, b: a != b,
            'in': lambda a, b: a in b,
        }

        op_func = operators.get(operator)
        if op_func:
            return op_func(data_value, value)
        return False

    def execute(self, rule, data):
        """Execute rule action"""
        return {
            'rule_id': rule['id'],
            'rule_name': rule.get('name', ''),
            'priority': rule['priority'],
            'output': rule['action']
        }

# Usage
engine = PriorityRuleEngine()

# High priority rule - VIP customer
engine.add_rule({
    'id': 'VIP_APPROVAL',
    'name': 'VIP Auto Approval',
    'conditions': [
        {'field': 'customer_type', 'operator': '==', 'value': 'VIP'}
    ],
    'action': {'decision': 'APPROVED', 'reason': 'VIP customer'}
}, priority=100)

# Normal rule
engine.add_rule({
    'id': 'NORMAL_APPROVAL',
    'name': 'Standard Approval',
    'conditions': [
        {'field': 'salary', 'operator': '>=', 'value': 30000},
        {'field': 'age', 'operator': '>=', 'value': 20}
    ],
    'action': {'decision': 'APPROVED', 'reason': 'ผ่านเกณฑ์'}
}, priority=50)

# Default rule (low priority)
engine.add_rule({
    'id': 'DEFAULT_REJECT',
    'name': 'Default Rejection',
    'conditions': [],  # Match all
    'action': {'decision': 'REJECTED', 'reason': 'ไม่ผ่านเกณฑ์'}
}, priority=0)

# Evaluate
data = {'salary': 25000, 'age': 25, 'customer_type': 'NORMAL'}
result = engine.evaluate(data, stop_on_first_match=True)
print(result)
```

### Example 3: Rule Engine with Compound Conditions

```python
class AdvancedRuleEngine:
    def evaluate_condition(self, condition, data):
        """Evaluate compound conditions (AND/OR)"""

        # Simple condition
        if 'field' in condition:
            return self.check_simple_condition(condition, data)

        # AND condition
        if 'all' in condition:
            return all(
                self.evaluate_condition(c, data)
                for c in condition['all']
            )

        # OR condition
        if 'any' in condition:
            return any(
                self.evaluate_condition(c, data)
                for c in condition['any']
            )

        # NOT condition
        if 'not' in condition:
            return not self.evaluate_condition(condition['not'], data)

        return False

    def check_simple_condition(self, condition, data):
        field = condition['field']
        operator = condition['operator']
        value = condition['value']

        # Support nested fields (e.g., "customer.age")
        data_value = self.get_nested_value(data, field)

        if data_value is None:
            return False

        return self.compare(data_value, operator, value)

    def get_nested_value(self, data, field):
        """Get value from nested dict using dot notation"""
        keys = field.split('.')
        value = data
        for key in keys:
            if isinstance(value, dict):
                value = value.get(key)
            else:
                return None
        return value

    def compare(self, a, operator, b):
        ops = {
            '>=': lambda x, y: x >= y,
            '<=': lambda x, y: x <= y,
            '>': lambda x, y: x > y,
            '<': lambda x, y: x < y,
            '==': lambda x, y: x == y,
            '!=': lambda x, y: x != y,
            'in': lambda x, y: x in y,
            'not_in': lambda x, y: x not in y,
            'contains': lambda x, y: y in x,
            'starts_with': lambda x, y: str(x).startswith(str(y)),
            'ends_with': lambda x, y: str(x).endswith(str(y)),
        }
        return ops.get(operator, lambda x, y: False)(a, b)

# Usage
engine = AdvancedRuleEngine()

# Complex condition
rule = {
    'id': 'COMPLEX_RULE',
    'condition': {
        'all': [  # AND
            {'field': 'customer.age', 'operator': '>=', 'value': 20},
            {
                'any': [  # OR
                    {'field': 'customer.type', 'operator': '==', 'value': 'VIP'},
                    {
                        'all': [  # Nested AND
                            {'field': 'salary', 'operator': '>=', 'value': 50000},
                            {'field': 'employment_months', 'operator': '>', 'value': 12}
                        ]
                    }
                ]
            }
        ]
    },
    'action': {'decision': 'APPROVED'}
}

data = {
    'customer': {'age': 25, 'type': 'NORMAL'},
    'salary': 60000,
    'employment_months': 24
}

result = engine.evaluate_condition(rule['condition'], data)
print(f"Match: {result}")  # True
```

### Example 4: Rule Engine with Actions

```python
class ActionRuleEngine:
    def __init__(self):
        self.rules = []
        self.results = {}

    def execute_actions(self, rule, data):
        """Execute various action types"""
        actions = rule.get('actions', [])
        results = []

        for action in actions:
            action_type = action.get('type')

            if action_type == 'set':
                # Set value in results
                self.results[action['field']] = action['value']
                results.append({'set': {action['field']: action['value']}})

            elif action_type == 'calculate':
                # Calculate expression
                expr = action['expression']
                value = self.evaluate_expression(expr, data)
                self.results[action['field']] = value
                results.append({'calculate': {action['field']: value}})

            elif action_type == 'call':
                # Call function
                func_name = action['function']
                args = action.get('args', {})
                result = self.call_function(func_name, args, data)
                results.append({'call': {func_name: result}})

            elif action_type == 'return':
                # Return value
                results.append({'return': action['value']})

        return results

    def evaluate_expression(self, expr, data):
        """Simple expression evaluator"""
        # Replace field names with values
        for key, value in data.items():
            expr = expr.replace(key, str(value))
        return eval(expr)  # Note: Use safe_eval in production

    def call_function(self, func_name, args, data):
        """Call registered function"""
        functions = {
            'send_notification': lambda args, data: f"Sent to {args.get('to')}",
            'log_event': lambda args, data: f"Logged: {args.get('event')}",
        }
        return functions.get(func_name, lambda a, d: None)(args, data)

# Usage
engine = ActionRuleEngine()

rule = {
    'id': 'LOAN_CALCULATION',
    'condition': {'field': 'approved', 'operator': '==', 'value': True},
    'actions': [
        {'type': 'calculate', 'field': 'max_amount', 'expression': 'salary * 15'},
        {'type': 'calculate', 'field': 'interest_rate', 'expression': '7.0 - credit_bonus'},
        {'type': 'set', 'field': 'status', 'value': 'APPROVED'},
        {'type': 'call', 'function': 'send_notification', 'args': {'to': 'customer'}},
        {'type': 'return', 'value': {'decision': 'APPROVED'}}
    ]
}

data = {'salary': 50000, 'credit_bonus': 1.5, 'approved': True}
results = engine.execute_actions(rule, data)
print(results)
```

### Example 5: Production-Ready Rule Engine Structure

```python
class ProductionRuleEngine:
    """
    Production-ready rule engine with logging, caching, and error handling
    """

    def __init__(self, config=None):
        self.rules = []
        self.cache = {}
        self.config = config or {}
        self.logger = self.setup_logger()

    def setup_logger(self):
        import logging
        logger = logging.getLogger('RuleEngine')
        logger.setLevel(logging.DEBUG)
        return logger

    def load_rules(self, rules_source):
        """Load rules from source (DB, file, etc.)"""
        self.logger.info(f"Loading rules from {rules_source}")
        # Implementation depends on source
        pass

    def evaluate(self, rule_set_id, data):
        """
        Main evaluation method

        Args:
            rule_set_id: ID of rule set to evaluate
            data: Input data dict

        Returns:
            EvaluationResult object
        """
        start_time = time.time()

        try:
            # Validate input
            self.validate_input(data)

            # Get rules for rule set
            rules = self.get_rules(rule_set_id)

            # Evaluate rules
            matched_rules = []
            for rule in rules:
                if self.match_rule(rule, data):
                    matched_rules.append(rule)

            # Conflict resolution
            selected_rule = self.resolve_conflicts(matched_rules)

            # Execute action
            if selected_rule:
                result = self.execute_rule(selected_rule, data)
            else:
                result = self.default_result()

            # Log execution
            execution_time = time.time() - start_time
            self.log_execution(rule_set_id, data, result, execution_time)

            return result

        except Exception as e:
            self.logger.error(f"Error evaluating rules: {e}")
            raise

    def validate_input(self, data):
        """Validate input data"""
        if not isinstance(data, dict):
            raise ValueError("Data must be a dictionary")

    def get_rules(self, rule_set_id):
        """Get rules by rule set ID"""
        # Check cache first
        cache_key = f"rules_{rule_set_id}"
        if cache_key in self.cache:
            return self.cache[cache_key]

        # Load from store
        rules = [r for r in self.rules if r.get('rule_set') == rule_set_id]

        # Sort by priority
        rules.sort(key=lambda x: x.get('priority', 0), reverse=True)

        # Cache
        self.cache[cache_key] = rules

        return rules

    def match_rule(self, rule, data):
        """Match rule against data"""
        try:
            condition = rule.get('condition', {})
            return self.evaluate_condition(condition, data)
        except Exception as e:
            self.logger.warning(f"Error matching rule {rule.get('id')}: {e}")
            return False

    def resolve_conflicts(self, matched_rules):
        """Select one rule from multiple matches"""
        if not matched_rules:
            return None

        # Use highest priority rule
        return matched_rules[0]

    def execute_rule(self, rule, data):
        """Execute rule action"""
        action = rule.get('action', {})
        return {
            'rule_id': rule['id'],
            'rule_name': rule.get('name', ''),
            'output': action,
            'timestamp': datetime.now().isoformat()
        }

    def default_result(self):
        """Return default result when no rules match"""
        return {
            'rule_id': None,
            'output': {'decision': 'NO_MATCH'},
            'timestamp': datetime.now().isoformat()
        }

    def log_execution(self, rule_set_id, data, result, execution_time):
        """Log rule execution for audit"""
        self.logger.info(
            f"Rule evaluation: set={rule_set_id}, "
            f"rule={result.get('rule_id')}, "
            f"time={execution_time:.3f}s"
        )
```

---

## 10. Diagnostic Questions

ตอบคำถามเหล่านี้เพื่อทดสอบความเข้าใจ:

1. Rule Engine มีขั้นตอนการทำงานหลักอะไรบ้าง?
2. Pattern Matching คืออะไร?
3. Conflict Resolution ทำเมื่อไหร่ และทำอย่างไร?
4. Working Memory คืออะไร?
5. Forward Chaining ต่างจาก Backward Chaining อย่างไร?
6. ทำไม Rule Priority ถึงสำคัญ?
7. ถ้า Rule Engine ช้า จะ optimize อย่างไร?
8. Infinite loop เกิดจากอะไร และป้องกันอย่างไร?
9. ทำไมต้อง log rule execution?
10. Default rule สำคัญอย่างไร?

---

## 11. Explainer for 5-year-old

ลองนึกถึง **ตู้หยอดเหรียญ** นะ

1. **หยอดเหรียญ** = ใส่ข้อมูล
2. **ตู้ตรวจสอบ** = Rule Engine ดูว่าเหรียญถูกขนาด
3. **เลือกของ** = ตัดสินใจว่าได้อะไร
4. **ของออกมา** = ผลลัพธ์

ถ้าเหรียญ 5 บาท → ได้ขนม
ถ้าเหรียญ 10 บาท → ได้น้ำ

Rule Engine ก็ทำงานแบบนี้แหละ ดูว่าข้อมูลตรงกับกฎไหน แล้วทำตามกฎนั้น!

---

## 12. Comparison Table

### ขั้นตอนการทำงาน

| ขั้นตอน | คำอธิบาย | ตัวอย่าง |
|---------|---------|---------|
| **Load** | โหลด rules เข้า memory | ดึง rules จาก database |
| **Match** | จับคู่ data กับ conditions | salary >= 30000? |
| **Resolve** | เลือก rule ที่จะใช้ | ใช้ rule ที่ priority สูงสุด |
| **Execute** | ทำตาม action | return {approved: true} |

### Conflict Resolution Strategies

| Strategy | คำอธิบาย | ใช้เมื่อ |
|----------|---------|---------|
| **Priority** | เลือก rule priority สูงสุด | ทั่วไป |
| **Specificity** | เลือก rule ที่ specific ที่สุด | Complex rules |
| **Recency** | เลือก rule ที่สร้างล่าสุด | Evolving rules |
| **First Match** | ใช้ rule แรกที่ match | Simple cases |
| **All Match** | ใช้ทุก rule ที่ match | Aggregation |

---

## 13. 7-Day Rapid Learning Roadmap

### สำหรับหัวข้อนี้โดยเฉพาะ (ใช้เวลา 2-3 ชั่วโมง)

| เวลา | กิจกรรม |
|------|---------|
| 0:00-0:20 | อ่าน Abstract และ Core Principles |
| 0:20-0:40 | ศึกษา Playbook (Rule Engine Flow) |
| 0:40-1:10 | ศึกษา Example 1-2 (Simple + Priority) |
| 1:10-1:40 | ศึกษา Example 3-4 (Compound + Actions) |
| 1:40-2:10 | ศึกษา Example 5 (Production) |
| 2:10-2:40 | ลองเขียน simple rule engine ด้วยตัวเอง |
| 2:40-3:00 | อ่าน Pitfalls และตอบ Diagnostic Questions |

### เป้าหมายหลังจบ:
- [ ] อธิบายการทำงานของ rule engine ได้
- [ ] เข้าใจ pattern matching และ conflict resolution
- [ ] รู้ว่า rules execute อย่างไร
- [ ] debug rule execution ได้
