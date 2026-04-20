PATH: /tasks/us-001-create-job-opening/task-012-frontend-hook-server-action.md

# 🔧 Task: Frontend — useCreateJob Mutation Hook + Server Action

## 🆔 Metadata
- ID: TASK-012
- Related User Story: US-001
- Layer: Frontend
- Estimation: 2 points

---

## 📝 Description

Implement the data-fetching layer for the Create Job Opening flow:

1. **`useCreateJob` hook** (`apps/web/src/features/jobs/hooks/use-create-job.ts`) — a TanStack Query `useMutation` hook that calls the backend `POST /api/v1/t/:tenantSlug/jobs` endpoint.
2. **Server Action** (`apps/web/src/features/jobs/actions/create-job.action.ts`) — optional progressive-enhancement path using Next.js Server Actions. Must validate input with `createJobRequestSchema` before forwarding to the API.

Both MUST use the `CreateJobRequest` and `JobResponse` types from `@ats/contracts` to ensure type safety across the network boundary.

---

## ⚙️ Steps

### useCreateJob hook

1. Create `apps/web/src/features/jobs/hooks/use-create-job.ts`:
   ```typescript
   'use client';
   import { useMutation, useQueryClient } from '@tanstack/react-query';
   import type { CreateJobRequest, JobResponse } from '@ats/contracts/jobs';

   export function useCreateJob(tenantSlug: string) {
     const queryClient = useQueryClient();

     return useMutation<JobResponse, Error, CreateJobRequest>({
       mutationFn: async (data) => {
         const res = await fetch(`/api/v1/t/${tenantSlug}/jobs`, {
           method: 'POST',
           headers: { 'Content-Type': 'application/json' },
           body: JSON.stringify(data),
         });
         if (!res.ok) {
           const err = await res.json();
           throw new Error(err?.error?.message ?? 'Failed to create job');
         }
         const body = await res.json();
         return body.data as JobResponse;
       },
       onSuccess: () => {
         queryClient.invalidateQueries({ queryKey: ['jobs', tenantSlug] });
       },
     });
   }
   ```

### Server Action (progressive enhancement)

2. Create `apps/web/src/features/jobs/actions/create-job.action.ts`:
   ```typescript
   'use server';
   import { redirect } from 'next/navigation';
   import { auth } from '@/lib/auth';
   import { createJobRequestSchema } from '@ats/contracts/jobs';

   export async function createJobAction(tenantSlug: string, formData: FormData) {
     const session = await auth();
     if (!session) redirect('/login');

     const raw = Object.fromEntries(formData.entries());
     const parsed = createJobRequestSchema.safeParse(raw);
     if (!parsed.success) {
       return { success: false, errors: parsed.error.flatten().fieldErrors };
     }

     const res = await fetch(
       `${process.env.API_INTERNAL_URL}/api/v1/t/${tenantSlug}/jobs`,
       {
         method: 'POST',
         headers: {
           'Content-Type': 'application/json',
           Authorization: `Bearer ${session.accessToken}`,
           'X-Tenant-ID': session.user.tenantId,
         },
         body: JSON.stringify(parsed.data),
       },
     );

     if (!res.ok) {
       const err = await res.json();
       return { success: false, apiError: err?.error?.message };
     }

     const body = await res.json();
     redirect(`/t/${tenantSlug}/jobs/${body.data.id}`);
   }
   ```

3. Add `API_INTERNAL_URL` to `packages/config-ts/env.schema.ts` if not already present.

---

## ✅ Acceptance Criteria

- **Given** `useCreateJob('acme')` is called with a valid `CreateJobRequest`  
  **When** the mutation resolves  
  **Then** `data` is a `JobResponse` with `id`, `status = 'draft'`, and the `jobs` query cache is invalidated

- **Given** the API returns `400` with field errors  
  **When** the mutation rejects  
  **Then** `error.message` contains the API error message (not an unhandled promise rejection)

- **Given** `createJobAction` is called without a valid session  
  **When** the action runs  
  **Then** `redirect('/login')` is called

- **Given** `createJobAction` is called with `title = 'AB'`  
  **When** Zod validates the FormData  
  **Then** `{ success: false, errors: { title: [...] } }` is returned without making an API call

---

## 🔗 Dependencies

- TASK-009 (Zod schema + types from `@ats/contracts`)
- TASK-010 (API endpoint must be deployed and reachable)
