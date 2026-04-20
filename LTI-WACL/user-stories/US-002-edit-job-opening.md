PATH: /user-stories/US-002-edit-job-opening.md

# 🧾 User Story: Edit Job Opening

## 🆔 Metadata
- ID: US-002
- Feature: Job Management
- Priority: High
- Status: Draft
- Owner: PM

## 🎯 User Story
**As a** Recruiter or Admin  
**I want** to modify the details of an existing job opening  
**So that** I can correct information or update the job profile without losing existing applications

## 🧠 Business Context
- Related Use Case(s): Editar Vacante (UC2)
- Entities involved: JobOpening, Company, User
- Business Rules:
  - The user must have role Recruiter or Admin.
  - The JobOpening must exist within the user's Company.
  - A closed JobOpening must not allow critical operational changes.
  - Editing the job opening does NOT affect existing PipelineStages that were already snapshot-copied.
  - The `sourceTemplateId` reference is read-only and not modifiable post-creation.
- Assumptions:
  - Basic change history is not shown to candidates — it is internal.
- Dependencies:
  - JobOpening must already exist (US-001).

## ✅ Acceptance Criteria (BDD + Edge Cases)

### 🎯 Happy Path
- **Given** an authenticated Recruiter and an existing Open JobOpening belonging to their Company  
  **When** the Recruiter modifies the title, description, location, or other editable fields and submits  
  **Then** the JobOpening record is updated with the new values, existing Applications remain intact, and the snapshot PipelineStages are not altered

### ⚠️ Edge Cases

#### Case 1: Editing a Closed job opening
- **Given** a Recruiter attempts to edit a JobOpening with status `Closed`  
  **When** the Recruiter submits the edit form  
  **Then** the system returns a 409 Conflict error indicating the job opening is closed and cannot be modified  
- Handling:
  - Block edits to closed job openings at the API layer; display a clear message in the UI.

#### Case 2: Concurrent edit conflict
- **Given** two Recruiters simultaneously edit the same JobOpening  
  **When** the second Recruiter submits after the first already saved  
  **Then** the system processes the last write and persists it, returning the updated record to the second Recruiter  
- Handling:
  - Last-write-wins for MVP; add optimistic concurrency (ETag) in a future iteration.

#### Case 3: Unauthorized role
- **Given** an authenticated user with role Interviewer or HiringManager  
  **When** the user attempts to edit a job opening  
  **Then** the system returns a 403 Forbidden error  
- Handling:
  - Only Recruiter and Admin may edit job openings.

### 🚫 Validation Rules
- `title` must not be set to blank.
- `description` must not be set to blank.
- `workMode` must remain within enum values `Onsite`, `Hybrid`, `Remote` if provided.
- `status` cannot be changed via the edit endpoint; use dedicated close endpoint (US-003).
- The edited JobOpening must belong to the same `companyId` as the authenticated user.

## 📊 Business Impact
- KPI Affected: Data quality of job openings, Recruiter operational efficiency
- Expected Impact: Reduces miscommunication caused by outdated job descriptions without disrupting active pipelines
- Success Criteria:
  - Zero Applications are orphaned or lost after a job opening edit
  - Edits are persisted within the same request cycle with no data loss
