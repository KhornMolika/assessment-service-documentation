# ⚠ Known Limitations & Technical Debt

While the Assessment Service is robust, certain architectural limitations and areas of technical debt should be acknowledged for future roadmap planning.

## ✦ Known Limitations

* **AI Grading Determinism**: LLM outputs can occasionally be unpredictable. While prompt engineering mitigates this, edge-case hallucinated grades are possible, necessitating the manual review fallback.
* **Real-time Synchronization Latency**: During massive spikes in concurrent real-time participants (e.g., 5,000+ simultaneously joining), the single-threaded Node.js event loop may experience slight delays broadcasting WebSocket events.
* **Media Processing**: The current infrastructure relies on basic file uploads. Large video or high-res image uploads inside subjective answers may strain the backend memory if not offloaded to direct S3 pre-signed URLs.

## ✦ Technical Debt

* **End-to-End (E2E) Test Coverage**: While core unit tests exist, comprehensive Cypress or Playwright E2E tests for complex user flows (like the real-time synchronous progression) are currently sparse.
* **Redis Bottleneck on Pub/Sub**: The system currently relies on a single Redis instance for both background job queuing (BullMQ) and WebSocket message broadcasting. As the application scales, these concerns should be split into dedicated Redis clusters.
* **Monolithic Repository Structure**: The frontend and backend are currently loosely coupled but housed within the same project space. Transitioning to a strict Nx Monorepo or entirely separate repositories would improve CI/CD isolation.
* **Database Connection Pooling**: Under extreme load spikes (e.g. 10,000 users joining a real-time session within a 5-second window), PgBouncer should be introduced in front of PostgreSQL to prevent connection exhaustion.
* **Soft Deletes vs GDPR**: Currently, participants are soft-deleted. To comply fully with GDPR "Right to be Forgotten", a hard-deletion cleanup CRON job should be implemented to wipe PII permanently after a tenant deletes a participant.
