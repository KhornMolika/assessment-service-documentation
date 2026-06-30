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
        LLM{OpenAI GPT-4}
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
    Client {
        uuid id PK
        string name
        string brandingColor
        timestamp createdAt
    }
    
    Participant {
        uuid id PK
        uuid clientId FK
        string name
        string identifier
    }

    Assessment {
        uuid id PK
        uuid clientId FK
        string title
        string mode "SELF_PACED | REAL_TIME"
        jsonb settings
    }

    QuestionBank {
        uuid id PK
        uuid clientId FK
        string title
    }

    AnswerSheet {
        uuid id PK
        uuid assessmentId FK
        uuid participantId FK
        float totalScore
        string status
    }

    Client ||--o{ QuestionBank : owns
    Client ||--o{ Assessment : manages
    Client ||--o{ Participant : invites

    Assessment ||--o{ QuestionBank : includes
    Assessment ||--o{ AnswerSheet : generates
    Participant ||--o{ AnswerSheet : attempts
    QuestionBank ||--o{ QuestionRound : contains
    AnswerSheet ||--o{ AnswerEntry : tracks
    QuestionRound ||--o{ AnswerEntry : evaluated-by
```

### ✦ Data Dictionary Highlights
* **Client**: Represents the tenant organization utilizing the platform.
* **QuestionBank**: A collection of reusable questions.
* **Assessment**: The core configuration mapping a set of QuestionBanks to specific execution settings (e.g., real-time vs. self-paced).
* **AnswerSheet**: The master record of a participant's attempt for a specific assessment.
* **AnswerEntry**: A granular record of a single question's response, grading outcome, and AI feedback.
