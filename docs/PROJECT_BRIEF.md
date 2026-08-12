> Repo: **Keynest**. Companion to `docs/SHARED_CONTEXT.md`.

# Keynest — Auth / identity field brief

**Read first:** [`SHARED_CONTEXT.md`](./SHARED_CONTEXT.md)  
**Field:** Authentication & identity  
**Working name:** Keynest  
**Say it:** KEY-nest  
**Status:** Not started  
**Priority:** Mid/low as standalone — only if auth depth isn’t already proven elsewhere; high keyword value if done properly

---

## One-sentence pitch

A real identity service: OIDC/OAuth, session/refresh security, orgs + RBAC, MFA, audit log — Clerk/Auth0 core, not “JWT login.”

---

## Paid analog / why it exists

Auth products are paid because session theft, refresh rotation, SSO, and org permissions are easy to get subtly wrong. Keynest proves threat-aware auth engineering.

---

## Depth requirement

| Capability | Minimum |
|---|---|
| **OIDC/OAuth** | Login with at least one social provider + email/password or magic link |
| **Sessions** | Server sessions or refresh-token rotation with reuse detection |
| **RBAC** | Orgs, roles, permission checks on APIs |
| **MFA** | TOTP (or WebAuthn stretch) |
| **Audit** | Login, logout, refresh reuse, admin permission changes logged |
| **Security basics** | CSRF strategy, secure cookies, secret management, rate limit login |
| **Proof** | Threat notes in README; demo of refresh reuse revocation; test suite around auth core |

**Shallow fail:** `jsonwebtoken.sign` on login with tokens in localStorage and a roles string on the user row.

---

## Suggested architecture

```
Apps → Keynest (authorize, token, userinfo)
     → Postgres (users, orgs, sessions, audit)
     → Redis (rate limits, session secondary optional)
Admin UI for orgs/members
```

**Stack bias:** TypeScript; well-known libraries OK (openid-client, iron-session, etc.) but **you** own flows and threat model; Docker Compose; sample resource API that enforces RBAC.

---

## MVP vertical slice

1. Email + OAuth login to a sample app.
2. Org with admin/member roles gating routes.
3. Refresh rotation with reuse → revoke family.
4. TOTP enroll + challenge.
5. Audit log page.
6. Security section in README (cookie flags, CSRF, storage choices).

---

## Resume bullets (draft)

- Built **Keynest**, an identity service with OIDC login, refresh-token rotation, org RBAC, and TOTP MFA.
- Implemented audit logging and refresh reuse detection that revokes stolen token families.
- Documented session threat model and enforced RBAC on a sample multi-tenant API.

**Best resume variants:** Backend, Full-stack; keyword-dense for security-adjacent JDs.

---

## Interview ammo

- Access vs refresh tokens; rotation and reuse detection.
- Why httpOnly cookies vs bearer in JS-readable storage.
- CSRF for cookie sessions.
- RBAC vs ABAC at this scope.

---

## Explicit non-goals

- Full enterprise IdP (SAML everywhere, SCIM v2 complete) on day one
- Replacing cloud IAM

---

## Overlap warnings

- Tidegate **validates** tokens; Keynest **issues** them — keep boundaries clean if both exist.
- DevStack may already have JWT/RBAC — Keynest must be deeper (OIDC/MFA/rotation/audit) or user should deepen DevStack instead of a thin duplicate. Confirm with user before scaffolding.

---

## Agent kickoff checklist

- [ ] Confirm not duplicating shallow DevStack auth
- [ ] Threat model doc first page of README
- [ ] MVP auth flows + tests
- [ ] Update Status when demoable
