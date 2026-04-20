# USER STORIES
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

---

PATH: /user-stories/US-002-edit-job-opening.md

# 🧾 User Story: Edit Job Opening

## 🆔 Metadata
- ID: US-002
- Feature: Job Management
- Priority: High
- Status: Draft
- Owner: PM

## 🎯 User Story
**As a** Recruiter or Admin  
**I want** to modify the details of an existing job opening  
**So that** I can correct information or update the job profile without losing existing applications

## 🧠 Business Context
- Related Use Case(s): Editar Vacante (UC2)
- Entities involved: JobOpening, Company, User
- Business Rules:
  - The user must have role Recruiter or Admin.
  - The JobOpening must exist within the user's Company.
  - A closed JobOpening must not allow critical operational changes.
  - Editing the job opening does NOT affect existing PipelineStages that were already snapshot-copied.
  - The `sourceTemplateId` reference is read-only and not modifiable post-creation.
- Assumptions:
  - Basic change history is not shown to candidates — it is internal.
- Dependencies:
  - JobOpening must already exist (US-001).

## ✅ Acceptance Criteria (BDD + Edge Cases)

### 🎯 Happy Path
- **Given** an authenticated Recruiter and an existing Open JobOpening belonging to their Company  
  **When** the Recruiter modifies the title, description, location, or other editable fields and submits  
  **Then** the JobOpening record is updated with the new values, existing Applications remain intact, and the snapshot PipelineStages are not altered

### ⚠️ Edge Cases

#### Case 1: Editing a Closed job opening
- **Given** a Recruiter attempts to edit a JobOpening with status `Closed`  
  **When** the Recruiter submits the edit form  
  **Then** the system returns a 409 Conflict error indicating the job opening is closed and cannot be modified  
- Handling:
  - Block edits to closed job openings at the API layer; display a clear message in the UI.

#### Case 2: Concurrent edit conflict
- **Given** two Recruiters simultaneously edit the same JobOpening  
  **When** the second Recruiter submits after the first already saved  
  **Then** the system processes the last write and persists it, returning the updated record to the second Recruiter  
- Handling:
  - Last-write-wins for MVP; add optimistic concurrency (ETag) in a future iteration.

#### Case 3: Unauthorized role
- **Given** an authenticated user with role Interviewer or HiringManager  
  **When** the user attempts to edit a job opening  
  **Then** the system returns a 403 Forbidden error  
- Handling:
  - Only Recruiter and Admin may edit job openings.

### 🚫 Validation Rules
- `title` must not be set to blank.
- `description` must not be set to blank.
- `workMode` must remain within enum values `Onsite`, `Hybrid`, `Remote` if provided.
- `status` cannot be changed via the edit endpoint; use dedicated close endpoint (US-003).
- The edited JobOpening must belong to the same `companyId` as the authenticated user.

## 📊 Business Impact
- KPI Affected: Data quality of job openings, Recruiter operational efficiency
- Expected Impact: Reduces miscommunication caused by outdated job descriptions without disrupting active pipelines
- Success Criteria:
  - Zero Applications are orphaned or lost after a job opening edit
  - Edits are persisted within the same request cycle with no data loss

---

PATH: /user-stories/US-003-close-job-opening.md

# 🧾 User Story: Close Job Opening

## 🆔 Metadata
- ID: US-003
- Feature: Job Management
- Priority: High
- Status: Draft
- Owner: PM

## 🎯 User Story
**As a** Recruiter or Admin  
**I want** to close a job opening when the position is no longer accepting candidates  
**So that** no new applications are accepted while the historical data and pipeline remain accessible

## 🧠 Business Context
- Related Use Case(s): Cerrar Vacante (UC3)
- Entities involved: JobOpening, Application, Company, User
- Business Rules:
  - Only JobOpenings with status `Open` can be closed.
  - Closing sets `status = Closed` and records `closedAt` timestamp.
  - A closed JobOpening does not accept new Applications.
  - Existing Applications and their StageTransitions are preserved.
  - A closed JobOpening cannot be reopened in MVP (post-MVP enhancement).
- Assumptions:
  - Closing a job opening does not automatically reject active Applications; the Recruiter handles that manually.
- Dependencies:
  - JobOpening must exist and be in `Open` status.

## ✅ Acceptance Criteria (BDD + Edge Cases)

### 🎯 Happy Path
- **Given** an authenticated Recruiter and an existing Open JobOpening  
  **When** the Recruiter triggers the close action  
  **Then** the JobOpening status changes to `Closed`, `closedAt` is set to current timestamp, and the endpoint rejects any subsequent new Application submissions for that job opening

### ⚠️ Edge Cases

#### Case 1: Closing an already-closed job opening
- **Given** a Recruiter attempts to close a JobOpening already in `Closed` status  
  **When** the close action is submitted  
  **Then** the system returns a 409 Conflict error indicating the job opening is already closed  
- Handling:
  - Idempotency check: if already closed, return error with current state.

#### Case 2: Closing a Draft job opening
- **Given** a Recruiter attempts to close a JobOpening in `Draft` status  
  **When** the close action is submitted  
  **Then** the system returns a 422 Unprocessable Entity error  
- Handling:
  - Only `Open` job openings can be transitioned to `Closed`. Provide clear status transition guidance.

#### Case 3: New application submitted to closed job opening
- **Given** a Candidate submits an application after the job opening is closed  
  **When** POST /applications is called with the closed job opening ID  
  **Then** the system returns a 422 error indicating the job opening is not accepting applications  
- Handling:
  - Validate JobOpening status before creating any Application.

### 🚫 Validation Rules
- Only `Open` → `Closed` transition is valid in MVP.
- `closedAt` is set server-side at time of close; it cannot be provided by the client.
- The JobOpening must belong to the authenticated user's Company.
- Role must be Recruiter or Admin.

## 📊 Business Impact
- KPI Affected: Pipeline integrity, Time-to-hire measurement accuracy
- Expected Impact: Prevents pipeline pollution from late applications after a position is filled, ensuring clean reporting
- Success Criteria:
  - No Applications created for closed job openings
  - `closedAt` timestamp present on 100% of closed job openings

---

# BACKLOG PRIORIZADO

PATH: /backlog/product-backlog.md

# LTI ATS — Product Backlog

**Version:** 1.0  
**Date:** 2026-04-20  
**Author:** Walter Alexander Cordero Lara  
**Prioritization Method:** MoSCoW  
**Scope:** MVP + Immediate Post-MVP  

---

## Legend

| Priority | Label | Description |
|----------|-------|-------------|
| **Must** | M | Non-negotiable for MVP. System cannot function without it. |
| **Should** | S | High value; include in MVP if feasible, else first post-MVP iteration. |
| **Could** | C | Nice-to-have; included only if time/resources allow. |
| **Won't** | W | Explicitly deferred to a future release cycle. |

---

## Feature: Job Management

> Core feature. No job opening = no pipeline = no ATS. All stories here are foundational.

| ID | Title | Priority | Short Description | Business Value | Dependencies |
|----|-------|----------|-------------------|----------------|--------------|
| US-001 | Create Job Opening | **Must** | Recruiter creates a vacante with title, description, dept, location, and pipeline template snapshot | Formally starts a recruitment cycle; enables pipeline tracking from day one | PipelineTemplate must exist; authenticated Recruiter or Admin |
| US-002 | Edit Job Opening | **Must** | Recruiter modifies an existing open job opening without affecting existing applications | Keeps job descriptions accurate without disrupting active pipelines | US-001 must exist |
| US-003 | Close Job Opening | **Must** | Recruiter closes a job opening, preventing new applications while preserving history | Ensures pipeline integrity and time-to-hire measurement accuracy | US-001 must be in Open status |

---

## Feature: Candidate Capture

> Entry point for all candidate data. Without capture, the pipeline is empty.

| ID | Title | Priority | Short Description | Business Value | Dependencies |
|----|-------|----------|-------------------|----------------|--------------|
| US-004 | Apply to Job Opening | **Must** | Public-facing application form allowing candidates to submit name, email, and CV | Standardizes candidate entry; drives pipeline volume | US-001 (Open JobOpening); PipelineStage snapshot must exist |
| US-005 | Register Candidate Manually | **Must** | Recruiter manually registers offline/referral candidates and optionally links them to a job opening | Ensures no candidate is lost from non-digital channels; expands talent pool | Authenticated Recruiter or Admin; optionally US-001 |

---

## Feature: Candidate Management

> Enables Recruiters to view, search, and manage candidate profiles. Core operational need.

| ID | Title | Priority | Short Description | Business Value | Dependencies |
|----|-------|----------|-------------------|----------------|--------------|
| US-006 | View Candidate Profile | **Must** | Unified profile view with personal data, CV, applications, evaluations, and internal notes | Centralizes all candidate info for informed, context-rich decisions | US-004 or US-005 |
| US-007 | Search Candidates | **Must** | Search and filter candidates by name, email, job opening, or application status with pagination | Reduces time to locate candidates as volume grows; enables talent pool reuse | Candidates must exist in the system |
| US-018 | View Candidate Summary | **Must** | Consolidated summary card with profile completeness, AI insights (when ready), linked vacancies, and current stage | Accelerates initial triage by 30–50% per candidate review; reduces multi-screen navigation | US-004 or US-005; UC-29 (US-026) for AI field population |
| US-019 | Execute Bulk Actions on Candidates | **Should** | Select multiple candidates and apply a single action (reject, move stage, tag, assign) simultaneously | Enables 10x throughput for high-volume pipelines without manual repetition | US-004 or US-005; US-010 for MoveStage action |
| US-008 | Update Candidate Information | **Could** | Recruiter edits editable candidate fields (name, phone, location, resumeUrl); email is immutable | Keeps profiles current without affecting linked application history | US-004 or US-005 |
| US-020 | Save Candidate Search Filter | **Could** | Recruiter saves a named combination of search filters for instant reuse in future sessions | Saves 3–5 minutes per session for high-frequency searches; improves operational consistency | Authenticated Recruiter, HiringManager, or Admin |

---

## Feature: Pipeline Management

> The Kanban pipeline is the central UX of any ATS. All stories here are MVP-critical.

