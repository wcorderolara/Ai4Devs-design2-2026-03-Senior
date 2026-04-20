PATH: /user-stories/US-012-mark-candidate-hired.md

# 🧾 User Story: Mark Candidate as Hired

## 🆔 Metadata
- ID: US-012
- Feature: Pipeline Management
- Priority: High
- Status: Draft
- Owner: PM

## 🎯 User Story
**As a** Recruiter or Admin  
**I want** to mark a candidate as hired for a specific job opening  
**So that** the recruitment process is formally closed with a successful outcome and the system records the hiring decision

## 🧠 Business Context
- Related Use Case(s): Marcar Candidato como Contratado (UC12)
- Entities involved: Application, PipelineStage, StageTransition, JobOpening, Candidate, User
- Business Rules:
  - Only Recruiter and Admin roles may mark a candidate as hired.
  - The Application must be in `status = Active` and in an eligible non-terminal stage.
  - Marking as hired sets `Application.status = Hired` and moves the Application to the single `stageType = Hired` PipelineStage of the job opening.
  - A StageTransition must be recorded atomically.
  - Only one `stageType = Hired` stage may exist per pipeline.
  - After marking as hired, the JobOpening may be closed manually (UC3) or left open for additional hires.
- Assumptions:
  - Only one Candidate per Application can be marked as Hired (each Application is independent).
  - Hiring a candidate does not automatically close the job opening.
- Dependencies:
  - Application must have `status = Active`.
  - A PipelineStage with `stageType = Hired` must exist for the JobOpening.

## ✅ Acceptance Criteria (BDD + Edge Cases)

### 🎯 Happy Path
- **Given** an authenticated Recruiter and an Application with `status = Active` in a non-terminal stage  
  **When** the Recruiter marks the candidate as hired  
  **Then** `Application.status` is updated to `Hired`, the Application is moved to the Hired PipelineStage, a StageTransition is recorded with `toStageId = Hired stage`, and all three fields (`currentStageId`, `stageUpdatedAt`, `status`) are updated in the same transaction

### ⚠️ Edge Cases

#### Case 1: Candidate is already hired for the same job opening
- **Given** an Application with `status = Hired`  
  **When** a Recruiter attempts to mark the same candidate as hired again  
  **Then** the system returns a 409 Conflict error indicating the application is already in a hired state  
- Handling:
  - Return error with current application state; no duplicate StageTransition is created.

#### Case 2: No Hired stage exists in the pipeline
- **Given** a JobOpening whose snapshot pipeline lacks a `stageType = Hired` stage  
  **When** a Recruiter tries to mark a candidate as hired  
  **Then** the system returns a 500 Internal Server Error (pipeline configuration error)  
- Handling:
  - This is a data integrity issue. Enforce exactly one Hired stage per pipeline at job opening creation (US-001 guard).

#### Case 3: Application is already rejected or withdrawn
- **Given** an Application with `status = Rejected` or `Withdrawn`  
  **When** a Recruiter tries to mark the candidate as hired  
  **Then** the system returns a 422 Unprocessable Entity error  
- Handling:
  - Terminal statuses prevent further movement. Display a clear error message.

#### Case 4: Unauthorized role
- **Given** an authenticated user with role HiringManager  
  **When** the user attempts to mark a candidate as hired  
  **Then** the system returns a 403 Forbidden error  
- Handling:
  - Only Recruiter and Admin may mark candidates as hired in MVP.

### 🚫 Validation Rules
- Application must have `status = Active`.
- Target stage must have `stageType = Hired`, `ownerType = JobOpening`, `ownerId = application.jobOpeningId`.
- Only one `stageType = Hired` stage may exist per pipeline; enforce uniqueness.
- `Application.status`, `Application.currentStageId`, and `Application.stageUpdatedAt` must update atomically with StageTransition INSERT.
- Role must be Recruiter or Admin.

## 📊 Business Impact
- KPI Affected: Time-to-hire (end date), Hiring conversion rate, Recruiter success metrics
- Expected Impact: Provides the definitive business outcome signal for each recruitment cycle, enabling accurate reporting on hiring velocity
- Success Criteria:
  - 100% of Hired applications have exactly one corresponding StageTransition to a Hired stage
  - Time-to-hire becomes calculable from `Application.appliedAt` to the Hired `StageTransition.changedAt`
