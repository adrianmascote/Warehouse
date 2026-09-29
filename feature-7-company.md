# Feature 7: Company Data Management 

## User Stories 

### US-7.1: Add Company Profile 
**As a** system administrator 
**I want to** add a new company profile 
**So that** the business is removed from the system if the account is closed. 

**Priority:** P1 
**Independent test:** Verify an admin can create a new company profile with a name and tax ID. 
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
 * The system must allow admins to create a company profile 
 * The system must allow admins to update the company's name, address, and contact details. 
 * The syste mmust allow admins to delete a company profile. 

 ## Key Entities 
 * Company 
 * Administrator 

 ## Initial Data Model
 * **Company** 
 * 'company_id' (Primary Key)
 * 'company_name' (String)
 * 'headquarters_address' (String)
 * 'contact_email' (String)

 ## Gherkin AC 

 ### US-7.1 
 **Scenario: Admin adds a company** 
 **Given** an admin os on the Company setup screen
 **when** the admin enters the company name and address 
 **And** clicks "Save" 
 **Then** the system should create a new company profile in the database. 

 ### US-7.2
 **Scenario: Admin updates company information** 
 **Given** a company profile exists 
 **When** the admin updates the address
 **And** clicks "Update" 
 **Then** the system should save the new address for the company. 

 ### US-7.3 
 **Scenario: Admin delets a company** 
 **Given** a comppany profile exists
 **When** the admin clicks "Delete" and confirms
 **Then** the system should remove the company profile completely. 





