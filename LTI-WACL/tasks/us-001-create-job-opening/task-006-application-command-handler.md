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
