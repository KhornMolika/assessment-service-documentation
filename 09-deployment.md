# ⚙ Deployment & Infrastructure Guide

The Assessment Service relies on a containerized, cloud-native architecture suitable for horizontal scaling.

## ✦ Infrastructure Requirements

* **Node.js**: v20+
* **Database**: PostgreSQL 16+
* **Cache/Broker**: Redis 7+

## ✦ Environment Variables

Both the frontend and backend require properly configured `.env` files containing:
* `DATABASE_URL`: The PostgreSQL connection string.
* `REDIS_URL`: The Redis connection string.
* `JWT_SECRET`: A highly secure, randomized cryptographic key.
* `OPENAI_API_KEY`: Credentials for the AI Grading Provider.

## ✦ Deployment Strategies

### 1. Docker & Kubernetes (Recommended for Scale)
Containerize both the Next.js frontend and NestJS backend.
* Use stateless backend nodes behind a Load Balancer.
* Scale WebSocket connections horizontally using the Redis adapter for `Socket.io`.

### 2. Vercel (Frontend) + Cloud Run (Backend)
* **Frontend**: Deploy the Next.js application to Vercel for edge caching, CDN benefits, and serverless API routes.
* **Backend**: Deploy the NestJS container to Google Cloud Run or AWS App Runner.
* **Database/Redis**: Managed services like Supabase, Aiven, or AWS RDS/ElastiCache.

## ✦ Continuous Integration (CI/CD)
The repository supports automated workflows for:
* **Linting & Formatting**: Enforcing ESLint and Prettier rules.
* **Type Checking**: Running `tsc --noEmit` across both codebases.
* **Testing**: Executing Jest unit and E2E tests before container builds.
