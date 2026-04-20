PATH: /user-stories/US-021-ai-rank-candidates.md

# 🧾 User Story: AI-Assisted Candidate Ranking

## 🆔 Metadata
- ID: US-021
- Feature: Pipeline & Tracking
- Priority: High
- Status: Draft
- Owner: PM

## 🎯 User Story
**As a** Recruiter  
**I want** to request an AI-generated ranking of candidates for a specific job opening  
**So that** I can prioritize which candidates to review first based on their fit with the job requirements, reducing manual screening time

## 🧠 Business Context
- Related Use Case(s): UC-24 Rankear candidatos automáticamente
- Entities involved: Application, JobOpening, Candidate, AiGenerationRun, AiSuggestion, AiSuggestionFeedback, User, Company
- Business Rules:
  - The AI ranking feature must be enabled for the tenant (Company).
  - The ranking applies to a specific JobOpening and evaluates all active Applications.
  - The ranking does NOT automatically change the stage or status of any Application.
  - Each ranking execution creates one `AiGenerationRun` with `purpose = CandidateRanking` and produces `AiSuggestion` records with `suggestionType = CandidateScore` and `CandidateRank`.
  - `Application.aiScore`, `Application.aiRank`, `Application.aiExplanation`, and `Application.aiScoredAt` are updated atomically when the ranking is applied.
  - The ranking result and explanation must only be visible to authorized users (Recruiter, HiringManager, Admin).
  - AI suggestions must be distinguishable from manual evaluations.
  - A failed AI execution (`AiGenerationRun.status = Failed`) must not update any Application fields.
- Assumptions:
  - The AI ranking is triggered on demand by the Recruiter, not automatically.
  - The JobOpening must have a title, description, and at least 2 active Applications for a meaningful ranking.
  - The LLM provider is accessed through an AI Gateway service (abstracted, provider-agnostic).
- Dependencies:
  - JobOpening must exist and be Open (US-001).
  - Applications must exist for the JobOpening (US-004 or US-005).
  - AI Gateway service must be configured for the tenant.

## ✅ Acceptance Criteria (BDD + Edge Cases)

### 🎯 Happy Path
- **Given** an authenticated Recruiter, an Open JobOpening with 5 active Applications, and AI assistance enabled for the Company  
  **When** the Recruiter requests a candidate ranking for the job opening  
  **Then** the system creates an `AiGenerationRun` with `purpose = CandidateRanking`, sends the job description and candidate profiles to the AI Gateway, receives scores and explanations, creates `AiSuggestion` records per Application, updates `Application.aiScore`, `Application.aiRank`, `Application.aiExplanation`, and `Application.aiScoredAt`, and presents the ranked list with brief explanations to the Recruiter

### ⚠️ Edge Cases

#### Case 1: JobOpening has insufficient information for ranking
- **Given** a JobOpening with only a title but no description or requirements  
  **When** the Recruiter requests a ranking  
  **Then** the system returns a warning indicating the ranking confidence will be low, and the `AiGenerationRun` records this as a partial execution  
- Handling:
  - Proceed with available data but flag the result with a "Low confidence" indicator. Do not block the request.

#### Case 2: AI Gateway returns an error or timeout
- **Given** the AI Gateway is unavailable or returns an error  
  **When** the ranking request is processed  
  **Then** `AiGenerationRun.status` is set to `Failed`, no Application fields are modified, and the Recruiter receives a clear error message with a "Retry" option  
- Handling:
  - Fail gracefully; Applications must remain in their current state. Log the failure in `AiGenerationRun.responseText`.

#### Case 3: Re-ranking after new applications arrive
- **Given** a JobOpening that already has a ranking from a previous run  
  **When** the Recruiter requests a new ranking  
  **Then** a new `AiGenerationRun` is created, new `AiSuggestion` records are produced, previous suggestions are marked `Superseded`, and Application fields are updated with the new scores  
- Handling:
  - Prior `AiSuggestion` records with `suggestionType = CandidateRank` for this JobOpening are set to `status = Superseded` before new ones are written.

#### Case 4: Recruiter provides feedback on a suggestion
- **Given** a Recruiter reviews the ranked list and disagrees with a score for a candidate  
  **When** the Recruiter marks the suggestion as rejected  
  **Then** an `AiSuggestionFeedback` record is created with `decision = Rejected` and the Recruiter's optional feedback text; the Application's `aiRank` is not automatically changed  
- Handling:
  - Feedback is captured for model improvement; it does not auto-correct the ranking in MVP.

### 🚫 Validation Rules
- AI ranking can only be triggered for a JobOpening with `status = Open`.
- At least 1 active Application must exist for the ranking to execute.
- `AiGenerationRun.provider` and `AiGenerationRun.modelVersion` are mandatory and set by the AI Gateway.
- Application fields (`aiScore`, `aiRank`, `aiExplanation`, `aiScoredAt`) are only updated on `AiGenerationRun.status = Succeeded`.
- The AI ranking does not modify `Application.status` or `Application.currentStageId`.
- Role must be Recruiter, HiringManager, or Admin to view ranking results.

## 📊 Business Impact
- KPI Affected: Time-to-screening-decision, Screening accuracy, Recruiter throughput per vacancy
- Expected Impact: Reduces manual initial screening time by up to 60% for high-volume job openings by surfacing the best-fit candidates first
- Success Criteria:
  - Ranking results delivered within 10 seconds for up to 50 candidates
  - `aiScore` and `aiRank` populated on 100% of active Applications after a successful run
  - Zero Application stage changes caused by AI ranking alone
