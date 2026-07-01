# ⛨ Question Types & Scoring Strategies

The Assessment Service utilizes a highly extensible `Grading Engine` module within the NestJS backend to evaluate a wide array of question formats.

## ⚙ Supported Question Formats

| Type Enum | Description | Grading Strategy |
|---|---|---|
| `MULTIPLE_CHOICE` | Single correct answer from a list of options. | Exact Match (Binary). |
| `MULTIPLE_RESPONSE` | Multiple correct answers from a list of options (checkboxes). | Partial Scoring available. |
| `TRUE_FALSE` | Binary boolean choice. | Exact Match (Binary). |
| `MATCHING` | Draggable pairs connecting items in Column A to Column B. | Partial Scoring available. |
| `SHORT_ANSWER` | Brief text input. | Regex/Exact Match or AI Evaluation. |
| `ESSAY` | Long-form markdown text input. | Manual Review or AI Evaluation. |

## ⚙ The Grading Pipeline

When an `AnswerSheet` is submitted via `POST /runtime/sessions/:sessionId/submit`, the backend executes the following pipeline:

1. **Snapshot Retrieval**: The system pulls the immutable `questionSnapshot` from the `AssessmentQuestion` record. This ensures that if an admin edits a question bank *after* a participant starts, the participant's grading logic remains tied to the original version they saw.
2. **Synchronous Grading**: The `Grading Engine` iterates over all objective questions (`MULTIPLE_CHOICE`, `TRUE_FALSE`, `MATCHING`).
3. **Asynchronous Dispatch**: Any subjective questions (`ESSAY`) are grouped and pushed to the BullMQ queue (`ai-grading-queue`).
4. **Finalization**: Once all queue jobs complete, the `AnswerSheet` status transitions to `GRADED`, the `totalScore` is calculated, and Webhooks are dispatched.

## ⚙ Partial Scoring Logic

For complex formats like `MULTIPLE_RESPONSE` and `MATCHING`, the system supports partial credit to reward partial knowledge.

**Example: MULTIPLE_RESPONSE**
* **Total Points**: 10
* **Correct Options**: A, B, C (3 total)
* **Participant Selected**: A, B, D
* **Calculation**:
  * Correct selections (A, B) = +2
  * Incorrect selections (D) = -1 (Penalty to prevent guessing all options)
  * Net correct = 1 out of 3.
  * Score Awarded = `(1 / 3) * 10 = 3.33 points`.
* *Note: The final score floor is 0; negative scores are not awarded.*

## ⚙ Manual Overrides

Regardless of the automated grading strategy (Objective or AI-driven), the system architecture allows administrators to perform manual overrides. 
Using the `PATCH /api/v1/assessments/{sessionId}/entries/{entryId}/review` endpoint, an admin can manually inject a `scoreAwarded` value, which forces the `AnswerSheet` to recalculate its totals and immediately update the participant's transcript.
