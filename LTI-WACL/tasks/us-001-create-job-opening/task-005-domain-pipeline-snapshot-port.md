PATH: /tasks/us-001-create-job-opening/task-005-domain-pipeline-snapshot-port.md

# 🔧 Task: Domain — PipelineTemplateRepository Port + PipelineStage Entity

## 🆔 Metadata
- ID: TASK-005
- Related User Story: US-001
- Layer: Domain
- Estimation: 2 points

---

## 📝 Description

Define the domain contracts for pipeline template resolution and stage snapshot creation. This task creates the `PipelineTemplateRepository` port interface and a lightweight `PipelineStageSnapshot` value type used to represent a stage copied from a template into a job's pipeline. These artifacts are consumed by the application-layer handler (`CreateJobHandler`) to enforce the business rule: *"A PipelineTemplate's stages are snapshot-copied to the new JobOpening at creation time."*

---

## ⚙️ Steps

1. Create `modules/jobs/domain/ports/pipeline-template.repository.ts`:
   ```typescript
   export interface PipelineStageSnapshot {
     readonly id: string;
     readonly tenantId: string;
     readonly ownerType: 'Job';
     readonly ownerId: string; // jobId (set at snapshot time)
     readonly name: string;
     readonly position: number;
     readonly isFinal: boolean;
     readonly stageType: 'standard' | 'rejected' | 'hired';
   }

   export interface PipelineTemplateRepository {
     /**
      * Returns the default template for the tenant, or null if none exists.
      */
     findDefault(tenantId: string): Promise<{ id: string; name: string; stages: Omit<PipelineStageSnapshot, 'ownerId' | 'ownerType'>[] } | null>;

     /**
      * Returns a template by ID scoped to the tenant, or null if not found.
      */
     findById(tenantId: string, templateId: string): Promise<{ id: string; name: string; stages: Omit<PipelineStageSnapshot, 'ownerId' | 'ownerType'>[] } | null>;

     /**
      * Persists a set of snapshot stages for a newly created job.
      * MUST be called inside the same transaction as Job.save().
      */
     saveStageSnapshots(snapshots: readonly PipelineStageSnapshot[]): Promise<void>;
   }

   export const PIPELINE_TEMPLATE_REPOSITORY = Symbol('PipelineTemplateRepository');
   ```

2. Validate business invariants expressed in the interface documentation:
   - `findDefault` returns `null` (not throws) when no template exists — the handler throws `NoPipelineTemplateFoundError`.
   - `findById` returns `null` when the template is not in the tenant — the handler throws `PipelineTemplateNotInTenantError`.
   - `saveStageSnapshots` is transactional — the handler wraps it inside `UnitOfWork.runInTransaction`.

3. Export `PipelineTemplateRepository`, `PipelineStageSnapshot`, and `PIPELINE_TEMPLATE_REPOSITORY` from `modules/jobs/domain/ports/index.ts`.

---

## ✅ Acceptance Criteria

- **Given** the `PipelineTemplateRepository` interface  
  **When** `findDefault` is called with a `tenantId` that has no template  
  **Then** the interface contract specifies `null` is returned (not a thrown error)

- **Given** a `PipelineStageSnapshot` value  
  **When** `ownerType` is read  
  **Then** it is always `'Job'` (never `'PipelineTemplate'`)

- **Given** `saveStageSnapshots` is called with an empty array  
  **When** the call resolves  
  **Then** no error is thrown (valid edge case: template with no stages)

- **Given** the interface is used in `CreateJobHandler`  
  **When** the concrete `PrismaJobRepository` is swapped for an in-memory fake  
  **Then** the handler compiles and behaves identically

---

## 🔗 Dependencies

- TASK-003 (domain errors `NoPipelineTemplateFoundError`, `PipelineTemplateNotInTenantError` must exist)
