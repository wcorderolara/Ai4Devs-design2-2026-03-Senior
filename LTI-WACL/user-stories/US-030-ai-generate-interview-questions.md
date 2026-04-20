PATH: /user-stories/US-030-ai-generate-interview-questions.md

# 🧾 User Story: AI-Generated Interview Questions

## 🆔 Metadata
- ID: US-030
- Feature: AI Assistance
- Priority: Medium
- Status: Draft
- Owner: PM

## 🎯 User Story
**As an** Interviewer or Recruiter  
**I want** to request AI-suggested interview questions tailored to the candidate's profile and the job opening's requirements  
**So that** I can prepare more relevant and consistent interviews faster, without spending time crafting questions from scratch

## 🧠 Business Context
- Related Use Case(s): UC-33 Generar preguntas de entrevista por IA
- Entities involved: AiGenerationRun, AiSuggestion, AiSuggestionFeedback, Candidate, JobOpening, Application, User, Company
- Business Rules:
  - The AI question generation feature must be enabled for the tenant.
  - Question generation requires sufficient context: at minimum a Candidate profile and a JobOpening description.
  - One `AiGenerationRun` is created with `purpose = InterviewQuestions` and `contextType = Interview` (or `Application` if no Interview entity is linked yet).
  - One `AiSuggestion` with `suggestionType = InterviewQuestionSet` is created per successful run, containing the question set in `contentJson`.
  - Suggested questions must be clearly labeled as AI-generated; they do not replace the Interviewer's judgment.
  - Suggested and adopted questions must remain distinguishable in the record.
  - The Interviewer may select, edit, or discard suggested questions; this feedback is captured in `AiSuggestionFeedback`.
  - A failed generation run (`AiGenerationRun.status = Failed`) must not surface partial results.
- Assumptions:
  - The request is triggered on demand by the Interviewer or Recruiter from the application or interview preparation view.
  - `contentJson` contains a structured list of question objects (question text, category, difficulty).
  - The LLM is accessed through the AI Gateway; the ATS does not call the LLM provider directly.
- Dependencies:
  - Candidate must have a `resumeUrl` or sufficient profile data.
  - JobOpening must have a title and description.
  - AI Gateway service must be configured for the Company.

## ✅ Acceptance Criteria (BDD + Edge Cases)

### 🎯 Happy Path
- **Given** an authenticated Interviewer, an Application with a Candidate profile (including CV) and an Open JobOpening with a description, and AI assistance enabled  
  **When** the Interviewer requests AI-generated interview questions  
  **Then** an `AiGenerationRun` is created with `purpose = InterviewQuestions`, the candidate profile and job description are sent to the AI Gateway, a question set is received, an `AiSuggestion` with `suggestionType = InterviewQuestionSet` is created with the questions in `contentJson`, and the suggested questions are displayed to the Interviewer with a visible "AI-generated" label

### ⚠️ Edge Cases

#### Case 1: Insufficient context for generation
- **Given** a Candidate has no CV and the JobOpening has no description  
  **When** the Interviewer requests AI questions  
  **Then** the system returns a warning indicating insufficient context, `AiGenerationRun` is not created, and the Interviewer is prompted to enrich the candidate profile or job description  
- Handling:
  - Validate that at least one of `Candidate.resumeUrl` (non-null) or a non-empty JobOpening description exists before triggering the generation.

#### Case 2: AI Gateway returns no questions
- **Given** the AI Gateway processes the request but returns an empty question set  
  **When** the generation completes  
  **Then** `AiGenerationRun.status = Succeeded`, but the `AiSuggestion.contentJson` contains an empty array; the UI displays "No questions generated — please retry"  
- Handling:
  - An empty question set is a valid (if unhelpful) response. Do not mark the run as Failed. Surface clearly in the UI.

#### Case 3: Interviewer selects and modifies a suggested question
- **Given** an Interviewer views the generated question set  
  **When** the Interviewer edits one question before adopting it  
  **Then** an `AiSuggestionFeedback` record is created with `decision = EditedBeforeAccept` and the edited question text in `feedbackText`  
- Handling:
  - Capture the edit decision for audit and model improvement. The original suggestion in `AiSuggestion.contentJson` is not modified.

#### Case 4: Multiple runs for the same Application
- **Given** a previous question set exists as an `AiSuggestion` for the same Application  
  **When** the Interviewer requests a new set  
  **Then** a new `AiGenerationRun` is created, a new `AiSuggestion` is produced, and the previous `AiSuggestion` is marked `status = Superseded`  
- Handling:
  - Previous suggestions are superseded, not deleted. Both remain visible in audit history.

### 🚫 Validation Rules
- At minimum one of `Candidate.resumeUrl` or a non-empty `JobOpening.description` must exist.
- `AiGenerationRun.provider` and `AiGenerationRun.modelVersion` are mandatory at run creation.
- `AiSuggestion.contentJson` must conform to the `InterviewQuestionSet` contract (array of question objects).
- Suggested questions must never be surfaced without an "AI-generated" label in the UI.
- `AiGenerationRun.status = Failed` must not produce any `AiSuggestion` record.
- Role must be Interviewer or Recruiter to trigger question generation.

## 📊 Business Impact
- KPI Affected: Interview preparation time, Interview quality consistency, Interviewer onboarding speed
- Expected Impact: Reduces interview preparation time by 50–70% for first-time interviewers; improves question relevance and consistency across interviewers for the same role
- Success Criteria:
  - AI question suggestions delivered within 10 seconds for 95% of requests
  - 100% of generated question sets labeled as AI-generated in the UI
  - Interviewer feedback (accept/reject/edit) captured for at least 80% of surfaced question sets
