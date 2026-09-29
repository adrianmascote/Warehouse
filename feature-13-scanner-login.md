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
**I want to** log out fo the scanner at the end of my shift 
**So that** the next worker can use the same physical device under their profile 

**Priority:** P2 
**Independent test:** Verify logging out securely ends the session and returns to the login screen. 
**Acceptance scenarios:** see ### US-13.3 under Gherkin AC 

## Functional Requirements
* The system must authenticate users via a Worker ID and a secure Password
* The system must track the active session, link all scanned actions to the Worker ID. 
* The system must follow role-based access protocols, differentiating between a "Worker" and "Manager" 
* The system must allow users to terminate their session safely. 

## Key Entities 

* Worker 
* Device (Scanner)
* Session

## Initial Data Model 
* **WorkerAuth** 
* 'worker_id' (Foreign Key linked to Worker)
* 'password_hash' (String)
* 'role' (String - e.g., "Standard", "Manager")
* **DeviceSession** 
* 'session_id' (Primary Key)
* 'worker_id' (Foreign Key)
* 'login_time' (DateTime)

## Gherkin AC 

### US-13.1 
**Scenario: Worker successfully logs into scanner** 
**Given** the worker is on the scanner login screen
**When** the worker enters a valid Worker ID and Password
**And** clicks "Login" 
**Then** the system should grant access and display the task menu. 

### US-13.2 
**Scenario: Worker attempts to access restricted area** 
**Given** a user with a "Standard" role is logged in
**When** the user attempts to open the Manager reporting screen 
**Then** the system should display an "Access Denied" Error message


### US-13.3 
**Scenario: Worker logs out at end of shift** 
**Given** a worker is logged into an active scanner session
**When** the worker clicks "Log Out" 
**Then** the system should end the session and return to the main login screen. 