| ID | Title | Priority | Short Description | Business Value | Dependencies |
|----|-------|----------|-------------------|----------------|--------------|
| US-009 | View Recruitment Pipeline | **Must** | Kanban board showing candidates per pipeline stage for a specific job opening | Provides real-time visibility into recruitment funnel; identifies bottlenecks immediately | US-001 (with stages); US-004 or US-005 |
| US-010 | Move Candidate to Another Stage | **Must** | Recruiter or HiringManager manually moves an application to a target stage with auditable StageTransition | Provides complete, auditable pipeline movement history for bottleneck analysis | Active Application; target PipelineStage must belong to same JobOpening |
| US-011 | Discard Candidate from Job Opening | **Must** | Recruiter rejects an application, moving it to the Rejected terminal stage with optional reason | Keeps active pipeline focused on viable candidates; preserves rejection history for compliance | Active Application; Rejected PipelineStage must exist in snapshot |
| US-012 | Mark Candidate as Hired | **Must** | Recruiter formally marks a candidate as hired, triggering final status update and StageTransition | Provides definitive business outcome signal; enables accurate time-to-hire reporting | Active Application; Hired PipelineStage must exist in snapshot |

---

## Feature: Evaluation & Collaboration

> Structured feedback drives quality hiring decisions. Both stories are MVP-essential.

| ID | Title | Priority | Short Description | Business Value | Dependencies |
|----|-------|----------|-------------------|----------------|--------------|
| US-013 | Register Candidate Feedback | **Must** | Team members post free-text qualitative notes on a candidate or application (internal only) | Creates a permanent, auditable record of team impressions; reduces decision bias | Application or Candidate must exist |
| US-014 | Register Interview Evaluation | **Must** | Interviewer submits a structured evaluation with score and recommendation after interviewing | Replaces subjective impressions with structured, comparable data for consistent decisions | Active Application; authenticated Interviewer/HiringManager/Recruiter |

---

## Feature: Operational Communication

> Candidate-facing email is fundamental to employer branding and process transparency.

| ID | Title | Priority | Short Description | Business Value | Dependencies |
|----|-------|----------|-------------------|----------------|--------------|
| US-015 | Send Communication to Candidate | **Must** | Recruiter sends an email to a candidate directly from the ATS; delivery and status are logged | Improves employer branding; provides auditable communication trail; reduces ghosting | Application must exist; Candidate must have valid email; Email Provider configured |

---

## Feature: Basic Administration

> User and role management is a security and operational prerequisite for multi-user operation.

| ID | Title | Priority | Short Description | Business Value | Dependencies |
|----|-------|----------|-------------------|----------------|--------------|
| US-016 | Manage Internal Users | **Must** | Admin creates, edits, and deactivates internal ATS users; deactivated users lose all access immediately | Ensures right people have right access; enables team onboarding within one Admin session | Active Company; Admin must be active |
| US-017 | Assign Roles and Permissions | **Must** | Admin assigns or changes the role of an existing user (Admin, Recruiter, HiringManager, Interviewer) | Ensures precisely scoped access; enables rapid role adjustments as the team evolves | US-016 (user must exist and be active) |

---

## Feature: Pipeline & Tracking

> Advanced pipeline visibility and automation execution. Foundational for scaled operations.

| ID | Title | Priority | Short Description | Business Value | Dependencies |
|----|-------|----------|-------------------|----------------|--------------|
| US-022 | Auto-Update Candidate Status via Automation | **Must** | System evaluates active automation rules on pipeline events and applies configured actions automatically | Eliminates 40–60% of manual status updates; improves process consistency at scale | US-027 (AutomationRule configured and active) |
| US-021 | AI-Assisted Candidate Ranking | **Should** | Recruiter triggers on-demand AI ranking of all active candidates for a job opening with explanations | Reduces manual screening time by up to 60% for high-volume openings; surfaces best-fit candidates first | Open JobOpening with active Applications; AI Gateway configured |
| US-023 | Monitor Pipeline SLA | **Could** | Dashboard overlay showing which candidates have exceeded or are at risk of exceeding stage time targets | Reduces average stage dwell time; expected 20–30% reduction in breach frequency after adoption | SlaPolicy configured by Admin; US-010 (stageEnteredAt populated) |

---

## Feature: Evaluation (Structured Scorecard)

> Extends basic feedback with template-driven scorecards for standardized, bias-reduced interviews.

| ID | Title | Priority | Short Description | Business Value | Dependencies |
|----|-------|----------|-------------------|----------------|--------------|
| US-024 | Evaluate Candidate with Structured Scorecard | **Must** | Interviewer completes a scorecard template with per-criterion scores; weighted average auto-computed | Standardizes interview feedback; enables apples-to-apples candidate comparison; reduces bias | ScorecardTemplate with active criteria (Admin-configured); Active Application |
| US-025 | View Candidate Activity History | **Could** | Chronological timeline of all candidate events: stage changes, evaluations, comments, automation, and AI actions | Single source of truth for candidate journey; reduces decision time by 20–30%; supports audit readiness | ActivityEvent records written by prior actions; Candidate must exist |

---

## Feature: Collaboration

> In-context team discussion directly on candidate records eliminates external tool dependency.

| ID | Title | Priority | Short Description | Business Value | Dependencies |
|----|-------|----------|-------------------|----------------|--------------|
| US-028 | Comment on a Candidate | **Must** | Team members post internal collaborative comments on a candidate's profile or application | Centralizes all candidate discussion within ATS; eliminates Slack/email context-switching per candidate | Candidate or Application must exist within same Company |
| US-029 | Mention Collaborator in Comment | **Must** | Recruiter uses @-mention in a comment to notify a specific colleague with an in-app notification | Replaces ad-hoc pings with in-context mentions; reduces response-to-feedback time by 30–50% | US-028 (Comment must exist); mentioned user must be active in same Company |

---

## Feature: Automation

> Rule engine configuration is the prerequisite for the automation execution story (US-022).

| ID | Title | Priority | Short Description | Business Value | Dependencies |
|----|-------|----------|-------------------|----------------|--------------|
| US-027 | Configure Automation Rule | **Must** | Admin creates automation rules with triggers, conditions, and actions scoped to global, job opening, or stage level | Enables 40–60% reduction in repetitive recruiter tasks; prerequisite for all automation execution | Automation module enabled for tenant; Admin role required |

---

## Feature: AI Assistance

> AI features that accelerate screening and interview preparation. High ROI on recruiter time.

| ID | Title | Priority | Short Description | Business Value | Dependencies |
|----|-------|----------|-------------------|----------------|--------------|
| US-026 | AI-Powered CV Summary Generation | **Must** | System auto-generates an AI summary of a candidate's CV on upload or on-demand; displayed with "AI-generated" label | Reduces CV reading from ~4 min to under 30 sec; enables 5–8x recruiter throughput at initial screening | Candidate must have non-null resumeUrl; AI Gateway configured; AI enabled for tenant |
| US-030 | AI-Generated Interview Questions | **Could** | Interviewer requests AI-tailored interview questions based on candidate profile and job description | Reduces interview prep time by 50–70%; improves question relevance and consistency across interviewers | Candidate profile data or JobOpening description; AI Gateway configured; AI enabled for tenant |

---

## Summary Table — Full Backlog by MoSCoW Priority

| ID | Title | Feature | Priority |
|----|-------|---------|----------|
| US-001 | Create Job Opening | Job Management | Must |
| US-002 | Edit Job Opening | Job Management | Must |
| US-003 | Close Job Opening | Job Management | Must |
| US-004 | Apply to Job Opening | Candidate Capture | Must |
| US-005 | Register Candidate Manually | Candidate Capture | Must |
| US-006 | View Candidate Profile | Candidate Management | Must |
| US-007 | Search Candidates | Candidate Management | Must |
| US-009 | View Recruitment Pipeline | Pipeline Management | Must |
| US-010 | Move Candidate to Another Stage | Pipeline Management | Must |
| US-011 | Discard Candidate from Job Opening | Pipeline Management | Must |
| US-012 | Mark Candidate as Hired | Pipeline Management | Must |
| US-013 | Register Candidate Feedback | Evaluation & Collaboration | Must |
| US-014 | Register Interview Evaluation | Evaluation & Collaboration | Must |
| US-015 | Send Communication to Candidate | Operational Communication | Must |
| US-016 | Manage Internal Users | Basic Administration | Must |
| US-017 | Assign Roles and Permissions | Basic Administration | Must |
| US-018 | View Candidate Summary | Candidate Management | Must |
| US-022 | Auto-Update Candidate Status | Pipeline & Tracking | Must |
| US-024 | Evaluate with Structured Scorecard | Evaluation | Must |
| US-026 | AI-Powered CV Summary Generation | AI Assistance | Must |
| US-027 | Configure Automation Rule | Automation | Must |
| US-028 | Comment on a Candidate | Collaboration | Must |
| US-029 | Mention Collaborator in Comment | Collaboration | Must |
| US-019 | Execute Bulk Actions on Candidates | Candidate Management | Should |
| US-021 | AI-Assisted Candidate Ranking | Pipeline & Tracking | Should |
| US-008 | Update Candidate Information | Candidate Management | Could |
| US-020 | Save Candidate Search Filter | Candidate Management | Could |
| US-023 | Monitor Pipeline SLA | Pipeline & Tracking | Could |
| US-025 | View Candidate Activity History | Evaluation | Could |
| US-030 | AI-Generated Interview Questions | AI Assistance | Could |

---

## Prioritization Rationale

### Overall Approach

Prioritization follows three axes applied in order:

1. **MVP Core Completeness** — Does the system remain functional as an ATS without this story? If not, it is Must.
2. **Dependency Chain** — Stories that are prerequisites for other Must/Should items are promoted to Must regardless of standalone value.
3. **Business Impact vs. Implementation Risk** — Higher-complexity stories with equivalent impact are downgraded to Should or Could when simpler alternatives cover the immediate need.

---

### Must — Why These 23 Stories Are Non-Negotiable

**Job Management (US-001, 002, 003):** The ATS cannot operate without the ability to open, maintain, and close positions. These three stories form the foundational entity lifecycle. US-001 is the root of the entire data model (JobOpening → PipelineStages → Applications).

**Candidate Capture (US-004, 005):** Without inbound candidates (public apply) and offline candidates (manual registration), the pipeline is always empty. Both channels must coexist to reflect real-world recruiting operations.

