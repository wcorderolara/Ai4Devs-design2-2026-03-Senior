PATH: /user-stories/US-001-create-job-opening.md

# 🧾 User Story: Create Job Opening

## 🆔 Metadata
- ID: US-001
- Feature: Job Management
- Priority: High
- Status: Draft
- Owner: PM

## 🎯 User Story
**As a** Recruiter or Admin  
**I want** to create a new job opening with title, department, location, work mode, and description  
**So that** I can formally start a recruitment process and allow candidates to apply

## 🧠 Business Context
- Related Use Case(s): Crear Vacante (UC1)
- Entities involved: JobOpening, Company, User, PipelineTemplate, PipelineStage
- Business Rules:
  - The job opening must be associated with a valid Company (tenant).
  - The creating user must have role Recruiter or Admin.
  - A PipelineTemplate must be selected or defaulted; its stages are snapshot-copied to the new JobOpening.
  - New job opening is created in status `Draft` or `Open`; it does NOT accept applications while in `Draft`.
  - A Company with status `Inactive` cannot create new job openings.
- Assumptions:
  - At least one active PipelineTemplate exists in the company.
  - The `isDefault` PipelineTemplate is used when none is explicitly selected.
- Dependencies:
  - PipelineTemplate must exist before a job opening can be created.
  - User must be authenticated with an active session.

## ✅ Acceptance Criteria (BDD + Edge Cases)

### 🎯 Happy Path
- **Given** an authenticated Recruiter with an active Company and at least one PipelineTemplate  
  **When** the Recruiter fills in title, description, and optionally department/location/workMode, then submits  
  **Then** a new JobOpening is persisted with status `Draft`, the selected PipelineTemplate's stages are snapshot-copied with `ownerType = JobOpening` and `ownerId = new JobOpening.id`, and the system returns the created job opening with its ID

### ⚠️ Edge Cases

#### Case 1: Missing required fields
- **Given** a Recruiter is on the job opening creation form  
  **When** the Recruiter submits without a title or description  
  **Then** the system rejects the request with a 400 validation error listing which fields are required  
- Handling:
  - Title and description are mandatory. The API must return field-level validation errors.

#### Case 2: No PipelineTemplate available
- **Given** a Recruiter attempts to create a job opening  
  **When** the company has no active PipelineTemplate defined  
  **Then** the system rejects creation with an error indicating that a pipeline template is required  
- Handling:
  - Block creation and prompt the Admin to configure a PipelineTemplate first.

#### Case 3: Inactive Company
- **Given** an authenticated user belonging to an Inactive Company  
  **When** the user attempts to create a job opening  
  **Then** the system returns a 403 Forbidden error  
- Handling:
  - Enforce company status check at the API layer before persisting.

#### Case 4: Unauthorized role
- **Given** an authenticated user with role Interviewer  
  **When** the user attempts to create a job opening  
  **Then** the system returns a 403 Forbidden error  
- Handling:
  - Role-based access control must block Interviewer and HiringManager from creating job openings.

### 🚫 Validation Rules
- `title` is required and must not be blank.
- `description` is required and must not be blank.
- `workMode` must be one of: `Onsite`, `Hybrid`, `Remote` if provided.
- `status` upon creation is always `Draft` (client cannot set it to `Open` on creation).
- The selected `sourceTemplateId` must belong to the same `companyId`.

## 📊 Business Impact
- KPI Affected: Time-to-hire, Number of open positions
- Expected Impact: Formalizes the start of a recruitment cycle, enabling pipeline tracking from day one
- Success Criteria:
  - 100% of job openings are created with a valid snapshot pipeline before accepting applications
  - Job opening creation takes under 2 minutes for a Recruiter
