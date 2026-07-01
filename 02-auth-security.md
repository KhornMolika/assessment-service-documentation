# ⛨ Authentication & Security Design

Security is integrated at multiple layers of the application, ensuring that tenant data is strictly isolated and participant data is secure.

## ⚙ Authentication Mechanisms

### ✦ Client Authentication (Workspace)
Clients interacting with the workspace utilize a robust authentication mechanism.

* **Strategy**: JWT-based stateless authentication via OAuth2 Client Credentials grant.
* **Token Request Payload**:
  To obtain an access token, clients submit a `POST /api/v1/auth/token` request with:
  ```json
  {
    "clientId": "{{clientId}}",
    "clientSecret": "{{clientSecret}}",
    "grant_type": "client_credentials"
  }
  ```
* **Token Response Payload**:
  A successful request returns the standard success wrapper containing the JWT:
  ```json
  {
      "success": true,
      "statusCode": 200,
      "timestamp": "2026-07-01T02:41:55.171Z",
      "path": "/api/v1/auth/token",
      "data": {
          "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJiNGZkNjc4NS0wZGQ3LTQ4YjMtYmRmMy00MzUxZjk5ZTRiYmQiLCJzbHVnIjoiYWNtZS1jb3JwMyIsInNjb3BlcyI6W10sImlhdCI6MTc4Mjg3MzcxNSwiZXhwIjoxNzgyODc3MzE1fQ.w6POv00-Un2_vjzDxKxmPnhYopiHCbW1ve3I88i1CP0",
          "token_type": "Bearer",
          "expires_in": 3600
      }
  }
  ```
* **Token Storage**: `HttpOnly` secure cookies to prevent XSS-based token exfiltration.
* **Payload Structure**:
  ```json
  {
    "sub": "client_uuid",
    "email": "workspace@organization.com",
    "iat": 1717200000,
    "exp": 1717286400
  }
  ```

### ✦ Participant Authentication
Participants accessing assessments do not require full accounts. Instead, they operate via time-limited, signed URLs or session tokens.

* **Strategy**: Short-lived Session Tokens.
* **Mechanism**: When a participant joins an assessment, a unique session token is generated. This token grants scoped access strictly to their specific `AnswerSheet` ID.

## ⚙ Security Policies

### ✦ Data Isolation
* All database queries interacting with sensitive resources enforce strict `clientId` scoping.
* Cross-tenant access is explicitly blocked at the service layer.

### ✦ Rate Limiting & Abuse Prevention
* **REST Endpoints**: Throttled using `@nestjs/throttler` backed by Redis to prevent brute force attacks and API abuse.
* **WebSockets**: Connection rates and message frequencies are strictly monitored and limited.

### ✦ Data Validation
* **Frontend**: Extensive use of Zod for client-side input validation and error feedback.
* **Backend**: Deep validation pipelines utilizing `class-validator` and `class-transformer` ensure no malformed or malicious payloads reach the database execution layer.
