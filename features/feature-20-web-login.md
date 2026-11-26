# Feature 20: Regular Web Login (Management Portal)

## User Stories

### US-20.1: Manager Web Login 
**As a** warehouse manager 
**I want to** log into the web application using my company email and password 
**So that** I can access the maangement dashboard, inventory_reports, and supplier ordering tools from a desktop browser

**Priority:** P1 
**Indpendent test:** Verify a manager with a valid email and password credentials is authenticated, issued a web session, and routed to the Management Dashboard. 
**Acceptance scenarios:** see ### US-20.1 under Gherkin AC 

### US-20.2: Web Portal Role Enforcement 
**As a** warehouse manager
**I want** the web portal to verify user roles upon login 
**So that** floor workers who only hold scanner permissions cannot access management-only web pages or APIs. 

**Priority:** P1
**Independent test:** Verify that a standard floor worker attempting to log into the management web portal receives a `403 Forbidden` response and cannot view management screens.
**Acceptance scenarios:** see ### US-20.2 under Gherkin AC 

### US-20.3: Web Session Logout
**As a** warehouse manager
**I want to** log out of the web portal when I finish my shift 
**So that** my account remains secure on shared office computers

**Priority:** P2 
**Independent test:** Verify clicking "Log Out" invalidates the active web session token and redirects the browser to the web login screen. 
**Acceptance scenarios:** see ### US-20.3 under Gherkin AC 

## Functional Requirements 
* The system must authenticate web users via a unique company `email` and securely hashed `password`.
* The system must issue a secure, HTTP-only session upon authentication and attatch the user's corresponding `role` (`Manager` or `Admin`) to the session. 
* The system must deny web portal access to accounts that do not have a `Manager` or `Admin` role, returning `403 Forbidden`.
* The system must invalidate the active web session immediately upon logout and redirect unauthenticated requests to the login view. 
* The system must log `login_time` and `logout_time` for auditability.

## Assumptions 
* Managers already have active accounts in the `Worker` and `WorkerAuthorization` tables with an assigned company email.
* The web application is served over HTTPS so credentials and session cookies are encrypted.

## Edge Cases 
**Invalid Credentials:** If the email does not exist or the password is incorrect, the system returns `401 Unauthorized` with a generic `"Invalid email or password"` message (never revealing which field was wrong).
**Deactivated Account:** If a former manager's account is marked inactive, loginis blocked with `403 Forbidden`.
**Expired Session:** If a manager's session expires while idle, the next API call returns `401 Unauthorized` and the UI redirects to `/web-login`.

## Success Criteria 
* Managersr can log in via desktop browser using email and password and reach the Management Dashboard.
* Invalid credentials and unauthorized roles are cleanly rejected without exposing system details. 
* Logging out destroys the session so the browser Back button cannot reload pages.

## Key Entities 
* **Manager/Worker**
* **WorkerAuthorization** 
* **WebSession**

## Initial Data Model 

### WorkerAuthorization (Extended from Feature 13)

| Field Name | Data Type | Constraints | Description | 
|---|---|---|---|
| `worker_id` | Integer | PK, FK, Not Null | Foreign key to the Worker table | 
| `email` | String | Unique, Not Null | Company email used for regular web login | 
| `password_hash` | String | Not Null | Securely hashed password | 
| `role` | String | Not Null | Access level (`Worker`, `Manager`, `Admin`) |
| `is_active` | Boolean | Not Null, Default: true | Indicates whether the account can log in | 

## WebSession 
| Field Name | Data Type | Constraints | Description | 
|---|---|---|---|
|`session_id` | integer | PK, Not Null | primary key for the web session | 
| `worker_id` | Integer | Fk, Not Null | Foreign key referencing the authenticated user | 
| `token_hash` | String | Not Null | Hashed session token |
| `login_time` | DateTime | Not Null | Exact date and time the web session started | 
| `expires_at` | DateTime | Not Null | Expiration timestamp for idle timeout | 
| `logout_time` | DateTime | Nullable | Exact date and time the user logged out (`NULL` while active) |

