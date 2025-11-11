# 📘 Python Data Engineer IoT Pipeline Handbook (Professional Learning Edition)

**ผู้เขียน:** Kitsana Mameesukh
**Edition:** Learning Edition
**Version:** 1.0
**อัปเดตล่าสุด:** พฤศจิกายน 2025

---

## 📚 สารบัญ (Table of Contents)

### [Module 1: Python Fundamentals & Environment](./module-1/module-1.md)
พื้นฐาน Python สำหรับ Data Engineering
- Data Structures: Pandas และ NumPy
- Virtual Environment Setup
- Time-Series Data Introduction
- [🧪 Labs & Exercises](./module-1/labs/lab-module-1.md)

### [Module 2: ETL - Data Loading & Cleansing](./module-2/module-2.md)
การโหลดและทำความสะอาดข้อมูล
- Optimized Loading (dtype optimization)
- Handling Missing Values
- Data Integrity Checks
- [🧪 Labs & Exercises](./module-2/labs/lab-module-2.md)

### [Module 3: ETL - Transformation & Optimization](./module-3/module-3.md)
การแปลงและเพิ่มประสิทธิภาพข้อมูล
- Feature Engineering
- Performance Benchmarking
- Chunking Strategies
- [🧪 Labs & Exercises](./module-3/labs/lab-module-3.md)

### [Module 4: Cloud Integration & Storage](./module-4/module-4.md)
การบูรณาการกับ Cloud และการจัดเก็บข้อมูล
- Cloud Storage & Data Lake
- Cloud SDKs (AWS, GCP, Azure)
- Data Partitioning Strategy
- [🧪 Labs & Exercises](./module-4/labs/lab-module-4.md)

### [Module 5: Airflow - Orchestration Basics](./module-5/module-5.md)
พื้นฐาน Workflow Orchestration
- DAGs, Tasks, Operators
- Dependencies & Scheduling
- Airflow UI Navigation
- [🧪 Labs & Exercises](./module-5/labs/lab-module-5.md)

### [Module 6: Airflow - Building the IoT Pipeline](./module-6/module-6.md)
สร้าง Pipeline สำหรับ IoT
- FileSensor Implementation
- PythonOperator Integration
- Cloud Operators
- [🧪 Labs & Exercises](./module-6/labs/lab-module-6.md)

### [Module 7: Data Quality & Testing](./module-7/module-7.md)
การตรวจสอบคุณภาพและการทดสอบ
- Data Validation
- Pipeline Idempotency
- Unit Testing for ETL
- [🧪 Labs & Exercises](./module-7/labs/lab-module-7.md)

### [Module 8: Project Summary & Future Scope](./module-8/module-8.md)
สรุปและแนวทางต่อไป
- Executive Summary
- Key Skills Mapping
- Real-Time & MLOps Vision
- [🧪 Final Project](./module-8/labs/lab-module-8.md)

---

## 🎯 วัตถุประสงค์ของคู่มือ

คู่มือเล่มนี้ออกแบบมาเพื่อให้ผู้เรียนสามารถ:

1. ✅ **เข้าใจพื้นฐาน Python** สำหรับงาน Data Engineering
2. ✅ **ออกแบบและพัฒนา ETL Pipeline** สำหรับข้อมูล IoT
3. ✅ **ใช้งาน Apache Airflow** ในการ Orchestrate Workflows
4. ✅ **ทำงานกับ Cloud Storage** และ Data Lake Architecture
5. ✅ **สร้าง Data Quality Framework** และ Testing Strategy
6. ✅ **นำไปประยุกต์ใช้** ในโปรเจคจริง

---

## 🛠️ เทคโนโลยีที่ใช้ (Tech Stack)

```
Programming:     Python 3.8+
Data Processing: Pandas, NumPy
Orchestration:   Apache Airflow
Cloud:           AWS, GCP, Azure (Conceptual)
Tools:           Jupyter, VS Code, Docker, Git
```

---

## 📊 ภาพรวม Data Engineering Lifecycle

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   INGEST    │────▶│  TRANSFORM  │────▶│    STORE    │────▶│    SERVE    │
│  (Extract)  │     │   (Clean &  │     │ (Data Lake) │     │ (Analytics) │
│             │     │  Enrich)    │     │             │     │             │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
       ▲                    ▲                    ▲                    ▲
       │                    │                    │                    │
       └────────────────────┴────────────────────┴────────────────────┘
                        Apache Airflow (Orchestration)
