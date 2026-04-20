PATH: /user-stories/US-017-assign-roles-permissions.md

# 🧾 User Story: Assign Roles and Basic Permissions

## 🆔 Metadata
- ID: US-017
- Feature: Basic Administration
- Priority: High
- Status: Draft
- Owner: PM

## 🎯 User Story
**As an** Admin  
**I want** to assign or change the role of an existing internal user  
**So that** each user's functional permissions are accurately reflect their responsibilities in the recruitment process

## 🧠 Business Context
- Related Use Case(s): Asignar Roles y Permisos Básicos (UC17)
- Entities involved: User, Company
- Business Rules:
  - Only Admin role can assign or change roles for other users.
  - In MVP, each User has exactly one role from the enum: `Admin`, `Recruiter`, `HiringManager`, `Interviewer`.
  - Role changes take effect immediately on the user's next authenticated action (JWT re-issuance required for active sessions).
  - An Admin cannot downgrade their own role to a non-Admin role.
  - The target User must exist and belong to the same Company as the Admin.
- Assumptions:
  - Role drives functional permissions (not just UI visibility); the backend enforces role checks on every protected endpoint.
  - Granular permissions per resource are a post-MVP enhancement (documented as technical debt).
- Dependencies:
  - User must exist (US-016).
  - Admin performing the action must have `status = Active`.

## ✅ Acceptance Criteria (BDD + Edge Cases)

### 🎯 Happy Path
- **Given** an authenticated Admin and an existing Active User in the same Company  
  **When** the Admin changes the User's role from `Recruiter` to `HiringManager`  
  **Then** `User.role` is updated to `HiringManager`, the change is persisted, and the affected user's permissions are governed by the new role on subsequent authenticated requests

### ⚠️ Edge Cases

#### Case 1: Admin attempts to downgrade their own role
- **Given** an authenticated Admin  
  **When** the Admin submits a role change for their own User ID to `Recruiter`  
  **Then** the system returns a 422 Unprocessable Entity error preventing role self-downgrade  
- Handling:
  - Prevent Admins from removing their own Admin role to avoid lockout. Display a clear error message.

#### Case 2: Target user belongs to a different Company
- **Given** an Admin provides a userId from a different Company  
  **When** the role assignment is submitted  
  **Then** the system returns a 404 Not Found error  
- Handling:
  - Enforce companyId scoping on every user lookup for role changes.

#### Case 3: Assigning an invalid role value
- **Given** an Admin submits a role assignment with value `SuperAdmin` (not in enum)  
  **When** the request is processed  
  **Then** the system returns a 400 validation error listing valid role values  
- Handling:
  - Validate `role` against the enum `(Admin, Recruiter, HiringManager, Interviewer)` before persisting.

#### Case 4: Role change for an Inactive user
- **Given** an Admin attempts to change the role of an Inactive user  
  **When** the role assignment is submitted  
  **Then** the system returns a 422 error indicating that roles cannot be changed for inactive users  
- Handling:
  - Reactivate the user first (via US-016 edit) before changing their role.

### 🚫 Validation Rules
- `role` must be one of: `Admin`, `Recruiter`, `HiringManager`, `Interviewer`.
- Target user must have `status = Active` to have their role changed.
- Target user must belong to the same `companyId` as the Admin.
- An Admin may not change their own role to a non-Admin value.
- Role change takes effect server-side immediately; active JWT sessions may retain old role until token refresh.

## 📊 Business Impact
- KPI Affected: Access control accuracy, System security, Recruiter onboarding speed
- Expected Impact: Ensures every team member has precisely scoped access, reducing unauthorized operations while enabling rapid role adjustments as the team evolves
- Success Criteria:
  - Role changes reflected on subsequent API calls within the same session after re-authentication
  - Zero users with roles outside the defined enum persisted in the database
