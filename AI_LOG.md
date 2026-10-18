# AI Assistance Disclosure Log (AI_LOG.md)

**Course:** EGN321 - Engineering Software Systems  
**Assignment:** Module 7 Assignment 7.2 (Failure Report: Diagnose, Recover, and Improve)  
**Author:** Faustino Fernandez  
**Repository:** https://github.com/faustinofernandezatlantis/EGN321-Module7-2  

---

## 1. Overview & Policy Compliance

This document records all artificial intelligence (AI) assistance utilized during the development, execution, diagnosis, and documentation of Module 7 Assignment 7.2. All AI interactions were conducted in accordance with institutional academic integrity policies and course requirements.

---

## 2. AI Tools Used

- **Primary AI Tool:** Gemini (Google DeepMind)
- **Role:** Technical Assistant, Log Analysis Consultant, and Markdown Documentation Specialist.

---

## 3. Scope of AI Assistance

AI tools were utilized for the following tasks:
1. **Failure Scenario Formulation:** Selecting a realistic operating system level fault (`chmod 444 sensor_history.db`) to evaluate database permission exceptions.
2. **Report Structuring:** Formatting the required 13-section failure investigation report (`FAILURE_REPORT.md`) according to assignment rubrics.
3. **Structured Logging Improvement:** Updating Python database error handling functions (`database.py`) to capture component names, retry counts, and exception types (`sqlite3.OperationalError`).
4. **Timeline Formatting:** Constructing the structured chronological evidence table mapping timestamps, component events, log outputs, and system interpretations.

---

## 4. Prompt & Interaction Audit

| Interaction Date | Primary Prompt / Focus | AI Output Summary | Human Verification & Modifications |
| :--- | :--- | :--- | :--- |
| **Oct 18, 2026** | "Draft a complete 13-section failure investigation report for a read-only SQLite fault." | Provided structured Markdown template covering pre-failure state, symptoms, and timeline table. | Verified all file paths, timestamps, and log snippets against active Colab execution. |
| **Oct 18, 2026** | "Improve database.py to capture exception types, component names, and path metadata." | Generated updated Python functions with explicit `try-except sqlite3.OperationalError` blocks. | Tested modified script inside Google Colab environment to confirm failure behavior. |
| **Oct 18, 2026** | "Generate repository documentation (README.md, FAILURE_REPORT.md, AI_LOG.md) in English." | Delivered complete repository file templates adhering to project submission requirements. | Uploaded and verified all files within the GitHub repository structure. |

---

## 5. Verification & Statement of Originality

All AI-generated code, log structures, and report templates were independently executed, validated, and verified within a Google Colab Linux environment. I confirm that I understand all technical concepts, recovery steps, and structural improvements documented in this repository.