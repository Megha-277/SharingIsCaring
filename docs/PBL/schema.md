# Database Schema: AarogyaRekha

This document outlines the relational database structure for the Health Pulse Digital Twin. It is designed to handle messy medical data while maintaining strict privacy and audit trails.

---

## 🏗️ Core Tables

### Users
The central entity for every account (Patients, Doctors, and Caregivers).
| Column | Type | Constraints | Description |
| :--- | :--- | :--- | :--- |
| `user_id` | INT | PK, AI | Unique User ID |
| `full_name` | VARCHAR(100) | NOT NULL | User's legal name |
| `email` | VARCHAR(100) | UNIQUE, NOT NULL | Login identifier |
| `password_hash` | VARCHAR(255) | NOT NULL | Bcrypt hashed password |
| `dob` | DATE | NOT NULL | Date of Birth |
| `gender` | ENUM | 'male','female','other' | User gender |
| `blood_group` | VARCHAR(5) | Nullable | Stored for fast emergency queries |
| `is_doctor` | BOOLEAN | DEFAULT FALSE | Privilege flag for doctor roles |
| `created_at` | TIMESTAMP | DEFAULT CURRENT | Account creation time |

### Patient_Identity
A 1:1 extension of the Users table for health-specific metadata.
| Column | Type | Constraints | Description |
| :--- | :--- | :--- | :--- |
| `identity_id` | INT | PK, AI | Internal ID |
| `user_id` | INT | FK, UNIQUE | Link to `Users.user_id` (on delete cascade) |
| `allergies` | TEXT | Nullable | Known medical allergies |
| `current_medications` | TEXT | Nullable | Currently active prescriptions |
| `emergency_contact_name` | VARCHAR(100) | Nullable | Primary contact person |
| `emergency_contact_phone` | VARCHAR(15) | Nullable | Primary contact phone |

---

## 📊 Medical Data Tables

### Medical_Records
The central fact table containing meta-information about every upload.
| Column | Type | Constraints | Description |
| :--- | :--- | :--- | :--- |
| `record_id` | INT | PK, AI | Unique Record ID |
| `user_id` | INT | FK, NOT NULL | The Patient who owns the record |
| `uploaded_by_user_id` | INT | FK | Who uploaded it (Doctor/Caregiver) |
| `record_type` | ENUM | NOT NULL | 'lab_report', 'prescription', 'clinical_note' |
| `doctor_name` | VARCHAR(100) | Nullable | Name of the visiting physician |
| `record_date` | DATE | NOT NULL | Date on the medical document |
| `file_path` | VARCHAR(255) | Nullable | Storage path for PDF/Image |

### Test_Results (Child of Medical_Records)
Stores specific numeric values extracted from lab reports by AI.
| Column | Type | Constraints | Description |
| :--- | :--- | :--- | :--- |
| `test_id` | INT | PK, AI | Unique Test ID |
| `record_id` | INT | FK, NOT NULL | Link to `Medical_Records.record_id` |
| `test_name` | VARCHAR(100) | NOT NULL | e.g., Glucose, Hemoglobin, TSH |
| `test_value` | DECIMAL(12,4) | NOT NULL | The numeric result |
| `unit` | VARCHAR(20) | Nullable | e.g., mg/dL, IU/L |

---

## 🔐 Access & Security

### Doctor_Access_Requests
Handles the workflow for temporary OTP-based doctor access.
| Column | Type | Constraints | Description |
| :--- | :--- | :--- | :--- |
| `request_id` | INT | PK, AI | Unique Request ID |
| `doctor_user_id` | INT | FK, NOT NULL | The Doctor requesting access |
| `patient_user_id` | INT | FK, NOT NULL | The Patient being requested |
| `status` | ENUM | DEFAULT 'pending' | 'pending','approved','rejected','expired' |
| `expires_at` | DATETIME | Nullable | Access cut-off time (e.g., +30 mins) |

### Audit_Log
An append-only trail of every time a record is viewed or edited.
| Column | Type | Constraints | Description |
| :--- | :--- | :--- | :--- |
| `log_id` | INT | PK, AI | Unique Log ID |
| `actor_user_id` | INT | FK, NOT NULL | Who performed the action |
| `target_user_id` | INT | FK, NOT NULL | Whose records were subject to action |
| `action` | ENUM | NOT NULL | 'view','upload','edit','delete','share' |
| `logged_at` | TIMESTAMP | DEFAULT CURRENT | Time of event |

---

## 🔗 Relationship Cardinality



!!! info "Legend"
    **1:1** (One-to-One) | **1:N** (One-to-Many) | **M:N** (Many-to-Many)

* **Users → Patient_Identity (1:1):** Every identity record belongs to one user.
* **Users → Medical_Records (1:N):** One user owns many records.
* **Medical_Records → Test_Results (1:N):** One lab report contains many specific test values.
* **Users ↔ Users (M:N via Family_Access):** A user can manage many family members, and be managed by many.
* **Users → Audit_Log (1:N):** One user's account can have many access events logged against it.