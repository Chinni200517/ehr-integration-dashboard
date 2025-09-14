# Implementation Guide — EHR Integration Dashboard

## Overview
This document explains how the integration works, architecture decisions, authentication, state management, error handling, and deployment recommendations for the EHR CRUD Dashboard assignment.

**Chosen EHR:** DrChrono (REST API). The deliverable can be adapted to Epic (SMART-on-FHIR) by replacing the client flow with SMART launch and FHIR resources.

---
## Architecture
- **Frontend:** Next.js (TypeScript). Pages for Patients, Appointments, Clinical, Billing, Settings.
- **Server / Integration layer:** Next.js API routes used as a secure proxy to call DrChrono APIs. Keeps client secrets server-side.
- **Optional DB:** PostgreSQL for token storage (encrypted), audit logs, app users & RBAC.
- **Auth:** OAuth2 Authorization Code (server-side). Access tokens + refresh tokens stored encrypted server-side.

Sequence (simplified):
1. User clicks "Connect EHR" -> redirects to DrChrono OAuth authorize URL.
2. DrChrono redirects back to `/api/auth/callback?code=...`.
3. Server exchanges `code` for access + refresh token; stores tokens encrypted.
4. Frontend calls Next.js API routes; server attaches `Authorization: Bearer <access_token>` when calling DrChrono.

---
## Key Implementation Details

### OAuth & Token Management
- Use Authorization Code flow with server-side exchange.
- Store tokens encrypted at rest (e.g., KMS or DB field encryption).
- Implement token refresh endpoint and proactively refresh before expiry.
- Do not expose client_secret on the frontend.

### Proxy Pattern (Security)
- All calls to EHR should go through server-side proxy API routes.
- Proxy enforces RBAC, records audit logs, and sanitizes responses to remove any accidental logs of PHI.

### State Management
- **React Query (TanStack)** for server state (patients, appointments).
- **React Context + useReducer** for session, user roles, and app-level settings.
- Keep PHI out of localStorage; use HttpOnly cookies for session tokens.

### Error Handling Strategy
- Standardize error response format: `{ code, message, details? }`.
- For upstream 401: attempt token refresh once; if refresh fails, require re-auth.
- For rate-limits (429): exponential backoff with jitter; surface friendly UI message.
- Log server-side errors with correlation IDs for tracing.

### Performance Optimizations
- Use pagination and server-side filtering for patient lists.
- Cache common responses with React Query; use `stale-while-revalidate` patterns.
- Use bulk export endpoints for large reports.

### Audit & Compliance
- Record user id, action type (READ/WRITE/DELETE), target resource id, timestamp, and client IP.
- Keep audit logs immutable if possible and retain according to org policy.
- Enforce TLS, encrypt tokens & backups, and implement RBAC with least privilege.

---
## Development & Deployment
- Use Vercel for Next.js hosting (automatic HTTPS).
- Store secrets in Vercel Environment Variables or a secrets manager.
- CI pipeline: run tests, lint, build; deploy to preview environment.

---
## Files of interest in this package
- `nextjs_starter/pages/api/auth.ts` — OAuth skeleton
- `nextjs_starter/pages/api/proxy/[...path].ts` — Proxy skeleton
- `API_Discovery.md` — endpoint list used for the assignment
- `postman_collection.json` — importable into Postman

---
## Known limitations & notes
- Some lab write endpoints or billing features may require DrChrono enablement; contact DrChrono support for vendor-specific access.
- Epic integration requires implementing SMART-on-FHIR and adapting resource models to FHIR R4.