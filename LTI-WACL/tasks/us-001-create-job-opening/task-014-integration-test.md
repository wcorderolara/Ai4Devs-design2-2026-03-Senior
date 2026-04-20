PATH: /tasks/us-001-create-job-opening/task-014-integration-test.md

# 🔧 Task: Testing — Integration Test (POST /api/v1/t/:tenantSlug/jobs)

## 🆔 Metadata
- ID: TASK-014
- Related User Story: US-001
- Layer: Testing
- Estimation: 3 points

---

## 📝 Description

Write Jest integration tests for the `POST /api/v1/t/:tenantSlug/jobs` endpoint using `supertest` and a real PostgreSQL test database (via Testcontainers). These tests verify the full request-to-response stack including middleware, RBAC, validation, handler, Prisma repository, and DB state. Tests MUST assert both HTTP response shape and actual database records. Multi-tenant isolation MUST be explicitly validated.

File: `modules/jobs/__tests__/integration/create-job.integration.spec.ts`

---

## ⚙️ Steps

1. Set up `TestHarness` in `apps/api/test/setup/build-test-app.ts` if not yet implemented:
   - Spins up an Express app with all modules registered.
   - Wraps a Testcontainers PostgreSQL instance.
   - Provides `harness.seedTenant()`, `harness.seedUser()`, `harness.seedPipelineTemplate()`, `harness.signUserToken()`, `harness.reset()`.

2. `beforeAll`: Start the harness.
3. `afterAll`: Shutdown the harness.
4. `beforeEach`: `harness.reset()` then seed:
   - Tenant `acme` (id: `tenant_acme`, status: `active`).
   - User `user_1` (role: `recruiter`, tenantId: `tenant_acme`).
   - Default PipelineTemplate with 3 stages for `tenant_acme`.

5. Test cases:

   **Happy path — draft:**
   ```
   POST /api/v1/t/acme/jobs
   Body: { title, department, location, workMode: 'remote', description, status: 'draft' }
   Headers: Authorization: Bearer <recruiter-token>, X-Tenant-ID: tenant_acme
   → 201, Location header matches /\/api\/v1\/t\/acme\/jobs\/.+/
   → data.status = 'draft', data.publishedAt = null
   → DB: jobs table has 1 row, pipeline_stages has 3 rows (ownerType='Job')
   ```

   **Happy path — open:**
   ```
   POST with status: 'open'
   → data.publishedAt is not null
   ```

   **Tenant mismatch:**
   ```
   X-Tenant-ID: tenant_OTHER
   → 403, error.code = 'TENANT_MISMATCH'
   ```

   **Unauthorized role (Interviewer):**
   ```
   User with role 'interviewer'
   → 403, error.code = 'FORBIDDEN'
   ```

   **Validation — title too short:**
   ```
   title: 'AB'
   → 400, errors array contains title field error
   ```

   **No pipeline template:**
   ```
   Seed tenant without a PipelineTemplate
   → 422, error.code = 'NO_PIPELINE_TEMPLATE_FOUND'
   ```

   **Inactive company:**
   ```
   Seed tenant with status: 'inactive'
   → 403, error.code = 'INACTIVE_COMPANY'
   ```

   **Multi-tenant isolation:**
   ```
   Create a job under tenant_acme
   Assert that tenant_other sees zero jobs in their pipeline query
   ```

---

## ✅ Acceptance Criteria

- **Given** a valid recruiter request with matching tenant  
  **When** `POST /api/v1/t/acme/jobs` is called  
  **Then** `response.status === 201` and `pipeline_stages` table has exactly 3 rows with `owner_type = 'Job'`

- **Given** `X-Tenant-ID` header does not match path slug  
  **When** the request is processed  
  **Then** `response.status === 403` with `TENANT_MISMATCH` code

- **Given** a user with role `interviewer`  
  **When** the request is processed  
  **Then** `response.status === 403` with `FORBIDDEN` code

- **Given** a request body with `title: 'AB'`  
  **When** validation runs  
  **Then** `response.status === 400` and no rows are inserted in the `jobs` table

- **Given** two requests from two different tenants  
  **When** each creates a job  
  **Then** `SELECT * FROM jobs WHERE tenant_id = 'tenant_A'` returns only tenant_A's jobs

---

## 🔗 Dependencies

- TASK-001 (DB schema must exist)
- TASK-008 (Prisma repositories)
- TASK-010 (API controller + routes + module)
