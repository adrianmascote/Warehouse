# Feature 7: Company Data Management 

## User Stories 

### US-7.1: Add Company Profile 
**As a** system administrator 
**I want to** add a new company profile 
**So that** warehouse operations and records are linked to the correct business entity.

**Priority:** P1 
**Independent test:** Verify an admin can create a new company profile with a name, tax ID, address, and contact email. 
**Acceptance scenarios:** see ### US-7.1 under Gherkin AC 

### US-7.2: Update Company Profile 
**As a** system administrator 
**I want to** update the company's information 
**So that** the system reflects the correct contact details and address if the business moves. 

**Priority:** P2 
**Independent test:** Verify an admin can edit the company's address and the system saves the update. 
**Acceptance scenarios:** see ### US-7.2 under Gherkin AC 

### US-7.3: Delete Company Profile 
**As a** system administrator
**I want to** delete a company profile 
**So that** the business is removed from the system if the company's account is closed. 

**Priority:** P3
**Independent test:** Verify an admin can delete a company profile from the database completely. 
**Acceptance scenarios:** see ### US-7.3 under Gherkin AC 

## Functional Requirements 
* The system must allow admins to create a company profile with a name, tax ID, headquarters address, and contact email.
* The system must allow admins to update the company's name, tax ID, address, and contact details. 
* The system must allow admins to delete a company profile if it is not tied to active warehouse records.

## Assumptions 
* Only users with the `Admin` (or `Manager`) role can access company configuration settings 
* Each company profile has a unique `tax_id`.

## Edge Cases
* **Duplicate Tax ID:** Attempting to create a company with an existing `tax_id` returns `409 Conflict`. 
* **Active Reference Protection:** Attempting to delete a company that is still linked to active warehouses or orders is blocked with `409 Conflict`.

## Success Criteria 
* Admins can perform full CRUD operations on company profiles.
* Updated company details are immediately reflected across the system. 

## Key Entities 
* **Company**
* **Administrator** 

## Initial Data Model 

### Company 
| Field Name | Data Type | Constraints | Description | 
|---|---|---|
| `company_id` | Integer | PK, Not Null | Primary key for the company |
| `company_name` | String | The name of the company | 
| `tax_id` | String | Unique, Not Null | Business tax identification number |
| `headquarters_address` | String | Not Null | The headquarters address of the company | 
| `contact_email` | String | Not Null | The primary contact email | 


## Gherkin AC 

### US-7.1 
**Scenario: Admin adds a company** 
**Given** an admin is on the Company setup screen
**When** the admin enters the company name, tax ID, address, and contact email
**And** clicks "Save" 
**Then** the system should create a new company profile in the database. 

### US-7.2
**Scenario: Admin updates company information** 
**Given** a company profile exists 
**When**the admin updates the address
**And** clicks "Update" 
**Then** the system should save the new address for the company. 

### US-7.3 
**Scenario: Admin delets a company** 
**Given** an unreferenced company profile exists
**When** the admin clicks "Delete" and confirms
**Then** the system should remove the company profile from the database.

## Data Ownership & Isolation 

| Rule | Requirement | 
|---|---|
| **Read scope** | Admins and Managers can view company profile details. | 
| **Write scope** | Only Admins can update or delete company profiles. | 
| **Create scope** | Only Admins can create new company profiles. | 
| **Cross-user-access** | Standard worker attempting access -> `403 Forbidden`. | 
| **UI scope** | Company Settings view is only visible to Admins. |
| **Implementation** | Role-based access control (RBAC) enforced via shared helper in `app/authorization/`. |

## API Requirements 

| Method | Endpoint | Auth | Purpose | 
|---|---|---|---|
| `GET` | `/api/companies` | Yes (Admin/Manager) | Fetches company profile details | 
| `POST` | `/api/companies` | Yes (Admin) | Creates a new company profile |
| `PATCH` | `/api/companies/{company_id}` | Yes (Admin) | Updates company name, tax ID, address, or email | 
| `DELETE` | `/api/companies/{company_id}` | Yes (Admin) | Deletes a company profile (if unreferenced) |

## Screen Requirements 

### [View: Company Settings] - route name `manage-company`
* Heading / purpose: Company Profile Management 
* Primary actions: **Save Company** (`oc-cta`)
* Secondary actions: **Edit**, **Delete** 
Fields: Company Name, Tax ID, Headquarters Address, Contact Email 
* **Empty state:** "No company profile configured."
* Loading / error: `<v-alert type="error">` for duplicate Tax ID or deletion conflicts.

## Test Coverage Map 
| Story | Scenario | Teset file | Test name | 
|---|---|---|---|
| US-7.1 | Admin adds a company | `backend/tests/company.test.js` | `System creates a new company profile` | 
| US-7.2 | Admin updates company information | `backend/tests/company.test.js` | `System deletes unreferenced company profile` |

## Agent Implementation Request
```text 
Implement Feature 7 from @features/feature-7-company.md on branch `feature/7-company`.

Follow layer order in @features/feature-7-company.md (models -> routes -> backend tests -> frontend -> frontend tests).
Map every Gherkin scenario in the Test Coverage Map; run `npm test` before finishing.
```

## Definition of Done 
* [ ] Backend and frontend implemented per this spec 
* [ ] Success Criteria met 
* [ ] All mapped tests pass (`npm test`)
* [ ] Test Coverage Map complete 
* [ ] `features/reference/data-model.md` updated (Added Company table)
* [ ] `features/reference/api.md` updated (Added company CRUD endpoints)

## Out of Scope 
* Multi-tenant billing or subscription management 
* Automated tax filing or accounting integrations






