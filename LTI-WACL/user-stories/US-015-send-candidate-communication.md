PATH: /user-stories/US-015-send-candidate-communication.md

# 🧾 User Story: Send Communication to Candidate

## 🆔 Metadata
- ID: US-015
- Feature: Operational Communication
- Priority: High
- Status: Draft
- Owner: PM

## 🎯 User Story
**As a** Recruiter  
**I want** to send an email notification to a candidate directly from the ATS  
**So that** the candidate is informed of their application status and the communication is logged in the system

## 🧠 Business Context
- Related Use Case(s): Enviar Comunicación al Candidato (UC15)
- Entities involved: Communication, Application, Candidate, User, Company
- Business Rules:
  - Only users with role Recruiter may send communications in MVP.
  - The channel in MVP is exclusively `Email`; other channels are blocked even if the enum supports them.
  - The Candidate must have a valid email address.
  - The Communication must be linked to an existing Application.
  - `candidateId` in Communication must match the `candidateId` of the referenced Application.
  - Only Communications with `status = Sent` may have `sentAt` populated.
  - If `scheduledAt` is set and `sentAt` is null, the communication is pending scheduled delivery.
- Assumptions:
  - MVP uses an external Email Provider (SMTP/API) for actual delivery.
  - Templates are simple or manually composed; no dynamic template engine required in MVP.
  - Failed deliveries (`status = Failed`) are visible to the Recruiter for follow-up.
- Dependencies:
  - Application must exist (US-004 or US-005).
  - Candidate must have a non-null `email`.
  - Email Provider must be configured.

## ✅ Acceptance Criteria (BDD + Edge Cases)

### 🎯 Happy Path
- **Given** an authenticated Recruiter, an existing Application, and a Candidate with a valid email  
  **When** the Recruiter composes a message with subject and body and sends immediately  
  **Then** a Communication record is created with `status = Pending`, the message is dispatched to the Email Provider, upon successful delivery `status` updates to `Sent` and `sentAt` is populated, and the communication appears in the Application's communication history

### ⚠️ Edge Cases

#### Case 1: Email delivery fails
- **Given** the Email Provider returns an error during delivery  
  **When** the communication is dispatched  
  **Then** the Communication record is updated to `status = Failed`, `sentAt` remains null, and the Recruiter is notified of the failure  
- Handling:
  - Persist the failure state; do not retry automatically in MVP. Surface the failed communication in the UI for manual follow-up.

#### Case 2: Candidate has no email registered
- **Given** a Candidate record with a null or missing `email`  
  **When** the Recruiter attempts to send a communication  
  **Then** the system returns a 422 Unprocessable Entity error indicating the candidate has no email on file  
- Handling:
  - Block communication creation if Candidate email is null; prompt Recruiter to update candidate profile first.

#### Case 3: Scheduled communication
- **Given** a Recruiter sets a `scheduledAt` datetime for future delivery  
  **When** the communication is saved  
  **Then** the Communication record is persisted with `status = Pending`, `scheduledAt` set, and `sentAt = null`; delivery occurs at the scheduled time  
- Handling:
  - In MVP, scheduled delivery is acknowledged but may rely on a basic background job. `sentAt` is only set upon actual delivery.

#### Case 4: Sending communication for a closed application
- **Given** an Application with `status = Rejected` or `Hired`  
  **When** a Recruiter attempts to send a communication  
  **Then** the system allows the communication (rejection/offer notifications are valid post-decision)  
- Handling:
  - Communications are allowed regardless of Application status; status check not required for this operation.

### 🚫 Validation Rules
- `channel` must be `Email` in MVP; other values are rejected with a 422 error.
- `subject` is required and must not be blank.
- `body` is required and must not be blank.
- `applicationId` must reference an existing Application within the authenticated user's Company.
- `candidateId` must match `application.candidateId`.
- Only `status = Sent` Communications may have a non-null `sentAt`.
- If `scheduledAt` is provided, it must be a future datetime at time of creation.
- Role must be Recruiter.

## 📊 Business Impact
- KPI Affected: Candidate experience, Response time to candidates, Communication audit trail
- Expected Impact: Improves employer branding by ensuring timely and documented candidate communications, reducing ghosting and improving candidate satisfaction
- Success Criteria:
  - 100% of sent communications logged with `status` and `sentAt`
  - Zero communications sent to candidates with null email addresses
