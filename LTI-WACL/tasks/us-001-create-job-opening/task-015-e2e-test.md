PATH: /tasks/us-001-create-job-opening/task-015-e2e-test.md

# 🔧 Task: Testing — E2E Test (Playwright — Create Job Opening)

## 🆔 Metadata
- ID: TASK-015
- Related User Story: US-001
- Layer: Testing
- Estimation: 3 points

---

## 📝 Description

Write Playwright E2E tests for the "Create Job Opening" user journey. Tests run against a full Docker stack (Next.js frontend + Express API + PostgreSQL). They simulate a Recruiter logging in, navigating to the new job form, filling it out, submitting, and verifying the redirect to the newly created job's detail page.

File: `apps/web/e2e/specs/jobs/create-job.spec.ts`

---

## ⚙️ Steps

1. Ensure `apps/web/e2e/playwright.config.ts` points to the Docker test stack base URL.
2. Create `apps/web/e2e/fixtures/auth.fixture.ts`:
   - `loginAsRecruiter(page, tenantSlug)` — fills login form and stores auth cookie.
3. Create `apps/web/e2e/fixtures/seed.fixture.ts`:
   - Before each test: seed `tenant`, `recruiter user`, and `default PipelineTemplate` via API or direct DB script.
   - After each test: cleanup via `DELETE FROM jobs WHERE tenant_id = 'e2e_tenant'`.

4. Test cases:

   **Happy path — draft job:**
   ```
   1. Login as recruiter for tenant 'e2e_tenant'
   2. Navigate to /t/e2e_tenant/jobs/new
   3. Fill in: title, department, location, workMode=Remote, description (≥ 20 chars)
   4. Leave status = Draft (default)
   5. Click "Create Job Opening"
   6. Assert: redirected to /t/e2e_tenant/jobs/<id>
   7. Assert: job detail page shows title and "Draft" badge
   ```

   **Client-side validation — title too short:**
   ```
   1. Navigate to /t/e2e_tenant/jobs/new
   2. Type 'AB' in title field
   3. Click "Create Job Opening"
   4. Assert: error message "Title must be at least 3 characters" is visible
   5. Assert: URL remains /t/e2e_tenant/jobs/new (no navigation)
   ```

   **Unauthenticated access:**
   ```
   1. Navigate to /t/e2e_tenant/jobs/new without logging in
   2. Assert: redirected to /login
   ```

   **Role guard — Interviewer cannot access form:**
   ```
   1. Login as user with role Interviewer
   2. Navigate to /t/e2e_tenant/jobs/new
   3. Assert: 403 page or redirect shown (no form rendered)
   ```

5. Use `expect(page).toHaveURL(...)` and `expect(page.getByRole('status')).toContainText('Draft')` for assertions.
6. Add screenshots on failure via `test.use({ screenshot: 'only-on-failure' })`.

---

## ✅ Acceptance Criteria

- **Given** a logged-in Recruiter on `/t/e2e_tenant/jobs/new`  
  **When** the form is completed with valid data and submitted  
  **Then** the browser navigates to `/t/e2e_tenant/jobs/<newId>` and the title is visible on the detail page

- **Given** `title = 'AB'` is entered  
  **When** the form is submitted  
  **Then** the inline error `"Title must be at least 3 characters"` is visible and no navigation occurs

- **Given** an unauthenticated user visits `/t/e2e_tenant/jobs/new`  
  **When** the page loads  
  **Then** they are redirected to `/login`

- **Given** all E2E tests run in CI against the Docker stack  
  **When** `pnpm e2e` is executed  
  **Then** all tests pass with zero flaky failures across 3 consecutive runs

---

## 🔗 Dependencies

- TASK-010 (API endpoint)
- TASK-011 (Frontend page + form)
- TASK-012 (useCreateJob hook)
