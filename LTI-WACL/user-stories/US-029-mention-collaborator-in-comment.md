PATH: /user-stories/US-029-mention-collaborator-in-comment.md

# 🧾 User Story: Mention Collaborator in a Comment

## 🆔 Metadata
- ID: US-029
- Feature: Collaboration
- Priority: High
- Status: Draft
- Owner: PM

## 🎯 User Story
**As a** Recruiter, Hiring Manager, or Interviewer  
**I want** to mention one or more colleagues in a comment using an @-mention  
**So that** the mentioned collaborators are aware of the discussion and can respond without missing important context about a candidate

## 🧠 Business Context
- Related Use Case(s): UC-32 Mencionar colaborador en comentario
- Entities involved: Comment, CommentMention, UserNotification, CommentThread, User, Company
- Business Rules:
  - Mentions are only valid for active Users (`status = Active`) belonging to the same Company.
  - Each `CommentMention` references one `mentionedUserId` per comment; the same user cannot be mentioned twice in one comment.
  - A mention does not create a formal assignment or ownership transfer; it is a notification mechanism only.
  - Upon mention creation, a `UserNotification` of type `Mention` is generated for each mentioned user.
  - `CommentMention` records are preserved even if the `Comment` is soft-deleted.
  - Mentions are traced individually per `CommentMention` record.
  - `notificationStatus` on `CommentMention` tracks whether the notification has been delivered and read.
- Assumptions:
  - The @-mention UI autocompletes from active users in the same Company.
  - A `UserNotification` is surfaced in the recipient's notification center within the ATS.
  - Email notification for mentions is out of scope for MVP (internal notification only).
- Dependencies:
  - A Comment must exist or be created simultaneously (US-028).
  - Mentioned users must be active within the same Company.

## ✅ Acceptance Criteria (BDD + Edge Cases)

### 🎯 Happy Path
- **Given** an authenticated Recruiter composing a comment on an Application  
  **When** the Recruiter types "@HiringManager" and selects a valid active user from the autocomplete, then submits the comment  
  **Then** a `Comment` is persisted, a `CommentMention` record is created with `mentionedUserId` and `notificationStatus = Pending`, a `UserNotification` of type `Mention` is created for the mentioned user, and the comment body renders the mention visually highlighted for all authorized readers

### ⚠️ Edge Cases

#### Case 1: Mentioning an inactive user
- **Given** a Recruiter types the name of a deactivated user in the mention field  
  **When** the autocomplete or submit is processed  
  **Then** the system returns a 422 validation error indicating the mentioned user is not active  
- Handling:
  - Exclude inactive users from the autocomplete list. Also enforce server-side on submit.

#### Case 2: Mentioning a user from another Company
- **Given** a Recruiter provides a `mentionedUserId` that belongs to a different Company  
  **When** the comment is submitted  
  **Then** the system returns a 422 error indicating the user is not a member of this tenant  
- Handling:
  - Validate `mentionedUserId.companyId = authenticated user's companyId` before persisting the `CommentMention`.

#### Case 3: Duplicate mention of the same user in one comment
- **Given** a Recruiter mentions the same user twice in one comment  
  **When** the comment is submitted  
  **Then** only one `CommentMention` is created for that user; duplicates are silently deduplicated  
- Handling:
  - Enforce unique constraint on `(commentId, mentionedUserId)`. Deduplicate at the application layer before persisting.

#### Case 4: Mentioned user sees notification but comment thread is resolved
- **Given** a comment mention targets a user and the thread is subsequently resolved  
  **When** the mentioned user clicks the notification  
  **Then** the user is taken to the comment thread, which is shown in "Resolved" state, with the comment still readable  
- Handling:
  - Resolved threads are always readable; the notification links directly to the comment anchor. Thread status is displayed prominently.

### 🚫 Validation Rules
- `mentionedUserId` must reference a User with `status = Active` and `companyId` matching the commenter's Company.
- Unique constraint on `(commentId, mentionedUserId)`.
- A mention does not transfer ownership or create a task assignment.
- `UserNotification.createdAt` is set server-side at mention creation time.
- `notificationStatus` transitions: `Pending → Delivered → Read`.
- Role must be Recruiter, HiringManager, or Interviewer.

## 📊 Business Impact
- KPI Affected: Inter-team coordination speed, Decision latency, Candidate review cycle time
- Expected Impact: Replaces ad-hoc Slack/email pings with in-context @-mentions, reducing average response-to-feedback time by 30–50% and keeping all communication traceable
- Success Criteria:
  - Mentioned users receive a `UserNotification` within 2 seconds of comment submission
  - Zero `CommentMention` records created for inactive or cross-tenant users
  - 100% of mention-triggered notifications link directly to the associated comment
