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

*Generated: 2026-04-20 | LTI ATS Product Team*
