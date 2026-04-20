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
