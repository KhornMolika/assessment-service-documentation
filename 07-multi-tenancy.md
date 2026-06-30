# ⛨ Multi-tenancy & Client Isolation Model

To support a SaaS operational model, the Assessment Service implements strict multi-tenancy. Every core resource is scoped to a specific `Client` (Tenant).

## ⚙ Data Model Isolation

The database architecture employs **Logical Isolation** via a shared database schema.

* Every tenant-owned table (e.g., `QuestionBank`, `Assessment`, `Participant`) contains a mandatory `clientId` foreign key.
* The `Client` table serves as the root of the hierarchy.

## ⚙ Query Scoping

The backend utilizes strict service-layer scoping to prevent data leakage.

```typescript
// Example of strict scoping
async getAssessmentById(clientId: string, assessmentId: string) {
  return this.repository.findOne({
    where: { 
      id: assessmentId,
      clientId: clientId // Strict isolation enforcement
    }
  });
}
```

## ⚙ Configuration & Branding
Each client profile supports custom configuration, allowing tenants to inject their own branding (logos, primary colors) which dynamically styles the participant-facing assessment interfaces.
