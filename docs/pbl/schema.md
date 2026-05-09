# Database Schema: AarogyaRekha

This document details the relational database structure for the Health Pulse Digital Twin. 

---

## 🏗️ Core User Tables

### Users
The central entity for every account in the system.

| Column | Type | Constraints | Description |
| :--- | :--- | :--- | :--- |
| `user_id` | INT | PK, AI | Primary key, auto-increment |
| `full_name` | VARCHAR(100) | NOT NULL | User's legal name |
| `email` | VARCHAR(100) | UNIQUE, NOT NULL | Login identifier |
| `password_hash` | VARCHAR(255) | NOT NULL | Bcrypt hash |
| `dob` | DATE | NOT NULL | Date of Birth |
| `gender` | ENUM | 'male', 'female', 'other' | - |
| `blood_group` | VARCHAR(5) | Nullable | Stored for fast emergency queries |
| `is_doctor` | BOOLEAN | DEFAULT FALSE | Privilege flag |
| `created_at` | TIMESTAMP | DEFAULT CURRENT | Account creation time |

### Patient_Identity
A 1:1 extension of the Users table for medical profiles.

| Column | Type | Constraints | Description |
| :--- | :--- | :--- | :--- |
| `identity_id` | INT | PK, AI | Primary key |
| `user_id` | INT | FK, UNIQUE, NOT NULL | Link to `Users(user_id)` (Cascade) |
| `allergies` | TEXT | Nullable | Known allergies |
| `current_medications` | TEXT | Nullable | Active meds |
| `emergency_contact_name` | VARCHAR(100) | Nullable | - |
| `emergency_contact_phone`| VARCHAR(15) | Nullable | - |

---

## 📊 Medical Data Structures

### Medical_Records
The central fact table for all uploaded documents.

| Column | Type | Constraints | Description |
| :--- | :--- | :--- | :--- |
| `record_id` | INT | PK, AI | Primary key |
| `user_id` | INT | FK, NOT NULL | Record owner (Cascade) |
| `uploaded_by_user_id` | INT | FK | Uploader (SET NULL on delete) |
| `record_type` | ENUM | NOT NULL | 'lab_report', 'prescription', 'clinical_note' |
| `title` | VARCHAR(200) | Nullable | - |
| `doctor_name` | VARCHAR(100) | Nullable | Denormalized for PBL scope |
| `hospital_name` | VARCHAR(100) | Nullable | - |
| `record_date` | DATE | NOT NULL | Date on the document |
| `file_path` | VARCHAR(255) | Nullable | Path to PDF/Image |
| `file_mime_type` | VARCHAR(50) | Nullable | e.g. 'application/pdf' |
| `notes` | TEXT | Nullable | - |

> **Indices:** `record_date` (Timeline sort), `record_type` (Filter), `(user_id, record_date)` (Composite)

### Test_Results
Detailed numeric values extracted from lab reports.

| Column | Type | Constraints | Description |
| :--- | :--- | :--- | :--- |
| `test_id` | INT | PK, AI | Primary key |
| `record_id` | INT | FK, NOT NULL | Link to `Medical_Records` (Cascade) |
| `test_name` | VARCHAR(100) | NOT NULL | e.g. Glucose, TSH |
| `test_value` | DECIMAL(12,4) | NOT NULL | Supports high precision |
| `unit` | VARCHAR(20) | Nullable | e.g. mg/dL |
| `normal_min` | DECIMAL(12,4) | Nullable | Reference range low |
| `normal_max` | DECIMAL(12,4) | Nullable | Reference range high |

### Prescriptions
Medicine details linked to prescription records.

| Column | Type | Constraints | Description |
| :--- | :--- | :--- | :--- |
| `prescription_id` | INT | PK, AI | Primary key |
| `record_id` | INT | FK, NOT NULL | Link to `Medical_Records` (Cascade) |
| `medicine_name` | VARCHAR(100) | NOT NULL | - |
| `dosage` | VARCHAR(100) | Nullable | e.g. 500mg twice daily |
| `duration` | VARCHAR(50) | Nullable | e.g. 7 days |

---

## 🔐 Access, Family, & Audit

### Family_Access
Self-join table for managing family permissions.

| Column | Type | Constraints | Description |
| :--- | :--- | :--- | :--- |
| `family_access_id` | INT | PK, AI | - |
| `owner_user_id` | INT | FK, NOT NULL | The user granting access |
| `member_user_id` | INT | FK, NOT NULL | The user receiving access |
| `relation` | ENUM | NOT NULL | 'parent', 'child', 'spouse', etc. |
| `can_manage` | BOOLEAN | DEFAULT TRUE | - |
| `granted_at` | DATETIME | DEFAULT CURRENT | - |

### Doctor_Access_Requests
Workflow for temporary record sharing.

| Column | Type | Constraints | Description |
| :--- | :--- | :--- | :--- |
| `request_id` | INT | PK, AI | - |
| `doctor_user_id` | INT | FK, NOT NULL | Link to `Users` (Cascade) |
| `patient_user_id` | INT | FK, NOT NULL | Link to `Users` (Cascade) |
| `status` | ENUM | DEFAULT 'pending'| 'pending', 'approved', 'rejected', 'expired' |
| `expires_at` | DATETIME | Nullable | Set upon approval |
| `approved_by_user_id`| INT | FK | Approving caregiver/patient |

### Audit_Log
Append-only trail for security and compliance.

| Column | Type | Constraints | Description |
| :--- | :--- | :--- | :--- |
| `log_id` | INT | PK, AI | - |
| `actor_user_id` | INT | FK, NOT NULL | Who performed action |
| `target_user_id` | INT | FK, NOT NULL | Subject of the action |
| `action` | ENUM | NOT NULL | 'view', 'upload', 'edit', 'delete', 'share' |
| `record_id` | INT | Nullable | Specific record affected |
| `logged_at` | TIMESTAMP | DEFAULT CURRENT | - |

---

## 🔗 Relationships & Cardinality

| Entities | Ratio | Participation | Meaning |
| :--- | :--- | :--- | :--- |
| **Users → Identity** | 1 : 1 | Total (ID), Partial (User) | Every ID must have a user; users might not have IDs yet. |
| **Users → Records** | 1 : N | Partial / Partial | One user owns zero or many records. |
| **Records → Tests** | 1 : N | Partial / Partial | Lab reports have many test values; notes have none. |
| **Users ↔ Users** | M : N | Partial / Partial | Junction table for family management (Self-join). |
| **Users → Doctor Req**| 1 : N | Partial / Partial | Doctors send many; Patients receive many. |
| **Users → Audit Log** | 1 : N | Partial / Partial | Tracks actors (who) and targets (whose data). |

!!! info "Key"
    **PK**: Primary Key | **FK**: Foreign Key | **IDX**: Index | **UQ**: Unique | **AI**: Auto-Increment