**Candidate Management (US-006, 007, 018):** Viewing and finding candidates are read operations that unlock all downstream decision-making. US-018 (Candidate Summary) is elevated to Must because it is the primary daily-use screen for Recruiters — it surfaces AI-generated data (US-026), pipeline status, and evaluations in a single view, directly reducing time-to-decision.

**Pipeline Management (US-009, 010, 011, 012):** These four stories together represent the complete Application lifecycle: visualize → move → reject → hire. Removing any one of them breaks the end-to-end flow. They are the core ATS differentiator versus a simple spreadsheet.

**Evaluation & Collaboration (US-013, 014):** Free-text feedback (US-013) and structured evaluations (US-014) are required for team alignment before any hiring decision. Without them, the ATS becomes a pipeline tracker without qualitative input — insufficient for professional recruiting.

**Operational Communication (US-015):** Candidate-facing email communication is a legal and operational requirement in any professional recruiting context. It directly affects candidate experience and employer branding metrics.

**Basic Administration (US-016, 017):** Multi-user access control is a security prerequisite. Without user management and role assignment, the system cannot enforce any of its role-based access rules, making all other stories unsecured.

**Pipeline & Tracking — Automation Execution (US-022):** Elevated to Must because it is the runtime counterpart of US-027 (rule configuration). Configuring rules without executing them produces no business value; both must ship together.

**Evaluation — Structured Scorecard (US-024):** The product's key differentiator for reducing hiring bias. Structured scorecards are explicitly called out in the product vision as a Priority 1 feature (LTI-WACL.md, Prioridad 1: Evaluación y feedback). This elevates US-024 above a simple Should.

**AI Assistance — CV Summary (US-026):** Elevated to Must because US-018 (Candidate Summary) depends on `aiSummaryStatus` to populate a core field in the summary view. Without US-026, the summary view renders a permanent "Not available" state for AI insights, degrading the flagship screen.

**Automation — Rule Configuration (US-027):** Prerequisite for US-022. Cannot ship automation execution without first allowing Admins to configure rules.

**Collaboration (US-028, 029):** Commenting (US-028) is the foundation for async team collaboration within the ATS. Mentions (US-029) extend it with notifications, making comments actionable. Both are in the product vision as Priority 2 (Colaboración en equipo). They are elevated to Must because they replace the need for external Slack/email threads — a critical adoption driver.

---

### Should — High Value, Post-MVP First Wave

**US-019 (Bulk Actions):** High operational value for teams processing 50+ applications per job opening, but requires US-010 (move) and US-011 (reject) to be stable first. Delivers 10x throughput gains but is not blocking day-one ATS use. Promoted to Should over Could because high-volume pipelines cannot be managed manually at scale for more than a few weeks.

**US-021 (AI Ranking):** Requires the AI Gateway and multiple existing Applications to be useful. Its value is real (60% reduction in screening time), but it depends on US-026 (AI infrastructure) and US-004/US-005 (volume of candidates). Best delivered as the second AI feature after the CV summary is stable.

---

### Could — Meaningful Enhancements, Lower Urgency

**US-008 (Update Candidate Info):** Basic CRUD on a candidate profile. Useful but not blocking any critical flow since the data is captured correctly on entry (US-004, 005). Recruiters can work around temporary inaccuracies.

**US-020 (Save Candidate Filter):** A productivity enhancement for repeat search patterns. Delivers 3–5 minutes saved per session. Not blocking any recruitment decision; purely an efficiency gain.

**US-023 (Pipeline SLA Monitoring):** Requires SlaPolicy configuration by Admin and populated `stageEnteredAt` data from US-010. This is a process maturity feature — it adds maximum value after teams have been using the system for 4–6 weeks and have sufficient data to identify bottlenecks. Premature for day-one MVP.

**US-025 (Candidate Activity History):** A powerful audit and compliance feature, but the timeline is only meaningful after substantial activity has been generated across multiple stories. Best delivered once US-010, US-013, US-014, US-022, US-028, and US-029 are all producing ActivityEvent records.

**US-030 (AI Interview Questions):** High perceived value for interviewers, but lower frequency of use than US-026 (CV summaries are used on every candidate; interview questions are only needed when an interview is scheduled). Also depends on AI Gateway being stable, which is validated via US-026 first.

---

### Won't (This Sprint / Cycle)

No stories from the provided set of US-001 through US-030 are formally classified as "Won't" — all 30 stories have clear business value and will be delivered across MVP and post-MVP iterations. The Could stories are deferred to a subsequent sprint cycle but remain on the roadmap.

Features explicitly out of scope for the current planning cycle (referenced in LTI-WACL.md under Prioridad 3) include:

- Analytics & Reporting dashboard (time-to-hire funnel, conversion by source)
- Multi-channel job posting integrations (LinkedIn, external job boards)
- Calendar integrations (Google Calendar / Outlook) for interview scheduling
- HRIS / Payroll system integrations
- Candidate self-service portal (status tracking, bidirectional communication)
- GDPR / data retention compliance tooling
- Employer branding and career page customization

These are acknowledged as future roadmap items and will be re-evaluated in a subsequent planning session once the MVP has been validated in production.

---

# TASKS USER STORY 001
---
# FILE: task-001-prisma-schema-migration.md
---

PATH: /tasks/us-001-create-job-opening/task-001-prisma-schema-migration.md

# 🔧 Task: Prisma Schema — Job, PipelineTemplate, PipelineStage Models + Migration

## 🆔 Metadata
- ID: TASK-001
- Related User Story: US-001
- Layer: Persistence
- Estimation: 3 points

---

## 📝 Description

Define the Prisma models for `Job`, `PipelineTemplate`, and `PipelineStage` in `apps/api/prisma/schema.prisma`. The `PipelineStage` model supports both template-owned and job-owned stages via `ownerType` + `ownerId` polymorphism (the pipeline snapshot pattern). Create and run the corresponding database migration, including all required indexes for tenant-scoped queries.

---

## ⚙️ Steps

1. Open `apps/api/prisma/schema.prisma`.
2. Add the `Job` model:
   ```prisma
   model Job {
     id               String    @id @default(cuid())
     tenantId         String    @map("tenant_id")
     title            String
     department       String
     location         String
     workMode         String    @map("work_mode")
     description      String    @db.Text
     status           String    @default("draft")
     sourceTemplateId String?   @map("source_template_id")
     createdBy        String    @map("created_by")
     createdAt        DateTime  @default(now()) @map("created_at")
     updatedAt        DateTime  @updatedAt @map("updated_at")
     publishedAt      DateTime? @map("published_at")
     closedAt         DateTime? @map("closed_at")

     tenant         Tenant          @relation(fields: [tenantId], references: [id], onDelete: Cascade)
     pipelineStages PipelineStage[]

     @@index([tenantId, status],    map: "idx_jobs_tenant_status")
     @@index([tenantId, createdAt], map: "idx_jobs_tenant_created_at")
     @@map("jobs")
   }
   ```
3. Add the `PipelineTemplate` model:
   ```prisma
   model PipelineTemplate {
     id        String    @id @default(cuid())
     tenantId  String    @map("tenant_id")
     name      String
     isDefault Boolean   @default(false) @map("is_default")
     createdAt DateTime  @default(now()) @map("created_at")
     updatedAt DateTime  @updatedAt @map("updated_at")

     tenant Tenant          @relation(fields: [tenantId], references: [id], onDelete: Cascade)
     stages PipelineStage[]

     @@unique([tenantId, isDefault], map: "uq_pipeline_templates_tenant_default")
     @@index([tenantId], map: "idx_pipeline_templates_tenant")
     @@map("pipeline_templates")
   }
   ```
4. Add the `PipelineStage` model (polymorphic via `ownerType` + `ownerId`):
   ```prisma
   model PipelineStage {
     id          String   @id @default(cuid())
     tenantId    String   @map("tenant_id")
     ownerType   String   @map("owner_type")   // 'PipelineTemplate' | 'Job'
     ownerId     String   @map("owner_id")
     name        String
     position    Int
     isFinal     Boolean  @default(false) @map("is_final")
     stageType   String   @default("standard") @map("stage_type")  // 'standard' | 'rejected' | 'hired'
     createdAt   DateTime @default(now()) @map("created_at")

     tenant           Tenant            @relation(fields: [tenantId], references: [id], onDelete: Cascade)
     pipelineTemplate PipelineTemplate? @relation(fields: [ownerId], references: [id], map: "fk_pipeline_stages_template")
     job              Job?              @relation(fields: [ownerId], references: [id], map: "fk_pipeline_stages_job")

     @@index([tenantId, ownerType, ownerId], map: "idx_pipeline_stages_owner")
     @@index([ownerId, position],            map: "idx_pipeline_stages_position")
     @@map("pipeline_stages")
   }
   ```
5. Run `pnpm prisma migrate dev --name add-jobs-pipeline-schema` from `apps/api/`.
6. Run `pnpm prisma generate` to regenerate the Prisma client.
7. Verify migration applied cleanly: `pnpm prisma migrate status`.

---

## ✅ Acceptance Criteria

- **Given** the schema file is updated and migration is run  
  **When** `pnpm prisma migrate status` is executed  
  **Then** all migrations are applied with no pending state

- **Given** a `Job` record is inserted for `tenantId = 'tenant_A'`  
  **When** a query filters by `tenantId = 'tenant_B'`  
  **Then** zero rows are returned (tenant isolation enforced at DB level)

- **Given** a `PipelineStage` with `ownerType = 'Job'`  
  **When** the linked `Job` is deleted  
  **Then** the `PipelineStage` records cascade-delete via `tenantId` constraint

- **Given** two `PipelineTemplate` records for the same tenant  
  **When** both have `isDefault = true`  
  **Then** the unique constraint `uq_pipeline_templates_tenant_default` prevents the second insert

---

## 🔗 Dependencies

- None (first task — no code dependencies)


---
# FILE: task-002-domain-value-objects.md
---

PATH: /tasks/us-001-create-job-opening/task-002-domain-value-objects.md

# 🔧 Task: Domain — Value Objects (JobId, JobTitle, JobStatus, WorkMode)

## 🆔 Metadata
- ID: TASK-002
- Related User Story: US-001
- Layer: Domain
- Estimation: 2 points

---

## 📝 Description

Implement the four Value Objects required by the `Job` aggregate root in `modules/jobs/domain/value-objects/`. Each VO MUST be immutable, self-validating, and throw a domain-specific error on invalid input. Use the `ValueObject<T>` base class from `@/shared/domain/value-object`. No VO may import from `infrastructure/` or `application/`.

