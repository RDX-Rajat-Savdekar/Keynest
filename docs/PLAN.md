# Keynest — Implementation Plan

**Product:** Self-hostable identity service (OIDC/OAuth, sessions/refresh security, orgs + RBAC, MFA, audit) — Clerk/Auth0 core, not “JWT login.”  
**Bar:** Interview-proof (Compose, demo, Loom, ≥1 measured metric, defendable threat model).  
**Constraint:** Build toward the **final product** continuously. No “phases.” Ship vertical capability one piece at a time; each merge leaves a runnable system closer to the pitch.

Related: [`SHARED_CONTEXT.md`](./SHARED_CONTEXT.md) · [`PROJECT_BRIEF.md`](./PROJECT_BRIEF.md) · [`../journal.md`](../journal.md)

---

## 1. Definition of done (final product)

Keynest is done for resume/demo when **all** of the following are true:

| Capability | Concrete acceptance |
|---|---|
| **Email auth** | Email + password **or** magic link; passwords hashed (Argon2id or bcrypt cost documented); verified email path or explicit MVP note |
| **OAuth / OIDC** | At least one social provider (e.g. Google); authorization code + PKCE where applicable; tokens never in localStorage |
| **Sessions** | Server sessions **or** refresh-token rotation with **reuse detection** that revokes the token family |
| **Access tokens** | Short-lived access tokens (JWT or opaque); refresh path documented; audience/issuer claims if JWT |
| **Orgs + RBAC** | Org create/join; roles `admin` / `member` (minimum); permission checks on protected APIs |
| **MFA** | TOTP enroll + challenge on login (or step-up); recovery codes optional but preferred |
| **Audit** | Persist: login, logout, refresh reuse, MFA enroll/disable, role/permission changes; UI or API to list |
| **Security basics** | httpOnly Secure cookies (or documented bearer-only resource API); CSRF strategy for cookie flows; rate limit login; secrets via env |
| **Sample resource API** | Separate demo app/API that **enforces** Keynest-issued identity + org RBAC on routes |
| **Admin UI** | Orgs/members, MFA status, audit log view (operator surface — not a marketing site) |
| **Ops** | `docker compose up` brings Keynest + Postgres + Redis + sample app; README: architecture, how to run, **threat model** |
| **Proof** | Automated tests on auth core (rotation, reuse revoke, RBAC deny); demo of refresh reuse → family revoke; **one honest measured number** (e.g. login RPS or p99 token validate under load) |
| **Docs** | Threat notes (cookie vs bearer, CSRF, storage, rotation); tradeoffs; Loom path |

**Shallow fail (do not ship as “done”):** `jsonwebtoken.sign` on login, tokens in localStorage, roles as a string on the user row with no org boundary, no audit, no reuse detection.

**Explicit non-goals (do not build day one):** full enterprise IdP (SAML everywhere, SCIM v2 complete), replacing cloud IAM, every OAuth provider, WebAuthn as MVP requirement (stretch after TOTP).

**Boundary with siblings:**
- **Tidegate** validates/enforces tokens at the edge — Keynest **issues** identity. Do not reimplement a gateway here.
- **DevStack** may have JWT/RBAC — Keynest must be deeper (OIDC, MFA, rotation, audit) or deepen DevStack instead. This plan assumes a **standalone IdP** with a sample consumer app.

---

## 2. Architecture (target system)

```
┌──────────────────┐         ┌─────────────────────────────────────┐
│ Sample app / API │ ──────► │ Keynest                             │
│ (resource + UI)  │  OIDC   │ /authorize · /token · /userinfo     │
│ RBAC on routes   │  cookies│ sessions · refresh · MFA · audit    │
└──────────────────┘  or     └──────────────┬──────────────────────┘
                      bearer                │
                         ┌──────────────────┼──────────────────┐
                         ▼                  ▼                  ▼
                   ┌──────────┐       ┌──────────┐       ┌──────────┐
                   │ Postgres │       │  Redis   │       │ Admin UI │
                   │ users    │       │ rate     │       │ orgs     │
                   │ orgs     │       │ limit    │       │ members  │
                   │ sessions │       │ (opt.    │       │ audit    │
                   │ audit    │       │  sess)   │       │ MFA      │
                   └──────────┘       └──────────┘       └──────────┘
```

### Locked stack decisions (revise only with a written tradeoff)

