PATH: /user-stories/US-018-view-candidate-summary.md

# 🧾 User Story: View Candidate Summary

## 🆔 Metadata
- ID: US-018
- Feature: Candidate Management
- Priority: High
- Status: Draft
- Owner: PM

## 🎯 User Story
**As a** Recruiter or Hiring Manager  
**I want** to view a consolidated operational summary of a candidate including profile completeness, AI-generated insights, linked vacancies, and current pipeline status  
**So that** I can make a fast, informed initial decision without navigating multiple screens

## 🧠 Business Context
- Related Use Case(s): UC-21 Visualizar resumen de candidato
- Entities involved: Candidate, Application, JobOpening, PipelineStage, Note, Evaluation, CandidateViewEvent, ActivityEvent
- Business Rules:
  - Only candidates belonging to the active tenant (Company) are shown.
  - The summary must include the status of the candidate's most recent or highest-priority active Application.
  - If a candidate has multiple Applications, each must be clearly distinguished within the summary.
  - Incomplete profile sections must be flagged as pending.
  - Viewing the summary generates a `CandidateViewEvent` record for audit and collaboration analytics.
  - `Candidate.lastViewedAt` is updated on each view.
  - The information visible depends on the viewer's role.
- Assumptions:
  - The AI summary (`Candidate.aiSummary`) is displayed when `aiSummaryStatus = Ready`; otherwise a "Pending" or "Not available" state is shown.
  - `Candidate.profileCompletion` (0–100) is a pre-computed integer that indicates how complete the profile is.
- Dependencies:
  - Candidate must exist (US-004 or US-005).
  - UC-29 (Resume Summary) must execute for `aiSummary` to be populated.

## ✅ Acceptance Criteria (BDD + Edge Cases)

### 🎯 Happy Path
- **Given** an authenticated Recruiter and a Candidate with at least one Application and a completed AI summary  
  **When** the Recruiter selects the candidate from the list  
  **Then** the system displays: personal data, `profileCompletion` percentage, AI summary (if `aiSummaryStatus = Ready`), list of all Applications with their current `PipelineStage`, recent Notes and Evaluations, and a `CandidateViewEvent` is recorded with `sourceModule`, `viewerUserId`, and `viewedAt`

### ⚠️ Edge Cases

#### Case 1: Candidate with no information beyond basic fields
- **Given** a Candidate with `profileCompletion` below 30% and no Applications  
  **When** the Recruiter views the summary  
  **Then** the system renders the available data and marks missing sections (CV, phone, location, applications) as pending with a visual indicator  
- Handling:
  - Incomplete state is valid; the summary view must render regardless of completeness.

#### Case 2: AI summary not yet generated
- **Given** a Candidate where `aiSummaryStatus = Pending` or `Failed`  
  **When** the summary is displayed  
  **Then** the AI summary section shows a "Summary not available" message instead of blank content  
- Handling:
  - Never render null/blank in the AI summary section; always show a human-readable status.

#### Case 3: Candidate with multiple Applications across job openings
- **Given** a Candidate with three active Applications in different JobOpenings  
  **When** the summary is loaded  
  **Then** each Application appears as a distinct card or row showing job title, current stage, and `Application.lastActivityAt`  
- Handling:
  - Applications are sorted by `lastActivityAt` descending; the most recent surfaces first.

#### Case 4: Unauthorized access (cross-tenant)
- **Given** a Recruiter requests a Candidate from a different Company  
  **When** the summary endpoint is called  
  **Then** the system returns a 404 Not Found error  
- Handling:
  - Enforce `companyId` scoping on every Candidate lookup; never expose cross-tenant data.

### 🚫 Validation Rules
- The requested Candidate must belong to the authenticated user's `companyId`.
- `CandidateViewEvent` must be persisted atomically with the response (fire-and-forget or sync, but not skippable).
- `Candidate.lastViewedAt` must be updated on each view.
- `aiSummary` is only shown when `aiSummaryStatus = Ready`.
- Role must be Recruiter or HiringManager.

## 📊 Business Impact
- KPI Affected: Time-to-screening-decision, Recruiter operational throughput
- Expected Impact: Reduces time spent navigating between tabs to assess a candidate; accelerates initial triage by 30–50% per candidate review
- Success Criteria:
  - Summary loads in under 1.5 seconds including AI summary field
  - 100% of summary views produce a `CandidateViewEvent` record