---

## ⚙️ Steps

1. Create `job-id.vo.ts`:
   - Wraps a non-empty `string`.
   - Throws `InvalidValueError` (from shared) if blank.
   - Exposes `get value(): string`.

2. Create `job-title.vo.ts`:
   - Trims input on creation.
   - Min length: 3 chars → throws `JobTitleTooShortError`.
   - Max length: 120 chars → throws `JobTitleTooLongError`.
   - Exposes `get value(): string`.

3. Create `job-status.vo.ts`:
   - Enum values: `'draft' | 'open' | 'closed' | 'archived'`.
   - Static factories: `draft()`, `open()`.
   - `canTransitionTo(next: JobStatus): boolean` — enforces state machine:
     - `draft → open | archived`
     - `open → closed | archived`
     - `closed → archived`
     - `archived → (none)`
   - `transitionTo(next: JobStatus): JobStatus` — throws `InvalidJobStatusTransitionError` if transition is illegal.

4. Create `work-mode.vo.ts`:
   - Enum values: `'remote' | 'hybrid' | 'onsite'`.
   - Throws `InvalidValueError` if value is not in enum.
   - Exposes `get value(): WorkModeValue`.

5. Export all four VOs from `modules/jobs/domain/value-objects/index.ts`.

---

## ✅ Acceptance Criteria

- **Given** `JobTitle.create('AB')`  
  **When** the VO is instantiated  
  **Then** `JobTitleTooShortError` is thrown with `actualLength = 2`

- **Given** `JobTitle.create('  ')`  
  **When** the VO is instantiated  
  **Then** `JobTitleTooShortError` is thrown (trimmed length = 0 < 3)

- **Given** `JobStatus.draft().transitionTo(JobStatus.create('closed'))`  
  **When** the transition is called  
  **Then** `InvalidJobStatusTransitionError` is thrown

- **Given** `JobStatus.draft().transitionTo(JobStatus.open())`  
  **When** the transition is called  
  **Then** a new `JobStatus` with value `'open'` is returned

- **Given** `WorkMode.create('fulltime' as any)`  
  **When** the VO is instantiated  
  **Then** `InvalidValueError` is thrown

---

## 🔗 Dependencies

- TASK-003 (domain errors must exist first for imports in `job-title.vo.ts` and `job-status.vo.ts`)


---
# FILE: task-003-domain-errors.md
---

PATH: /tasks/us-001-create-job-opening/task-003-domain-errors.md

# 🔧 Task: Domain — Job Domain Errors

## 🆔 Metadata
- ID: TASK-003
- Related User Story: US-001
- Layer: Domain
- Estimation: 1 point

---

## 📝 Description

Create `modules/jobs/domain/errors/job.errors.ts` with all domain-specific error classes for the `jobs` bounded context. Each error MUST extend `DomainError` (from `@/shared/domain/errors/domain-error`), carry a typed `readonly code` string, and include structured metadata as constructor parameters. No generic `Error` instances are permitted in the domain layer.

---

## ⚙️ Steps

1. Create `modules/jobs/domain/errors/job.errors.ts`.
2. Implement the following error classes:

   | Class | Code | Constructor params | When thrown |
   |---|---|---|---|
   | `JobTitleTooShortError` | `JOB_TITLE_TOO_SHORT` | `actualLength: number, minLength: number` | `JobTitle.create()` with < 3 chars |
   | `JobTitleTooLongError` | `JOB_TITLE_TOO_LONG` | `actualLength: number, maxLength: number` | `JobTitle.create()` with > 120 chars |
   | `InvalidJobStatusTransitionError` | `INVALID_JOB_STATUS_TRANSITION` | `from: string, to: string` | `JobStatus.transitionTo()` with illegal transition |
   | `JobNotFoundError` | `JOB_NOT_FOUND` | `jobId: string` | Repository returns null |
   | `NoPipelineTemplateFoundError` | `NO_PIPELINE_TEMPLATE_FOUND` | `tenantId: string` | No active template exists for the company |
   | `PipelineTemplateNotInTenantError` | `PIPELINE_TEMPLATE_NOT_IN_TENANT` | `templateId: string, tenantId: string` | `sourceTemplateId` belongs to a different tenant |
   | `InactiveCompanyError` | `INACTIVE_COMPANY` | `tenantId: string` | Company status is not `Active` |

3. Each error message must be human-readable and include the discriminating value(s).
4. Export all errors from `modules/jobs/domain/errors/index.ts`.

---

## ✅ Acceptance Criteria

- **Given** `new JobTitleTooShortError(2, 3)`  
  **When** `.message` is read  
  **Then** it contains both `2` (actual) and `3` (min) in the message string

- **Given** `new NoPipelineTemplateFoundError('tenant_abc')`  
  **When** `.code` is read  
  **Then** it equals `'NO_PIPELINE_TEMPLATE_FOUND'`

- **Given** any error in `job.errors.ts`  
  **When** `instanceof DomainError` is checked  
  **Then** it returns `true`

- **Given** `new InactiveCompanyError('tenant_X')`  
  **When** `.metadata` is accessed  
  **Then** `{ tenantId: 'tenant_X' }` is returned

---

## 🔗 Dependencies

- None (shared `DomainError` base class must exist in `@/shared/domain/errors/`)


---
# FILE: task-004-domain-aggregate-event-repository.md
---

PATH: /tasks/us-001-create-job-opening/task-004-domain-aggregate-event-repository.md

# 🔧 Task: Domain — Job Aggregate Root, JobCreatedEvent, JobRepository Port

## 🆔 Metadata
- ID: TASK-004
- Related User Story: US-001
- Layer: Domain
- Estimation: 3 points

---

## 📝 Description

Implement three domain artifacts in `modules/jobs/domain/`:

1. **`Job` Aggregate Root** (`entities/job.entity.ts`) — the central entity that encapsulates all invariants for a job opening. Must use `Clock` and `IdGenerator` ports (never `new Date()` or `crypto.randomUUID()` directly). Exposes `create()` static factory and `toPrimitives()` for serialization.

2. **`JobCreatedEvent`** (`events/job-created.event.ts`) — domain event raised inside `Job.create()`. Carries `tenantId`, `jobId`, `title`, `status`, and `createdBy` in its payload.

3. **`JobRepository` port** (`ports/job.repository.ts`) — interface defining persistence contract. Concrete implementations live in `infrastructure/` (not here).

---

## ⚙️ Steps

### Job Aggregate Root (`job.entity.ts`)

1. Extend `AggregateRoot<JobId>` from shared.
2. Define `JobProps` interface with all primitive-typed fields plus VOs.
3. Implement `static create(params: CreateJobParams, clock: Clock, idGen: IdGenerator): Job`:
   - Generates ID via `idGen.generate()`.
   - Creates all VOs (`JobTitle`, `JobStatus`, `WorkMode`).
   - Sets `publishedAt = clock.now()` only when `params.status.value === 'open'`; else `null`.
   - Calls `this.addDomainEvent(new JobCreatedEvent(...))`.
4. Implement `static reconstitute(props: JobProps): Job` — rebuilds from persistence (no events raised).
5. Implement `toPrimitives()` returning a plain object with primitive types only.
6. Add read-only getters: `id`, `tenantId`, `status`.
7. Add `sourceTemplateId?: string` to `JobProps` and `CreateJobParams` (nullable — stores which template was used for the snapshot).

### JobCreatedEvent (`events/job-created.event.ts`)

1. Extend `DomainEvent` from shared.
2. Set `eventName = 'jobs.job.created'` and `eventVersion = 1`.
3. Constructor receives `{ aggregateId, tenantId, title, status, createdBy, occurredAt, idGenerator }`.
4. `toPayload()` returns `{ jobId, tenantId, title, status, createdBy }`.

### JobRepository Port (`ports/job.repository.ts`)

1. Define `interface JobRepository`:
   - `findById(tenantId: string, id: JobId): Promise<Job | null>`
   - `save(job: Job): Promise<void>`
2. Export `JOB_REPOSITORY = Symbol('JobRepository')` for DI token.

---

## ✅ Acceptance Criteria

- **Given** `Job.create({ ...validParams, status: JobStatus.open() }, clock, idGen)`  
  **When** the factory is called  
  **Then** `job.toPrimitives().publishedAt` is not null and equals `clock.now()`

- **Given** `Job.create({ ...validParams, status: JobStatus.draft() }, clock, idGen)`  
  **When** the factory is called  
  **Then** `job.toPrimitives().publishedAt` is null

- **Given** `Job.create(validParams, clock, idGen)` is called  
  **When** `job.pullDomainEvents()` is called  
  **Then** exactly one `JobCreatedEvent` is returned with `eventName = 'jobs.job.created'`

- **Given** `Job.reconstitute(props)` is called  
  **When** `job.pullDomainEvents()` is called  
  **Then** zero domain events are returned

- **Given** `JobRepository` interface is used in a handler  
  **When** the concrete implementation is replaced with an in-memory fake  
  **Then** the handler compiles and runs without modification

---

## 🔗 Dependencies

- TASK-002 (Value Objects must exist)
- TASK-003 (Domain errors must exist)


---
# FILE: task-005-domain-pipeline-snapshot-port.md
---

PATH: /tasks/us-001-create-job-opening/task-005-domain-pipeline-snapshot-port.md

# 🔧 Task: Domain — PipelineTemplateRepository Port + PipelineStage Entity

## 🆔 Metadata
- ID: TASK-005
- Related User Story: US-001
- Layer: Domain
- Estimation: 2 points

---

## 📝 Description

Define the domain contracts for pipeline template resolution and stage snapshot creation. This task creates the `PipelineTemplateRepository` port interface and a lightweight `PipelineStageSnapshot` value type used to represent a stage copied from a template into a job's pipeline. These artifacts are consumed by the application-layer handler (`CreateJobHandler`) to enforce the business rule: *"A PipelineTemplate's stages are snapshot-copied to the new JobOpening at creation time."*

---

## ⚙️ Steps

