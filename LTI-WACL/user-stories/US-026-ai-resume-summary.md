PATH: /user-stories/US-026-ai-resume-summary.md

# 🧾 User Story: AI-Powered CV Summary Generation

## 🆔 Metadata
- ID: US-026
- Feature: AI Assistance
- Priority: High
- Status: Draft
- Owner: PM

## 🎯 User Story
**As the** System (triggered automatically on CV upload or on demand by a Recruiter)  
**I want** to generate an AI-powered summary of a candidate's CV  
**So that** Recruiters can quickly grasp the candidate's profile without reading the full document, accelerating the initial screening stage

## 🧠 Business Context
- Related Use Case(s): UC-29 Resumir CV automáticamente
- Entities involved: Candidate, AiGenerationRun, AiSuggestion, AiSuggestionFeedback, User, Company
- Business Rules:
  - AI summary generation requires `Candidate.resumeUrl` to be non-null and the document to be processable.
  - The AI feature must be enabled for the tenant.
  - `AiGenerationRun` is created with `purpose = ResumeSummary` and `contextType = Candidate`.
  - On success, one `AiSuggestion` with `suggestionType = ResumeSummary` is created.
  - `Candidate.aiSummary` is updated with the summary text, `Candidate.aiSummaryStatus = Ready`, and `Candidate.aiSummaryUpdatedAt` is set — all atomically when `AiGenerationRun.status = Succeeded`.
  - The summary is labeled as AI-generated and must never be presented as factual truth.
  - If the CV changes (new `resumeUrl`), the previous summary is marked `Stale` and a new run is triggered.
  - Absence of a summary does not block the main ATS workflow.
- Assumptions:
  - CV processing (extracting text from PDF/DOCX) is handled by the AI Gateway before passing to the LLM.
  - The generation is asynchronous; the Recruiter sees a "Generating…" indicator until the result is ready.
  - Regeneration can be requested manually by the Recruiter from the candidate profile.
- Dependencies:
  - Candidate must have a non-null `resumeUrl`.
  - AI Gateway service must be available.

## ✅ Acceptance Criteria (BDD + Edge Cases)

### 🎯 Happy Path
- **Given** a Candidate with a valid `resumeUrl` and AI assistance enabled for the Company  
  **When** the system processes a new CV upload or the Recruiter manually requests a summary  
  **Then** an `AiGenerationRun` is created with `purpose = ResumeSummary`, the CV is sent to the AI Gateway, a summary is received, an `AiSuggestion` with `suggestionType = ResumeSummary` is persisted, `Candidate.aiSummary` is populated, `Candidate.aiSummaryStatus = Ready`, and the summary is displayed in the candidate profile with an "AI-generated" label

### ⚠️ Edge Cases

#### Case 1: CV document cannot be processed (unreadable format)
- **Given** a Candidate's CV file is in an unsupported or corrupted format  
  **When** the AI Gateway attempts to extract text  
  **Then** `AiGenerationRun.status = Failed`, `Candidate.aiSummaryStatus = Failed`, and the Recruiter sees a "Summary could not be generated — document unreadable" message with an option to re-upload a supported format  
- Handling:
  - `Candidate.aiSummary` is not modified. `aiSummaryUpdatedAt` is updated to reflect the failed attempt timestamp.

#### Case 2: Candidate uploads a new CV after a summary exists
- **Given** a Candidate has `aiSummaryStatus = Ready` and uploads a new CV  
  **When** the system detects the `resumeUrl` change  
  **Then** `Candidate.aiSummaryStatus` is set to `Stale`, a new `AiGenerationRun` is triggered asynchronously, and the Recruiter sees "Summary is outdated — regenerating" until the new run completes  
- Handling:
  - Set `aiSummaryStatus = Stale` synchronously on CV update; regeneration is async.

#### Case 3: AI Gateway is unavailable
- **Given** the AI Gateway service is down when the summary is requested  
  **When** the generation attempt is made  
  **Then** `AiGenerationRun.status = Failed`, `Candidate.aiSummaryStatus = Failed`, and the Recruiter sees a "Summary unavailable — please retry later" message  
- Handling:
  - No retry logic in MVP. The Recruiter can manually retrigger from the profile view.

#### Case 4: Recruiter provides negative feedback on the summary
- **Given** the Recruiter reads the AI summary and finds it inaccurate  
  **When** the Recruiter marks the suggestion as rejected and provides feedback text  
  **Then** an `AiSuggestionFeedback` record is created with `decision = Rejected` and the feedback text; `Candidate.aiSummary` is not automatically cleared  
- Handling:
  - Feedback is captured for future model improvement. The summary remains visible until regenerated.

### 🚫 Validation Rules
- `Candidate.resumeUrl` must be non-null to trigger summary generation.
- `Candidate.aiSummary` is only updated when `AiGenerationRun.status = Succeeded`.
- `AiGenerationRun.provider` and `AiGenerationRun.modelVersion` are mandatory.
- The AI summary must always include an "AI-generated" label in the UI.
- `Candidate.aiSummaryStatus` transitions: `null → Pending → Ready | Failed | Stale`.
- Absence of `aiSummary` must not block any other ATS workflow.

## 📊 Business Impact
- KPI Affected: Screening time per candidate, Recruiter throughput, Time-to-first-decision
- Expected Impact: Reduces CV reading time from an average of 4 minutes to under 30 seconds per candidate for initial screening, enabling Recruiters to process 5–8x more candidates per hour
- Success Criteria:
  - AI summary generated and ready within 15 seconds of CV upload for 95% of cases
  - `aiSummaryStatus = Ready` on at least 90% of candidates with a valid `resumeUrl`
  - Zero summaries presented without the "AI-generated" label
