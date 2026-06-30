# ✦ AI Grading Provider Guide

The Assessment Service employs Large Language Models (LLMs) to automate the evaluation of subjective question formats, significantly reducing administrative overhead.

## ⚙ Integration Architecture

The system utilizes an abstraction layer over the OpenAI API, allowing for future expansion to other providers (e.g., Anthropic, Gemini).

```mermaid
%%{init: { 'theme': 'base', 'themeVariables': { 'textColor': '#0ea5e9', 'messageTextColor': '#0ea5e9', 'actorTextColor': '#0ea5e9', 'noteTextColor': '#0f172a', 'lineColor': '#38bdf8' } } }%%
sequenceDiagram
    autonumber
    actor Admin
    participant Frontend as Next.js Client
    participant Backend as NestJS API & BullMQ
    participant LLM as OpenAI GPT-4 API
    
    Backend->>LLM: Submit Prompt (Question Context + Rubric + Participant Answer)
    activate LLM
    Note right of LLM: AI processes prompt enforcing strict JSON schema
    LLM-->>Backend: Return JSON (suggestedScore, reasoning, keyPoints)
    deactivate LLM
    
    activate Backend
    Backend->>Backend: Validate JSON Schema via Zod
    Backend->>Backend: Persist AI grading insights to AnswerEntry
    Backend-->>Frontend: Broadcast updated entry via WebSockets / REST
    deactivate Backend
    
    Frontend->>Admin: Present Suggested Score, Reasoning & Key Points
    
    opt Administrator Override
        Admin->>Frontend: Adjust 'scoreAwarded' value manually
        Frontend->>Backend: PATCH /entries/:id/review
        Backend->>Backend: Save explicit manual score
    end
```

## ⚙ Prompt Engineering Strategy

The system constructs highly specific prompts to enforce deterministic JSON outputs. The prompt includes:
1. **The Question Context**: The original text and any provided attachments.
2. **The Ideal Answer/Rubric**: Criteria by which the answer should be judged.
3. **The Participant's Answer**: The raw text submitted by the user.
4. **The Output Schema**: Strict instructions to return JSON containing:
   * `suggestedScore` (Float)
   * `reasoning` (String)
   * `keyPointsAddressed` (Array of Strings)
   * `keyPointsMissed` (Array of Strings)
   * `flagForReview` (Boolean)

## ⚙ Fallbacks & Limitations
* **Latency**: AI grading is performed asynchronously via background job queues (BullMQ) to prevent blocking the main thread.
* **Hallucinations**: AI outputs are strictly treated as *suggestions*. Administrators always have the final authority to adjust the `scoreAwarded` and override the AI's recommendation.