1. Create `modules/jobs/domain/ports/pipeline-template.repository.ts`:
   ```typescript
   export interface PipelineStageSnapshot {
     readonly id: string;
     readonly tenantId: string;
     readonly ownerType: 'Job';
     readonly ownerId: string; // jobId (set at snapshot time)
     readonly name: string;
     readonly position: number;
     readonly isFinal: boolean;
     readonly stageType: 'standard' | 'rejected' | 'hired';
   }

   export interface PipelineTemplateRepository {
     /**
      * Returns the default template for the tenant, or null if none exists.
      */
     findDefault(tenantId: string): Promise<{ id: string; name: string; stages: Omit<PipelineStageSnapshot, 'ownerId' | 'ownerType'>[] } | null>;

     /**
      * Returns a template by ID scoped to the tenant, or null if not found.
      */
     findById(tenantId: string, templateId: string): Promise<{ id: string; name: string; stages: Omit<PipelineStageSnapshot, 'ownerId' | 'ownerType'>[] } | null>;

     /**
      * Persists a set of snapshot stages for a newly created job.
      * MUST be called inside the same transaction as Job.save().
      */
     saveStageSnapshots(snapshots: readonly PipelineStageSnapshot[]): Promise<void>;
   }

   export const PIPELINE_TEMPLATE_REPOSITORY = Symbol('PipelineTemplateRepository');
   ```

2. Validate business invariants expressed in the interface documentation:
   - `findDefault` returns `null` (not throws) when no template exists — the handler throws `NoPipelineTemplateFoundError`.
   - `findById` returns `null` when the template is not in the tenant — the handler throws `PipelineTemplateNotInTenantError`.
   - `saveStageSnapshots` is transactional — the handler wraps it inside `UnitOfWork.runInTransaction`.

3. Export `PipelineTemplateRepository`, `PipelineStageSnapshot`, and `PIPELINE_TEMPLATE_REPOSITORY` from `modules/jobs/domain/ports/index.ts`.

---

## ✅ Acceptance Criteria

- **Given** the `PipelineTemplateRepository` interface  
  **When** `findDefault` is called with a `tenantId` that has no template  
  **Then** the interface contract specifies `null` is returned (not a thrown error)

- **Given** a `PipelineStageSnapshot` value  
  **When** `ownerType` is read  
  **Then** it is always `'Job'` (never `'PipelineTemplate'`)

- **Given** `saveStageSnapshots` is called with an empty array  
  **When** the call resolves  
  **Then** no error is thrown (valid edge case: template with no stages)

- **Given** the interface is used in `CreateJobHandler`  
  **When** the concrete `PrismaJobRepository` is swapped for an in-memory fake  
  **Then** the handler compiles and behaves identically

---

## 🔗 Dependencies

- TASK-003 (domain errors `NoPipelineTemplateFoundError`, `PipelineTemplateNotInTenantError` must exist)


---
# FILE: task-006-application-command-handler.md
---

PATH: /tasks/us-001-create-job-opening/task-006-application-command-handler.md

# 🔧 Task: Application — CreateJobCommand + CreateJobHandler

## 🆔 Metadata
- ID: TASK-006
- Related User Story: US-001
- Layer: Domain
- Estimation: 3 points

---

## 📝 Description

Implement the application-layer command and handler for creating a job opening, located in `modules/jobs/application/commands/create-job/`. The handler orchestrates:
1. Company active status check (via `TenantContext`).
2. Pipeline template resolution (default or explicit `sourceTemplateId`).
3. `Job` aggregate creation.
4. Pipeline stage snapshot generation and persistence.
5. Atomic save of `Job` + snapshots inside a `UnitOfWork` transaction.
6. Domain event dispatch via `EventBus`.

The handler MUST NOT contain any Prisma/SQL logic — all persistence is delegated through ports.

---

## ⚙️ Steps

1. Create `create-job.command.ts`:
   ```typescript
   export class CreateJobCommand {
     constructor(
       public readonly title: string,
       public readonly department: string,
       public readonly location: string,
       public readonly workMode: 'remote' | 'hybrid' | 'onsite',
       public readonly description: string,
       public readonly status: 'draft' | 'open',
       public readonly sourceTemplateId?: string, // optional; uses default template if absent
     ) {}
   }
   ```

2. Create `create-job.handler.ts`:
   - Inject: `JobRepository`, `PipelineTemplateRepository`, `UnitOfWork`, `Clock`, `IdGenerator`, `EventBus`.
   - `execute(context: TenantContext, command: CreateJobCommand): Promise<JobDto>`:
     1. If `context.companyStatus !== 'active'` → throw `InactiveCompanyError`.
     2. Resolve template:
        - If `command.sourceTemplateId` is provided → call `pipelineTemplateRepo.findById(tenantId, sourceTemplateId)` → throw `PipelineTemplateNotInTenantError` if null.
        - Else → call `pipelineTemplateRepo.findDefault(tenantId)` → throw `NoPipelineTemplateFoundError` if null.
     3. Create `Job` aggregate via `Job.create({ ...fields, sourceTemplateId: template.id }, clock, idGen)`.
     4. Build `PipelineStageSnapshot[]` by mapping `template.stages` → set `ownerType = 'Job'` and `ownerId = job.id.value`, generate new IDs via `idGen.generate()`.
     5. Wrap in `unitOfWork.runInTransaction(async () => { await jobRepo.save(job); await pipelineTemplateRepo.saveStageSnapshots(snapshots); })`.
     6. Publish events: `await eventBus.publishAll(job.pullDomainEvents())`.
     7. Return `JobMapper.toDto(job)`.

---

## ✅ Acceptance Criteria

- **Given** a valid `CreateJobCommand` with no `sourceTemplateId` and a default template exists  
  **When** `handler.execute(context, command)` is called  
  **Then** a `Job` is saved, `PipelineStageSnapshot` records equal the template's stage count, and one `JobCreatedEvent` is published

- **Given** a valid `CreateJobCommand` with an explicit `sourceTemplateId` belonging to the same tenant  
  **When** `handler.execute(context, command)` is called  
  **Then** the snapshot uses the specified template's stages

- **Given** no `PipelineTemplate` exists for the tenant  
  **When** `handler.execute(context, command)` is called  
  **Then** `NoPipelineTemplateFoundError` is thrown and no Job is saved

- **Given** `context.companyStatus = 'inactive'`  
  **When** `handler.execute(context, command)` is called  
  **Then** `InactiveCompanyError` is thrown before any persistence call

- **Given** `sourceTemplateId` belongs to a different tenant  
  **When** `handler.execute(context, command)` is called  
  **Then** `PipelineTemplateNotInTenantError` is thrown

- **Given** the `JobRepository.save` throws inside the transaction  
  **When** `handler.execute` propagates the error  
  **Then** `saveStageSnapshots` is not committed (transaction rolled back)

---

## 🔗 Dependencies

- TASK-003 (domain errors)
- TASK-004 (Job aggregate + JobRepository port)
- TASK-005 (PipelineTemplateRepository port)
- TASK-007 (JobDto + JobMapper needed by handler return)


---
# FILE: task-007-application-dto-mapper.md
---

PATH: /tasks/us-001-create-job-opening/task-007-application-dto-mapper.md

# 🔧 Task: Application — JobDto, JobMapper, PipelineStageDto

## 🆔 Metadata
- ID: TASK-007
- Related User Story: US-001
- Layer: Domain
- Estimation: 1 point

---

## 📝 Description

Create the read-model DTOs and mapper used by the application layer to convert `Job` aggregates into serializable plain objects. These DTOs cross the boundary between `application/` and `interface/` layers. They MUST contain only primitives (no domain VOs) and MUST NOT import from `infrastructure/`.

---

## ⚙️ Steps

1. Create `modules/jobs/application/dtos/job.dto.ts`:
   ```typescript
   export interface JobDto {
     id: string;
     tenantId: string;
     title: string;
     department: string;
     location: string;
     workMode: string;
     description: string;
     status: string;
     sourceTemplateId: string | null;
     createdBy: string;
     createdAt: string;   // ISO 8601
     updatedAt: string;   // ISO 8601
     publishedAt: string | null;
     closedAt: string | null;
   }
   ```

2. Create `modules/jobs/application/dtos/pipeline-stage.dto.ts`:
   ```typescript
   export interface PipelineStageDto {
     id: string;
     name: string;
     position: number;
     isFinal: boolean;
     stageType: 'standard' | 'rejected' | 'hired';
   }
   ```

3. Create `modules/jobs/application/dtos/job.mapper.ts`:
   ```typescript
   export class JobMapper {
     static toDto(job: Job): JobDto {
       const raw = job.toPrimitives();
       return {
         id: raw.id,
         tenantId: raw.tenantId,
         title: raw.title,
         department: raw.department,
         location: raw.location,
         workMode: raw.workMode,
         description: raw.description,
         status: raw.status,
         sourceTemplateId: raw.sourceTemplateId ?? null,
         createdBy: raw.createdBy,
         createdAt: raw.createdAt.toISOString(),
         updatedAt: raw.updatedAt.toISOString(),
         publishedAt: raw.publishedAt?.toISOString() ?? null,
         closedAt: raw.closedAt?.toISOString() ?? null,
       };
     }
   }
   ```

4. Export all from `modules/jobs/application/dtos/index.ts`.

---

## ✅ Acceptance Criteria

- **Given** a `Job` aggregate with `status = 'draft'` and `publishedAt = null`  
  **When** `JobMapper.toDto(job)` is called  
  **Then** `dto.publishedAt` is `null` and `dto.status` is `'draft'`

- **Given** a `Job` aggregate with `status = 'open'` and `publishedAt = new Date('2026-01-01')`  
  **When** `JobMapper.toDto(job)` is called  
  **Then** `dto.publishedAt` equals `'2026-01-01T00:00:00.000Z'`

- **Given** `JobDto` is imported in `interface/http/`  
  **When** TypeScript strict mode is active  
  **Then** no type errors are produced

---

## 🔗 Dependencies

- TASK-004 (Job aggregate with `toPrimitives()`)


---
# FILE: task-008-infrastructure-repositories.md
---

PATH: /tasks/us-001-create-job-opening/task-008-infrastructure-repositories.md

# 🔧 Task: Infrastructure — PrismaJobRepository + PrismaPipelineTemplateRepository

## 🆔 Metadata
- ID: TASK-008
- Related User Story: US-001
- Layer: Persistence
- Estimation: 3 points

---

## 📝 Description

Implement the concrete Prisma-backed repository classes in `modules/jobs/infrastructure/persistence/`. These classes implement the ports defined in the domain layer (`JobRepository`, `PipelineTemplateRepository`). They also contain the persistence mappers that translate between Prisma models and domain aggregates/value types. All queries MUST include `tenantId` in the `where` clause to enforce tenant isolation.

