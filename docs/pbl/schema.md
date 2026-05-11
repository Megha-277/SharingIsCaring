# Database Schema Documentation

## 1. Schema Tables

### Users
**Core entity — every account**

| Column | Type | Constraints | Description |
|:---|:---|:---|:---|
| `user_id` | INT AI | **PK** | Primary key |
| `full_name` | VARCHAR(100) | NOT NULL | |
| `email` | VARCHAR(100) | UNIQUE NOT NULL | |
| `password_hash` | VARCHAR(255) | NOT NULL | bcrypt hash |
| `dob` | DATE | NOT NULL | |
| `gender` | ENUM | 'male','female','other' | |
| `blood_group` | VARCHAR(5) | Nullable | Stored here for fast emergency queries |
| `is_doctor` | BOOLEAN | DEFAULT FALSE | Privilege flag, not a separate role table |
| `created_at` | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | |

---

### Patient_Identity
**1:1 extension of Users**

| Column | Type | Constraints | Description |
|:---|:---|:---|:---|
| `identity_id` | INT AI | **PK** | Primary key |
| `user_id` | INT | **FK**, UNIQUE NOT NULL | → `Users(user_id)` ON DELETE CASCADE |
| `allergies` | TEXT | Nullable | |
| `current_medications` | TEXT | Nullable | |
| `emergency_contact_name` | VARCHAR(100) | Nullable | |
| `emergency_contact_phone`| VARCHAR(15) | Nullable | |

---

### Family_Access
**M:N self-join on Users**

