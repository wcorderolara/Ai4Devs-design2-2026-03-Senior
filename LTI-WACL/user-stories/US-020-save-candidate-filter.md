PATH: /user-stories/US-020-save-candidate-filter.md

# 🧾 User Story: Save Candidate Search Filter

## 🆔 Metadata
- ID: US-020
- Feature: Candidate Management
- Priority: Medium
- Status: Draft
- Owner: PM

## 🎯 User Story
**As a** Recruiter  
**I want** to save a combination of search filters I use frequently under a custom name  
**So that** I can reapply the same filters instantly in future sessions without reconstructing them manually

## 🧠 Business Context
- Related Use Case(s): UC-23 Guardar filtro de búsqueda de candidatos
- Entities involved: SavedFilter, User, Company
- Business Rules:
  - A `SavedFilter` is scoped to a module context: `Candidate`, `Application`, or `JobOpening`.
  - `name` must be unique per `ownerUserId + module`.
  - `filterJson` must not be empty; it serializes the active filter criteria.
  - `visibility` can be `Private` (only the creator sees it) or `Shared` (visible to team members with appropriate role).
  - Creating a `Shared` filter requires explicit permission (Recruiter role is sufficient in MVP; Admin approval may be added post-MVP).
  - Filters never reference data from another Company.
- Assumptions:
  - Sort order (`sortJson`) is persisted alongside filter criteria so the full search context is restored.
  - A saved filter is not a snapshot of results — it saves criteria only, applied against live data on reuse.
- Dependencies:
  - User must be authenticated with role Recruiter, HiringManager, or Admin.

## ✅ Acceptance Criteria (BDD + Edge Cases)

### 🎯 Happy Path
- **Given** an authenticated Recruiter with an active search using filters for status `Active` and a specific JobOpening  
  **When** the Recruiter clicks "Save filter", provides the name "Active Java Devs Q2", chooses `Private` visibility, and confirms  
  **Then** a `SavedFilter` record is persisted with `module = Candidate`, `filterJson` containing the current criteria, `sortJson` containing the current sort order, and the filter appears in the Recruiter's saved filter list for reuse

### ⚠️ Edge Cases

#### Case 1: Duplicate filter name for the same user and module
- **Given** a Recruiter already has a saved filter named "Active Java Devs Q2" in the `Candidate` module  
  **When** the Recruiter tries to save another filter with the same name  
  **Then** the system returns a 409 Conflict error and offers the Recruiter the option to overwrite the existing filter or choose a different name  
- Handling:
  - Enforce unique constraint on `(ownerUserId, module, name)`. Surface the conflict in the UI with both options.

#### Case 2: Saving a filter with no active criteria
- **Given** a Recruiter opens the save dialog with no filters applied  
  **When** the Recruiter submits without any active criteria  
  **Then** the system returns a 400 validation error indicating that at least one filter criterion is required  
- Handling:
  - `filterJson` must not be empty or equivalent to `{}`.

#### Case 3: Reusing a shared filter that was updated by another user
- **Given** a Recruiter applies a `Shared` filter created by a colleague  
  **When** the filter is applied  
  **Then** the system applies the current saved `filterJson` criteria against live data at the time of reuse  
- Handling:
  - Filters are not versioned in MVP. The latest saved state is always used. This is expected behavior.

#### Case 4: Deleting a saved filter
- **Given** a Recruiter selects a private saved filter to delete  
  **When** the delete action is confirmed  
  **Then** the `SavedFilter` record is removed and no longer appears in the saved filter list  
- Handling:
  - Deletion is a hard delete for `SavedFilter`; no soft delete required as filters have no downstream dependencies.

### 🚫 Validation Rules
- `name` must not be blank and must be unique per `ownerUserId + module`.
- `module` must be one of: `Candidate`, `Application`, `JobOpening`.
- `filterJson` must be non-empty valid JSON.
- `visibility` must be `Private` or `Shared`.
- The `SavedFilter` must be scoped to the authenticated user's `companyId`.
- A `Shared` filter must not expose data from another Company when applied.

## 📊 Business Impact
- KPI Affected: Recruiter session efficiency, Time to access candidate subsets, Operational consistency
- Expected Impact: Eliminates repeated filter reconstruction for common searches, saving 3–5 minutes per session for high-frequency recruiters
- Success Criteria:
  - Saved filters applied in under 500ms (filter criteria retrieval, not search execution)
  - Shared filters visible to all Recruiters within the same Company
