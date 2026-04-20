PATH: /user-stories/US-028-comment-on-candidate.md

# 🧾 User Story: Comment on a Candidate

## 🆔 Metadata
- ID: US-028
- Feature: Collaboration
- Priority: High
- Status: Draft
- Owner: PM

## 🎯 User Story
**As a** Recruiter, Hiring Manager, or Interviewer  
**I want** to post collaborative comments on a candidate's profile or application  
**So that** the hiring team can share context, questions, and observations in one place without relying on external chat tools

## 🧠 Business Context
- Related Use Case(s): UC-31 Comentar candidato
- Entities involved: CommentThread, Comment, ActivityEvent, Candidate, Application, User, Company
- Business Rules:
  - Comments are stored within a `CommentThread` associated to either a Candidate (`contextType = Candidate`) or an Application (`contextType = Application`).
  - Comments are always internal; they must never be visible to the Candidate.
  - The `Comment.authorUserId` and `Comment.createdAt` are immutable after creation.
  - A `Comment.body` must not be empty or consist only of whitespace.
  - Soft delete is supported via `Comment.deletedAt`; deleted comments are hidden from the UI but preserved in the database.
  - An `ActivityEvent` of type `Commented` is recorded when a Comment is posted.
  - `CommentThread` carries visibility through `Comment.visibilityScope`; each comment can be `Internal`, `HiringTeam`, or `AdminOnly`.
  - A `CommentThread` can be `Resolved` by authorized users, but comments within a resolved thread remain readable.
- Assumptions:
  - A new `CommentThread` is created automatically if one does not yet exist for the given context (Candidate or Application).
  - The Recruiter cannot edit a posted comment in MVP; they can only soft-delete it.
- Dependencies:
  - Candidate or Application must exist within the same Company.

## ✅ Acceptance Criteria (BDD + Edge Cases)

### 🎯 Happy Path
- **Given** an authenticated Recruiter and an existing Application  
  **When** the Recruiter types a non-empty comment in the application's comment section and submits  
  **Then** a `CommentThread` is created (if none exists for the Application), a `Comment` record is persisted with `body`, `authorUserId`, `createdAt`, `visibilityScope = Internal`, and the comment appears immediately in the collaboration section; an `ActivityEvent` of type `Commented` is recorded for the Application

### ⚠️ Edge Cases

#### Case 1: Empty or whitespace-only comment
- **Given** a Recruiter opens the comment input  
  **When** the Recruiter submits with an empty or whitespace-only `body`  
  **Then** the system returns a 400 validation error indicating the comment body cannot be empty  
- Handling:
  - Server-side validation: strip whitespace and reject if resulting string is empty.

#### Case 2: Comment posted in a resolved thread
- **Given** a `CommentThread` with `status = Resolved`  
  **When** a Recruiter attempts to post a new comment to that thread  
  **Then** the comment is rejected with a 422 error indicating the thread is resolved; the Recruiter must reopen the thread first  
- Handling:
  - Check `CommentThread.status` before inserting a new Comment. Surface a "Reopen thread" action in the UI.

#### Case 3: Soft-deleting a comment
- **Given** a Recruiter posted a comment and wants to remove it  
  **When** the Recruiter selects "Delete" on their own comment  
  **Then** `Comment.deletedAt` is set to now, the comment body is replaced with "Deleted" in the UI, and the record is preserved in the database  
- Handling:
  - Only the comment's author or an Admin may soft-delete a comment. Enforce this at the API layer.

#### Case 4: Unauthorized visibility access
- **Given** a Comment with `visibilityScope = AdminOnly`  
  **When** a Recruiter loads the comment section  
  **Then** the AdminOnly comment is not returned in the API response for the Recruiter's role  
- Handling:
  - Filter `Comment` records by `visibilityScope` based on the viewer's role before returning results.

### 🚫 Validation Rules
- `body` must be non-empty after whitespace trimming.
- `contextType` must be one of: `Application`, `Candidate`.
- `contextId` must reference a valid Application or Candidate within the authenticated user's Company.
- `authorUserId` is set server-side from the authenticated session; clients cannot override it.
- Comments on resolved threads are blocked until the thread is reopened.
- `deletedAt` comments are excluded from UI rendering but included in audit exports.
- Role must be Recruiter, HiringManager, or Interviewer.

## 📊 Business Impact
- KPI Affected: Team collaboration efficiency, Decision speed, Dependency on external tools
- Expected Impact: Centralizes all candidate-specific team discussion within the ATS, reducing context-switching by eliminating the need for Slack/email threads per candidate
- Success Criteria:
  - Comments visible to all authorized team members within the same Company in real time
  - 100% of posted comments generate a corresponding `ActivityEvent` of type `Commented`
  - Zero comments surfaced outside their `visibilityScope` boundary