| Concern | Choice | Why |
|---|---|---|
| Language | **TypeScript (Node)** | Aligns with portfolio; strict types; fast iteration |
| HTTP framework | **Fastify** or **Hono** (pick one and stick) | Explicit middleware; good for auth middleware demos |
| Primary DB | **Postgres** | Users, orgs, memberships, sessions/refresh families, audit — real persistence |
| Rate limit / ephemeral | **Redis** | Login brute-force limits; optional session secondary index |
| Password hashing | **Argon2id** (preferred) or bcrypt with documented cost | Industry baseline; document parameters |
| OAuth client | Well-known libs OK (`openid-client`, provider SDKs) | Own the **flows and threat model**; do not invent crypto |
| Access token | Short-lived **JWT** (RS256 or EdDSA) **or** opaque + introspection — pick one, document | Interview ammo: why not long-lived localStorage JWT |
| Refresh | **Refresh rotation + family reuse detection** | Depth bar; must be tested |
| Cookies | `HttpOnly`, `Secure`, `SameSite` documented; CSRF for cookie session APIs | Security basics in brief |
| Admin / sample UI | **React + Vite** | Operator + sample app surfaces |
| Deploy | **Docker Compose** | Interview-proof local path |
| Migrations | **Drizzle** or **Prisma** — pick one | Schema as code; keep `docs/SCHEMA.md` true |

### Design principles

1. **Threat model first** — README security section is not an afterthought; write it as capabilities land.
2. **Tokens are not the product** — sessions, rotation, RBAC, MFA, and audit are. Access tokens are a delivery mechanism.
3. **Org is the tenancy boundary** — membership + role checks on every protected resource API path.
4. **Fail closed** — missing role → 403; reuse of refresh → revoke family + audit; rate limit login.
5. **Secrets stay out of the repo** — `.env.example` only; signing keys generated/loaded at runtime.
6. **Own the flows** — libraries may speak OIDC/OAuth; you own rotation, cookie flags, CSRF, MFA challenge, audit events.
7. **Observability of Keynest itself** — structured JSON logs for auth events (no secrets in logs).

---

## 3. Repository layout (create as you build)

```
Keynest/
├── AGENTS.md
├── README.md                 # how to run, architecture, threat model, metrics
├── journal.md                # session continuity
├── docs/
│   ├── SHARED_CONTEXT.md
│   ├── PROJECT_BRIEF.md
│   ├── PLAN.md               # this file
│   ├── SCHEMA.md             # users/orgs/sessions/audit (write early, keep true)
│   ├── THREAT_MODEL.md       # cookie/CSRF/rotation/MFA (can start in README, split when long)
│   └── ADRs/                 # short decisions (JWT vs opaque, cookie vs bearer, etc.)
├── docker-compose.yml
├── .env.example
├── packages/ or services/
│   ├── server/               # Keynest IdP: authorize, token, userinfo, admin APIs
│   ├── admin-ui/             # orgs, members, audit, MFA management
│   └── sample-app/           # consumer app + resource API enforcing RBAC
├── scripts/
│   ├── load-test.ts          # login / token validate throughput
│   └── seed.ts               # demo org + users
└── tests/                    # unit + integration (Compose Postgres/Redis)
```

Start as a single Node service + one UI if that ships faster. Split sample-app when the consumer boundary clarifies the demo. Shared types (`@keynest/shared`) once two packages exist.

---

## 4. Data model (write `docs/SCHEMA.md` before bulk coding)

### Users & credentials

- User: id, email (unique), email_verified, password_hash nullable (OAuth-only users), created_at, disabled
- OAuth identity: provider, provider_subject, user_id (unique per provider+subject)
- MFA: TOTP secret (encrypted at rest), enabled_at, recovery codes hashed

### Orgs & RBAC

- Org: id, name, slug, created_at
- Membership: org_id, user_id, role (`admin` | `member`), unique (org_id, user_id)
- Permission checks: map role → actions (e.g. `org:admin`, `resource:write`); keep RBAC simple — not full ABAC

### Sessions & refresh

- Session or refresh family: family_id, user_id, org_id?, created_at, revoked_at
- Refresh token: family_id, token_hash, parent_hash?, expires_at, used_at, replaced_by
- **Reuse rule:** presenting an already-used/replaced refresh → revoke entire family + audit `refresh_reuse`

### Audit

- event_type, actor_user_id, org_id?, ip, user_agent, metadata JSON (no secrets), created_at
- Minimum events: `login_success`, `login_failure`, `logout`, `refresh_reuse`, `mfa_enroll`, `mfa_challenge`, `role_change`, `org_member_add/remove`

Indexes: users `(email)`, memberships `(org_id, user_id)`, refresh `(token_hash)`, audit `(org_id, created_at DESC)`, `(actor_user_id, created_at DESC)`.

---

## 5. Build order (piece by piece → final product)

Work top to bottom. Each item should leave `compose up` (or a documented subset) runnable. **Do not** batch as “Phase A/B/C.” Check boxes here or in the journal as you go.

### Foundations

- [ ] Scaffold tooling: pnpm (or npm), TypeScript strict, ESLint/Prettier, shared `tsconfig`
- [ ] `docker-compose.yml`: Postgres + Redis + service placeholders
- [ ] `.env.example` + README “how to run” stub
- [ ] Write `docs/SCHEMA.md` (tables, indexes, retention for audit if any)
- [ ] Migrations (Drizzle or Prisma) matching schema
- [ ] Health endpoint + structured logging

### Identity core (email)

- [ ] Register + login (email/password); Argon2id; generic error messages on failure
- [ ] Rate limit login by IP + email (Redis)
- [ ] Server session cookie **or** issue access + refresh with rotation from day one of token path
- [ ] Logout + session/refresh revoke
- [ ] Unit tests: password verify, rate limit keying

