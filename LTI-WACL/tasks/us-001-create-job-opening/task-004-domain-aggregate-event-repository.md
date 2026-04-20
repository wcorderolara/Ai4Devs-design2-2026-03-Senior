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
