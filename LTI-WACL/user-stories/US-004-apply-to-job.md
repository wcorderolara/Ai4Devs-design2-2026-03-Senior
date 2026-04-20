PATH: /user-stories/US-004-apply-to-job.md

# 🧾 User Story: Apply to Job Opening

## 🆔 Metadata
- ID: US-004
- Feature: Candidate Capture
- Priority: High
- Status: Draft
- Owner: PM

## 🎯 User Story
**As a** Candidate  
**I want** to submit my application and CV for a specific open job opening  
**So that** I am registered in the company's ATS and my profile enters the recruitment pipeline

## 🧠 Business Context
- Related Use Case(s): Postular a Vacante (UC4)
- Entities involved: Candidate, Application, JobOpening, PipelineStage, StageTransition, Company
- Business Rules:
  - The JobOpening must have status `Open` to accept applications.
  - A Candidate can only have one active Application per JobOpening.
  - On submission, the Application is created with `status = Active` and placed at the first PipelineStage (lowest `sequence`) of the job opening's snapshot pipeline.
  - A StageTransition record must be inserted (with `fromStageId = null`) atomically with the Application.
  - `Application.currentStageId` and `Application.stageUpdatedAt` must be set in the same transaction.
  - Candidate email is mandatory.
- Assumptions:
  - The Candidate does not need to be authenticated; the apply form is public.
  - A new Candidate record is created if no existing Candidate with the same email exists within the Company.
- Dependencies:
  - JobOpening must exist and be Open (US-001).
  - At least one PipelineStage with `ownerType = JobOpening` must exist for the job opening.

## ✅ Acceptance Criteria (BDD + Edge Cases)

### 🎯 Happy Path
- **Given** an Open JobOpening and a new Candidate  
  **When** the Candidate submits the application form with firstName, lastName, email, and optional CV  
  **Then** a Candidate record is created (or matched by email), an Application is created with `status = Active` at the first PipelineStage, a StageTransition is recorded with `fromStageId = null`, and `Application.currentStageId` is set atomically

### ⚠️ Edge Cases

#### Case 1: Candidate applies to the same job opening twice
- **Given** a Candidate with an existing active Application for a specific JobOpening  
  **When** the same Candidate submits another application for the same job opening  
  **Then** the system returns a 409 Conflict error indicating a duplicate active application  
- Handling:
  - Enforce unique constraint on `(candidateId, jobOpeningId)` for active Applications. Return the existing application reference.

#### Case 2: Applying to a Closed job opening
- **Given** a JobOpening with status `Closed`  
  **When** a Candidate submits an application  
  **Then** the system returns a 422 error indicating the position is no longer accepting applications  
- Handling:
  - Validate status before persisting; display a user-friendly message on the public apply form.

#### Case 3: Missing required candidate fields
- **Given** a Candidate submits the form without email or name  
  **When** the form is submitted  
  **Then** the system returns a 400 validation error listing missing required fields  
- Handling:
  - Validate `firstName`, `lastName`, and `email` before persisting any record.

#### Case 4: Invalid email format
- **Given** a Candidate provides a malformed email address  
  **When** the form is submitted  
  **Then** the system returns a 400 validation error indicating invalid email format  
- Handling:
  - Apply EmailAddress Value Object syntactic validation before persistence.

### 🚫 Validation Rules
- `email` is required and must pass syntactic validation (EmailAddress Value Object).
- `firstName` and `lastName` are required and must not be blank (PersonName Value Object).
- One active Application per `(candidateId, jobOpeningId)` combination.
- JobOpening must have `status = Open`.
- CV file upload is optional but must be stored as `resumeUrl` if provided.

## 📊 Business Impact
- KPI Affected: Number of applications per job opening, Source of hire, Conversion rate (applied → hired)
- Expected Impact: Standardizes candidate entry into the pipeline, reducing manual data entry and information loss
- Success Criteria:
  - 100% of submitted applications create a corresponding StageTransition atomically
  - Zero duplicate active Applications per Candidate + JobOpening combination
