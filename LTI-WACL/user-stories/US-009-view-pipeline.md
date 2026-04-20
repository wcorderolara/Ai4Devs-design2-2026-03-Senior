PATH: /user-stories/US-009-view-pipeline.md

# 🧾 User Story: View Recruitment Pipeline

## 🆔 Metadata
- ID: US-009
- Feature: Pipeline Management
- Priority: High
- Status: Draft
- Owner: PM

## 🎯 User Story
**As a** Recruiter, Hiring Manager, or Admin  
**I want** to view all candidates organized by pipeline stage for a specific job opening  
**So that** I have a clear operational picture of where each candidate stands in the recruitment process

## 🧠 Business Context
- Related Use Case(s): Visualizar Pipeline de Vacante (UC9)
- Entities involved: JobOpening, Application, PipelineStage, Candidate, Company, User
- Business Rules:
  - The pipeline view is scoped to a single JobOpening.
  - Stages displayed must be the snapshot PipelineStages with `ownerType = JobOpening` for that specific job opening.
  - Each stage shows all Applications currently at `currentStageId = that stage`.
  - Applications with `status = Rejected`, `Hired`, or `Withdrawn` may be visually separated or hidden by default.
  - The authenticated user must belong to the same Company as the JobOpening.
- Assumptions:
  - The pipeline is presented as a Kanban-style board ordered by `PipelineStage.sequence`.
  - Candidate card in each column shows at minimum: full name, application date, and current stage.
- Dependencies:
  - JobOpening must exist with at least one PipelineStage (US-001).
  - At least one Application must exist for the view to be non-empty.

## ✅ Acceptance Criteria (BDD + Edge Cases)

### 🎯 Happy Path
- **Given** an authenticated Recruiter and a JobOpening with Applications distributed across multiple stages  
  **When** the Recruiter opens the pipeline view for that job opening  
  **Then** the system displays a Kanban board with one column per PipelineStage ordered by sequence, each column listing the Candidates whose Application.currentStageId matches that stage

### ⚠️ Edge Cases

#### Case 1: Job opening with no applications
- **Given** a JobOpening exists but has received zero Applications  
  **When** the Recruiter opens the pipeline view  
  **Then** the system displays the Kanban columns (stages) with empty candidate lists and a message indicating no candidates have applied yet  
- Handling:
  - Render stage columns regardless; empty state is a valid view.

#### Case 2: Job opening belongs to a different Company
- **Given** a Recruiter tries to view the pipeline of a JobOpening from another Company  
  **When** the pipeline endpoint is called  
  **Then** the system returns a 404 Not Found error  
- Handling:
  - Enforce companyId scoping on JobOpening lookup before returning pipeline data.

#### Case 3: Closed job opening pipeline
- **Given** a JobOpening has status `Closed`  
  **When** the Recruiter opens the pipeline view  
  **Then** the system displays the historical pipeline state as read-only, indicating the job opening is closed  
- Handling:
  - Closed pipelines are visible but no stage-move actions are available. Display a "Closed" badge prominently.

### 🚫 Validation Rules
- The `jobOpeningId` must belong to the authenticated user's Company.
- PipelineStages displayed must have `ownerType = JobOpening` and `ownerId = jobOpeningId`.
- `Application.currentStageId` is the source of truth for the candidate's column placement.
- Stages must be ordered ascending by `sequence`.

## 📊 Business Impact
- KPI Affected: Pipeline visibility, Time spent in each stage, Conversion rate per stage
- Expected Impact: Provides real-time visibility into the recruitment funnel, enabling faster identification of bottlenecks and stale applications
- Success Criteria:
  - Pipeline view loads in under 2 seconds for job openings with up to 500 candidates
  - Every active Application appears in exactly one pipeline column
