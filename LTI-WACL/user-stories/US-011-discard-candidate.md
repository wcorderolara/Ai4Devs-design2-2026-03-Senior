PATH: /user-stories/US-011-discard-candidate.md

# 🧾 User Story: Discard Candidate from Job Opening

## 🆔 Metadata
- ID: US-011
- Feature: Pipeline Management
- Priority: High
- Status: Draft
- Owner: PM

## 🎯 User Story
**As a** Recruiter or Hiring Manager  
**I want** to discard a candidate from a specific job opening  
**So that** the pipeline reflects only active candidates while the candidate's history is preserved for future reference

## 🧠 Business Context
- Related Use Case(s): Descartar Candidato (UC11)
- Entities involved: Application, PipelineStage, StageTransition, Candidate, JobOpening, User
- Business Rules:
  - Only Recruiter and HiringManager roles may discard candidates.
  - Discarding sets `Application.status = Rejected` and moves the Application to a stage with `stageType = Rejected`.
  - A StageTransition must be recorded with the terminal Rejected stage as `toStageId`.
  - `Application.currentStageId`, `Application.stageUpdatedAt`, and `Application.status` must all be updated atomically.
  - An optional `rejectionReason` may be provided and stored on the Application.
  - Discarding an Application does NOT delete the Candidate record; the Candidate remains in the system.
- Assumptions:
  - The Candidate may still be eligible for future job openings.
  - Discarding is a soft operation — no physical deletion occurs.
- Dependencies:
  - Application must have `status = Active`.
  - A PipelineStage with `stageType = Rejected` must exist for the JobOpening.

## ✅ Acceptance Criteria (BDD + Edge Cases)

### 🎯 Happy Path
- **Given** an authenticated Recruiter and an Application with `status = Active`  
  **When** the Recruiter discards the candidate with an optional rejection reason  
  **Then** `Application.status` is set to `Rejected`, `Application.rejectionReason` is stored if provided, the Application is moved to the Rejected PipelineStage, and a StageTransition is recorded atomically

### ⚠️ Edge Cases

#### Case 1: Discarding an already-rejected application
- **Given** an Application already in `status = Rejected`  
  **When** a Recruiter attempts to discard the candidate again  
  **Then** the system returns a 409 Conflict error indicating the application is already in a rejected state  
- Handling:
  - Idempotency check: return error if already rejected; surface current application state.

#### Case 2: No Rejected stage exists in the pipeline
- **Given** a JobOpening whose snapshot PipelineStages do not include a stage with `stageType = Rejected`  
  **When** a Recruiter tries to discard a candidate  
  **Then** the system returns a 500 Internal Server Error (configuration error) and logs the issue  
- Handling:
  - This is a data integrity issue. Enforce that every pipeline snapshot contains exactly one Rejected and one Hired stage at creation time (US-001 guard).

#### Case 3: Unauthorized role
- **Given** an authenticated user with role Interviewer or Admin attempting to discard a candidate  
  **When** the discard action is submitted  
  **Then** the system returns a 403 Forbidden error  
- Handling:
  - Only Recruiter and HiringManager may trigger the discard action.

### 🚫 Validation Rules
- Application must have `status = Active` to be discarded.
- A PipelineStage with `stageType = Rejected` and `ownerType = JobOpening` must exist for the application's JobOpening.
- `Application.status`, `Application.currentStageId`, and `Application.stageUpdatedAt` must update atomically with the StageTransition INSERT.
- `rejectionReason` is optional but must be a non-empty string if provided.
- Role must be Recruiter or HiringManager.

## 📊 Business Impact
- KPI Affected: Pipeline cleanliness, Funnel conversion rate, Time-to-hire accuracy
- Expected Impact: Keeps active pipeline focused on viable candidates while preserving full rejection history for compliance and analytics
- Success Criteria:
  - Discarded Applications removed from active pipeline view within the same request
  - 100% of discarded Applications have a corresponding StageTransition to a Rejected stage
