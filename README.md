# Module 7 Assignment 7.2: Failure Report — Diagnose, Recover, and Improve

**Course:** EGN321 - Engineering Software Systems  
**Author:** Faustino Fernandez  
**Repository:** https://github.com/faustinofernandezatlantis/EGN321-Module7-2  

---

## Overview

This repository contains Assignment 7.2, focusing on failure diagnosis, unattended recovery, log analysis, and structural logging design improvements for an unattended sensor monitoring system.

A deliberate, realistic failure—revoking file write permissions on the SQLite database (`chmod 444 sensor_history.db`)—was introduced to evaluate how system evidence (logs, timestamps, and health state indicators) reveals root causes versus downstream symptoms.

---

## Repository Structure

```text
EGN321-Module7-2/
├── FAILURE_REPORT.md       # Complete 13-Section Failure Investigation Report
├── README.md               # Repository Overview and Instructions
├── AI_LOG.md               # Disclosure of AI Tool Usage
├── app.py                  # Flask Web Server & Dashboard REST API
├── database.py             # SQLite Database Interface with Enhanced Logging
├── monitor_engine.py       # Health Assessment Engine & Threshold Evaluator
├── sensor_generator.py     # Sensor Reading Simulator & Database Writer
├── config.py               # Global System Configuration & Thresholds
├── requirements.txt        # Python Dependencies
└── screenshots/            # Failure and Recovery Evidence
    ├── dashboard_before_failure.png
    ├── failure_log_evidence.png
    └── recovery_verification.png

Key System Features & Logging Enhancements
Enhanced Exception Handling: Captures sqlite3.OperationalError explicitly when write operations fail.

Structured Log Attributes: All log outputs include [COMPONENT], [PATH], [RETRY], and [EXCEPTION] metadata tags.

Audit Schema: SQLite system_logs table stores component and exception_type columns to assist root-cause diagnosis.

Resilient Architecture: Fallback logging ensures text-based log files (app_activity.log) capture failure events even when the SQLite database becomes read-only or unreachable.

How to Run & Reproduce the Failure Test in Google Colab
1. Install Dependencies

pip install Flask

2. Start Services

nohup python3 app.py > flask.log 2>&1 &
nohup python3 monitor_engine.py > monitor.log 2>&1 &
nohup python3 sensor_generator.py > sensor.log 2>&1 &

3. Simulate Failure (Read-Only Database)

chmod 444 sensor_history.db

4. Review Log Evidence

tail -n 20 app_activity.log

5. Restore System Operation

chmod 664 sensor_history.db
pkill -f "python3"
nohup python3 app.py > flask.log 2>&1 &
nohup python3 monitor_engine.py > monitor.log 2>&1 &
nohup python3 sensor_generator.py > sensor.log 2>&1 &


Verification

Full analysis, timeline of events, and cause verification procedures are documented in FAILURE_REPORT.md.