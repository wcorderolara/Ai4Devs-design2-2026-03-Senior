PATH: /user-stories/US-006-view-candidate-profile.md

# 🧾 User Story: View Candidate Profile

## 🆔 Metadata
- ID: US-006
- Feature: Candidate Management
- Priority: High
- Status: Draft
- Owner: PM

## 🎯 User Story
**As a** Recruiter, Hiring Manager, or Admin  
**I want** to view the unified profile of a candidate including their personal data, CV, applications, evaluations, and internal notes  
**So that** I can make informed decisions about the candidate's progress in any active recruitment process

## 🧠 Business Context
- Related Use Case(s): Consultar Perfil de Candidato (UC6)
- Entities involved: Candidate, Application, JobOpening, PipelineStage, Evaluation, Note, Communication, User
- Business Rules:
  - The actor must be authenticated with role Recruiter, HiringManager, or Admin.
  - All data shown belongs to the authenticated user's Company (multi-tenant isolation).
  - Notes are internal and must never be visible to the Candidate.
  - The profile aggregates all Applications across multiple job openings for the same Candidate.
- Assumptions:
  - The Candidate profile is read-only in this view; editing is handled by US-008.
  - CV is presented as a download link via `resumeUrl`.
- Dependencies:
  - Candidate must exist in the system (US-004 or US-005).

## ✅ Acceptance Criteria (BDD + Edge Cases)

### 🎯 Happy Path
- **Given** an authenticated Recruiter and an existing Candidate in their Company  
  **When** the Recruiter navigates to the Candidate's profile page  
  **Then** the system displays: personal data (name, email, phone, location), resumeUrl link, list of all Applications with their current PipelineStage and status, all Evaluations associated to those Applications, all internal Notes, and Communication history

### ⚠️ Edge Cases

#### Case 1: Candidate not found
- **Given** a Recruiter requests a Candidate profile with a non-existent or different-company ID  
  **When** the profile page is loaded  
  **Then** the system returns a 404 Not Found error  
- Handling:
  - Enforce companyId scoping on every Candidate lookup. Never expose cross-tenant data.

#### Case 2: Candidate with no applications
- **Given** a Candidate registered manually without linking to any job opening  
  **When** the profile is viewed  
  **Then** the system displays the Candidate's personal data with an empty Applications section and a prompt to link them to a job opening  
- Handling:
  - Applications section renders as empty list; profile is still accessible.

#### Case 3: Unauthorized role
- **Given** an authenticated user with role Interviewer  
  **When** the user attempts to access a candidate profile directly  
  **Then** the system returns a 403 Forbidden error  
- Handling:
  - Interviewers can only access evaluations for applications they are explicitly assigned to (post-MVP). In MVP, block Interviewer access to full candidate profile.

### 🚫 Validation Rules
- The requested Candidate must belong to the same `companyId` as the authenticated user.
- Notes displayed must be `contextType = Candidate` or `contextType = Application` for Applications belonging to this Candidate.
- Evaluations displayed must belong to Applications of this Candidate.
- Communications displayed must have `candidateId` matching this Candidate.

## 📊 Business Impact
- KPI Affected: Decision quality per candidate, Time spent per hiring decision
- Expected Impact: Centralizes all candidate information in one view, reducing context-switching and improving decision consistency
- Success Criteria:
  - All Candidate data (applications, notes, evaluations, communications) accessible from a single profile view
  - Profile load time under 2 seconds for candidates with up to 10 applications
