PATH: /user-stories/US-024-structured-scorecard-evaluation.md

# 🧾 User Story: Evaluate Candidate with Structured Scorecard

## 🆔 Metadata
- ID: US-024
- Feature: Evaluation
- Priority: High
- Status: Draft
- Owner: PM

## 🎯 User Story
**As an** Interviewer, Hiring Manager, or Recruiter  
**I want** to complete a structured scorecard evaluation for a candidate after an interview  
**So that** the hiring team has standardized, comparable scores per criterion that reduce bias and support consistent decisions

## 🧠 Business Context
- Related Use Case(s): UC-27 Evaluar candidato con scorecard estructurado
- Entities involved: Evaluation, ScorecardTemplate, ScorecardCriterion, ScorecardResponse, Application, JobOpening, PipelineStage, User
- Business Rules:
  - The evaluator must have role Interviewer, HiringManager, or Recruiter.
  - A `ScorecardTemplate` must be associated with the JobOpening (`JobOpening.defaultScorecardTemplateId`) or selected manually.
  - The Evaluation is linked to an Application; the `ScorecardTemplate` determines the criteria to respond.
  - One `ScorecardResponse` record is created per `(evaluationId, criterionId)` combination; duplicates are not allowed.
  - All `ScorecardCriterion` records with `isRequired = true` must have a `ScorecardResponse` before `Evaluation.submittedAt` can be set.
  - `Evaluation.structuredScore` is computed as a weighted average of `ScorecardResponse.score` values when `submittedAt` is set.
  - A submitted Evaluation (`submittedAt != null`) cannot be altered without creating a new revision (immutability enforced).
  - `scorecardTemplateId` on `Evaluation` links to which template version was used.
- Assumptions:
  - The Interviewer can save a draft before submitting (draft = `completedAt = null`, `submittedAt = null`).
  - Multiple Interviewers may each complete their own scorecard for the same Application independently.
- Dependencies:
  - `ScorecardTemplate` must exist with at least one active `ScorecardCriterion` (configured by Admin).
  - Application must be in `status = Active`.

## ✅ Acceptance Criteria (BDD + Edge Cases)

### 🎯 Happy Path
- **Given** an authenticated Interviewer, an Application with `status = Active`, and a JobOpening with a `defaultScorecardTemplateId`  
  **When** the Interviewer opens the scorecard form, rates all required criteria, adds a summary, selects `recommendation = Yes`, and submits  
  **Then** one `ScorecardResponse` per criterion is persisted, `Evaluation.submittedAt` is set to now, `Evaluation.structuredScore` is computed as the weighted average, `Evaluation.recommendation` is saved, and the evaluation appears in the Application's evaluation list for all authorized team members

### ⚠️ Edge Cases

#### Case 1: Required criterion left unanswered at submission
- **Given** an Interviewer attempts to submit a scorecard with one or more `isRequired = true` criteria unanswered  
  **When** the submit action is triggered  
  **Then** the system returns a 400 validation error listing the missing required criteria by name  
- Handling:
  - Block submission; do not set `submittedAt`. Highlight missing criteria in the UI.

#### Case 2: Attempting to edit a submitted scorecard
- **Given** an Evaluation with `submittedAt != null`  
  **When** the Interviewer attempts to modify a `ScorecardResponse`  
  **Then** the system returns a 422 Unprocessable Entity error indicating the evaluation is already submitted and immutable  
- Handling:
  - Submitted scorecards are immutable in MVP. Post-MVP: allow revision with full trazabilidad.

#### Case 3: Score outside the criterion's defined scale
- **Given** a `ScorecardCriterion` with `scaleMin = 1` and `scaleMax = 5`  
  **When** the Interviewer submits a `ScorecardResponse.score = 7`  
  **Then** the system returns a 400 validation error for that criterion's score  
- Handling:
  - Validate `score >= scaleMin AND score <= scaleMax` per criterion before persisting.

#### Case 4: ScorecardTemplate becomes inactive after a draft is started
- **Given** an Interviewer starts a draft scorecard and the Admin deactivates the linked `ScorecardTemplate` before the Interviewer submits  
  **When** the Interviewer attempts to submit the draft  
  **Then** the system allows submission using the template version that was active at draft creation  
- Handling:
  - Submission references the `scorecardTemplateId` already linked to the Evaluation record; template status does not block in-progress submissions.

### 🚫 Validation Rules
- All `isRequired = true` criteria must have a `ScorecardResponse` before `submittedAt` can be set.
- `ScorecardResponse.score` must respect `ScorecardCriterion.scaleMin` and `scaleMax` when defined.
- Only one `ScorecardResponse` per `(evaluationId, criterionId)` combination.
- `Evaluation.structuredScore` is computed server-side at submission time; clients cannot set it directly.
- A submitted Evaluation (`submittedAt != null`) is immutable.
- Role must be Interviewer, HiringManager, or Recruiter.

## 📊 Business Impact
- KPI Affected: Hiring decision consistency, Bias reduction, Time-to-decision after interview
- Expected Impact: Standardizes interview feedback across the team, eliminating subjective free-text-only evaluations and enabling apples-to-apples candidate comparison
- Success Criteria:
  - 100% of submitted Evaluations with a ScorecardTemplate have all required criteria answered
  - `structuredScore` computed and stored on 100% of submitted Evaluations with a template
  - Scorecard results accessible to Recruiter and HiringManager within the same session as submission
