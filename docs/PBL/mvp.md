# AarogyaRekha-Health Pulse: Digital Twin of Health

> **Vision:** Transforming fragmented, messy medical paper records into a structured, searchable, and intelligent health dashboard.

---

## The 4 Pillars of AarogyaRekha

### 1. Smart Ingestion
* **The "Magic Email":** Get a unique address (e.g., `user123@vitanode.com`) for direct lab results ingestion.
* **One-Way Push:** Authorized hospitals use an API to "post" reports securely.
* **AI Extraction & OCR:** * Converts images into database entries (e.g., "Glucose: 95 mg/dL").
    * **Review & Confirm:** Human-in-the-loop feature to verify AI extraction (crucial for doctor's handwriting).
* **Direct Upload:** Scan or upload pictures of reports and prescriptions instantly.

### 2. Intelligent Visualization
=== "Patient View"
    Less technical terms, interactive graphs for vitals, and simple tablet names.
=== "Doctor View"
    A "TL;DR" summary including technical terms, overall medical history, and quick access to original PDFs/prescriptions.
=== "Emergency QR"
    A lock-screen-accessible code for first responders containing:
    * Allergies & Blood Type
    * Present Medications
    * Recent Tests & Emergency Contacts

### 3. Proactive Health Logic
!!! example "Date-Delta Alerts"
    - 🟢 **Green (Current):** 0 - 6 months
    - 🟡 **Yellow (Stale):** 6 months - 2 years (Trigger Reminders)
    - 🔴 **Red (Outdated):** 2+ years

* **Drug Interaction Checker:** Warns if new prescriptions conflict with existing meds or known allergies. Lists side effects and components.
* **Automatic Reminders:** Opt-in reminders based on AI-extracted prescription duration.
* **Health Streaks:** Visual representation of test frequency with 1-line report verdicts.

### 4. Safety & Verification
* **Swipe-to-Verify:** A UI for users to confirm AI data extraction, ensuring 100% accuracy.
* **Temporary Access:** Provide doctors with 30-minute access via OTP for remote consultations.

---

## 🛠 Advanced Features
* **Family Profiles:** Manage accounts for children or elderly family members.
* **Voice Query:** Ask naturally: *"When was the last time I had a blood sugar checkup?"*