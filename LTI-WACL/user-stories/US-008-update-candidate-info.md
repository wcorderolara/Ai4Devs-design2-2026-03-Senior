PATH: /user-stories/US-008-update-candidate-info.md

# 🧾 User Story: Update Candidate Information

## 🆔 Metadata
- ID: US-008
- Feature: Candidate Management
- Priority: Medium
- Status: Draft
- Owner: PM

## 🎯 User Story
**As a** Recruiter or Admin  
**I want** to update a candidate's editable personal information  
**So that** the candidate's profile remains accurate without losing the history of their recruitment process

## 🧠 Business Context
- Related Use Case(s): Actualizar Información de Candidato (UC8)
- Entities involved: Candidate, Company, User
- Business Rules:
  - Only attributes explicitly defined as editable may be changed: `firstName`, `lastName`, `phone`, `currentLocation`, `resumeUrl`.
  - `email` is an identifier and must not be changed through this flow in MVP to avoid identity conflicts.
  - Editing a Candidate does not affect any linked Applications, Evaluations, or Notes.
  - Only Recruiter and Admin roles may edit Candidate profiles.
- Assumptions:
  - The Recruiter updates candidate information based on new data provided by the candidate (e.g., updated CV or phone number).
- Dependencies:
  - Candidate must exist in the system (US-004 or US-005).

## ✅ Acceptance Criteria (BDD + Edge Cases)

### 🎯 Happy Path
- **Given** an authenticated Recruiter and an existing Candidate in their Company  
  **When** the Recruiter updates the Candidate's phone number and location and submits  
  **Then** the Candidate record is updated with the new values, and all existing Applications and Evaluations remain unchanged

### ⚠️ Edge Cases

#### Case 1: Attempting to change the candidate's email
- **Given** a Recruiter submits an update with a modified `email` field  
  **When** the request is processed  
  **Then** the system ignores the `email` field change and returns the updated record without modifying email  
- Handling:
  - Strip `email` from the update payload silently in MVP, or return a 400 error with explanation that email is immutable.

#### Case 2: Candidate not found or belongs to another Company
- **Given** a Recruiter provides an ID that does not correspond to a Candidate in their Company  
  **When** the update is submitted  
  **Then** the system returns a 404 Not Found error  
- Handling:
  - Enforce companyId scoping on every Candidate update lookup.

#### Case 3: Invalid resume URL format
- **Given** a Recruiter submits a `resumeUrl` with a malformed URL string  
  **When** the update is submitted  
  **Then** the system returns a 400 validation error specifying that resumeUrl must be a valid URL  
- Handling:
  - Validate URL format server-side before persisting.

### 🚫 Validation Rules
- `email` is immutable in MVP; reject or ignore changes to this field.
- `firstName` and `lastName` must not be set to blank if provided.
- `resumeUrl` must be a valid URL format if provided.
- The Candidate must belong to the same `companyId` as the authenticated user.
- Role must be Recruiter or Admin.

## 📊 Business Impact
- KPI Affected: Data accuracy of candidate records, Recruiter operational efficiency
- Expected Impact: Ensures candidate profiles stay current, improving the quality of data used for hiring decisions
- Success Criteria:
  - Candidate profile updates are persisted without affecting linked Application history
  - Zero applications orphaned or corrupted due to candidate data edits
