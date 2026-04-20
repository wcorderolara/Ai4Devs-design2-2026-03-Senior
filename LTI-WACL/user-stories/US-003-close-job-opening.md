PATH: /user-stories/US-003-close-job-opening.md

# 🧾 User Story: Close Job Opening

## 🆔 Metadata
- ID: US-003
- Feature: Job Management
- Priority: High
- Status: Draft
- Owner: PM

## 🎯 User Story
**As a** Recruiter or Admin  
**I want** to close a job opening when the position is no longer accepting candidates  
**So that** no new applications are accepted while the historical data and pipeline remain accessible

## 🧠 Business Context
- Related Use Case(s): Cerrar Vacante (UC3)
- Entities involved: JobOpening, Application, Company, User
- Business Rules:
  - Only JobOpenings with status `Open` can be closed.
  - Closing sets `status = Closed` and records `closedAt` timestamp.
  - A closed JobOpening does not accept new Applications.
  - Existing Applications and their StageTransitions are preserved.
  - A closed JobOpening cannot be reopened in MVP (post-MVP enhancement).
- Assumptions:
  - Closing a job opening does not automatically reject active Applications; the Recruiter handles that manually.
- Dependencies:
  - JobOpening must exist and be in `Open` status.

## ✅ Acceptance Criteria (BDD + Edge Cases)

### 🎯 Happy Path
- **Given** an authenticated Recruiter and an existing Open JobOpening  
  **When** the Recruiter triggers the close action  
  **Then** the JobOpening status changes to `Closed`, `closedAt` is set to current timestamp, and the endpoint rejects any subsequent new Application submissions for that job opening

### ⚠️ Edge Cases

#### Case 1: Closing an already-closed job opening
- **Given** a Recruiter attempts to close a JobOpening already in `Closed` status  
  **When** the close action is submitted  
  **Then** the system returns a 409 Conflict error indicating the job opening is already closed  
- Handling:
  - Idempotency check: if already closed, return error with current state.

#### Case 2: Closing a Draft job opening
- **Given** a Recruiter attempts to close a JobOpening in `Draft` status  
  **When** the close action is submitted  
  **Then** the system returns a 422 Unprocessable Entity error  
- Handling:
  - Only `Open` job openings can be transitioned to `Closed`. Provide clear status transition guidance.

#### Case 3: New application submitted to closed job opening
- **Given** a Candidate submits an application after the job opening is closed  
  **When** POST /applications is called with the closed job opening ID  
  **Then** the system returns a 422 error indicating the job opening is not accepting applications  
- Handling:
  - Validate JobOpening status before creating any Application.

### 🚫 Validation Rules
- Only `Open` → `Closed` transition is valid in MVP.
- `closedAt` is set server-side at time of close; it cannot be provided by the client.
- The JobOpening must belong to the authenticated user's Company.
- Role must be Recruiter or Admin.

## 📊 Business Impact
- KPI Affected: Pipeline integrity, Time-to-hire measurement accuracy
- Expected Impact: Prevents pipeline pollution from late applications after a position is filled, ensuring clean reporting
- Success Criteria:
  - No Applications created for closed job openings
  - `closedAt` timestamp present on 100% of closed job openings