---

## ⚙️ Steps

### PrismaJobRepository (`prisma-job.repository.ts`)

1. Implement `JobRepository` interface.
2. `findById(tenantId, id)`:
   - Query: `prisma.job.findFirst({ where: { id: id.value, tenantId } })`.
   - Return `null` if not found; else call `JobPersistenceMapper.toDomain(raw)`.
3. `save(job)`:
   - Call `JobPersistenceMapper.toPersistence(job)` to get plain data.
   - Use `prisma.job.upsert({ where: { id }, create: data, update: { ...editableFields } })`.
   - `update` block MUST NOT overwrite `tenantId`, `createdBy`, or `createdAt`.

### JobPersistenceMapper (`job.mapper.ts`)

1. `toDomain(raw: PrismaJob): Job` — calls `Job.reconstitute(...)` with all VOs re-constructed.
2. `toPersistence(job: Job): Omit<PrismaJob, 'tenant' | 'pipelineStages'>` — calls `job.toPrimitives()` and maps to DB field names.

### PrismaPipelineTemplateRepository (`prisma-pipeline-template.repository.ts`)

1. Implement `PipelineTemplateRepository` interface.
2. `findDefault(tenantId)`:
   - Query: `prisma.pipelineTemplate.findFirst({ where: { tenantId, isDefault: true }, include: { stages: { orderBy: { position: 'asc' } } } })`.
   - Return `null` if not found.
   - Map to `{ id, name, stages: [...] }` with `Omit<PipelineStageSnapshot, 'ownerId' | 'ownerType'>[]`.
3. `findById(tenantId, templateId)`:
   - Same query but `where: { id: templateId, tenantId }`.
   - MUST include `tenantId` in filter to prevent cross-tenant access.
4. `saveStageSnapshots(snapshots)`:
   - Use `prisma.pipelineStage.createMany({ data: snapshots })`.
   - Each snapshot has `ownerType = 'Job'` and `ownerId = jobId`.

---

## ✅ Acceptance Criteria

- **Given** `PrismaJobRepository.findById('tenant_A', jobId)` is called  
  **When** the job exists under `tenant_B` but not `tenant_A`  
  **Then** `null` is returned (not the cross-tenant record)

- **Given** `PrismaPipelineTemplateRepository.findById('tenant_A', 'template_from_tenant_B')`  
  **When** the query runs  
  **Then** `null` is returned (tenant isolation enforced in `where` clause)

- **Given** `saveStageSnapshots([snapshot1, snapshot2])` is called  
  **When** the call resolves  
  **Then** exactly two `pipeline_stages` rows exist with `owner_type = 'Job'` and the correct `owner_id`

- **Given** `PrismaJobRepository.save(job)` is called twice with the same `job.id`  
  **When** the second call resolves  
  **Then** only one row exists in the database (upsert behavior)

---

## 🔗 Dependencies

- TASK-001 (Prisma schema + migration must exist)
- TASK-004 (Job aggregate and JobRepository port)
- TASK-005 (PipelineTemplateRepository port and PipelineStageSnapshot type)


---
# FILE: task-009-validation-zod-schema.md
---

PATH: /tasks/us-001-create-job-opening/task-009-validation-zod-schema.md

# 🔧 Task: Validation — Zod Contract Schema (packages/contracts)

## 🆔 Metadata
- ID: TASK-009
- Related User Story: US-001
- Layer: Validation
- Estimation: 1 point

---

## 📝 Description

Create the shared Zod schemas for the Create Job Opening request and response in `packages/contracts/src/jobs/`. These schemas are the single source of truth for input validation — imported by both the Express API layer and the Next.js frontend. They MUST enforce all business rules from the user story validation section and MUST NOT import from any `apps/` or `modules/` code.

---

## ⚙️ Steps

1. Create `packages/contracts/src/jobs/create-job.schema.ts`:
   ```typescript
   import { z } from 'zod';

   export const workModeSchema = z.enum(['remote', 'hybrid', 'onsite']);
   export type WorkMode = z.infer<typeof workModeSchema>;

   export const jobStatusSchema = z.enum(['draft', 'open', 'closed', 'archived']);
   export const creatableJobStatusSchema = z.enum(['draft', 'open']);
   export type CreatableJobStatus = z.infer<typeof creatableJobStatusSchema>;

   export const createJobRequestSchema = z.object({
     title:            z.string().trim().min(3, 'Title must be at least 3 characters').max(120),
     department:       z.string().trim().min(2).max(80),
     location:         z.string().trim().min(2).max(120),
     workMode:         workModeSchema,
     description:      z.string().trim().min(20, 'Description must be at least 20 characters').max(10_000),
     status:           creatableJobStatusSchema.default('draft'),
     sourceTemplateId: z.string().cuid().optional(),
   });
   export type CreateJobRequest = z.infer<typeof createJobRequestSchema>;

   export const jobResponseSchema = z.object({
     id:               z.string(),
     tenantId:         z.string(),
     title:            z.string(),
     department:       z.string(),
     location:         z.string(),
     workMode:         workModeSchema,
     description:      z.string(),
     status:           jobStatusSchema,
     sourceTemplateId: z.string().nullable(),
     createdBy:        z.string(),
     createdAt:        z.string().datetime(),
     updatedAt:        z.string().datetime(),
     publishedAt:      z.string().datetime().nullable(),
     closedAt:         z.string().datetime().nullable(),
   });
   export type JobResponse = z.infer<typeof jobResponseSchema>;
   ```

2. Export from `packages/contracts/src/jobs/index.ts`:
   ```typescript
   export * from './create-job.schema';
   ```

3. Export from `packages/contracts/src/index.ts`:
   ```typescript
   export * from './jobs';
   ```

4. Run `pnpm --filter @ats/contracts build` to verify no TypeScript errors.

---

## ✅ Acceptance Criteria

- **Given** `createJobRequestSchema.safeParse({ title: 'AB', ... })`  
  **When** parsed  
  **Then** `success = false` with a `title` field error (`min(3)` violated)

- **Given** `createJobRequestSchema.safeParse({ workMode: 'fulltime', ... })`  
  **When** parsed  
  **Then** `success = false` with a `workMode` field error (invalid enum value)

- **Given** `createJobRequestSchema.safeParse({ ..., status: 'closed', ... })`  
  **When** parsed  
  **Then** `success = false` (`creatableJobStatusSchema` only allows `draft` or `open`)

- **Given** `createJobRequestSchema.safeParse({ ..., status: undefined })`  
  **When** parsed  
  **Then** `success = true` and `data.status = 'draft'` (default applied)

- **Given** the schema is imported in both `apps/api` and `apps/web`  
  **When** `pnpm typecheck` runs  
  **Then** zero TypeScript errors in both apps

---

## 🔗 Dependencies

- None (standalone package; no dependency on other tasks)


---
# FILE: task-010-api-controller-routes-module.md
---

PATH: /tasks/us-001-create-job-opening/task-010-api-controller-routes-module.md

# 🔧 Task: API — JobsController, jobs.routes.ts, jobs.module.ts

## 🆔 Metadata
- ID: TASK-010
- Related User Story: US-001
- Layer: API
- Estimation: 3 points

---

## 📝 Description

Implement the HTTP interface layer for the `jobs` bounded context: the Express controller, router with RBAC guard, and the composition root module that wires all dependencies together. The route is: `POST /api/v1/t/:tenantSlug/jobs`. The controller MUST NOT contain business logic — it delegates entirely to `CreateJobHandler`. Validation is via Zod schema from `@ats/contracts`.

---

## ⚙️ Steps

### JobsController (`interface/http/jobs.controller.ts`)

1. Constructor receives `CreateJobHandler`.
2. `create` method:
   ```typescript
   public create = async (req: Request, res: Response, next: NextFunction): Promise<void> => {
     try {
       const parsed = createJobRequestSchema.safeParse(req.body);
       if (!parsed.success) throw ValidationError.fromZod(parsed.error);

       const cmd = new CreateJobCommand(
         parsed.data.title,
         parsed.data.department,
         parsed.data.location,
         parsed.data.workMode,
         parsed.data.description,
         parsed.data.status,
         parsed.data.sourceTemplateId,
       );

       const dto = await this.createJobHandler.execute(req.tenantContext, cmd);

       res
         .status(201)
         .location(`/api/v1/t/${req.tenantContext.tenantSlug}/jobs/${dto.id}`)
         .json({ data: dto, meta: { requestId: req.requestId } });
     } catch (err) {
       next(err);
     }
   };
   ```

### Router (`interface/http/jobs.routes.ts`)

1. Use `Router({ mergeParams: true })`.
2. Register: `router.post('/', authMiddleware(), tenantResolver(), rbac('jobs:create'), controller.create)`.
3. Mount in `app.ts` under `/api/v1/t/:tenantSlug/jobs`.

### Composition Root (`jobs.module.ts`)

1. Create `JobsModule` class with a static `register(prisma: PrismaClient, ...ports): JobsModule` factory.
2. Wire dependencies:
   - `PrismaJobRepository` ← `prisma`
   - `PrismaPipelineTemplateRepository` ← `prisma`
   - `PrismaUnitOfWork` ← `prisma`
   - `SystemClock` ← from shared infra
   - `CuidIdGenerator` ← from shared infra
   - `InProcessEventBus` ← from shared infra
   - `CreateJobHandler` ← all above
   - `JobsController` ← `CreateJobHandler`
3. Expose `router(): Router` that returns `buildJobsRouter(this.controller)`.

### Error Handler Integration

4. Verify that `NoPipelineTemplateFoundError` maps to `422 Unprocessable Entity` in the global error handler middleware (`shared/interface/middleware/error-handler.ts`).
5. Verify that `InactiveCompanyError` maps to `403 Forbidden`.

---

## ✅ Acceptance Criteria

- **Given** `POST /api/v1/t/acme/jobs` with a valid body and `Authorization: Bearer <recruiter-token>`  
  **When** the request is processed  
  **Then** response is `201 Created` with `Location` header and `data.id` in the body

- **Given** `POST /api/v1/t/acme/jobs` with role `Interviewer`  
  **When** the RBAC middleware evaluates `jobs:create`  
  **Then** response is `403 Forbidden`

