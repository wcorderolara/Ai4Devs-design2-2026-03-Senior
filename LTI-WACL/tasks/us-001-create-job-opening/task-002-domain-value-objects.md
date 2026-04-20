PATH: /tasks/us-001-create-job-opening/task-002-domain-value-objects.md

# 🔧 Task: Domain — Value Objects (JobId, JobTitle, JobStatus, WorkMode)

## 🆔 Metadata
- ID: TASK-002
- Related User Story: US-001
- Layer: Domain
- Estimation: 2 points

---

## 📝 Description

Implement the four Value Objects required by the `Job` aggregate root in `modules/jobs/domain/value-objects/`. Each VO MUST be immutable, self-validating, and throw a domain-specific error on invalid input. Use the `ValueObject<T>` base class from `@/shared/domain/value-object`. No VO may import from `infrastructure/` or `application/`.

---

## ⚙️ Steps

1. Create `job-id.vo.ts`:
   - Wraps a non-empty `string`.
   - Throws `InvalidValueError` (from shared) if blank.
   - Exposes `get value(): string`.

2. Create `job-title.vo.ts`:
   - Trims input on creation.
   - Min length: 3 chars → throws `JobTitleTooShortError`.
   - Max length: 120 chars → throws `JobTitleTooLongError`.
   - Exposes `get value(): string`.

3. Create `job-status.vo.ts`:
   - Enum values: `'draft' | 'open' | 'closed' | 'archived'`.
   - Static factories: `draft()`, `open()`.
   - `canTransitionTo(next: JobStatus): boolean` — enforces state machine:
     - `draft → open | archived`
     - `open → closed | archived`
     - `closed → archived`
     - `archived → (none)`
   - `transitionTo(next: JobStatus): JobStatus` — throws `InvalidJobStatusTransitionError` if transition is illegal.

4. Create `work-mode.vo.ts`:
   - Enum values: `'remote' | 'hybrid' | 'onsite'`.
   - Throws `InvalidValueError` if value is not in enum.
   - Exposes `get value(): WorkModeValue`.

5. Export all four VOs from `modules/jobs/domain/value-objects/index.ts`.

---

## ✅ Acceptance Criteria

- **Given** `JobTitle.create('AB')`  
  **When** the VO is instantiated  
  **Then** `JobTitleTooShortError` is thrown with `actualLength = 2`

- **Given** `JobTitle.create('  ')`  
  **When** the VO is instantiated  
  **Then** `JobTitleTooShortError` is thrown (trimmed length = 0 < 3)

- **Given** `JobStatus.draft().transitionTo(JobStatus.create('closed'))`  
  **When** the transition is called  
  **Then** `InvalidJobStatusTransitionError` is thrown

- **Given** `JobStatus.draft().transitionTo(JobStatus.open())`  
  **When** the transition is called  
  **Then** a new `JobStatus` with value `'open'` is returned

- **Given** `WorkMode.create('fulltime' as any)`  
  **When** the VO is instantiated  
  **Then** `InvalidValueError` is thrown

---

## 🔗 Dependencies

- TASK-003 (domain errors must exist first for imports in `job-title.vo.ts` and `job-status.vo.ts`)
