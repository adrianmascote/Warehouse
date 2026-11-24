# Feature 13: Scanner Login & Worker Authentication 

## User Stories 

### US-13.1 Worker Login
**As a** warehouse worker
**I want to** log into an RF scanner using my Worker ID and PIN 
**So that** the system can track the asks I complete during my shift 
 
**Priority:** P1 
**Indpendent test:** Verify the system authenticates valid worker credentials and grants access to the scanner dashboard. 
**Acceptance scenarios:** see ### US-13.1 under Gherkin AC 

### US-13.2: Role-Based Access Control 
**As a** warehouse manager 
**I want to** restrict scanner functions based on the worker's role
**So that** standard workers cannot access or alter management reporting. 

**Priority:** P1
**Indpendent test:** Verify that the user with a "Worker" role is denied access to management-only screens. 
**Acceptance scenarios:** see ### US-13.2 under Gherkin A

### US-13.3: Worker Logout 
**As a** warehouse worker
**I want to** log out of the scanner at the end of my shift 
**So that** the next worker can use the same physical device under their profile 

**Priority:** P2 
**Independent test:** Verify logging out securely ends the session and returns to the login screen. 
**Acceptance scenarios:** see ### US-13.3 under Gherkin AC 

## Functional Requirements
* The system must authenticate users via a Worker ID and a secure Password
* The system must track the active session, link all scanned actions to the Worker ID. 
* The system must follow role-based access protocols, differentiating between a "Worker" and "Manager" 
* The system must allow users to terminate their session safely. 

## Assumptions 
* Workers already have active profile made in the system by human resources.
* The scanners are connected to the warehouse's internal network/Wi-Fi.

## Success Criteria 
* Valid credentials start a session and route the user to the main menu
* Invalid credentials display a clear error without revealing if the ID or password was wrong. 
* Active session tokens are destroyed upon logout. 

## Key Entities 

* Worker 
* DeviceSession

## Initial Data Model 

### WorkerAuthorization 

| Field Name | Data Type | Constraints | Description | 
|---|---|---|---|
| `worker_id` | Integer | PK,FK, Not Null | for auth, Foreign key to the Worker Table | 
| `password_hash` | String | Not Null | The securely hashed password | 
| `role` | String | Not Null | The system access level (e.g., Worker, Manager) |

### DeciveSession 

| Field Name | Data Type | Constraints | Description |
|---|---|---|
| `session_id` | Integer | PK, Not Null  Primary key for the device session |   
| `worker_id` | Integer | FK, Not Null | Foreign key referencing teh Worker logged in | 
| `login_time` | DateTime | PK, Not Null | The exact date and time the session started | 
| `logout_time` | DateTime | Nullable | The exact date and time the session ended (Null whle active) |




## Gherkin AC 

### US-13.1 
**Scenario:** Worker successfully logs into scanner 
**Given** the worker is on the scanner login screen
**When** the worker enters a valid Worker ID and Password
**And** clicks "Login" 
**Then** the system should grant access and display the task menu. 

### US-13.2 
**Scenario: Worker attempts to access restricted area** 
**Given** a user with a "Worker" role is logged in
**When** the user attempts to open the user attempts to open the Manager reporting screen 
**Then** the system should display an "Access Denied" Error message


### US-13.3 
**Scenario: Worker logs out at end of shift** 
**Given:** a worker is logged into an active scanner session
**When:** the worker clicks "Log Out" 
**Then:** the system should end the session and return to the main login screen. 


## Data Ownership & Isolaton

| Rule | Requirement | 
|---|---|
| **Read scope** | Workers can only view their own active session details | 
| **Write scope** | Only when row matches `id` and `req.user.id` | 
| **Create scope** | New sessions are created when authentication is valid | 
| **Cross-user-access** | Another user's resource -> `404` (not `403`) | 
| **UI scope** | UI restricts navigation based on the `role` returned in the session |


## API Requirements 

| Method | Endpoint | Auth | Purpose | 
|---|---|---|---|
| `POST` | `/api/auth/login` | No | Vaidates credentials and creates a session | 
| `POST` | `/api/auth/logout` | Yes | Destroys the active session and instantiates a `logout_time` | 
| `GET` | `/api/auth/me` | Yes | Fetches the current user's profile and role for UI rendering | 

## Screen Requirements 

### [View: Scanner Login] - route name `login` 
* Heading / purpose: Worker Authentication 
* Primary action: **Login** (`oc-cta` when it is the primary labled )
* Fields: `Worker ID` (text input), `Password` (password input)
* Loading / error: `<v-alert type="error">` for "Invalid Credentials" 

### [View: Main Menu Dashboard] - route name `login`
* Heading / purpose: Main Task Menu
* Primary action: **Select Task** (`oc-cta` when it is primary labeled CTA)
* Secondary action: **Log out** 

## Test Coverage Map
| Story | Scenario | Test file | Test name | 
|---|---|---|---|
| US-13.1 | Worker successfully logs into scanner | `backend/tests.auth.test.js` | `System authenticates valid worker` | 
| US-13.2 | Worker attempts to access restricted area | `backend/tests/auth.test.js` | `System blocks worker from manager views` | 
| US-13.3 | Worker logs out at end of shift | `backend/tests/auth.test.js` | `System terminates session on logout` | 

## Agent Implementation request 

```text 
Implement Feature 13 from @features/feature-13-scanner-login.md on branch `feature/13-scanner-login`. 

Follow layer order in @features/framework.md (models -> routes -> backend tests -> frontend -> frontend tests).
Map every Gherkin scenarion in the TEst Coverage Map; run `npm test` to verify the order lifecycle strict status transitions. 
```

## Definition of Done 
* [ ] Backend and frontend implemented per this spec (FR-00N satisfied)
* [ ] Success Criteria (SC-00N) met 
* [ ] All mapped tests pass (`npm test`)
* [ ] Test Coverage Map complete 
* [ ] `features/reference/data-model.md` updated (if schema changed)
* [ ] `features/reference/api.md` updated (if API changed)
* [ ] `features/reference/behavior.md` updated (if product rules changed)

## Out of Scope 
* Biometric login (fingerprint / FaceID)
* Two-factor authentication (2FA) via SMS or email 


