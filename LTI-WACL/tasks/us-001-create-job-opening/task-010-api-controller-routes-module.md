PATH: /tasks/us-001-create-job-opening/task-010-api-controller-routes-module.md

# 🔧 Task: API — JobsController, jobs.routes.ts, jobs.module.ts

## 🆔 Metadata
- ID: TASK-010
- Related User Story: US-001
- Layer: API
- Estimation: 3 points

---

## 📝 Description

Implement the HTTP interface layer for the `jobs` bounded context: the Express controller, router with RBAC guard, and the composition root module that wires all dependencies together. The route is: `POST /api/v1/t/:tenantSlug/jobs`. The controller MUST NOT contain business logic — it delegates entirely to `CreateJobHandler`. Validation is via Zod schema from `@ats/contracts`.

---

## ⚙️ Steps

### JobsController (`interface/http/jobs.controller.ts`)

1. Constructor receives `CreateJobHandler`.
2. `create` method:
   ```typescript
   public create = async (req: Request, res: Response, next: NextFunction): Promise<void> => {
     try {
       const parsed = createJobRequestSchema.safeParse(req.body);
       if (!parsed.success) throw ValidationError.fromZod(parsed.error);

       const cmd = new CreateJobCommand(
         parsed.data.title,
         parsed.data.department,
         parsed.data.location,
         parsed.data.workMode,
         parsed.data.description,
         parsed.data.status,
         parsed.data.sourceTemplateId,
       );

       const dto = await this.createJobHandler.execute(req.tenantContext, cmd);

       res
         .status(201)
         .location(`/api/v1/t/${req.tenantContext.tenantSlug}/jobs/${dto.id}`)
         .json({ data: dto, meta: { requestId: req.requestId } });
     } catch (err) {
       next(err);
     }
   };
   ```

### Router (`interface/http/jobs.routes.ts`)

1. Use `Router({ mergeParams: true })`.
2. Register: `router.post('/', authMiddleware(), tenantResolver(), rbac('jobs:create'), controller.create)`.
3. Mount in `app.ts` under `/api/v1/t/:tenantSlug/jobs`.

### Composition Root (`jobs.module.ts`)

1. Create `JobsModule` class with a static `register(prisma: PrismaClient, ...ports): JobsModule` factory.
2. Wire dependencies:
   - `PrismaJobRepository` ← `prisma`
   - `PrismaPipelineTemplateRepository` ← `prisma`
   - `PrismaUnitOfWork` ← `prisma`
   - `SystemClock` ← from shared infra
   - `CuidIdGenerator` ← from shared infra
   - `InProcessEventBus` ← from shared infra
   - `CreateJobHandler` ← all above
   - `JobsController` ← `CreateJobHandler`
3. Expose `router(): Router` that returns `buildJobsRouter(this.controller)`.

### Error Handler Integration

4. Verify that `NoPipelineTemplateFoundError` maps to `422 Unprocessable Entity` in the global error handler middleware (`shared/interface/middleware/error-handler.ts`).
5. Verify that `InactiveCompanyError` maps to `403 Forbidden`.

---

## ✅ Acceptance Criteria

- **Given** `POST /api/v1/t/acme/jobs` with a valid body and `Authorization: Bearer <recruiter-token>`  
  **When** the request is processed  
  **Then** response is `201 Created` with `Location` header and `data.id` in the body

- **Given** `POST /api/v1/t/acme/jobs` with role `Interviewer`  
  **When** the RBAC middleware evaluates `jobs:create`  
  **Then** response is `403 Forbidden`

- **Given** a request body with `title: 'AB'` (too short)  
  **When** Zod validation runs  
  **Then** response is `400 Bad Request` with field-level `errors` array

- **Given** `X-Tenant-ID: tenant_OTHER` but path slug is `acme` (tenant_acme)  
  **When** `tenantResolver()` middleware runs  
  **Then** response is `403 Forbidden` with code `TENANT_MISMATCH`

- **Given** the company is `Inactive`  
  **When** `CreateJobHandler` throws `InactiveCompanyError`  
  **Then** the error handler returns `403 Forbidden`

- **Given** no `PipelineTemplate` exists  
  **When** `CreateJobHandler` throws `NoPipelineTemplateFoundError`  
  **Then** the error handler returns `422 Unprocessable Entity`

---

## 🔗 Dependencies

- TASK-006 (CreateJobCommand + CreateJobHandler)
- TASK-007 (JobDto)
- TASK-008 (Prisma repositories)
- TASK-009 (Zod schema from contracts)
