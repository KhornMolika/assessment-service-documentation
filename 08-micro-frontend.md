# ⊞ Micro-frontend Integration Guide

The Assessment Service frontend is built using Next.js. For organizations seeking to embed this service within a larger monolithic portal (e.g., an LMS or HR system), there are several robust integration pathways.

## ⚙ Iframe Embedding (Primary Method)

The simplest and most secure integration method involves embedding the participant-facing assessment URLs via `<iframe>`.

### ✦ Security & CORS
To protect against clickjacking while allowing authorized embedding, the system dynamically configures the `Content-Security-Policy` and `X-Frame-Options` headers.
* **Tenant Configuration**: The tenant must add their parent domain (e.g., `https://lms.organization.com`) to their `allowedOrigins` array in the Assessment Service workspace.
* **Middleware Execution**: The Next.js middleware and NestJS backend read this configuration and dynamically append the `frame-ancestors` directive for that specific session.

### ✦ EmbedDetector & Dynamic CSS
The frontend employs an `EmbedDetector` hook that analyzes `window.top !== window.self` and URL query parameters (`?embed=true`).
When embedded:
1. The global UI shells (like `WorkspaceShell` and `PageHeaderCard`) are automatically hidden.
2. Background colors become transparent, allowing the iframe to seamlessly blend into the parent application's DOM.
3. The dark mode theme is synchronized dynamically via `postMessage`.

### ✦ Cross-Window Communication
We use `window.postMessage()` to establish a two-way communication channel between the iframe and the parent window.

**Events emitted by the Iframe:**
* `ASSESSMENT_LOADED`: Fired when the React tree successfully mounts.
* `ASSESSMENT_COMPLETED`: Fired when the participant submits the final question. Includes a payload with the `sessionId` and preliminary `score`.

**Events accepted by the Iframe:**
* `THEME_UPDATE`: The parent window can push `{ theme: 'dark' }` into the iframe to force the assessment to match the parent's dark mode toggle instantly without reloading.

## ⚙ Integration Modal

The frontend provides a built-in `IntegrationModal` for workspace administrators. This modal auto-generates the exact HTML and React code snippets required for embedding, dynamically injecting the current assessment's ID, the tenant's API keys, and the correct environmental URLs.

## ⚙ Deep Linking & SSO

To integrate the admin workspace natively without iframes:
1. Configure an external Identity Provider (IdP).
2. Generate JWTs at the parent application level matching our expected payload structure.
3. Pass the JWT seamlessly via URL parameters, or use a shared domain cookie if the parent and the Assessment Service operate on the same root domain (e.g., `app.domain.com` and `assessments.domain.com`).

## ⚙ Module Federation (Advanced)

Next.js supports Webpack Module Federation (`@module-federation/nextjs-mf`). This allows you to expose specific React components (like the `ResultSheetDetailView` or the `RealtimeLeaderboard`) to be dynamically loaded into remote Next.js or React applications. This bypasses iframe boundaries entirely, sharing the React context tree, though it requires tightly coupled build pipelines.
