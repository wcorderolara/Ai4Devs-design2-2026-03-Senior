PATH: /user-stories/US-007-search-candidates.md

# 🧾 User Story: Search Candidates

## 🆔 Metadata
- ID: US-007
- Feature: Candidate Management
- Priority: High
- Status: Draft
- Owner: PM

## 🎯 User Story
**As a** Recruiter, Hiring Manager, or Admin  
**I want** to search and filter candidates by name, email, job opening, or application status  
**So that** I can quickly locate relevant candidates as the volume of applicants grows

## 🧠 Business Context
- Related Use Case(s): Buscar Candidato (UC7)
- Entities involved: Candidate, Application, JobOpening, PipelineStage, Company
- Business Rules:
  - Search results must be scoped to the authenticated user's Company.
  - Filters can be combined: name/email (text search), jobOpeningId, application status.
  - Results must include the Candidate's current Application stage per job opening.
  - Inactive or withdrawn candidates must be filterable but included in results by default.
- Assumptions:
  - Search is case-insensitive for name and email fields.
  - Pagination is applied to results (page size default: 20).
- Dependencies:
  - Candidates must exist in the system.

## ✅ Acceptance Criteria (BDD + Edge Cases)

### 🎯 Happy Path
- **Given** an authenticated Recruiter with multiple Candidates registered in their Company  
  **When** the Recruiter enters a name fragment in the search field and optionally selects a job opening filter  
  **Then** the system returns a paginated list of matching Candidates with their name, email, and current Application stage for the selected job opening

### ⚠️ Edge Cases

#### Case 1: No candidates match the search criteria
- **Given** a Recruiter searches with a name that does not match any Candidate  
  **When** the search is executed  
  **Then** the system returns an empty result set with a 200 OK response and a "No candidates found" message  
- Handling:
  - Return empty array with total count = 0. Do not return 404.

#### Case 2: Search with no filters applied
- **Given** a Recruiter submits the search form without any filter  
  **When** the search is executed  
  **Then** the system returns the first page of all Candidates in the Company  
- Handling:
  - Default to returning all Candidates paginated; no filter is a valid query.

#### Case 3: Search across multiple job openings
- **Given** a Recruiter searches by candidate name without filtering by job opening  
  **When** a Candidate has Applications in multiple job openings  
  **Then** the system returns the Candidate once, listing all their active Applications  
- Handling:
  - Deduplicate Candidate records in results; aggregate Applications per Candidate.

### 🚫 Validation Rules
- All results must be scoped to the authenticated user's `companyId`.
- Text search on `firstName`, `lastName`, and `email` must be case-insensitive.
- `jobOpeningId` filter must belong to the same Company if provided.
- `status` filter must match values from `Application.status` enum: `Active`, `Rejected`, `Hired`, `Withdrawn`.
- Pagination parameters `page` and `pageSize` must be positive integers; `pageSize` maximum is 100.

## 📊 Business Impact
- KPI Affected: Recruiter efficiency, Talent pool reuse rate
- Expected Impact: Reduces time to find and re-engage candidates from the existing pool, enabling faster hiring cycles
- Success Criteria:
  - Search results returned in under 1 second for up to 10,000 candidate records
  - Recruiters can locate a specific candidate with 2 or fewer filter interactions