- **Given** a request body with `title: 'AB'` (too short)  
  **When** Zod validation runs  
  **Then** response is `400 Bad Request` with field-level `errors` array

- **Given** `X-Tenant-ID: tenant_OTHER` but path slug is `acme` (tenant_acme)  
  **When** `tenantResolver()` middleware runs  
  **Then** response is `403 Forbidden` with code `TENANT_MISMATCH`

- **Given** the company is `Inactive`  
  **When** `CreateJobHandler` throws `InactiveCompanyError`  
  **Then** the error handler returns `403 Forbidden`

- **Given** no `PipelineTemplate` exists  
  **When** `CreateJobHandler` throws `NoPipelineTemplateFoundError`  
  **Then** the error handler returns `422 Unprocessable Entity`

---

## 🔗 Dependencies

- TASK-006 (CreateJobCommand + CreateJobHandler)
- TASK-007 (JobDto)
- TASK-008 (Prisma repositories)
- TASK-009 (Zod schema from contracts)


---
# FILE: task-011-frontend-create-job-page.md
---

PATH: /tasks/us-001-create-job-opening/task-011-frontend-create-job-page.md

# 🔧 Task: Frontend — Create Job Page + JobForm Component

## 🆔 Metadata
- ID: TASK-011
- Related User Story: US-001
- Layer: Frontend
- Estimation: 5 points

---

## 📝 Description

Build the UI for creating a job opening. This consists of:
1. A Next.js App Router page at `/t/[tenantSlug]/jobs/new/page.tsx` (Server Component — handles auth guard and data fetching for pipeline templates).
2. A `JobForm` client component using React Hook Form + Zod resolver + shadcn/ui components. The form renders all required fields and submits via the `useCreateJob` mutation hook.

All field validation must reuse the `createJobRequestSchema` from `@ats/contracts` (no duplicated Zod schemas in the frontend).

---

## ⚙️ Steps

### Page (`apps/web/src/app/t/[tenantSlug]/jobs/new/page.tsx`)

1. Mark as `async` Server Component.
2. Validate session with Auth.js — redirect to `/login` if unauthenticated.
3. Fetch available pipeline templates from the API (via internal fetch with tenant context).
4. Pass `pipelineTemplates` as props to `<JobForm />`.
5. Render a breadcrumb: `Jobs > New Job Opening`.

### JobForm (`apps/web/src/features/jobs/components/job-form.tsx`)

1. Mark `'use client'`.
2. Use `useForm<CreateJobRequest>` from React Hook Form with `zodResolver(createJobRequestSchema)`.
3. Render the following fields using shadcn/ui:
   - `title` → `<Input>` with label "Job Title" (required)
   - `department` → `<Input>` with label "Department"
   - `location` → `<Input>` with label "Location"
   - `workMode` → `<Select>` with options: Remote, Hybrid, Onsite
   - `description` → `<Textarea>` with label "Description" (required, min 20 chars)
   - `sourceTemplateId` → `<Select>` listing available templates; default pre-selected if `isDefault = true`
   - `status` → `<RadioGroup>` with options: Draft, Open (default: Draft)
4. Display inline field-level error messages from `formState.errors`.
5. Submit button: "Create Job Opening" — disabled while `isPending`.
6. On success: `router.push(`/t/${tenantSlug}/jobs/${newJob.id}`)`.
7. On error: display a toast notification with the error message.

---

## ✅ Acceptance Criteria

- **Given** the user navigates to `/t/acme/jobs/new`  
  **When** the page renders  
  **Then** the form is visible with all required fields and the breadcrumb shows `Jobs > New Job Opening`

- **Given** the user submits the form with `title = 'AB'`  
  **When** React Hook Form validates  
  **Then** an inline error `"Title must be at least 3 characters"` appears below the title field without making an API call

- **Given** the user selects `workMode = 'Remote'`  
  **When** the form renders  
  **Then** the select displays `'Remote'` as the selected value

- **Given** the form is submitted with all valid fields  
  **When** the API returns `201 Created`  
  **Then** the user is redirected to `/t/acme/jobs/<newJobId>`

- **Given** the API returns `422` (no pipeline template)  
  **When** the form submit handler catches the error  
  **Then** a toast notification displays `"A pipeline template must be configured by your Admin"`

- **Given** the user is unauthenticated  
  **When** `/t/acme/jobs/new` is accessed  
  **Then** they are redirected to `/login`

---

## 🔗 Dependencies

- TASK-009 (Zod schema from `@ats/contracts`)
- TASK-012 (useCreateJob hook must exist before form can be wired up)


---
# FILE: task-012-frontend-hook-server-action.md
---

PATH: /tasks/us-001-create-job-opening/task-012-frontend-hook-server-action.md

# 🔧 Task: Frontend — useCreateJob Mutation Hook + Server Action

## 🆔 Metadata
- ID: TASK-012
- Related User Story: US-001
- Layer: Frontend
- Estimation: 2 points

---

## 📝 Description

Implement the data-fetching layer for the Create Job Opening flow:

1. **`useCreateJob` hook** (`apps/web/src/features/jobs/hooks/use-create-job.ts`) — a TanStack Query `useMutation` hook that calls the backend `POST /api/v1/t/:tenantSlug/jobs` endpoint.
2. **Server Action** (`apps/web/src/features/jobs/actions/create-job.action.ts`) — optional progressive-enhancement path using Next.js Server Actions. Must validate input with `createJobRequestSchema` before forwarding to the API.

Both MUST use the `CreateJobRequest` and `JobResponse` types from `@ats/contracts` to ensure type safety across the network boundary.

---

## ⚙️ Steps

### useCreateJob hook

1. Create `apps/web/src/features/jobs/hooks/use-create-job.ts`:
   ```typescript
   'use client';
   import { useMutation, useQueryClient } from '@tanstack/react-query';
   import type { CreateJobRequest, JobResponse } from '@ats/contracts/jobs';

   export function useCreateJob(tenantSlug: string) {
     const queryClient = useQueryClient();

     return useMutation<JobResponse, Error, CreateJobRequest>({
       mutationFn: async (data) => {
         const res = await fetch(`/api/v1/t/${tenantSlug}/jobs`, {
           method: 'POST',
           headers: { 'Content-Type': 'application/json' },
           body: JSON.stringify(data),
         });
         if (!res.ok) {
           const err = await res.json();
           throw new Error(err?.error?.message ?? 'Failed to create job');
         }
         const body = await res.json();
         return body.data as JobResponse;
       },
       onSuccess: () => {
         queryClient.invalidateQueries({ queryKey: ['jobs', tenantSlug] });
       },
     });
   }
   ```

### Server Action (progressive enhancement)

2. Create `apps/web/src/features/jobs/actions/create-job.action.ts`:
   ```typescript
   'use server';
   import { redirect } from 'next/navigation';
   import { auth } from '@/lib/auth';
   import { createJobRequestSchema } from '@ats/contracts/jobs';

   export async function createJobAction(tenantSlug: string, formData: FormData) {
     const session = await auth();
     if (!session) redirect('/login');

     const raw = Object.fromEntries(formData.entries());
     const parsed = createJobRequestSchema.safeParse(raw);
     if (!parsed.success) {
       return { success: false, errors: parsed.error.flatten().fieldErrors };
     }

     const res = await fetch(
       `${process.env.API_INTERNAL_URL}/api/v1/t/${tenantSlug}/jobs`,
       {
         method: 'POST',
         headers: {
           'Content-Type': 'application/json',
           Authorization: `Bearer ${session.accessToken}`,
           'X-Tenant-ID': session.user.tenantId,
         },
         body: JSON.stringify(parsed.data),
       },
     );

     if (!res.ok) {
       const err = await res.json();
       return { success: false, apiError: err?.error?.message };
     }

     const body = await res.json();
     redirect(`/t/${tenantSlug}/jobs/${body.data.id}`);
   }
   ```

3. Add `API_INTERNAL_URL` to `packages/config-ts/env.schema.ts` if not already present.

---

## ✅ Acceptance Criteria

- **Given** `useCreateJob('acme')` is called with a valid `CreateJobRequest`  
  **When** the mutation resolves  
  **Then** `data` is a `JobResponse` with `id`, `status = 'draft'`, and the `jobs` query cache is invalidated

- **Given** the API returns `400` with field errors  
  **When** the mutation rejects  
  **Then** `error.message` contains the API error message (not an unhandled promise rejection)

- **Given** `createJobAction` is called without a valid session  
  **When** the action runs  
  **Then** `redirect('/login')` is called

- **Given** `createJobAction` is called with `title = 'AB'`  
  **When** Zod validates the FormData  
  **Then** `{ success: false, errors: { title: [...] } }` is returned without making an API call

---

## 🔗 Dependencies

- TASK-009 (Zod schema + types from `@ats/contracts`)
- TASK-010 (API endpoint must be deployed and reachable)


---
# FILE: task-013-unit-tests.md
---

PATH: /tasks/us-001-create-job-opening/task-013-unit-tests.md

# 🔧 Task: Testing — Unit Tests (Job Entity + CreateJobHandler)

## 🆔 Metadata
- ID: TASK-013
- Related User Story: US-001
- Layer: Testing
- Estimation: 3 points

---

## 📝 Description

Write Jest unit tests for the domain and application layers. Tests MUST use fake/in-memory implementations of all ports (no Prisma, no network, no Redis). All tests MUST be deterministic and run in < 100ms each. Tests live next to the files they test (co-located, not in a separate `__tests__` directory at this level).

---

## ⚙️ Steps

### 1. Fake implementations (test doubles)

Create in `apps/api/test/fakes/`:
- `in-memory-job.repository.ts` — implements `JobRepository` with a `Map<string, Job>`.
- `in-memory-pipeline-template.repository.ts` — implements `PipelineTemplateRepository` with preset templates and a `Map` for snapshots.
- `fake-clock.ts` — implements `Clock` with a fixed `Date` (constructor arg).
- `fake-id-generator.ts` — implements `IdGenerator` returning values from a pre-seeded array.
- `in-memory-event-bus.ts` — implements `EventBus` storing events in `published: DomainEvent[]`.
- `noop-unit-of-work.ts` — implements `UnitOfWork` that runs the callback directly (no transaction).

### 2. Job entity unit tests (`job.entity.spec.ts`)

File: `modules/jobs/domain/entities/job.entity.spec.ts`

