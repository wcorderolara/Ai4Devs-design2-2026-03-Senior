PATH: /user-stories/US-019-bulk-actions-candidates.md

# 🧾 User Story: Execute Bulk Actions on Candidates

## 🆔 Metadata
- ID: US-019
- Feature: Candidate Management
- Priority: High
- Status: Draft
- Owner: PM

## 🎯 User Story
**As a** Recruiter  
**I want** to select multiple candidates or applications and apply a bulk action to all of them at once  
**So that** I can process high volumes of applications efficiently without repeating the same operation one by one

## 🧠 Business Context
- Related Use Case(s): UC-22 Ejecutar acciones masivas sobre candidatos
- Entities involved: Application, Candidate, BulkOperation, BulkOperationItem, ActivityEvent, Company, User
- Business Rules:
  - Only Recruiters may execute bulk actions.
  - Permitted bulk action types: `MoveStage`, `Reject`, `Tag`, `Assign`, `Archive`.
  - All selected records must belong to the same tenant (Company).
  - The system must validate each record individually before applying the action.
  - Records that fail validation are excluded and reported; valid records proceed.
  - Each bulk operation creates one `BulkOperation` record (Aggregate Root) and one `BulkOperationItem` per selected record.
  - `Application.bulkUpdatedAt` and `Application.bulkUpdateBatchId` are updated for each affected Application.
  - A `BulkOperation` in `Completed` status cannot be reopened.
  - An `ActivityEvent` of type `BulkUpdated` is recorded for each affected Application.
- Assumptions:
  - The Recruiter has applied filters to narrow the list before selecting records.
  - The confirmation step is mandatory; no immediate execution without user confirmation.
- Dependencies:
  - Applications must exist (US-004 or US-005).
  - For `MoveStage`, the target stage must be valid for each Application's JobOpening (US-010).

## ✅ Acceptance Criteria (BDD + Edge Cases)

### 🎯 Happy Path
- **Given** an authenticated Recruiter with a filtered list of Applications  
  **When** the Recruiter selects 10 Applications, chooses `Reject` as bulk action, and confirms  
  **Then** a `BulkOperation` record is created with `status = Running`, one `BulkOperationItem` per selected Application is created, each valid Application is updated to `status = Rejected`, `BulkOperation.status` transitions to `Completed`, and `successCount` reflects the number of successfully processed records

### ⚠️ Edge Cases

#### Case 1: Some records fail validation
- **Given** a Recruiter selects 10 Applications, but 3 are already in a terminal state (`Hired` or `Rejected`)  
  **When** the bulk reject action is submitted  
  **Then** the system processes the 7 valid records, marks the 3 invalid `BulkOperationItem` records as `Skipped`, `BulkOperation.status` is `PartiallyCompleted`, and the response details which records were skipped and why  
- Handling:
  - Never fail the entire batch because of partial invalidity. Report per-item results.

#### Case 2: Recruiter cancels at confirmation step
- **Given** a Recruiter has selected records and chosen a bulk action  
  **When** the Recruiter cancels at the confirmation dialog  
  **Then** no `BulkOperation` is created, no records are modified, and the filtered list remains unchanged  
- Handling:
  - The BulkOperation record is only created after confirmation, not at selection time.

#### Case 3: Cross-company record selection attempt
- **Given** a Recruiter submits a bulk action payload containing Application IDs from another Company  
  **When** the bulk operation is processed  
  **Then** the system filters out any IDs not belonging to the authenticated user's Company; those items are marked as `Skipped` with reason "Not found in tenant"  
- Handling:
  - Enforce `companyId` scoping at the item validation layer.

#### Case 4: MoveStage bulk action with invalid target stage
- **Given** a Recruiter performs a `MoveStage` bulk action but provides a `toStageId` that does not belong to one of the Applications' JobOpenings  
  **When** the operation is processed  
  **Then** the affected `BulkOperationItem` records are marked as `Failed`, and valid items still proceed  
- Handling:
  - Validate `toStageId.ownerId = application.jobOpeningId` per item before applying the move.

### 🚫 Validation Rules
- `operationType` must be one of: `MoveStage`, `Reject`, `Tag`, `Assign`, `Archive`.
- All `targetId` values must belong to the same `companyId` as the authenticated user.
- `payloadJson` must be consistent with `operationType` (e.g., `MoveStage` requires `toStageId`).
- A `BulkOperation` in `Completed` or `Failed` status cannot be retried; a new operation must be created.
- Role must be Recruiter.
- Minimum 2 records required for a bulk action to be valid (single-record operations use standard endpoints).

## 📊 Business Impact
- KPI Affected: Recruiter throughput, Time spent on administrative pipeline tasks, Pipeline processing speed
- Expected Impact: Enables recruiters to process 10x more applications per session without manual repetition, directly reducing time-to-hire for high-volume pipelines
- Success Criteria:
  - Bulk operations on up to 100 records complete within 5 seconds
  - Every record in a `BulkOperation` has a corresponding `BulkOperationItem` with a final status
  - `successCount + failureCount` always equals the total number of items submitted