### Refresh rotation (depth bar)

- [ ] Refresh endpoint rotates: new refresh, invalidate old, same family
- [ ] Reuse detection → revoke family + audit event
- [ ] Integration test: reuse of old refresh fails and kills siblings
- [ ] Document in threat model / README

### Orgs + RBAC

- [ ] Create org; add member; assign admin/member
- [ ] Middleware: require auth + require role/permission
- [ ] Sample resource API routes gated by org role (read vs write)
- [ ] Tests: member cannot perform admin actions; cross-org deny

### OAuth / OIDC provider login

- [ ] One social provider (Google recommended for demo ergonomics)
- [ ] Link or create user; no password required for OAuth-only
- [ ] PKCE / state / nonce as required by the flow; document cookie/callback settings
- [ ] Sample app login via Keynest (authorization code or session cookie handoff)

### MFA (TOTP)

- [ ] Enroll: show secret/QR; confirm with code; store encrypted secret
- [ ] Login challenge when MFA enabled
- [ ] Disable MFA (admin or self with re-auth)
- [ ] Tests: wrong TOTP fails; reuse of recovery code (if implemented) fails closed

### Audit + Admin UI

- [ ] Persist required audit events
- [ ] Admin UI: org members, roles, MFA status, audit list (filter by org/time)
- [ ] Sample app UI: login, org switcher (if multi-org), protected page demo

### Hardening toward resume bar

- [ ] CSRF strategy implemented + documented (double-submit or SameSite+origin check — pick and defend)
- [ ] Cookie flags documented; no tokens in localStorage in sample app
- [ ] Secret/key management: signing keys via env or generated volume; rotate story in docs
- [ ] Load test script + one measured number in README (date + machine noted)
- [ ] Architecture diagram + tradeoffs in README
- [ ] Loom script: OAuth/email login → MFA → admin change → refresh reuse revoke → audit line
- [ ] Draft 3–4 resume bullets with **real** numbers only
- [ ] Update `PROJECT_BRIEF.md` Status when demoable

### Optional deepeners (only after bar above)

- WebAuthn, magic-link email, SCIM lite, SAML one IdP, opaque tokens + introspection, multi-region session sticky notes, device sessions UI

---

## 6. Engineering practices (always)

| Practice | How it applies here |
|---|---|
| **Vertical slices** | Prefer “login → protected resource → audit row” over horizontal stubs |
| **Schema first** | Migrations + `SCHEMA.md` before UI polish |
| **Security tests** | Rotation/reuse, RBAC deny, MFA challenge — these are the product |
| **ADRs** | Short note when changing token format, cookie policy, or MFA storage |
| **Secrets** | Never commit `.env` or real OAuth client secrets |
| **CI** | Lint + unit/integration on PR once tests exist |
| **Honest metrics** | Load numbers only from scripts you ran |
| **README as product** | Runner can demo without reading the brief |
| **Journal** | Update `journal.md` each significant session |

### Testing strategy

1. **Unit:** password hash verify, TOTP verify, permission map, refresh family state machine
2. **Integration:** Compose Postgres + Redis; register → login → refresh → reuse → revoke; RBAC on sample API
3. **Load:** separate script for login or token validate; required before “shipped,” not every commit
4. **UI:** light Playwright or manual Loom path — auth core tests > pixel tests

### API style

- OIDC-ish surfaces where they earn the demo: `/authorize`, `/token`, `/userinfo` (documented subset is OK)
- Admin/resource REST under `/v1` with stable errors `{ error: { code, message } }`
- Never return refresh tokens in JSON if cookie model is chosen for browser apps — pick one model and stay consistent in the sample app

---

## 7. Interview ammo (create while building)

Capture answers in README or `docs/THREAT_MODEL.md`:

- Access vs refresh; why rotation; how reuse detection works
- Why httpOnly cookies vs bearer in JS-readable storage
- CSRF for cookie-authenticated APIs
- RBAC vs ABAC at this scope; why org is the tenancy unit
- What happens when a refresh token is stolen (family revoke + audit)
- Password hashing parameters and why not plain bcrypt cost 1

---

## 8. Resume bullets (template — fill metrics when real)

- Built **Keynest**, an identity service with OIDC login, refresh-token rotation, org RBAC, and TOTP MFA.
- Implemented audit logging and refresh reuse detection that revokes stolen token families.
- Documented session threat model and enforced RBAC on a sample multi-tenant API.
- Load-tested login/token path and reported measured throughput/latency at stated concurrency.

---

## 9. Current state

| Item | Status |
|---|---|
| Repo seeded (briefs, README, agents entry) | Done |
| Code / Compose / schema | Not started |
| Plan + journal | Done (this session) |

**Next concrete work:** scaffolding (tooling + Compose Postgres/Redis) and `docs/SCHEMA.md`, then email register/login + session/refresh with a protected sample route (first end-to-end slice).
