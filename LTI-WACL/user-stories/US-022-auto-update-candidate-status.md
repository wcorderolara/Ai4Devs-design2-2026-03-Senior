PATH: /user-stories/US-022-auto-update-candidate-status.md

# 🧾 User Story: Automatically Update Candidate Status via Automation Rules

## 🆔 Metadata
- ID: US-022
- Feature: Pipeline & Tracking
- Priority: High
- Status: Draft
- Owner: PM

## 🎯 User Story
**As a** System (triggered automatically)  
**I want** to evaluate active automation rules when a pipeline event occurs and apply configured actions on eligible applications  
**So that** routine status updates and stage transitions happen without manual Recruiter intervention, keeping the pipeline consistent and reducing operational load

## 🧠 Business Context
- Related Use Case(s): UC-25 Actualizar estado de candidatos automáticamente
- Entities involved: Application, PipelineStage, AutomationRule, AutomationCondition, AutomationAction, AutomationExecution, AutomationExecutionItem, ActivityEvent, User, Company
- Business Rules:
  - Only `AutomationRule` records with `status = Active` are evaluated.
  - An automation rule is evaluated when its `triggerType` event occurs (e.g., `ApplicationCreated`, `StageChanged`, `EvaluationSubmitted`, `SlaBreached`).
  - No automation rule may skip mandatory pipeline stages defined by the JobOpening's snapshot.
  - Automation must not override decisions that have been manually locked or that are in a terminal state (`Hired`, `Rejected`).
  - Each execution creates one `AutomationExecution` with one `AutomationExecutionItem` per action.
  - `Application.updatedByType` is set to `Automation` when the last update originated from an automation.
  - `Application.lastStageChangeAt` is updated atomically with any stage change triggered by automation.
  - All executions are auditable: `AutomationExecution` records the trigger event type, origin, and result.
- Assumptions:
  - The Automation Engine is a separate service that receives domain events from the API and invokes actions back through the API.
  - Automation executions are asynchronous; the Recruiter sees the result reflected in the UI shortly after the trigger event.
- Dependencies:
  - `AutomationRule` must be configured and active (US-027 / UC-30).
  - The triggering event must have already been persisted in the system (e.g., a StageTransition for `StageChanged`).

## ✅ Acceptance Criteria (BDD + Edge Cases)

### 🎯 Happy Path
- **Given** an active `AutomationRule` with `triggerType = EvaluationSubmitted` and action `MoveToStage` for a specific JobOpening  
  **When** an Interviewer submits a completed Evaluation for an Application  
  **Then** the Automation Engine evaluates the rule's conditions against the Application, all conditions pass, an `AutomationExecution` is created with `status = Running`, the Application is moved to the configured next stage (via a `StageTransition` INSERT), `Application.updatedByType = Automation`, `Application.lastStageChangeAt` is updated, the `AutomationExecution` transitions to `Succeeded`, and an `ActivityEvent` of type `AutomationExecuted` is recorded

### ⚠️ Edge Cases

#### Case 1: Application does not meet rule conditions
- **Given** an active AutomationRule with conditions that an Application does not satisfy  
  **When** the trigger event fires  
  **Then** the `AutomationExecution` is created with `status = Skipped`, no Application fields are modified, and the execution is logged with "Conditions not met"  
- Handling:
  - `Skipped` is a valid and expected terminal state. It must be logged for auditability.

#### Case 2: Automation targets a terminal-state Application
- **Given** an Application with `status = Hired`  
  **When** an automation rule fires and its action would move the Application to another stage  
  **Then** the `AutomationExecutionItem` is marked as `Blocked`, the Application is not modified, and the Recruiter can review the blocked case manually  
- Handling:
  - Block automation actions on terminal Applications. `AutomationExecution.status = Blocked`.

#### Case 3: Action execution fails (e.g., invalid stage reference)
- **Given** an automation action specifies a `toStageId` that no longer exists in the JobOpening's snapshot  
  **When** the action is executed  
  **Then** the `AutomationExecutionItem.status = Failed`, `AutomationExecution.status = Failed`, and the system logs the failure reason without modifying the Application  
- Handling:
  - Fail the specific item and bubble up to the execution. Do not partially apply multi-action rules.

#### Case 4: Rule becomes inactive during execution
- **Given** an Admin deactivates an AutomationRule while an execution is in `Running` state  
  **When** the execution completes  
  **Then** the execution finishes with its current state; the deactivation only prevents future evaluations  
- Handling:
  - In-flight executions complete. Deactivation is not retroactive.

### 🚫 Validation Rules
- Only `AutomationRule.status = Active` rules are evaluated.
- Automation must not modify Applications in terminal states (`Hired`, `Rejected`, `Withdrawn`).
- Automation must not skip mandatory pipeline stages.
- `AutomationExecution` must record at least `jobOpeningId` or `applicationId`.
- `finishedAt` is mandatory for all terminal `AutomationExecution` statuses.
- `Application.updatedByType` must be set to `Automation` when automation triggers a change.

## 📊 Business Impact
- KPI Affected: Recruiter operational overhead, Pipeline throughput, Process standardization
- Expected Impact: Eliminates 40–60% of manual status updates for routine pipeline transitions, reducing administrative burden and improving consistency across the team
- Success Criteria:
  - 100% of automation-triggered changes have a corresponding `AutomationExecution` record
  - Zero terminal-state Applications modified by automation
  - Execution audit trail available per Application and per rule
