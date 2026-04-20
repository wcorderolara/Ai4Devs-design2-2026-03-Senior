PATH: /tasks/us-001-create-job-opening/task-011-frontend-create-job-page.md

# 🔧 Task: Frontend — Create Job Page + JobForm Component

## 🆔 Metadata
- ID: TASK-011
- Related User Story: US-001
- Layer: Frontend
- Estimation: 5 points

---

## 📝 Description

Build the UI for creating a job opening. This consists of:
1. A Next.js App Router page at `/t/[tenantSlug]/jobs/new/page.tsx` (Server Component — handles auth guard and data fetching for pipeline templates).
2. A `JobForm` client component using React Hook Form + Zod resolver + shadcn/ui components. The form renders all required fields and submits via the `useCreateJob` mutation hook.

All field validation must reuse the `createJobRequestSchema` from `@ats/contracts` (no duplicated Zod schemas in the frontend).

---

## ⚙️ Steps

### Page (`apps/web/src/app/t/[tenantSlug]/jobs/new/page.tsx`)

1. Mark as `async` Server Component.
2. Validate session with Auth.js — redirect to `/login` if unauthenticated.
3. Fetch available pipeline templates from the API (via internal fetch with tenant context).
4. Pass `pipelineTemplates` as props to `<JobForm />`.
5. Render a breadcrumb: `Jobs > New Job Opening`.

### JobForm (`apps/web/src/features/jobs/components/job-form.tsx`)

1. Mark `'use client'`.
2. Use `useForm<CreateJobRequest>` from React Hook Form with `zodResolver(createJobRequestSchema)`.
3. Render the following fields using shadcn/ui:
   - `title` → `<Input>` with label "Job Title" (required)
   - `department` → `<Input>` with label "Department"
   - `location` → `<Input>` with label "Location"
   - `workMode` → `<Select>` with options: Remote, Hybrid, Onsite
   - `description` → `<Textarea>` with label "Description" (required, min 20 chars)
   - `sourceTemplateId` → `<Select>` listing available templates; default pre-selected if `isDefault = true`
   - `status` → `<RadioGroup>` with options: Draft, Open (default: Draft)
4. Display inline field-level error messages from `formState.errors`.
5. Submit button: "Create Job Opening" — disabled while `isPending`.
6. On success: `router.push(`/t/${tenantSlug}/jobs/${newJob.id}`)`.
7. On error: display a toast notification with the error message.

---

## ✅ Acceptance Criteria

- **Given** the user navigates to `/t/acme/jobs/new`  
  **When** the page renders  
  **Then** the form is visible with all required fields and the breadcrumb shows `Jobs > New Job Opening`

- **Given** the user submits the form with `title = 'AB'`  
  **When** React Hook Form validates  
  **Then** an inline error `"Title must be at least 3 characters"` appears below the title field without making an API call

- **Given** the user selects `workMode = 'Remote'`  
  **When** the form renders  
  **Then** the select displays `'Remote'` as the selected value

- **Given** the form is submitted with all valid fields  
  **When** the API returns `201 Created`  
  **Then** the user is redirected to `/t/acme/jobs/<newJobId>`

- **Given** the API returns `422` (no pipeline template)  
  **When** the form submit handler catches the error  
  **Then** a toast notification displays `"A pipeline template must be configured by your Admin"`

- **Given** the user is unauthenticated  
  **When** `/t/acme/jobs/new` is accessed  
  **Then** they are redirected to `/login`

---

## 🔗 Dependencies

- TASK-009 (Zod schema from `@ats/contracts`)
- TASK-012 (useCreateJob hook must exist before form can be wired up)