| Column | Type | Constraints | Description |
|:---|:---|:---|:---|
| `family_access_id` | INT AI | **PK** | Primary key |
| `owner_user_id` | INT | **FK**, NOT NULL | → `Users(user_id)` ON DELETE CASCADE |
| `member_user_id` | INT | **FK**, NOT NULL | → `Users(user_id)` ON DELETE CASCADE |
| `relation` | ENUM | NOT NULL | 'parent','child','spouse','sibling','other' |
| `can_manage` | BOOLEAN | DEFAULT TRUE | |
| `granted_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |
| `(owner, member)` | UNIQUE KEY | **UQ** | Prevents duplicate access pairs |

---

### Medical_Records
**Central fact table**

| Column | Type | Constraints | Description |
|:---|:---|:---|:---|
| `record_id` | INT AI | **PK** | Primary key |
| `user_id` | INT | **FK**, NOT NULL | → `Users(user_id)` ON DELETE CASCADE (owner) |
| `uploaded_by_user_id` | INT | **FK** | → `Users(user_id)` ON DELETE SET NULL |
| `record_type` | ENUM | NOT NULL | 'lab_report','prescription','clinical_note' |
| `title` | VARCHAR(200) | Nullable | |
| `doctor_name` | VARCHAR(100) | Nullable | Denormalized for PBL scope |
| `hospital_name` | VARCHAR(100) | Nullable | |
| `record_date` | DATE | NOT NULL | |
| `file_path` | VARCHAR(255) | Nullable | Path to uploaded PDF/image |
| `file_mime_type` | VARCHAR(50) | Nullable | e.g. 'application/pdf', 'image/jpeg' |
| `notes` | TEXT | Nullable | |
| `created_at` | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | |

**Indexes:**
* **IDX**: `record_date` — INDEX for timeline sort
* **IDX**: `record_type` — INDEX for category filter
* **IDX**: `(user_id, record_date)` — COMPOSITE INDEX — primary query pattern

---

### Test_Results
**Child of Medical_Records**

| Column | Type | Constraints | Description |
|:---|:---|:---|:---|
| `test_id` | INT AI | **PK** | Primary key |
| `record_id` | INT | **FK**, NOT NULL | → `Medical_Records(record_id)` ON DELETE CASCADE |
| `test_name` | VARCHAR(100) | NOT NULL | |
| `test_value` | DECIMAL(12,4) | | Supports low-precision values like TSH |
| `unit` | VARCHAR(20) | Nullable | e.g. mg/dL, IU/L |
| `normal_min` | DECIMAL(12,4) | Nullable | |
| `normal_max` | DECIMAL(12,4) | Nullable | |

**Indexes:**
* **IDX**: `test_name` — INDEX for search by test name

---

### Prescriptions
**Child of Medical_Records**

| Column | Type | Constraints | Description |
|:---|:---|:---|:---|
| `prescription_id` | INT AI | **PK** | Primary key |
| `record_id` | INT | **FK**, NOT NULL | → `Medical_Records(record_id)` ON DELETE CASCADE |
| `medicine_name` | VARCHAR(100) | NOT NULL | |
| `dosage` | VARCHAR(100) | Nullable | e.g. 500mg twice daily |
| `duration` | VARCHAR(50) | Nullable | e.g. 7 days |

---

### Doctor_Access_Requests
**Access workflow with expiry**

| Column | Type | Constraints | Description |
|:---|:---|:---|:---|
| `request_id` | INT AI | **PK** | Primary key |
| `doctor_user_id` | INT | **FK**, NOT NULL | → `Users(user_id)` ON DELETE CASCADE |
| `patient_user_id` | INT | **FK**, NOT NULL | → `Users(user_id)` ON DELETE CASCADE |
| `request_message` | TEXT | Nullable | |
| `status` | ENUM | DEFAULT 'pending' | 'pending','approved','rejected','expired' |
| `requested_at` | DATETIME | DEFAULT CURRENT_TIMESTAMP | |
| `expires_at` | DATETIME | Nullable | NULL until approved; set on approval |
| `approved_by_user_id` | INT | **FK** | → `Users(user_id)` ON DELETE SET NULL (approver) |

---

### Audit_Log
**Append-only access trail**

| Column | Type | Constraints | Description |
|:---|:---|:---|:---|
| `log_id` | INT AI | **PK** | Primary key |
| `actor_user_id` | INT | **FK**, NOT NULL | → `Users(user_id)` — who performed action |
| `target_user_id` | INT | **FK**, NOT NULL | → `Users(user_id)` — whose records were accessed |
| `action` | ENUM | NOT NULL | 'view','upload','edit','delete','share' |
| `record_id` | INT | Nullable | Specific record; NULL for account-level actions |
| `logged_at` | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP | |

**Indexes:**
* **IDX**: `(target_user_id, logged_at)` — COMPOSITE INDEX — "who accessed my records" query

---

## 2. Relationship Cardinality & Participation

### Legend
* **PK** — Primary Key
* **FK** — Foreign Key
* **IDX / UQ** — Index or Unique Constraint
* **M:N** — Many-to-Many
* **1:N** — One-to-Many
* **1:1** — One-to-One

### Cardinality Table

| Entities | Ratio | Participation | Meaning |
|:---|:---|:---|:---|
| **Users → Patient_Identity** | 1 : 1 | Total (Identity), Partial (Users) | Every identity record must belong to exactly one user. A user may or may not have created their identity record yet. |
| **Users → Medical_Records (owner)** | 1 : N | Partial both sides | One user can own zero or many medical records. Each record belongs to exactly one user. |
| **Users → Medical_Records (uploader)** | 1 : N | Partial both sides | One user (doctor/caregiver) can upload zero or many records. Nullable — SET NULL on delete. |
| **Medical_Records → Test_Results** | 1 : N | Partial both sides | One lab record can contain many test values. Not all records have test results. |
| **Medical_Records → Prescriptions** | 1 : N | Partial both sides | One prescription record can list many medicines. Only 'prescription' type uses this. |
| **Users ↔ Users (Family_Access)** | M : N | Partial both sides | Junction table for self-join. A user can grant/receive access to many others. |
| **Users → Requests (doctor)** | 1 : N | Partial both sides | One doctor (is_doctor = TRUE) can send many access requests. |
| **Users → Requests (patient)** | 1 : N | Partial both sides | One patient can receive many access requests from different doctors. |
| **Users → Requests (approver)** | 1 : N | Partial both sides | One user (patient/caregiver) can approve many requests. Nullable until approved. |
| **Users → Audit_Log (actor)** | 1 : N | Partial both sides | One user can perform many logged actions. |
| **Users → Audit_Log (target)** | 1 : N | Partial both sides | One user's account can be the subject of many logged access events. |


<iframe src="../diagram.html" width="100%" height="800px" frameborder="0"></iframe>