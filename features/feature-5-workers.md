# Feature 5: Worker Data Management 

## User Stories 

### US-5.1: Add New Worker 
**As a** warehouse manager
**I want to** add a new worker profile to the system 
**So that** new hires can be assigned to tasks and log into the warehouse system. 

**Priority:** P1 
**Independent test:** Verify that a manager can create a new worker profile with a name, job role, and shift, and it saves to the database.
**Acceptance scenarios:** see ### US-5.1 under Gherkin AC 

### US-5.2: Update Worker Profile 
**As a** warehouse manager 
**I want to** update an existing worker's details or role 
**So that** the system reflects promotions, shift changes, or new job tasks. 

**Priority:** P2 
**Independent test:** Verify a manager can edit a worker's role and the system saves the new information. 
**Acceptance scenarios:** see ### US-5.2 under Gherkin AC 

### US-5.3: Deactive Worker Profile 
**As a** warehouse manager 
**I want to** deactivate a worker's profile 
**So that** former employees no longer have access to the warehouse system. 

**Priority:** P2 
**Independent test:** Verify a manager can deactivate a worker profile, changing its status to `Inactive` and removing it from the active roster.  
**Acceptance scenarios:** see ### US-5.3 under Gherkin AC 

## Functional Requirements 
* The system must allow managers to add new workers to the roster with a full name, job role, and shift. 
* The system must allow managers to update a worker's details such as `job_role` or `shift`. 
* The system must allow managers to deactivate workers by setting `status` to `Inactive` rather than permanently deleting the record, preserving historical audit logs. 
* Deactivating a worker must prevent them from logging into RF scanners (Feature 13) or the web portal (Feature 20).

## Assumptions 
* Standard floor workers cannot create, edit, or deactivate worker profiles. 
* Deactivated workers remain in the database for historical traceability across `PickTask`, `ReceivingLog`, and `InventoryMovement`. 

## Edge Cases 
* **Deactivating a Worker with Active Tasks:** If a worker currently has an `In Progress` `PickTask` or is assigned to an active `Route`, the system warns the manager to reassign those tasks before or immediately after deactivation. 
* **Self-Deactivation:** A manager cannot deactivate their own active account (`400 Bad Request`).

## Success Criteria 
* Managers can create, view, update, and deactivate worker profiles cleanly. 
* Deactivated workers disappear from the active roster immediately and are blocked from authenticating. 


## Key Entities 
* **Worker** 
* **Manager** 

## Initial Data Model 

### Worker 

| Field Name | Data Type | Constraints | Description | 
|---|---|---|---| 
| `worder_id` | Integer | PK, Not Null | Primary key for the worker |
| `full_name` | String | Not Null | The full name of the worker | 
| `job_role` | String | Not Null | ENUM: Picker, Receiver, Driver, Manager |
| `shift` | String | Nullable | Assigned work shift (e.g., Day, Swing, Night) |
| `status` | String | Not Null, Default: `Active` | ENUM: `Active`, `Inactive` |

## Gherkin AC 
### US-5.1 
**Scenario: Manager adds a new worker**
**Given** a manager is on the Workers management screen 
**When** the manager enters a new worker's name, job role, and shift
**And** clicks "Save" 
**Then** the system creates a new worker profile with status `Active`
**And** the worker appears in the active roster. 

### US-5.2 
**Scenario: Manager updates a worker role** 
**Given** a worker profile exists in the system 
**When** the manager changes the worker's role from "Picker" to "Receiver" 
**And** clicks "Update" 
**Then** the system should save the new role for that worker

### US-5.3
**Scenario: Manager deactivates a worker**
**Given** a worker profile exists in the system 
**When** the manager clicks "Deactivate" on the worker profile 
**Then** the system changes the worker's status to `Inactive`
**And** the worker is removed from the active roster 

## Data Ownership & Isolation 
| Rule | Requirement | 
|---|---|
| **Read scope** | Managers can view all active and inactive workers. Workers can only view their own profile (`/api/auth/me`). |
| **Write scope** | Only Managers can update or deactivate worker profiles. | 
| **Create scope** | Only Managers can create new worker profiles. | 
| **Cross-user-access** | Standard worker attempting POST/PATCH -> `403 Forbidden`. |
| **UI scope** | Worker Management Dashboard is only visible to Managers. | 
| **Implementation** | Role-based access control (RBAC) enforced via shared helper in `app/authorization/`. |

## API Requirements 

| Method | Endpoint | Auth | Purpose | 
|---|---|---|---|
| `GET` | `/api/workers` | Yes (Manager) | Fetches worker roster (filterable by `status` and `job_role`) |
| `POST` | `/api/workers` | Yes (Manager) | Creates a new worker profile | 
| `PATCH` | `/api/workers/{worker_id}` | Yes (Manager) | Updates worker name, job role, or shift |
| `PATCH` | `/api/workers/{worker_id}/deactivate` | Yes (Manager) | Sets worker `status` to `Inactive` | 

## Screen Requirements 
### [View: Worker Management Dashboard] - route name `manage-workers`
* Heading / purpose: Manage Warehouse Workers
* Primary action: **Add New Worker** (`oc-cta`)
* Secondary actions: **Edit**, **Deactivate**
* Fields: Full Name, Job Role Dropdown, Shift Dropdown, Status Filter (`Active` / `Inactive`)
* **Empty state:** "No active workers found."
* Loading / error: `<v-alert type = "error">` if a manager attempts self-deactivation or validation fails.

## Test Coverage Map 
| Story | Scenario | Test file | Test name | 
|---|---|---|---|
| US-5.1 | Manager adds a new worker | `backend/tests/workers.test.js` | `System creates a new active worker profile` | 
| US-5.2 | Manager updates a worker role | `backend/tests/workers/test.js` | `System updates existing worker role and shift` | 
| US-5.3 | Manager deactivates a worker | `backend/tests/workers.test.js` | `System sets worker status to Inactive on deactivate` | 


## Agent Implementation Request 
``` text 
Implement Feature 5 from @features/feature-5-workers.md on branch `feature/5-workers`.

Follow layer order in @features/framework.md (models -> routes -> backend tests -> frontend -> frontend tests). 
Map eveyr Gherkin scenario in the TEst Coverage Map; run `npm test` before finishing. 
```

## Definition of Done 
* [ ] Backend and frontend implemented per this spec 
* [ ] Success Criteria met 
* [ ] All mapped tests pass (`npm test`) 
* [ ] Test Coverage Map complete 
* [ ] `features/reference/data-model.md` updated (Added Worker table)
* [ ] `features/reference/api.md` updated (Added worker CRUD endpoints)

## Out of Scope 
* Payroll, hourly wage tracking, or timecard clock-in/clock-out
* HR onboarding documents or tax forms. 


