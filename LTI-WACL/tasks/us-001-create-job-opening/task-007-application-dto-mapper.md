PATH: /tasks/us-001-create-job-opening/task-007-application-dto-mapper.md

# 🔧 Task: Application — JobDto, JobMapper, PipelineStageDto

## 🆔 Metadata
- ID: TASK-007
- Related User Story: US-001
- Layer: Domain
- Estimation: 1 point

---

## 📝 Description

Create the read-model DTOs and mapper used by the application layer to convert `Job` aggregates into serializable plain objects. These DTOs cross the boundary between `application/` and `interface/` layers. They MUST contain only primitives (no domain VOs) and MUST NOT import from `infrastructure/`.

---

## ⚙️ Steps

1. Create `modules/jobs/application/dtos/job.dto.ts`:
   ```typescript
   export interface JobDto {
     id: string;
     tenantId: string;
     title: string;
     department: string;
     location: string;
     workMode: string;
     description: string;
     status: string;
     sourceTemplateId: string | null;
     createdBy: string;
     createdAt: string;   // ISO 8601
     updatedAt: string;   // ISO 8601
     publishedAt: string | null;
     closedAt: string | null;
   }
   ```

2. Create `modules/jobs/application/dtos/pipeline-stage.dto.ts`:
   ```typescript
   export interface PipelineStageDto {
     id: string;
     name: string;
     position: number;
     isFinal: boolean;
     stageType: 'standard' | 'rejected' | 'hired';
   }
   ```

3. Create `modules/jobs/application/dtos/job.mapper.ts`:
   ```typescript
   export class JobMapper {
     static toDto(job: Job): JobDto {
       const raw = job.toPrimitives();
       return {
         id: raw.id,
         tenantId: raw.tenantId,
         title: raw.title,
         department: raw.department,
         location: raw.location,
         workMode: raw.workMode,
         description: raw.description,
         status: raw.status,
         sourceTemplateId: raw.sourceTemplateId ?? null,
         createdBy: raw.createdBy,
         createdAt: raw.createdAt.toISOString(),
         updatedAt: raw.updatedAt.toISOString(),
         publishedAt: raw.publishedAt?.toISOString() ?? null,
         closedAt: raw.closedAt?.toISOString() ?? null,
       };
     }
   }
   ```

4. Export all from `modules/jobs/application/dtos/index.ts`.

---

## ✅ Acceptance Criteria

- **Given** a `Job` aggregate with `status = 'draft'` and `publishedAt = null`  
  **When** `JobMapper.toDto(job)` is called  
  **Then** `dto.publishedAt` is `null` and `dto.status` is `'draft'`

- **Given** a `Job` aggregate with `status = 'open'` and `publishedAt = new Date('2026-01-01')`  
  **When** `JobMapper.toDto(job)` is called  
  **Then** `dto.publishedAt` equals `'2026-01-01T00:00:00.000Z'`

- **Given** `JobDto` is imported in `interface/http/`  
  **When** TypeScript strict mode is active  
  **Then** no type errors are produced

---

## 🔗 Dependencies

- TASK-004 (Job aggregate with `toPrimitives()`)
