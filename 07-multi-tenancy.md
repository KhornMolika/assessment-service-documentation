# ⛨ Multi-tenancy & Client Isolation Model

To support a SaaS operational model, the Assessment Service implements strict multi-tenancy. Every core resource is scoped to a specific `Client` (Tenant).

## ⚙ Data Model Isolation

The database architecture employs **Logical Isolation** via a shared database schema.

* Every tenant-owned table (e.g., `QuestionBank`, `Assessment`, `Participant`) contains a mandatory `clientId` foreign key.
* The `Client` table serves as the root of the hierarchy, storing tenant-specific metadata, webhook configurations, and security policies (like `allowedOrigins`).
* We deliberately avoid physical isolation (separate databases per tenant) to minimize infrastructure overhead and simplify cross-tenant reporting for super admins, relying heavily on application-level filtering.

## ⚙ Application-Level Enforcement

The backend utilizes strict service-layer scoping to prevent data leakage, backed by global NestJS interceptors and guards.

### ✦ ClientAuthGuard & Context Injection
Every incoming request (except public routes) is intercepted by the `ClientAuthGuard`.
1. The guard validates the JWT token.
2. It extracts the `clientId` (sub) from the token.
3. It resolves the full `Client` entity from the database (cached via Redis).
4. It injects the `Client` entity into the request object.

Controllers then extract this using the `@CurrentClient()` decorator:

```typescript
@Get(':id')
async getAssessmentById(
  @CurrentClient() client: Client,
  @Param('id') assessmentId: string
) {
  return this.repository.findOne({
    where: { 
      id: assessmentId,
      clientId: client.id // Strict isolation enforcement
    }
  });
}
```

### ✦ Redis Caching
To prevent the tenant lookup from hammering the PostgreSQL database on every single API call, the resolved `Client` entity is cached in Redis. When a super admin updates a client's profile (e.g., suspends them or changes their allowed origins), the Redis cache is aggressively invalidated.

## ⚙ Configuration & Branding
Each client profile supports custom configuration:
* **Branding**: Allows tenants to inject their own logos and primary colors, which dynamically styles the participant-facing assessment interfaces using CSS variables.
* **Webhooks**: Tenants can configure a `webhookUrl` and a `webhookSecret`. The backend dispatches signed HTTP POST requests (e.g., `assessment.completed`) to these endpoints, allowing tenants to natively integrate grading results back into their own proprietary systems.
* **CORS Policies**: Tenants configure an array of `allowedOrigins`. The backend dynamic CORS middleware validates the `Origin` header against the specific tenant's allowlist before permitting cross-origin iframe embedding.
