PATH: /tasks/us-001-create-job-opening/task-001-prisma-schema-migration.md

# 🔧 Task: Prisma Schema — Job, PipelineTemplate, PipelineStage Models + Migration

## 🆔 Metadata
- ID: TASK-001
- Related User Story: US-001
- Layer: Persistence
- Estimation: 3 points

---

## 📝 Description

Define the Prisma models for `Job`, `PipelineTemplate`, and `PipelineStage` in `apps/api/prisma/schema.prisma`. The `PipelineStage` model supports both template-owned and job-owned stages via `ownerType` + `ownerId` polymorphism (the pipeline snapshot pattern). Create and run the corresponding database migration, including all required indexes for tenant-scoped queries.

---

## ⚙️ Steps

1. Open `apps/api/prisma/schema.prisma`.
2. Add the `Job` model:
   ```prisma
   model Job {
     id               String    @id @default(cuid())
     tenantId         String    @map("tenant_id")
     title            String
     department       String
     location         String
     workMode         String    @map("work_mode")
     description      String    @db.Text
     status           String    @default("draft")
     sourceTemplateId String?   @map("source_template_id")
     createdBy        String    @map("created_by")
     createdAt        DateTime  @default(now()) @map("created_at")
     updatedAt        DateTime  @updatedAt @map("updated_at")
     publishedAt      DateTime? @map("published_at")
     closedAt         DateTime? @map("closed_at")

     tenant         Tenant          @relation(fields: [tenantId], references: [id], onDelete: Cascade)
     pipelineStages PipelineStage[]

     @@index([tenantId, status],    map: "idx_jobs_tenant_status")
     @@index([tenantId, createdAt], map: "idx_jobs_tenant_created_at")
     @@map("jobs")
   }
   ```
3. Add the `PipelineTemplate` model:
   ```prisma
   model PipelineTemplate {
     id        String    @id @default(cuid())
     tenantId  String    @map("tenant_id")
     name      String
     isDefault Boolean   @default(false) @map("is_default")
     createdAt DateTime  @default(now()) @map("created_at")
     updatedAt DateTime  @updatedAt @map("updated_at")

     tenant Tenant          @relation(fields: [tenantId], references: [id], onDelete: Cascade)
     stages PipelineStage[]

     @@unique([tenantId, isDefault], map: "uq_pipeline_templates_tenant_default")
     @@index([tenantId], map: "idx_pipeline_templates_tenant")
     @@map("pipeline_templates")
   }
   ```
4. Add the `PipelineStage` model (polymorphic via `ownerType` + `ownerId`):
   ```prisma
   model PipelineStage {
     id          String   @id @default(cuid())
     tenantId    String   @map("tenant_id")
     ownerType   String   @map("owner_type")   // 'PipelineTemplate' | 'Job'
     ownerId     String   @map("owner_id")
     name        String
     position    Int
     isFinal     Boolean  @default(false) @map("is_final")
     stageType   String   @default("standard") @map("stage_type")  // 'standard' | 'rejected' | 'hired'
     createdAt   DateTime @default(now()) @map("created_at")

     tenant           Tenant            @relation(fields: [tenantId], references: [id], onDelete: Cascade)
     pipelineTemplate PipelineTemplate? @relation(fields: [ownerId], references: [id], map: "fk_pipeline_stages_template")
     job              Job?              @relation(fields: [ownerId], references: [id], map: "fk_pipeline_stages_job")

     @@index([tenantId, ownerType, ownerId], map: "idx_pipeline_stages_owner")
     @@index([ownerId, position],            map: "idx_pipeline_stages_position")
     @@map("pipeline_stages")
   }
   ```
5. Run `pnpm prisma migrate dev --name add-jobs-pipeline-schema` from `apps/api/`.
6. Run `pnpm prisma generate` to regenerate the Prisma client.
7. Verify migration applied cleanly: `pnpm prisma migrate status`.

---

## ✅ Acceptance Criteria

- **Given** the schema file is updated and migration is run  
  **When** `pnpm prisma migrate status` is executed  
  **Then** all migrations are applied with no pending state

- **Given** a `Job` record is inserted for `tenantId = 'tenant_A'`  
  **When** a query filters by `tenantId = 'tenant_B'`  
  **Then** zero rows are returned (tenant isolation enforced at DB level)

- **Given** a `PipelineStage` with `ownerType = 'Job'`  
  **When** the linked `Job` is deleted  
  **Then** the `PipelineStage` records cascade-delete via `tenantId` constraint

- **Given** two `PipelineTemplate` records for the same tenant  
  **When** both have `isDefault = true`  
  **Then** the unique constraint `uq_pipeline_templates_tenant_default` prevents the second insert

---

## 🔗 Dependencies

- None (first task — no code dependencies)
