PATH: /user-stories/US-016-manage-users.md

# 🧾 User Story: Manage Internal Users

## 🆔 Metadata
- ID: US-016
- Feature: Basic Administration
- Priority: High
- Status: Draft
- Owner: PM

## 🎯 User Story
**As an** Admin  
**I want** to create, edit, and deactivate internal users of the ATS  
**So that** I can control who participates in the recruitment process and with what level of access

## 🧠 Business Context
- Related Use Case(s): Gestionar Usuarios (UC16)
- Entities involved: User, Company, Role (enum on User)
- Business Rules:
  - Only users with role Admin may create, edit, or deactivate other users.
  - User `email` must be unique within a Company.
  - A deactivated user (`status = Inactive`) cannot log in or perform any operations.
  - Deactivation is a soft operation; the User record and all historical data authored by that user are preserved.
  - An Admin cannot deactivate themselves.
- Assumptions:
  - Users are internal to the company; candidates are separate entities and not managed here.
  - Password management and authentication flow are outside the scope of this story (handled by AuthModule).
- Dependencies:
  - Company must be Active.
  - Admin user performing the action must have `status = Active`.

## ✅ Acceptance Criteria (BDD + Edge Cases)

### 🎯 Happy Path
- **Given** an authenticated Admin for an Active Company  
  **When** the Admin creates a new user with `fullName`, `email`, and a valid `role`  
  **Then** a User record is persisted with `status = Active`, scoped to the Admin's `companyId`, and the new user can log in and operate according to their assigned role

### ⚠️ Edge Cases

#### Case 1: Duplicate email within the same Company
- **Given** an Admin attempts to create a user with an email already registered in the same Company  
  **When** the creation request is submitted  
  **Then** the system returns a 409 Conflict error indicating the email is already in use  
- Handling:
  - Enforce unique constraint on `(companyId, email)`. Surface a clear message to the Admin.

#### Case 2: Admin attempts to deactivate themselves
- **Given** an authenticated Admin  
  **When** the Admin submits a deactivation request for their own User ID  
  **Then** the system returns a 422 Unprocessable Entity error preventing self-deactivation  
- Handling:
  - Block self-deactivation to prevent lockout. Display a clear error message.

#### Case 3: Deactivating a user who owns active job openings
- **Given** an Admin deactivates a Recruiter who created open JobOpenings  
  **When** the deactivation is processed  
  **Then** the User is deactivated, their JobOpenings remain Open, and ownership records are preserved (`createdByUserId` remains unchanged)  
- Handling:
  - Deactivation is independent of authored records; ownership references are read-only historical data.

#### Case 4: Unauthorized role attempts user management
- **Given** an authenticated user with role Recruiter  
  **When** the user attempts to create or deactivate another user  
  **Then** the system returns a 403 Forbidden error  
- Handling:
  - Only Admin role may access user management endpoints.

### 🚫 Validation Rules
- `email` must pass EmailAddress Value Object validation.
- `email` must be unique within the same `companyId`.
- `fullName` must not be blank.
- `role` must be one of: `Admin`, `Recruiter`, `HiringManager`, `Interviewer`.
- Self-deactivation is forbidden.
- Performing user must have role `Admin` and `status = Active`.

## 📊 Business Impact
- KPI Affected: System access control, Onboarding speed for new recruiters, Security compliance
- Expected Impact: Ensures the right people have the right access at every stage of the hiring process, reducing unauthorized actions and data exposure
- Success Criteria:
  - New users can be onboarded and operational within the same Admin session
  - Deactivated users are blocked from all operations within the same request cycle after deactivation
