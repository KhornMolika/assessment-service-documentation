# ⊞ Micro-frontend Integration Guide

The Assessment Service frontend is built using Next.js. For organizations seeking to embed this service within a larger monolithic portal, there are several integration pathways.

## ⚙ Iframe Embedding

The simplest integration method involves embedding the participant-facing assessment URLs via `<iframe>`.

* **Security**: Ensure the parent domain is added to the `Content-Security-Policy` `frame-ancestors` directive on the backend to allow embedding.
* **Communication**: Use `window.postMessage()` to pass completion events (e.g., `ASSESSMENT_COMPLETED`) from the iframe back to the parent window.

## ⚙ Deep Linking & SSO

To integrate the admin workspace natively:
1. Configure an external Identity Provider (IdP).
2. Generate JWTs at the parent application level.
3. Pass the JWT seamlessly via URL parameters or shared cookies (if operating under the same root domain).

## ⚙ Module Federation (Advanced)

Next.js supports Webpack Module Federation (`@module-federation/nextjs-mf`). This allows you to expose specific React components (like the `ResultSheetDetailView`) to be dynamically loaded into remote Next.js or React applications without relying on iframes.
