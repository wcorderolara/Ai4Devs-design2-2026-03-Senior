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
