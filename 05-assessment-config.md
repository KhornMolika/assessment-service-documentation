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
