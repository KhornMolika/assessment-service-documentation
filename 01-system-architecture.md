# ❖ System Architecture Overview

This document outlines the architectural blueprint of the Assessment Service, detailing the interactions between the client, backend, and external dependencies.

## ⚙ High-Level Architecture

The system is designed as a decoupled, multi-tier application leveraging modern web technologies for high performance, real-time interactivity, and scalable grading capabilities.

```mermaid
%%{init: { 'theme': 'base', 'themeVariables': { 'textColor': '#0ea5e9', 'lineColor': '#38bdf8' } } }%%
flowchart TB
    subgraph User Tier
        Browser([Web Browser])
    end

    subgraph Presentation Tier
        Client[Next.js Client App]
    end

    subgraph Application Tier
        API[NestJS API Gateway]
        Sockets[WebSocket Server]
        Workers[Background Workers]
    end

    subgraph Data & Services Tier
        DB[(PostgreSQL 16)]
        Cache[(Redis 7)]
        LLM{DeepSeek API}
    end

    Browser -- "HTTPS (REST)" --> Client
    Browser -- "WSS (Real-time)" --> Sockets
    
    Client -- "Internal API / TRPC" --> API
    API -- "Pub/Sub" --> Sockets
    API -- "Queue Jobs" --> Workers
    
    API -- "TypeORM" --> DB
    Workers -- "TypeORM" --> DB
    
    API -- "BullMQ / Caching" --> Cache
    Workers -- "REST API" --> LLM
```

### ✦ Core Components
* **Frontend (Next.js 16)**: Delivers a responsive, server-side rendered application. Utilizes TailwindCSS and customized Shadcn components for premium aesthetics.
* **Backend (NestJS 11)**: Manages business logic, real-time sessions, and asynchronous background tasks.
* **Database (PostgreSQL 16)**: Primary persistent storage for clients, assessments, configurations, and result transcripts.
* **Cache & Message Broker (Redis 7)**: Handles WebSocket state distribution across horizontally scaled backend nodes and manages background job queues (BullMQ).
* **AI Evaluation Engine**: Interfaces with OpenAI to provide automated qualitative grading for subjective assessment types.

## ⛨ Entity Relationship Diagram (ERD)

The core relational data model is designed to support multi-tenancy and complex assessment lifecycles.

```mermaid
%%{init: { 'theme': 'base', 'themeVariables': { 'textColor': '#0ea5e9', 'lineColor': '#38bdf8', 'primaryTextColor': '#0ea5e9' } } }%%
erDiagram
    CLIENT ||--o{ TOPIC : owns
    TOPIC ||--o{ QUESTION : contains
    TOPIC ||--o{ QUESTION_BANK : contains
    TOPIC ||--o{ ASSESSMENT : contains

    QUESTION_BANK ||--o{ QUESTION_BANK_QUESTION : has
    QUESTION ||--o{ QUESTION_BANK_QUESTION : "linked via"

    ASSESSMENT ||--|| ASSESSMENT_SETTING : "configured by"
    ASSESSMENT ||--o{ ASSESSMENT_QUESTION : includes
    QUESTION ||--o{ ASSESSMENT_QUESTION : "snapshotted in"
    ASSESSMENT ||--o{ ASSESSMENT_PARTICIPANT : assigns
    PARTICIPANT ||--o{ ASSESSMENT_PARTICIPANT : "enrolled via"

    ASSESSMENT ||--o{ ANSWER_SHEET : "sessions for"
    ASSESSMENT_PARTICIPANT ||--o{ ANSWER_SHEET : starts
    ANSWER_SHEET ||--o{ ANSWER_ENTRY : records
    ASSESSMENT_QUESTION ||--o{ ANSWER_ENTRY : "answered by"
    ANSWER_ENTRY ||--o{ AI_GRADING_JOB : "graded by"

    ASSESSMENT_QUESTION {
        uuid id PK
        uuid assessmentId FK
        uuid questionId FK
        int order
        decimal points
        enum questionType
        jsonb questionSnapshot
    }

    ANSWER_SHEET {
        uuid id PK
        uuid assessmentId FK
        enum status
        decimal totalScore
        string grade
        boolean isPassed
        timestamp startedAt
        timestamp submittedAt
    }

    ANSWER_ENTRY {
        uuid id PK
        uuid answerSheetId FK
        uuid assessmentQuestionId FK
        jsonb response
        decimal scoreAwarded
        enum gradingStatus
    }
```

### ✦ Data Dictionary Highlights
* **Client**: Represents the tenant organization utilizing the platform.
* **QuestionBank**: A collection of reusable questions.
* **Assessment**: The core configuration mapping a set of QuestionBanks to specific execution settings (e.g., real-time vs. self-paced).
* **AnswerSheet**: The master record of a participant's attempt for a specific assessment.
* **AnswerEntry**: A granular record of a single question's response, grading outcome, and AI feedback.
