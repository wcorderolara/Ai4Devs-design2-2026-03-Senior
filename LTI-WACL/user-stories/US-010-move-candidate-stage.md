PATH: /user-stories/US-010-move-candidate-stage.md

# 🧾 User Story: Move Candidate to Another Pipeline Stage

## 🆔 Metadata
- ID: US-010
- Feature: Pipeline Management
- Priority: High
- Status: Draft
- Owner: PM

## 🎯 User Story
**As a** Recruiter or Hiring Manager  
**I want** to move a candidate's application from one pipeline stage to another  
**So that** the system accurately reflects the candidate's progress and the stage transition is permanently recorded

## 🧠 Business Context
- Related Use Case(s): Mover Candidato de Etapa (UC10)
- Entities involved: Application, PipelineStage, StageTransition, JobOpening, Candidate, User
- Business Rules:
  - Only users with role Recruiter or HiringManager may move candidates between stages.
  - The target stage must belong to the same JobOpening (ownerType = JobOpening, ownerId = jobOpeningId).
  - `Application.currentStageId` and `Application.stageUpdatedAt` must be updated atomically in the same transaction as the StageTransition INSERT.
  - Candidates in terminal stages (`stageType = Hired` or `stageType = Rejected`) cannot be moved without explicit override (not available in MVP).
  - Application status must remain consistent with the target stage's `stageType`.
- Assumptions:
  - Movement can be forward or backward between non-terminal stages.
  - A comment on the transition is optional but recommended.
- Dependencies:
  - Application must be in `status = Active` (US-004 or US-005).
  - Target PipelineStage must have `ownerType = JobOpening`.

## ✅ Acceptance Criteria (BDD + Edge Cases)

### 🎯 Happy Path
- **Given** an authenticated Recruiter and an Application in `status = Active` at a non-terminal stage  
  **When** the Recruiter moves the candidate to the next stage  
  **Then** a StageTransition record is inserted with `fromStageId = previous stage`, `toStageId = new stage`, `changedByUserId = recruiter`, `changedAt = now`; `Application.currentStageId` and `Application.stageUpdatedAt` are updated in the same transaction; the pipeline view reflects the new position immediately

### ⚠️ Edge Cases

#### Case 1: Moving to a terminal Hired stage
- **Given** a Recruiter moves a candidate to the `stageType = Hired` stage  
  **When** the move is submitted  
  **Then** the StageTransition is recorded, `Application.status` is updated to `Hired`, and `Application.currentStageId` reflects the Hired stage  
- Handling:
  - This action is the same as US-012 (Mark as Hired); unify the flow — stage movement to a Hired stage triggers the hired outcome automatically.

#### Case 2: Attempting to move from a Hired terminal stage
- **Given** an Application already in `stageType = Hired`  
  **When** a Recruiter tries to move the candidate to a different stage  
  **Then** the system returns a 422 Unprocessable Entity error indicating the application is in a terminal state  
- Handling:
  - Terminal stages block further movement in MVP. Display a clear error message.

#### Case 3: Target stage belongs to a different job opening
- **Given** a Recruiter provides a `toStageId` that belongs to a different JobOpening  
  **When** the move request is submitted  
  **Then** the system returns a 422 error indicating the target stage is not valid for this application  
- Handling:
  - Validate that `toStageId.ownerId = application.jobOpeningId` before persisting.

#### Case 4: Application not active
- **Given** an Application with `status = Rejected` or `Withdrawn`  
  **When** a Recruiter tries to move the candidate  
  **Then** the system returns a 422 error indicating the application is not in an active state  
- Handling:
  - Block moves on non-active applications.

### 🚫 Validation Rules
- `toStageId` must have `ownerType = JobOpening` and `ownerId = application.jobOpeningId`.
- Application must have `status = Active` to be moveable.
- `Application.currentStageId` and `Application.stageUpdatedAt` must be updated atomically with StageTransition INSERT.
- `changedByUserId` must be set to the authenticated user's ID.
- Role must be Recruiter or HiringManager.

## 📊 Business Impact
- KPI Affected: Time-to-hire, Conversion rate per stage, Stage duration metrics
- Expected Impact: Provides complete, auditable pipeline movement history enabling bottleneck analysis and process optimization
- Success Criteria:
  - 100% of stage movements produce a corresponding StageTransition record
  - Zero desynchronization between `Application.currentStageId` and the latest StageTransition
