PATH: /user-stories/US-025-view-candidate-activity-history.md

# 🧾 User Story: View Candidate Activity History

## 🆔 Metadata
- ID: US-025
- Feature: Evaluation
- Priority: Medium
- Status: Draft
- Owner: PM

## 🎯 User Story
**As a** Hiring Manager or Recruiter  
**I want** to view a chronological activity timeline for a candidate showing stage changes, evaluations, comments, automation events, and AI actions  
**So that** I have a complete, auditable picture of everything that has happened to a candidate before making a final hiring decision

## 🧠 Business Context
- Related Use Case(s): UC-28 Consultar historial de actividad del candidato
- Entities involved: ActivityEvent, Application, StageTransition, Evaluation, Comment, CandidateViewEvent, AutomationExecution, AiSuggestion, User, Candidate
- Business Rules:
  - The activity history is displayed in chronological order (oldest to newest by `ActivityEvent.occurredAt`).
  - Each event must identify whether it was performed by a `User`, `Automation`, `AI`, or `System`.
  - Visibility of events depends on the viewer's role: `AdminOnly` events are hidden from Recruiter and Interviewer.
  - Manual events and automated/AI events must be visually distinguishable.
  - The history is read-only; no actions can be taken from this view.
  - All events are scoped to the authenticated user's Company (multi-tenant isolation).
- Assumptions:
  - `ActivityEvent` captures all relevant events: `Viewed`, `Commented`, `Mentioned`, `StageChanged`, `EvaluationSubmitted`, `BulkUpdated`, `AutomationExecuted`, `AiSuggested`.
  - The timeline covers the candidate across all Applications (not just one JobOpening).
- Dependencies:
  - `ActivityEvent` records must have been written for prior events (each module writes its own ActivityEvent on significant actions).
  - Candidate must exist.

## ✅ Acceptance Criteria (BDD + Edge Cases)

### 🎯 Happy Path
- **Given** an authenticated Hiring Manager and a Candidate with a rich activity history across two Applications  
  **When** the Hiring Manager opens the activity history view  
  **Then** the system renders a chronological timeline of all `ActivityEvent` records for the candidate and their Applications, each showing event type, actor (user name or "System/Automation/AI"), timestamp, and a brief description, with automated events visually tagged differently from manual events

### ⚠️ Edge Cases

#### Case 1: No activity events recorded
- **Given** a newly registered Candidate with no activity beyond creation  
  **When** the history view is opened  
  **Then** the system displays an empty timeline with a message "No activity recorded yet"  
- Handling:
  - Empty state is valid; do not return 404.

#### Case 2: Admin-only events hidden from Recruiter
- **Given** an `ActivityEvent` with `visibilityScope = AdminOnly` exists for a Candidate  
  **When** a Recruiter views the activity history  
  **Then** the AdminOnly event is not included in the timeline rendered for the Recruiter  
- Handling:
  - Filter `ActivityEvent` records by `visibilityScope` based on the viewer's role before returning results.

#### Case 3: Activity history for a candidate in multiple job openings
- **Given** a Candidate with Applications in three different JobOpenings, each with their own events  
  **When** the history is loaded  
  **Then** all events from all Applications appear in a single unified chronological timeline, each event labeled with the related JobOpening title for context  
- Handling:
  - Aggregate `ActivityEvent` records across `contextType = Application` (all Application IDs for this Candidate) and `contextType = Candidate`.

#### Case 4: Cross-tenant access attempt
- **Given** a HiringManager requests the activity history for a Candidate from another Company  
  **When** the request is processed  
  **Then** the system returns a 404 Not Found error  
- Handling:
  - Enforce `companyId` scoping on all `ActivityEvent` and Candidate lookups.

### 🚫 Validation Rules
- All `ActivityEvent` records returned must have `companyId` matching the authenticated user's Company.
- Events with `visibilityScope = AdminOnly` are excluded for Recruiter and Interviewer roles.
- Events with `visibilityScope = HiringTeam` are excluded for Interviewer role.
- Timeline is sorted ascending by `occurredAt`; ties broken by `id`.
- `actorUserId` must be non-null when `actorType = User`.
- Role must be HiringManager, Recruiter, or Admin.

## 📊 Business Impact
- KPI Affected: Hiring decision quality, Time-to-decision, Compliance and audit readiness
- Expected Impact: Provides a single source of truth for candidate journey history, eliminating the need to cross-reference multiple screens and reducing decision time by 20–30%
- Success Criteria:
  - Activity history loads in under 2 seconds for candidates with up to 200 events
  - Manual and automated events are visually distinct in 100% of cases
  - Zero cross-tenant events surfaced in any timeline view
