# ⟳ API Reference

This document serves as an overview of the core REST API boundaries utilized by the Assessment Service client.

## ⚙ Core Resource Boundaries

The API is fully RESTful and versioned (e.g., `/api/v1/`).

### ✦ Assessments
* `POST /assessments` — Create a new assessment configuration.
* `GET /assessments` — Retrieve a paginated list of assessments.
* `GET /assessments/:id` — Retrieve full assessment details including configured question banks.
* `PATCH /assessments/:id/settings` — Update assessment behavior (timing, modes).
* `POST /assessments/:id/join` — Generate a participant session for an assessment.

### ✦ Runtime & Sessions
* `POST /runtime/sessions/start` — Initialize a participant's attempt and fetch the first batch of questions.
* `POST /runtime/sessions/:sessionId/answers` — Submit a response for a specific question round.
* `POST /runtime/sessions/:sessionId/submit` — Finalize the assessment and trigger grading pipelines.
* `GET /runtime/sessions/:sessionId/result` — Fetch the final graded transcript.

### ✦ Question Banks
* `POST /banks` — Create a new question bank.
* `GET /banks/:id` — Retrieve questions within a specific bank.
* `POST /banks/:id/questions` — Append a new question round to a bank.

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