```

---

## 🎓 เหมาะสำหรับใครบ้าง

- 👨‍💻 Software Engineers ที่ต้องการเปลี่ยนสาย Data Engineer
- 📊 Data Analysts ที่ต้องการพัฒนา Pipeline Skills
- 🔌 IoT Engineers ที่ต้องการจัดการข้อมูล Sensor
- 🎓 นักศึกษาที่สนใจ Data Engineering

---

## 📖 วิธีการเรียนรู้

### แนวทาง Learning by Doing

แต่ละ Module ประกอบด้วย:

1. **📝 แนวคิดและทฤษฎี** - อธิบายหลักการพื้นฐาน
2. **💻 ตัวอย่างโค้ด** - แสดง Best Practices
3. **🧪 Labs & Exercises** - ฝึกปฏิบัติจริง
4. **📊 Diagrams** - ภาพประกอบความเข้าใจ
5. **❓ Challenge Questions** - กระตุ้นการคิดวิเคราะห์

---

## ⏱️ เวลาที่แนะนำต่อ Module

| Module | เวลาศึกษา | เวลา Lab | รวม |
|--------|-----------|----------|-----|
| Module 1 | 2 ชม. | 1 ชม. | 3 ชม. |
| Module 2 | 2 ชม. | 1.5 ชม. | 3.5 ชม. |
| Module 3 | 2.5 ชม. | 2 ชม. | 4.5 ชม. |
| Module 4 | 2 ชม. | 1.5 ชม. | 3.5 ชม. |
| Module 5 | 3 ชม. | 2 ชม. | 5 ชม. |
| Module 6 | 3 ชม. | 2.5 ชม. | 5.5 ชม. |
| Module 7 | 2.5 ชม. | 2 ชม. | 4.5 ชม. |
| Module 8 | 2 ชม. | 3 ชม. | 5 ชม. |
| **รวม** | **19 ชม.** | **16 ชม.** | **35 ชม.** |

---

## 🚀 Quick Start Guide

### ขั้นตอนการเริ่มต้น

1. **เตรียม Environment**
   ```bash
   python --version  # ต้อง 3.8+
   pip install pandas numpy jupyter
   ```

2. **Clone หรือ Download คู่มือ**
   - ตรวจสอบให้แน่ใจว่ามีโฟลเดอร์ module-1 ถึง module-8

3. **เริ่มเรียน Module 1**
   - อ่าน [module-1/module-1.md](./module-1/module-1.md)
   - ทำ Lab ใน [module-1/labs/lab-module-1.md](./module-1/labs/lab-module-1.md)

4. **เรียนต่อเนื่องตามลำดับ Module**

---

## 📦 Requirements

### Software Requirements
```
Python >= 3.8
pandas >= 1.3.0
numpy >= 1.21.0
apache-airflow >= 2.0.0 (Module 5+)
jupyter >= 1.0.0
```

### Hardware Requirements
- RAM: อย่างน้อย 4 GB (แนะนำ 8 GB+)
- Storage: อย่างน้อย 10 GB ว่าง
- CPU: Multi-core สำหรับ Parallel Processing

---

## 🗺️ Learning Path Roadmap

```
START
  │
  ├─▶ Module 1: Python Basics ────────────▶ (Foundation)
  │
  ├─▶ Module 2: Data Loading ─────────────▶ (ETL Part 1)
  │
  ├─▶ Module 3: Transformation ───────────▶ (ETL Part 2)
  │
  ├─▶ Module 4: Cloud Storage ────────────▶ (Infrastructure)
  │
  ├─▶ Module 5: Airflow Basics ──────────▶ (Orchestration)
  │
  ├─▶ Module 6: Build IoT Pipeline ──────▶ (Integration)
  │
  ├─▶ Module 7: Quality & Testing ───────▶ (Best Practices)
  │
  └─▶ Module 8: Summary & Next Steps ────▶ (Career Growth)
                                              │
                                            FINISH
```

---

## 💡 Tips สำหรับการเรียนรู้

1. **📝 จดบันทึก** - เขียนสิ่งที่เรียนรู้ด้วยคำพูดของตัวเอง
2. **💻 Code Along** - พิมพ์โค้ดตามด้วยตัวเอง อย่า Copy-Paste
3. **🔄 Practice Repeatedly** - ทำ Lab ซ้ำจนเข้าใจ
4. **🤔 Ask Why** - ถามตัวเองว่า "ทำไมต้องทำแบบนี้?"
5. **🚀 Experiment** - ลองปรับแต่งโค้ดและดูผลลัพธ์
6. **📚 Read Documentation** - อ่าน Official Docs ของแต่ละ Library

---

## 🔗 Additional Resources

### Official Documentation
- [Python Documentation](https://docs.python.org/3/)
- [Pandas Documentation](https://pandas.pydata.org/docs/)
- [NumPy Documentation](https://numpy.org/doc/)
- [Apache Airflow Documentation](https://airflow.apache.org/docs/)

### Community & Support
- Stack Overflow (Tag: python, pandas, airflow)
- Reddit: r/dataengineering
- GitHub Discussions

---

## 📞 ติดต่อและ Feedback

หากพบข้อผิดพลาดหรือมีข้อเสนอแนะ:
- **Email:** kitsana.mameesukh@example.com
- **GitHub:** [Repository Link]

---

## 📄 License & Usage

คู่มือนี้จัดทำขึ้นเพื่อการศึกษา สามารถนำไปใช้และแชร์ได้อย่างอิสระ

---

## 🎬 พร้อมเริ่มต้นแล้วใช่ไหม?

**👉 เริ่มเลย: [Module 1: Python Fundamentals & Environment](./module-1/module-1.md)**

---

**Happy Learning! 🚀**

> *"Data Engineering is not just about moving data, it's about building reliable systems that enable data-driven decisions."*
>
> — Kitsana Mameesukh

---

**จัดทำโดย:** Kitsana Mameesukh
**สำหรับ:** นักเรียน Data Engineering ทุกท่าน
