PATH: /user-stories/US-023-monitor-pipeline-sla.md

# 🧾 User Story: Monitor Pipeline SLA

## 🆔 Metadata
- ID: US-023
- Feature: Pipeline & Tracking
- Priority: Medium
- Status: Draft
- Owner: PM

## 🎯 User Story
**As a** Recruiter or Admin  
**I want** to view which candidates have exceeded or are at risk of exceeding the time targets configured for each pipeline stage  
**So that** I can proactively intervene on stalled applications before they become bottlenecks

## 🧠 Business Context
- Related Use Case(s): UC-26 Monitorear SLA de pipeline
- Entities involved: Application, PipelineStage, SlaPolicy, SlaBreachEvent, JobOpening, Company, User
- Business Rules:
  - SLA thresholds are defined at two levels: `PipelineStage.slaHours` (simple, per stage) and `SlaPolicy` (advanced, per job opening or stage).
  - `Application.stageEnteredAt` records when the Application entered its current stage; SLA elapsed time is calculated from this field.
  - SLA status is classified as: "On Track" (within `warningHours`), "At Risk" (between `warningHours` and `thresholdHours`), or "Breached" (`thresholdHours` exceeded).
  - A `SlaBreachEvent` is created when a breach is first detected for a given Application + Stage combination; duplicates for the same combination are prevented.
  - SLA calculations must respect the tenant's timezone.
  - Breaching SLA does not automatically change the Application status unless an `AutomationRule` is configured to do so (US-022).
  - Stages without a configured SLA threshold are shown without a compliance classification.
- Assumptions:
  - The SLA monitoring view is a dashboard or board overlay on the existing pipeline view.
  - `warningHours` must be less than `thresholdHours` when defined.
- Dependencies:
  - SlaPolicy must be configured by an Admin.
  - Applications must have `stageEnteredAt` populated (set atomically with stage moves in US-010).

## ✅ Acceptance Criteria (BDD + Edge Cases)

### 🎯 Happy Path
- **Given** an authenticated Recruiter, a JobOpening with active Applications, and SLA policies configured for each stage  
  **When** the Recruiter accesses the SLA monitoring view for the job opening  
  **Then** the system calculates elapsed time per Application using `Application.stageEnteredAt`, compares against the applicable `SlaPolicy.thresholdHours` and `warningHours`, and renders each Application with a color-coded SLA status: "On Track", "At Risk", or "Breached"

### ⚠️ Edge Cases

#### Case 1: Stage has no SLA configured
- **Given** an Application in a PipelineStage with no `slaHours` and no applicable `SlaPolicy`  
  **When** the SLA view is loaded  
  **Then** the Application is displayed without a SLA classification and without a compliance indicator  
- Handling:
  - Render "No SLA configured" for stages without a threshold. Do not calculate or display a misleading status.

#### Case 2: SLA breach detection on existing application
- **Given** an Application has been in a stage for longer than `SlaPolicy.thresholdHours`  
  **When** the SLA monitoring check runs (on view load or scheduled evaluation)  
  **Then** a `SlaBreachEvent` is created with `status = Open` if one does not already exist for this `applicationId + pipelineStageId` combination  
- Handling:
  - Idempotency: check for an existing open `SlaBreachEvent` before inserting a new one.

#### Case 3: Application advances stage after breach
- **Given** a `SlaBreachEvent` with `status = Open` exists for an Application  
  **When** the Recruiter manually moves the Application to the next stage  
  **Then** the `SlaBreachEvent` is updated to `status = Resolved` and `resolvedAt` is set to the current timestamp  
- Handling:
  - Stage transition (US-010) should trigger SlaBreachEvent resolution for the previous stage.

#### Case 4: Policy deactivated while breach is open
- **Given** a `SlaPolicy` is deactivated by an Admin  
  **When** the SLA view is loaded  
  **Then** existing open `SlaBreachEvent` records remain visible but new breaches are no longer generated for that policy  
- Handling:
  - Only active policies participate in new breach detection.

### 🚫 Validation Rules
- `Application.stageEnteredAt` must be non-null for SLA calculations to apply.
- `SlaPolicy.warningHours` must be less than `SlaPolicy.thresholdHours` when both are defined.
- `SlaPolicy` must have at least `jobOpeningId` or `pipelineStageId`.
- A single `SlaBreachEvent` per `(applicationId, pipelineStageId)` in `Open` status; no duplicates.
- SLA elapsed time is calculated server-side using UTC timestamps; display is adjusted per tenant timezone.
- Role must be Recruiter or Admin.

## 📊 Business Impact
- KPI Affected: Time-to-hire, Stage duration averages, Pipeline throughput, Candidate experience
- Expected Impact: Reduces average stage dwell time by surfacing stalled applications proactively; expected 20–30% reduction in breach frequency after adoption
- Success Criteria:
  - SLA status visible per Application on the pipeline board without additional navigation
  - 100% of SLA breaches generate a corresponding `SlaBreachEvent` record
  - Resolved breaches (`resolvedAt` set) when Application advances past the breached stage