Test cases:
- `create()` with `status = 'draft'` → `publishedAt = null`.
- `create()` with `status = 'open'` → `publishedAt = clock.now()`.
- `create()` raises exactly one `JobCreatedEvent` with correct payload.
- `reconstitute()` raises zero domain events.
- `JobTitle.create('AB')` throws `JobTitleTooShortError`.
- `JobStatus.draft().transitionTo(JobStatus.open())` returns `JobStatus('open')`.
- `JobStatus.draft().transitionTo(JobStatus.create('closed'))` throws `InvalidJobStatusTransitionError`.

### 3. CreateJobHandler unit tests (`create-job.handler.spec.ts`)

File: `modules/jobs/application/commands/create-job/create-job.handler.spec.ts`

Test cases:
- **Happy path (draft):** valid command → job saved, snapshots equal template stage count, one `JobCreatedEvent` published.
- **Happy path (open):** `status = 'open'` → `dto.publishedAt` is not null.
- **Explicit template:** `sourceTemplateId` provided → uses that template's stages.
- **No template:** `PipelineTemplateRepository.findDefault` returns null → `NoPipelineTemplateFoundError` thrown, no job saved.
- **Cross-tenant template:** `findById` returns null → `PipelineTemplateNotInTenantError` thrown.
- **Inactive company:** `context.companyStatus = 'inactive'` → `InactiveCompanyError` thrown before any repo call.
- **Title too short:** `command.title = 'AB'` → `JobTitleTooShortError` thrown, no job saved, zero events published.

---

## ✅ Acceptance Criteria

- **Given** all unit tests for `job.entity.spec.ts` and `create-job.handler.spec.ts`  
  **When** `pnpm test --filter apps/api` is run  
  **Then** all tests pass with zero failures

- **Given** `FakeClock` is constructed with `new Date('2026-01-15T10:00:00.000Z')`  
  **When** `handler.execute(context, openCmd)` returns  
  **Then** `dto.publishedAt === '2026-01-15T10:00:00.000Z'`

- **Given** `InMemoryJobRepository` starts empty  
  **When** `NoPipelineTemplateFoundError` is thrown  
  **Then** `inMemoryJobRepo.size === 0` (no partial write)

- **Given** domain layer coverage is measured  
  **When** `jest --coverage` runs  
  **Then** `modules/jobs/domain/` has ≥ 95% line and branch coverage

---

## 🔗 Dependencies

- TASK-003 (domain errors)
- TASK-004 (Job aggregate)
- TASK-006 (CreateJobHandler)


---
# FILE: task-014-integration-test.md
---

PATH: /tasks/us-001-create-job-opening/task-014-integration-test.md

# 🔧 Task: Testing — Integration Test (POST /api/v1/t/:tenantSlug/jobs)

## 🆔 Metadata
- ID: TASK-014
- Related User Story: US-001
- Layer: Testing
- Estimation: 3 points

---

## 📝 Description

Write Jest integration tests for the `POST /api/v1/t/:tenantSlug/jobs` endpoint using `supertest` and a real PostgreSQL test database (via Testcontainers). These tests verify the full request-to-response stack including middleware, RBAC, validation, handler, Prisma repository, and DB state. Tests MUST assert both HTTP response shape and actual database records. Multi-tenant isolation MUST be explicitly validated.

File: `modules/jobs/__tests__/integration/create-job.integration.spec.ts`

---

## ⚙️ Steps

1. Set up `TestHarness` in `apps/api/test/setup/build-test-app.ts` if not yet implemented:
   - Spins up an Express app with all modules registered.
   - Wraps a Testcontainers PostgreSQL instance.
   - Provides `harness.seedTenant()`, `harness.seedUser()`, `harness.seedPipelineTemplate()`, `harness.signUserToken()`, `harness.reset()`.

2. `beforeAll`: Start the harness.
3. `afterAll`: Shutdown the harness.
4. `beforeEach`: `harness.reset()` then seed:
   - Tenant `acme` (id: `tenant_acme`, status: `active`).
   - User `user_1` (role: `recruiter`, tenantId: `tenant_acme`).
   - Default PipelineTemplate with 3 stages for `tenant_acme`.

5. Test cases:

   **Happy path — draft:**
   ```
   POST /api/v1/t/acme/jobs
   Body: { title, department, location, workMode: 'remote', description, status: 'draft' }
   Headers: Authorization: Bearer <recruiter-token>, X-Tenant-ID: tenant_acme
   → 201, Location header matches /\/api\/v1\/t\/acme\/jobs\/.+/
   → data.status = 'draft', data.publishedAt = null
   → DB: jobs table has 1 row, pipeline_stages has 3 rows (ownerType='Job')
   ```

   **Happy path — open:**
   ```
   POST with status: 'open'
   → data.publishedAt is not null
   ```

   **Tenant mismatch:**
   ```
   X-Tenant-ID: tenant_OTHER
   → 403, error.code = 'TENANT_MISMATCH'
   ```

   **Unauthorized role (Interviewer):**
   ```
   User with role 'interviewer'
   → 403, error.code = 'FORBIDDEN'
   ```

   **Validation — title too short:**
   ```
   title: 'AB'
   → 400, errors array contains title field error
   ```

   **No pipeline template:**
   ```
   Seed tenant without a PipelineTemplate
   → 422, error.code = 'NO_PIPELINE_TEMPLATE_FOUND'
   ```

   **Inactive company:**
   ```
   Seed tenant with status: 'inactive'
   → 403, error.code = 'INACTIVE_COMPANY'
   ```

   **Multi-tenant isolation:**
   ```
   Create a job under tenant_acme
   Assert that tenant_other sees zero jobs in their pipeline query
   ```

---

## ✅ Acceptance Criteria

- **Given** a valid recruiter request with matching tenant  
  **When** `POST /api/v1/t/acme/jobs` is called  
  **Then** `response.status === 201` and `pipeline_stages` table has exactly 3 rows with `owner_type = 'Job'`

- **Given** `X-Tenant-ID` header does not match path slug  
  **When** the request is processed  
  **Then** `response.status === 403` with `TENANT_MISMATCH` code

- **Given** a user with role `interviewer`  
  **When** the request is processed  
  **Then** `response.status === 403` with `FORBIDDEN` code

- **Given** a request body with `title: 'AB'`  
  **When** validation runs  
  **Then** `response.status === 400` and no rows are inserted in the `jobs` table

- **Given** two requests from two different tenants  
  **When** each creates a job  
  **Then** `SELECT * FROM jobs WHERE tenant_id = 'tenant_A'` returns only tenant_A's jobs

---

## 🔗 Dependencies

- TASK-001 (DB schema must exist)
- TASK-008 (Prisma repositories)
- TASK-010 (API controller + routes + module)


---
# FILE: task-015-e2e-test.md
---

PATH: /tasks/us-001-create-job-opening/task-015-e2e-test.md

# 🔧 Task: Testing — E2E Test (Playwright — Create Job Opening)

## 🆔 Metadata
- ID: TASK-015
- Related User Story: US-001
- Layer: Testing
- Estimation: 3 points

---

## 📝 Description

Write Playwright E2E tests for the "Create Job Opening" user journey. Tests run against a full Docker stack (Next.js frontend + Express API + PostgreSQL). They simulate a Recruiter logging in, navigating to the new job form, filling it out, submitting, and verifying the redirect to the newly created job's detail page.

File: `apps/web/e2e/specs/jobs/create-job.spec.ts`

---

## ⚙️ Steps

1. Ensure `apps/web/e2e/playwright.config.ts` points to the Docker test stack base URL.
2. Create `apps/web/e2e/fixtures/auth.fixture.ts`:
   - `loginAsRecruiter(page, tenantSlug)` — fills login form and stores auth cookie.
3. Create `apps/web/e2e/fixtures/seed.fixture.ts`:
   - Before each test: seed `tenant`, `recruiter user`, and `default PipelineTemplate` via API or direct DB script.
   - After each test: cleanup via `DELETE FROM jobs WHERE tenant_id = 'e2e_tenant'`.

4. Test cases:

   **Happy path — draft job:**
   ```
   1. Login as recruiter for tenant 'e2e_tenant'
   2. Navigate to /t/e2e_tenant/jobs/new
   3. Fill in: title, department, location, workMode=Remote, description (≥ 20 chars)
   4. Leave status = Draft (default)
   5. Click "Create Job Opening"
   6. Assert: redirected to /t/e2e_tenant/jobs/<id>
   7. Assert: job detail page shows title and "Draft" badge
   ```

   **Client-side validation — title too short:**
   ```
   1. Navigate to /t/e2e_tenant/jobs/new
   2. Type 'AB' in title field
   3. Click "Create Job Opening"
   4. Assert: error message "Title must be at least 3 characters" is visible
   5. Assert: URL remains /t/e2e_tenant/jobs/new (no navigation)
   ```

   **Unauthenticated access:**
   ```
   1. Navigate to /t/e2e_tenant/jobs/new without logging in
   2. Assert: redirected to /login
   ```

   **Role guard — Interviewer cannot access form:**
   ```
   1. Login as user with role Interviewer
   2. Navigate to /t/e2e_tenant/jobs/new
   3. Assert: 403 page or redirect shown (no form rendered)
   ```

5. Use `expect(page).toHaveURL(...)` and `expect(page.getByRole('status')).toContainText('Draft')` for assertions.
6. Add screenshots on failure via `test.use({ screenshot: 'only-on-failure' })`.

---

## ✅ Acceptance Criteria

- **Given** a logged-in Recruiter on `/t/e2e_tenant/jobs/new`  
  **When** the form is completed with valid data and submitted  
  **Then** the browser navigates to `/t/e2e_tenant/jobs/<newId>` and the title is visible on the detail page

- **Given** `title = 'AB'` is entered  
  **When** the form is submitted  
  **Then** the inline error `"Title must be at least 3 characters"` is visible and no navigation occurs

- **Given** an unauthenticated user visits `/t/e2e_tenant/jobs/new`  
  **When** the page loads  
  **Then** they are redirected to `/login`

- **Given** all E2E tests run in CI against the Docker stack  
  **When** `pnpm e2e` is executed  
  **Then** all tests pass with zero flaky failures across 3 consecutive runs

---

## 🔗 Dependencies

- TASK-010 (API endpoint)
- TASK-011 (Frontend page + form)
- TASK-012 (useCreateJob hook)
