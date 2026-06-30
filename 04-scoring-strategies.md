# ⊞ Question Types & Scoring Strategies

The Assessment Service supports a wide variety of question formats, each possessing distinct validation, rendering, and scoring logics.

## ⚙ Supported Question Types

| Type | Description | Client Interaction | Grading Strategy |
|---|---|---|---|
| **Multiple Choice** | Standard selection from predefined options. | Radio / Checkbox | Deterministic |
| **Matching** | Pairing items from two distinct columns. | Drag & Drop | Deterministic (Partial) |
| **Fill in the Blank** | Text inputs embedded within a larger sentence structure. | Text Input | Deterministic (Partial) |
| **Rating** | Selection along a linear scale (e.g., 1-5). | Sliders / Stars | Deterministic |
| **True / False** | Binary selection. | Toggle / Button | Deterministic |
| **Short Answer / Essay** | Open-ended text responses. | Text Area | Qualitative (AI Evaluated) |

## ⚙ Scoring Methodologies

### ✦ Deterministic Scoring (Strict)
Used for True/False and standard Multiple Choice questions. The score awarded is binary: either full points (`1.0 * maxPoints`) or zero (`0.0`).

### ✦ Deterministic Scoring (Partial Credit)
Used for Matching and Fill-in-the-Blank. 
* Points are calculated proportionally based on the number of correct elements.
* *Formula:* `(Correct Elements / Total Elements) * Max Points`
* Precision is maintained using standard floating-point operations clamped to two decimal places to ensure clean rendering.

### ✦ Qualitative AI Scoring
Used for subjective formats like Essay.
* The participant's response is passed to the AI Grading pipeline.
* The AI returns a suggested score, reasoning, and key points missed/addressed.
* Administrators have the final authority to override AI-suggested scores.
