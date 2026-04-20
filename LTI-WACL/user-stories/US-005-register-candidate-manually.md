PATH: /user-stories/US-005-register-candidate-manually.md

# 🧾 User Story: Register Candidate Manually

## 🆔 Metadata
- ID: US-005
- Feature: Candidate Capture
- Priority: High
- Status: Draft
- Owner: PM

## 🎯 User Story
**As a** Recruiter or Admin  
**I want** to manually register a candidate in the system and optionally link them to a job opening  
**So that** candidates sourced offline or through external channels are captured in the ATS pipeline

## 🧠 Business Context
- Related Use Case(s): Registrar Candidato Manualmente (UC5)
- Entities involved: Candidate, Application, JobOpening, PipelineStage, StageTransition, Company, User
- Business Rules:
  - The user must be authenticated with role Recruiter or Admin.
  - A Candidate record is created with at least an email identifier.
  - If a JobOpening is selected, an Application is created atomically with its initial StageTransition.
  - A Candidate may be registered without linking to a job opening (talent pool entry).
  - Duplicate email within the same Company is not allowed; the system must detect and prevent it.
- Assumptions:
  - The Recruiter has obtained the candidate's consent to store their data.
  - Source field may be set manually to reflect the offline channel (e.g., "Referral", "Job Fair").
- Dependencies:
  - Company must be active.
  - If linking to a job opening, that JobOpening must exist and have status `Open`.

## ✅ Acceptance Criteria (BDD + Edge Cases)

### 🎯 Happy Path
- **Given** an authenticated Recruiter and an Open JobOpening  
  **When** the Recruiter submits the manual candidate form with firstName, lastName, email, and selects a job opening  
  **Then** a Candidate record is persisted, an Application is created with `status = Active` at the first PipelineStage of the job opening, and a StageTransition is recorded with `fromStageId = null`

### ⚠️ Edge Cases

#### Case 1: Candidate email already exists in the Company
- **Given** a Recruiter attempts to register a candidate with an email already in the system  
  **When** the form is submitted  
  **Then** the system returns a 409 Conflict error and surfaces the existing Candidate profile  
- Handling:
  - Prevent duplicate Candidate records per Company. Offer the Recruiter the option to link the existing Candidate to the new job opening instead.

#### Case 2: Register candidate without linking to a job opening
- **Given** a Recruiter registers a candidate without selecting a job opening  
  **When** the form is submitted  
  **Then** the Candidate record is created with no Application, available in the talent pool for future assignment  
- Handling:
  - Application creation is optional. The Candidate entity must be independent of any JobOpening.

#### Case 3: Linking to a Closed job opening
- **Given** a Recruiter tries to link the new Candidate to a Closed JobOpening  
  **When** the form is submitted  
  **Then** the system returns a 422 error indicating the selected job opening is not accepting applications  
- Handling:
  - Validate JobOpening status before creating the Application.

### 🚫 Validation Rules
- `email` is required and must pass EmailAddress Value Object validation.
- `firstName` and `lastName` are required (PersonName Value Object).
- Candidate `email` must be unique within the Company.
- If `jobOpeningId` is provided, the job opening must have `status = Open` and belong to the same Company.
- `source` is optional; if provided, it is stored as-is on the Application record.

## 📊 Business Impact
- KPI Affected: Source of hire diversity, Talent pool size, Pipeline completeness
- Expected Impact: Ensures no candidate is lost due to offline sourcing; expands the company's talent pool inside the ATS
- Success Criteria:
  - Manually registered candidates appear in pipeline within the same session
  - Zero Candidate records with duplicate emails per Company
