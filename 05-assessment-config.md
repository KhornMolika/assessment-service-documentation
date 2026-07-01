# ⚙ Assessment Configuration Reference

Assessments are highly configurable structures that dictate how question banks are delivered to participants.

## ✦ Execution Modes

### 1. Self-Paced Mode
Participants complete the assessment independently on their own time.
* **Navigation**: Configurable to allow backward navigation or strict forward-only progression.
* **Pacing**: Governed strictly by the overarching assessment timer.
* **Result Release**: Configurable (e.g., immediate release upon submission or manual release by an admin).

### 2. Real-Time Mode (Instructor-Led)
A synchronous environment where an instructor controls the pacing.
* **Navigation**: Locked. The host advances the slides for all participants simultaneously via WebSockets.
* **Scoring**: Immediate feedback is often rendered between rounds depending on the host's configuration.
* **State Management**: Managed via a Redis-backed Socket.io room.

## ✦ Global Settings

| Setting | Type | Description |
|---|---|---|
| `timeLimit` | Integer (seconds) | Maximum duration allowed for the entire session. |
| `passMark` | Float (percentage) | The threshold percentage required to pass. |
| `gradeLabels` | Array | Custom grading bands (e.g., A, B, C) mapped to score percentages. |
| `shuffleQuestions` | Boolean | Randomizes the order of question delivery per participant. |
| `shuffleOptions` | Boolean | Randomizes the order of multiple-choice options. |

## ✦ Execution Lifecycle & State Machine
The `Assessment` entity tracks states: `DRAFT` -> `PUBLISHED` -> `ARCHIVED`.
Only `DRAFT` assessments can have their `AssessmentQuestion` relationships modified. Once published, the question set becomes immutable to ensure all participants are graded against the exact same snapshot, even if the underlying `QuestionBank` is modified later.

## ✦ Real-Time Sync Mechanisms (Instructor-Led)
For Real-Time assessments, the backend utilizes `Socket.io`. 
* **State Propagation**: The Redis adapter ensures that if the Host is connected to Node A and Participant 1 is connected to Node B, the `ADVANCE_SLIDE` event emitted by the Host propagates across the pub/sub cluster.
* **Late Joiners**: The current active question index and leaderboard state are cached in Redis under the `sessionId`. When a participant joins late, they immediately receive the latest cached state to sync their UI instantly.
