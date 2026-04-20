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
