# ⟳ API Reference

This document serves as an exhaustive overview of the core REST API boundaries utilized by the Assessment Service client.

## ⚙ Core Resource Boundaries

The API is fully RESTful and versioned under `/api/v1/`.

### ✦ Health
* `GET /api/v1/health` — Health check endpoint

### ✦ Realtime
* `POST /api/v1/runtime/real-time/{assessmentId}/start` — Starts a real-time assessment session

### ✦ Assessments
* `GET /api/v1/assessments` — Get a paginated list of all assessments globally
* `GET /api/v1/topics/{topicId}/assessments` — Get a paginated list of assessments by topic ID
* `POST /api/v1/topics/{topicId}/assessments` — Create an assessment for a given topic
* `GET /api/v1/assessments/{id}` — Get an assessment by ID
* `PATCH /api/v1/assessments/{id}` — Update an assessment by ID (DRAFT only)
* `DELETE /api/v1/assessments/{id}` — Delete an assessment by ID
* `POST /api/v1/assessments/{id}/publish` — Publish an assessment by ID
* `POST /api/v1/assessments/{id}/archive` — Archive an assessment by ID
* `GET /api/v1/assessments/{id}/questions` — Get questions for an assessment by ID
* `POST /api/v1/assessments/{id}/questions` — Add questions to an assessment (DRAFT only)
* `PUT /api/v1/assessments/{id}/questions` — Replace all questions in an assessment (DRAFT only)
* `DELETE /api/v1/assessments/{id}/questions/{assessmentQuestionId}` — Remove a question from an assessment (DRAFT only)
* `GET /api/v1/assessments/{id}/settings` — Get settings of an assessment by ID
* `PATCH /api/v1/assessments/{id}/settings` — Update settings of an assessment by ID
* `GET /api/v1/assessments/{id}/participants` — Get participants of an assessment by ID
* `POST /api/v1/assessments/{id}/join` — Public endpoint to join a real-time assessment
* `DELETE /api/v1/assessments/{id}/participants/{participantId}` — Remove a participant from an assessment
* `POST /api/v1/assessments/{sessionId}/recalculate` — Recalculate assessment session after manual grading/override
* `PATCH /api/v1/assessments/{sessionId}/entries/{entryId}/review` — Save manual review score for an answer entry
* `POST /api/v1/assessments/{sessionId}/entries/{entryId}/ai-grading/retry` — Retry AI grading for an assessment entry

### ✦ Questions
* `GET /api/v1/questions` — Get a paginated list of all questions globally
* `GET /api/v1/questions/{id}` — Get a question by ID
* `PATCH /api/v1/questions/{id}` — Update a question by ID
* `DELETE /api/v1/questions/{id}` — Delete a question by ID
* `GET /api/v1/topics/{topicId}/questions` — List all questions for a topic
* `POST /api/v1/topics/{topicId}/questions` — Create a new question in a topic

### ✦ Topics
* `POST /api/v1/topics` — Create a new topic
* `GET /api/v1/topics` — List all topics
* `GET /api/v1/topics/{id}` — Get a topic by ID
* `PATCH /api/v1/topics/{id}` — Update a topic by ID
* `DELETE /api/v1/topics/{id}` — Delete a topic by ID

### ✦ Question Banks
* `GET /api/v1/banks` — Get a paginated list of all banks globally
* `GET /api/v1/banks/{id}` — Get a question bank by ID
* `PATCH /api/v1/banks/{id}` — Update a question bank by ID
* `DELETE /api/v1/banks/{id}` — Delete a question bank by ID
* `GET /api/v1/banks/{id}/questions` — List all questions in a question bank
* `POST /api/v1/banks/{id}/questions` — Add questions to a bank
* `DELETE /api/v1/banks/{id}/questions/{questionId}` — Remove a question from a bank
* `POST /api/v1/topics/{topicId}/banks` — Create a new question bank in a topic
* `GET /api/v1/topics/{topicId}/banks` — List all question banks for a topic

### ✦ Participants
* `GET /api/v1/participants` — Get a paginated list of all participants
* `POST /api/v1/participants` — Create a new participant
* `GET /api/v1/participants/{id}` — Get a single participant detail by ID
* `PATCH /api/v1/participants/{id}` — Update a participant by ID
* `DELETE /api/v1/participants/{id}` — Soft delete a participant by ID

### ✦ Runtime
* `POST /api/v1/runtime/sessions/start` — Starts an assessment session
* `POST /api/v1/runtime/sessions/{sessionId}/answers` — Saves or updates an answer
* `POST /api/v1/runtime/sessions/{sessionId}/submit` — Submits a completed session
* `GET /api/v1/runtime/sessions/{sessionId}/result` — Returns result based on assessment showResults setting

### ✦ Clients
* `POST /api/v1/clients` — [Admin] Provision new client — secret shown once
* `GET /api/v1/clients` — [Admin] List all clients
* `GET /api/v1/clients/me` — Get own client profile
* `PATCH /api/v1/clients/me` — Update own client configuration
* `GET /api/v1/clients/{id}` — [Admin] Get client by ID
* `PATCH /api/v1/clients/{id}` — [Admin] Update any client
* `POST /api/v1/clients/{id}/rotate-secret` — [Admin] Rotate client secret
* `PATCH /api/v1/clients/{id}/suspend` — [Admin] Suspend a client
* `PATCH /api/v1/clients/{id}/activate` — [Admin] Activate a client

### ✦ Auth
* `POST /api/v1/auth/token` — OAuth2 client credentials grant — returns Bearer token

### ✦ Reports
* `GET /api/v1/assessments/{assessmentId}/sessions/{sessionId}/report` — Get full session report for one participant attempt
* `GET /api/v1/sessions/{sessionId}/report` — Get full session report by session id
* `GET /api/v1/assessments/{assessmentId}/report` — Get aggregated report for all participants in an assessment
* `GET /api/v1/participants/{participantId}/report` — Get cross-assessment report for one participant

## ⚙ Standard Payload Structures

### ✦ Success Response
```json
{
  "data": {
    "id": "uuid",
    ...
  },
  "meta": {
    "timestamp": "2026-06-30T12:00:00Z"
  }
}
```

### ✦ Error Response
```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "The provided configuration is invalid.",
    "details": ["'points' must be a positive number"]
  }
}
```
