PATH: /tasks/us-001-create-job-opening/task-009-validation-zod-schema.md

# 🔧 Task: Validation — Zod Contract Schema (packages/contracts)

## 🆔 Metadata
- ID: TASK-009
- Related User Story: US-001
- Layer: Validation
- Estimation: 1 point

---

## 📝 Description

Create the shared Zod schemas for the Create Job Opening request and response in `packages/contracts/src/jobs/`. These schemas are the single source of truth for input validation — imported by both the Express API layer and the Next.js frontend. They MUST enforce all business rules from the user story validation section and MUST NOT import from any `apps/` or `modules/` code.

---

## ⚙️ Steps

1. Create `packages/contracts/src/jobs/create-job.schema.ts`:
   ```typescript
   import { z } from 'zod';

   export const workModeSchema = z.enum(['remote', 'hybrid', 'onsite']);
   export type WorkMode = z.infer<typeof workModeSchema>;

   export const jobStatusSchema = z.enum(['draft', 'open', 'closed', 'archived']);
   export const creatableJobStatusSchema = z.enum(['draft', 'open']);
   export type CreatableJobStatus = z.infer<typeof creatableJobStatusSchema>;

   export const createJobRequestSchema = z.object({
     title:            z.string().trim().min(3, 'Title must be at least 3 characters').max(120),
     department:       z.string().trim().min(2).max(80),
     location:         z.string().trim().min(2).max(120),
     workMode:         workModeSchema,
     description:      z.string().trim().min(20, 'Description must be at least 20 characters').max(10_000),
     status:           creatableJobStatusSchema.default('draft'),
     sourceTemplateId: z.string().cuid().optional(),
   });
   export type CreateJobRequest = z.infer<typeof createJobRequestSchema>;

   export const jobResponseSchema = z.object({
     id:               z.string(),
     tenantId:         z.string(),
     title:            z.string(),
     department:       z.string(),
     location:         z.string(),
     workMode:         workModeSchema,
     description:      z.string(),
     status:           jobStatusSchema,
     sourceTemplateId: z.string().nullable(),
     createdBy:        z.string(),
     createdAt:        z.string().datetime(),
     updatedAt:        z.string().datetime(),
     publishedAt:      z.string().datetime().nullable(),
     closedAt:         z.string().datetime().nullable(),
   });
   export type JobResponse = z.infer<typeof jobResponseSchema>;
   ```

2. Export from `packages/contracts/src/jobs/index.ts`:
   ```typescript
   export * from './create-job.schema';
   ```

3. Export from `packages/contracts/src/index.ts`:
   ```typescript
   export * from './jobs';
   ```

4. Run `pnpm --filter @ats/contracts build` to verify no TypeScript errors.

---

## ✅ Acceptance Criteria

- **Given** `createJobRequestSchema.safeParse({ title: 'AB', ... })`  
  **When** parsed  
  **Then** `success = false` with a `title` field error (`min(3)` violated)

- **Given** `createJobRequestSchema.safeParse({ workMode: 'fulltime', ... })`  
  **When** parsed  
  **Then** `success = false` with a `workMode` field error (invalid enum value)

- **Given** `createJobRequestSchema.safeParse({ ..., status: 'closed', ... })`  
  **When** parsed  
  **Then** `success = false` (`creatableJobStatusSchema` only allows `draft` or `open`)

- **Given** `createJobRequestSchema.safeParse({ ..., status: undefined })`  
  **When** parsed  
  **Then** `success = true` and `data.status = 'draft'` (default applied)

- **Given** the schema is imported in both `apps/api` and `apps/web`  
  **When** `pnpm typecheck` runs  
  **Then** zero TypeScript errors in both apps

---

## 🔗 Dependencies

- None (standalone package; no dependency on other tasks)