## Gherkin AC 
### US-20.1 
**Scenario: Manager logs into the web portal** 
**Given** a manager is on the web loggin screen 
**When** the manager enters a valid company email and password and cliicks "Sign In" 
**Then** the system creates a new `WebSession`
**And** redirects the manager to the Management Dashboard. 

### US-20.2 
**Scenario: Standard worker attempts to log into the management web portal** 
**Given** a user with the `Worker` role is on the Web Login screen 
**When** the user submits valid credentials 
**Then** the system rejects the request with `403 Forbidden`
**And** displays an error stating they do not ahve web portal permissions. 

### US-20.3 
**Scenario: Manager logs out of the web portal**
**Given** a manager is logged into an active web session 
**When** the manager clicks "Log Out" in the navigation bar 
**Then** the system records `logout_time`, clears the session token, and redirects to the web login screen. 

## Data Ownership & Isolation 

| Rule | Requirement |
|---|---|
| `POST` | `/api/auth/web/login` | No | Validates email/password and role, creates a `WebSession`, and returns user profile | 
| `POST` | `/api/auth/web/logout` | Yes | Invalidates the active `WebSession` and records `logout_time` | 
| `GET` | `/api/auth/me` | yes | Returns the authenticated user's `worker_id`, `email`, and `role` for UI rendering |

**Login request body:** 
```json 
{
    "email":"manager@warehouse.com",
    "password" : "SecurePassword123!"
}
```
**Success response** 
```json
{
    "worker_id":101, 
    "email": "manager@warehouse.com",
    "role" : "Manager", 
    "message" : "Login successful." 
}
```
## Screen Requirements 

### [View: manager Web Login] - route name `web-login`
* Heading / purpose: Warehouse Management Portal Sign In 
* Primary action: **Sign In** (`oc-cta`)
* Fields: `Email`(email input), `Password` (password input with show/hide toggle)
* Loading / error: Loading state on submit button; `<v-alert type = "error">` for invalid credentials or unauthorized role

### [View: Management Dashboard] - route name `manager-Dashboard`
* Heading / Purpose: Warehouse Management Dashboard 
* Secondary action: **Log Out** button in the top navigation bar 

## Test Coverage Map 

| Story | Scenario | Test file | Test name | 
|---|---|---|---|
| US-20.1 | Manager logs into the web portal | `backend/tests/web_auth.test.js` | `System authenticates valid manager and creates WebSession`|
| US-20.1 | Invalid credentials rejected | `backend/tests/web_auth.test.js` | `System rejects invalid email or password with 401` |
| US-20.2 | Standard worker attempts web portal login | `backend/tests/web_auth.test.js` | `System blocks standard worker from web portal with 403` | 
| US-20.3 | Manager logs out of the web portal | `backend/tests/web_auth.test.js` | `System invalidates WebSession on logout` | 

## Agent Implementation Request

```text
Implement Feature 20 from @features/feature-20-web-login.md on branch `feature/20-web-login`.

Follow layer order in @features/framework.md (models -> routes -> backend tests -> frontend -> frontend tests). 
Map every Gherkin scenario in the Test Coverage Map; run `npm test` before finishing. Ensure `POST /api/auth/web/login` enforces role checks and never leaks whether the email or password failed. 
```

## Definition of Done 
- [ ] Backend and frontend implemented per this spec 
- [ ] Success Criteria met (`npm test`)
- [ ] All mapped tests pass (`npm test`)
- [ ] Test Coverage Map complete
- [ ] `features/reference/data-model.md` updated (Added `WebSession` and `email` on `WorkerAuthorization`)
- [ ] `features/reference/api.md` updated (Added web auth endpoints)
- [ ] `features/reference/behavior.md` updated (Added web login)

## Out of Scope 
* Self-service password reset via email 
* Single Sign-On (SSO/OAuth) or Two-Factor Authentication 


