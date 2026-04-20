PATH: /user-stories/US-027-configure-automation-rule.md

# 🧾 User Story: Configure Automation Rule

## 🆔 Metadata
- ID: US-027
- Feature: Automation
- Priority: High
- Status: Draft
- Owner: PM

## 🎯 User Story
**As an** Admin  
**I want** to create and configure automation rules with triggers, conditions, and actions  
**So that** routine pipeline operations are executed automatically without requiring manual Recruiter intervention

## 🧠 Business Context
- Related Use Case(s): UC-30 Configurar regla de automatización
- Entities involved: AutomationRule, AutomationCondition, AutomationAction, Company, User
- Business Rules:
  - Only Admin role may create, edit, or deactivate automation rules.
  - The tenant's automation module must be enabled.
  - Every `AutomationRule` requires at minimum: one `triggerType`, zero or more `AutomationCondition` records, and at least one `AutomationAction`.
  - An `AutomationRule` saved with `status = Inactive` does not execute; it must be explicitly activated.
  - Rules are scoped by `scopeType`: `Global` (all job openings), `JobOpening` (specific vacancy), or `PipelineStage` (specific stage). When `scopeType != Global`, `scopeId` is mandatory.
  - Rules must not affect data from other tenants.
  - Configuration changes to an active rule apply only to future executions; in-flight executions complete with their original configuration.
  - `AutomationCondition.sequence` must be unique per rule and determines evaluation order.
  - `AutomationAction.sequence` must be unique per rule and determines execution order.
- Assumptions:
  - Condition `logicalJoin` connects conditions sequentially (AND/OR chain). The last condition's `logicalJoin` value is ignored.
  - `configJson` in `AutomationAction` follows the contract defined by `actionType` (validated server-side).
- Dependencies:
  - The automation module must be active for the tenant.
  - For `scopeType = JobOpening` or `PipelineStage`, the referenced entity must exist and belong to the same Company.

## ✅ Acceptance Criteria (BDD + Edge Cases)

### 🎯 Happy Path
- **Given** an authenticated Admin with the automation module enabled  
  **When** the Admin creates a rule with `triggerType = StageChanged`, one condition (Application status equals Active), and one action (`MoveToStage` targeting the Interview stage), then activates the rule  
  **Then** an `AutomationRule` is persisted with `status = Active`, the corresponding `AutomationCondition` and `AutomationAction` records are created, and the rule is available for evaluation on the next `StageChanged` event within its scope

### ⚠️ Edge Cases

#### Case 1: Rule saved without any action
- **Given** an Admin fills in the trigger and conditions but does not add any action  
  **When** the Admin saves the rule  
  **Then** the system returns a 400 validation error indicating that at least one action is required  
- Handling:
  - Block save if no `AutomationAction` is present. Display a clear message in the configuration UI.

#### Case 2: Admin saves rule as inactive
- **Given** an Admin configures a valid rule but chooses to save it without activating  
  **When** the save action is submitted  
  **Then** the `AutomationRule` is persisted with `status = Inactive` and does not trigger any executions until explicitly activated  
- Handling:
  - Inactive rules are valid and fully stored. They appear in the rule list with a clear "Inactive" badge.

#### Case 3: Inconsistent action configuration
- **Given** an Admin sets `actionType = MoveToStage` but provides a `configJson` without a `toStageId`  
  **When** the rule is saved  
  **Then** the system returns a 400 validation error specifying that `configJson` is incompatible with `actionType = MoveToStage`  
- Handling:
  - Validate `configJson` against the contract of each `actionType` server-side at save time.

#### Case 4: `scopeId` not provided for non-global scope
- **Given** an Admin sets `scopeType = JobOpening` but does not provide `scopeId`  
  **When** the rule is saved  
  **Then** the system returns a 400 validation error indicating that `scopeId` is required when `scopeType` is not `Global`  
- Handling:
  - Enforce: if `scopeType != Global` then `scopeId` must be a valid UUID referencing an entity in the same Company.

### 🚫 Validation Rules
- At least one `AutomationAction` is required per rule.
- `triggerType` must be one of the defined enum values.
- `scopeId` is required when `scopeType != Global`; the referenced entity must belong to the same `companyId`.
- `AutomationCondition.sequence` must be unique per rule.
- `AutomationAction.sequence` must be unique per rule.
- `configJson` must conform to the contract of its `actionType`.
- `valueJson` in `AutomationCondition` must be compatible with the specified `operator`.
- Role must be Admin.

## 📊 Business Impact
- KPI Affected: Recruiter operational overhead, Process standardization, Pipeline throughput
- Expected Impact: Enables HR teams to automate 40–60% of repetitive status updates and notifications, freeing Recruiters to focus on high-value tasks like candidate evaluation
- Success Criteria:
  - Automation rules created and activated within a single Admin session
  - 100% of active rules evaluated on every matching trigger event
  - Zero automation rules execute across tenant boundaries
