PATH: /user-stories/US-013-register-candidate-feedback.md

# 🧾 User Story: Register Candidate Feedback

## 🆔 Metadata
- ID: US-013
- Feature: Evaluation & Collaboration
- Priority: High
- Status: Draft
- Owner: PM

## 🎯 User Story
**As a** Recruiter, Hiring Manager, or Interviewer  
**I want** to record qualitative feedback and observations about a candidate at any stage of the process  
**So that** the team has a shared internal record of impressions that informs the hiring decision

## 🧠 Business Context
- Related Use Case(s): Registrar Feedback de Candidato (UC13)
- Entities involved: Note, Application, Candidate, User, PipelineStage
- Business Rules:
  - Feedback is stored as a Note, internal-only and never visible to the Candidate.
  - A Note must be associated to at least one context: `contextType = Application` or `contextType = Candidate`.
  - The author must be an authenticated user with role Recruiter, HiringManager, or Interviewer.
  - Notes cannot be physically deleted in MVP (soft delete reserved for post-MVP).
  - Multiple team members can leave Notes on the same Application or Candidate.
- Assumptions:
  - Feedback in this story is free-text qualitative commentary (structured scorecard evaluation is covered in US-014).
  - A Note linked to an Application captures stage-specific context automatically.
- Dependencies:
  - Application must exist (US-004 or US-005) if contextType is Application.
  - Candidate must exist if contextType is Candidate.

## ✅ Acceptance Criteria (BDD + Edge Cases)

### 🎯 Happy Path
- **Given** an authenticated Recruiter and an existing Application  
  **When** the Recruiter submits a Note with non-empty content linked to the Application  
  **Then** a Note record is persisted with `contextType = Application`, `contextId = application.id`, `authorUserId = recruiter.id`, and the note appears in the application's feedback section for all authorized team members

### ⚠️ Edge Cases

#### Case 1: Submitting an empty note
- **Given** a Recruiter opens the feedback form  
  **When** the Recruiter submits with blank content  
  **Then** the system returns a 400 validation error indicating content is required  
- Handling:
  - `content` must be a non-empty, non-whitespace string.

#### Case 2: Note on a Candidate without an Application
- **Given** a Candidate exists in the talent pool without any Application  
  **When** a Recruiter submits a Note with `contextType = Candidate`  
  **Then** the Note is created successfully linked directly to the Candidate  
- Handling:
  - Support both `contextType = Application` and `contextType = Candidate`; both are valid contexts.

#### Case 3: Unauthorized access to Note content
- **Given** an Interviewer submits a Note on an Application  
  **When** another Interviewer from the same Company views the Application  
  **Then** the Note is visible to all authorized internal team members of that Company  
- Handling:
  - Notes are scoped to Company; all internal users with appropriate roles can read notes within their Company.

#### Case 4: Note on a Candidate from another Company
- **Given** a Recruiter provides a `contextId` referencing a Candidate from a different Company  
  **When** the note is submitted  
  **Then** the system returns a 404 Not Found error  
- Handling:
  - Validate that the referenced Candidate or Application belongs to the authenticated user's Company.

### 🚫 Validation Rules
- `content` must be non-empty and non-whitespace.
- `contextType` must be one of: `Application`, `Candidate`.
- `contextId` must be a valid UUID referencing an existing Application or Candidate within the same Company.
- `authorUserId` is set server-side from the authenticated session; clients cannot override it.
- Role must be Recruiter, HiringManager, or Interviewer.

## 📊 Business Impact
- KPI Affected: Decision quality, Team collaboration effectiveness, Bias reduction
- Expected Impact: Creates a permanent, auditable record of team impressions that supports more consistent and defensible hiring decisions
- Success Criteria:
  - All feedback Notes are visible to authorized team members within the same Company
  - Zero Notes physically deleted; soft delete mechanism respected
