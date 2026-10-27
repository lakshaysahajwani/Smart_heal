# SmartHeal — Self-Healing Student Results Portal

BCA final-year project built with Flask, SQLite, HTML, CSS, and JavaScript. It extends the student/admin portal with health checks, structured incident logging, controlled chaos tests, and automatic recovery demonstrations.

> **Safety note:** Chaos tests should never deliberately corrupt `smartheal.db`. Database-fault tests use the disposable `smartheal_test.db`, which is restored from `smartheal_test_backup.db`. Keep backups of all data before demonstrations.

## Features

- Student and administrator login with Werkzeug password hashing.
- Admin can create student accounts and update attendance, marks, and assignments.
- Student dashboard (`templates/dashboard.html`) and student panel.
- `/api/health` health check and `/api/resilience` circuit/fallback status.
- Structured incident records with detected, diagnosed, action, recovered timestamps, detection time, recovery time, mechanism, and details.
- Retry with exponential backoff for transient dependency failures.
- Circuit-breaker state and cached fallback when the controlled dependency simulator fails.
- Isolated test-database recovery from a known-good backup after a test table failure or invalid record.
- Bounded 100-query local load test; this is not a production load-testing benchmark.
- `watchdog.py` restarts the Flask process if it exits or repeatedly fails its local health check.
- `chaos_script.py` runs four controlled API chaos scenarios.

## Windows Quick Start

1. Install Python 3.10 or newer from [python.org](https://www.python.org/downloads/) and select **Add Python to PATH**.
2. Extract the whole project folder.
3. Double-click `Run_SmartHeal.bat`.
4. Open [http://127.0.0.1:5000/](http://127.0.0.1:5000/) in Chrome.
5. Keep the terminal window open. It runs the watchdog, which starts and supervises the app.

Or run manually in Command Prompt from the project folder:

```bat
py -3 -m pip install -r requirements.txt
py -3 watchdog.py
```

## Admin Account

On a database with no admin user, first initialization creates the default local admin account `admin` with password `SmartHeal@2026`.

For safer setups, set environment variables before initialization:

```bat
set ADMIN_USERNAME=your_admin_name
set ADMIN_PASSWORD=use_a_strong_password
set SECRET_KEY=use_a_long_random_secret
py -3 app.py
```

If the database already contains an admin, changing these environment variables does not change that existing user's password.

**Security:** Do not publish real credentials, secret keys, or a database containing private student information to a public repository. Change any default credentials before sharing or deploying the application.

## Run the Chaos Tests

With dependencies installed, run:

```bat
py -3 chaos_script.py
```

The script exercises dependency outage, isolated test database failure, invalid test record, and bounded load testing. It uses Flask's test client and does not require a browser.

The report printed by the script contains the results from that run. Re-run tests and retain the actual output for your resilience report.

You can also use the Error Lab on the home page. With auto-recovery disabled, click **Run Recovery** after generating a test failure before starting another one. With auto-recovery enabled, recovery and verification happen automatically.

## Test the Process Watchdog

1. Start `py -3 watchdog.py`.
2. Open Task Manager, find the Python process running `app.py`, and end **that child process only**. Leave `watchdog.py` running.
3. Watch the terminal and `watchdog.log`. The watchdog should report the exited process and start another one.
4. Refresh [http://127.0.0.1:5000/](http://127.0.0.1:5000/).

Perform this only on your local test machine. Do not expose Flask's development server directly to the public internet.

## Incident Lifecycle and Metrics

Each chaos test attempts to record:

1. **Detected** — failure identified and timestamp stored.
2. **Diagnosed** — failure type identified.
3. **Action taken** — recovery action begins.
4. **Recovered** — recovery verification succeeds.

Detection and recovery times are recorded in milliseconds. These are runtime measurements, not guaranteed performance values. A successful API response does not replace repeated testing or the demonstration required by the project brief.

## Known Limitations

- The external dependency is a controlled in-process simulator, not a real third-party provider or internet outage.
- The isolated database backup is a local file copy; it does not demonstrate disaster recovery if the entire disk is lost.
- The 100-query test is a small local smoke test, not proof that the app can handle ten times production traffic.
- The watchdog supervises the Flask process on the same computer. It cannot recover from machine power loss, operating-system failure, or a watchdog process crash.
- SQLite and Flask's development server are suitable for a student demonstration, not a high-traffic production results portal.
- The project does not currently automatically roll back a bad application release.
- Do not claim every failure is healed. Record any failed recovery honestly.

## Project Layout

```text
SelfHealingStudentPortal/
├── app.py
├── watchdog.py
├── chaos_script.py
├── Run_SmartHeal.bat
├── requirements.txt
├── README.md
├── RESILIENCE_REPORT_TEMPLATE.md
├── style.css
├── script.js
└── templates/
    ├── index.html
    ├── login.html
    ├── admin.html
    ├── student.html
    └── dashboard.html
```

The local database, test database backups, and runtime logs may be created during execution and are intentionally omitted from this public-facing file listing.

## Technology Stack

- Python
- Flask
- SQLite
- HTML, CSS, JavaScript
- Git and GitHub

## Project Purpose

SmartHeal demonstrates how a student portal can detect selected software failures, attempt recovery, verify outcomes, and record incidents. Its effectiveness should be evaluated using actual test results and documented limitations.
