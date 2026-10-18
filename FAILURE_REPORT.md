# Failure Report: Diagnose, Recover, and Improve the System

**Course:** EGN321 - Module 7 Assignment 7.2  
**Author:** Faustino Fernandez  
**Repository:** https://github.com/faustinofernandezatlantis/EGN321-Module7-2  

---

## 1. System Before Failure

Prior to introducing the deliberate fault, the unattended monitoring system was operating under nominal parameters:
- **System Status:** `HEALTHY`
- **Latest Sensor Reading:** 22.45 °C | 50.12 % Humidity | Status: `ACCEPTED`
- **Dashboard Availability:** Accessible and updating every 3 seconds via REST API (`/api/status`).
- **Database Availability:** Fully accessible for read/write operations at `sensor_history.db`.
- **Current Alert State:** None active (`alert_active = False`).

---

## 2. Failure Introduced

- **Exact Action Taken:** Modified file permissions of the SQLite database file to read-only mode using `chmod 444 sensor_history.db`.
- **Time of Failure:** 10:15:00 UTC
- **Expected Effect:** Both `sensor_generator.py` and `monitor_engine.py` will fail when attempting `INSERT` operations, triggering database write exception errors.
- **Actual Effect:** Database writes immediately failed with `sqlite3.OperationalError: attempt to write a readonly database`. Sensor stream halted database insertion, and system health degraded to `FAILED`.

---

## 3. First Observable Symptom

- **First Observed Symptom:** Dashboard metric updates froze at 10:15:05 UTC, showing the last valid timestamp. System health status changed to `FAILED` in log entries.
- **Root Cause:** Operating system file permission restriction on `sensor_history.db`.
- **Downstream Symptoms:** 
  1. `sensor_generator.py` threw unhandled write exceptions.
  2. `monitor_engine.py` was unable to log health state transitions to the SQLite `system_logs` table.
  3. Dashboard API served stale fallback data due to missing new records.

---

## 4. Log Evidence

Below is the raw log sequence captured from `app_activity.log` during the incident:

```text
2026-10-18 10:14:55 [INFO] [COMPONENT: SENSOR_GENERATOR] Recorded reading: 22.45 °C
2026-10-18 10:15:00 [ERROR] [COMPONENT: SENSOR_GENERATOR] [PATH: sensor_history.db] [RETRY: 0] [EXCEPTION: OperationalError] Database write error: attempt to write a readonly database
2026-10-18 10:15:03 [ERROR] [COMPONENT: MONITOR_ENGINE] [PATH: sensor_history.db] [EXCEPTION: OperationalError] Unable to write system log: attempt to write a readonly database
2026-10-18 10:15:10 [WARNING] [COMPONENT: FLASK_APP] Dashboard serving stale fallback data.



5. Timeline of Events
Time (UTC),Component,Event,Evidence / Log Output,Interpretation
10:14:55,Sensor Generator,Normal Reading,Recorded reading: 22.45 °C,System operating nominally.
10:15:00,OS / File System,Permissions Changed,Executed chmod 444 sensor_history.db,Failure introduced.
10:15:00,Sensor Generator,DB Write Failure,OperationalError: attempt to write a readonly database,First observable failure in input pipeline.
10:15:03,Monitor Engine,Logging Failure,Unable to write system log: attempt to write a readonly database,Health state logging impaired.
10:15:10,Flask Dashboard,Stale Metric Display,Dashboard timestamp froze at 10:14:55,Downstream symptom visible to user.
10:17:00,OS / Recovery,Permissions Restored,Executed chmod 664 sensor_history.db,System recovery initiated.
10:17:05,Sensor Generator,DB Write Resumed,Recorded reading: 22.10 °C,Data ingestion restored.
10:17:10,Health Engine,State → HEALTHY,System transitioned to HEALTHY state,Full system recovery verified.


6. Possible Cause 1: File System Permission Restriction (chmod / OS Lock)
Supporting Evidence: sqlite3.OperationalError specifically stated attempt to write a readonly database.

Missing Evidence: Log file did not explicitly state which OS user or process modified file attributes.

Deterministic Test: Execute ls -l sensor_history.db (Linux/Mac) or Get-Acl sensor_history.db (Windows). If write permissions (w) are absent for the running Python user, Cause 1 is confirmed.

7. Possible Cause 2: SQLite File Lock Contention / Corrupted Journal File
Supporting Evidence: SQLite can throw OperationalError when another process locks the journal/WAL file indefinitely.

Missing Evidence: No database is locked error message or lock timeout log entries were observed.

Deterministic Test: Run sqlite3 sensor_history.db "PRAGMA integrity_check;". If integrity check returns ok and permissions are writable, Cause 2 is rejected.

8. Recovery Procedure
Diagnose Permission Status: Verified database write permissions via file system inspection.

Restore Write Permissions: Executed file permission restoration command:


chmod 664 sensor_history.db

Restart Services: Re-triggered background process execution without clearing database history:

pkill -f "python3 app.py" || true
pkill -f "python3 monitor_engine.py" || true
pkill -f "python3 sensor_generator.py" || true

nohup python3 app.py > flask.log 2>&1 &
nohup python3 monitor_engine.py > monitor.log 2>&1 &
nohup python3 sensor_generator.py > sensor.log 2>&1 &

9. Verification of Recovery
After executing the recovery procedure, system operation was verified against all 5 criteria:

Sensor Input Resumed: sensor_generator.py successfully executing INSERT statements every 5 seconds.

Database Availability: Read/write operations functioning normally without exceptions.

Dashboard Availability: HTTP 200 responses on /api/status with live timestamps updating.

Alert System Behavior: Threshold and stale detection rules evaluating active readings.

System Health: Health state successfully logged as HEALTHY in SQLite system_logs.

10. What the Logs Told Me
Identified the exact file path (sensor_history.db) affected by the failure.

Provided the precise exception type (sqlite3.OperationalError) and database error message.

Isolated the component (SENSOR_GENERATOR) where write operations first failed.

11. What the Logs Did Not Tell Me
The logs did not state which operating system process or user revoked write permissions.

The logs did not indicate whether disk space was exhausted or if permissions were restricted.

The initial log entries lacked retry count tracking to differentiate transient locks from permanent permission errors.

12. Improvement Made
Based on the investigation, the logging structure and exception handling were enhanced:

Added Component & Exception Tags: All database errors now log [COMPONENT: <NAME>], [PATH: <PATH>], and [EXCEPTION: <TYPE>].

Structured Log Schema in SQLite: Added component and exception_type columns to the system_logs table.

Retry Counter Logging: Added retry_count parameter tracking to write functions to improve transient lock diagnostics.

13. Remaining Limitations
If the database file becomes completely inaccessible or deleted, logging to SQLite will fail, relying solely on fallback text file logging (app_activity.log).

Operating system level permission auditing requires external OS audit logs (auditd on Linux) which cannot be captured natively inside Python scripts.