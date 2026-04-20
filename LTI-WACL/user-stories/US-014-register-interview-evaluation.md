PATH: /user-stories/US-014-register-interview-evaluation.md

# 🧾 User Story: Register Interview Evaluation

## 🆔 Metadata
- ID: US-014
- Feature: Evaluation & Collaboration
- Priority: High
- Status: Draft
- Owner: PM

## 🎯 User Story
**As a** Recruiter, Hiring Manager, or Interviewer  
**I want** to submit a structured evaluation with a score and recommendation after interviewing a candidate  
**So that** the team has comparable, structured data to support consistent and unbiased hiring decisions

## 🧠 Business Context
- Related Use Case(s): Registrar Evaluación de Entrevista (UC14)
- Entities involved: Evaluation, Application, PipelineStage, User
- Business Rules:
  - Only authenticated users with role Recruiter, HiringManager, or Interviewer may submit evaluations.
  - An Evaluation is linked to a specific Application and optionally to a PipelineStage.
  - `recommendation` is required when the Evaluation is marked as completed (`completedAt` is not null).
  - A draft Evaluation (with `completedAt = null`) is valid and does not count for hiring decisions.
  - Multiple users may submit separate Evaluations for the same Application.
  - `score` is optional and represents a numeric rating alongside the structured recommendation.
- Assumptions:
  - An Evaluation corresponds to a completed interview or formal assessment.
  - The same user may revise their draft Evaluation before marking it complete.
- Dependencies:
  - Application must exist and be in `status = Active`.
  - PipelineStage referenced must belong to the same JobOpening as the Application.

## ✅ Acceptance Criteria (BDD + Edge Cases)

### 🎯 Happy Path
- **Given** an authenticated Interviewer and an Application with an active interview stage  
  **When** the Interviewer submits an Evaluation with `recommendation = Yes`, optional `score`, optional `summary`, and marks it complete (`completedAt = now`)  
  **Then** an Evaluation record is persisted with `evaluatorUserId = interviewer.id`, `applicationId`, `stageId` (if provided), `recommendation`, and `completedAt`; the evaluation is visible to authorized team members

### ⚠️ Edge Cases

#### Case 1: Completing an evaluation without a recommendation
- **Given** an Interviewer submits an Evaluation and sets `completedAt` without providing `recommendation`  
  **When** the request is processed  
  **Then** the system returns a 400 validation error indicating `recommendation` is required when completing an evaluation  
- Handling:
  - Enforce: if `completedAt` is not null, then `recommendation` must not be null.

#### Case 2: Submitting a draft evaluation
- **Given** an Interviewer partially fills in an evaluation and saves without completing  
  **When** the request is submitted with `completedAt = null`  
  **Then** the Evaluation is stored as a draft, `completedAt` remains null, and the record does not influence hiring decisions  
- Handling:
  - Draft evaluations are valid. Flag them visually in the UI as "Draft" to distinguish from completed evaluations.

#### Case 3: Two evaluators submit for the same application
- **Given** both a Hiring Manager and an Interviewer submit separate Evaluations for the same Application  
  **When** both submissions are processed  
  **Then** both Evaluations are persisted independently and both appear in the Application's evaluation list  
- Handling:
  - Multiple Evaluations per Application are allowed; one per evaluator per stage is a recommended practice but not enforced in MVP.

#### Case 4: Evaluation linked to a stage from a different job opening
- **Given** an Interviewer provides a `stageId` that belongs to a different JobOpening  
  **When** the evaluation is submitted  
  **Then** the system returns a 422 Unprocessable Entity error  
- Handling:
  - Validate that `stageId.ownerId = application.jobOpeningId` if `stageId` is provided.

### 🚫 Validation Rules
- `applicationId` must reference an Application within the authenticated user's Company.
- `recommendation` is required when `completedAt` is not null; must be one of: `StrongNo`, `No`, `Yes`, `StrongYes`.
- `score` is optional; if provided, must be a positive decimal.
- `stageId` is optional; if provided, must have `ownerType = JobOpening` and belong to the application's job opening.
- `evaluatorUserId` is set server-side from the authenticated session.
- Role must be Recruiter, HiringManager, or Interviewer.

## 📊 Business Impact
- KPI Affected: Hiring decision quality, Evaluator consistency, Bias reduction in selection
- Expected Impact: Replaces subjective impressions with structured, comparable scorecards that improve alignment across the hiring team
- Success Criteria:
  - 100% of completed Evaluations have a non-null `recommendation`
  - Evaluation data available to all authorized team members for comparison before final hiring decision
