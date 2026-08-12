# Shared context for agents (copied into Keynest)

> This repo is **Keynest** (KEY-nest). Field: Auth / identity.
> Read `docs/SHARED_CONTEXT.md` then `docs/PROJECT_BRIEF.md` before building.

---


---

## Who this is for

**Owner:** Rajat Savdekar  
**Primary goal:** get a software engineering job (new-grad / early-career fullstack–backend leaning, USC MS CS ~May 2026).  
**Location:** LA / remote-friendly US hiring.

**Contact / proof surfaces (existing):**
- Site: https://rajatsavdekar.dev/
- GitHub: https://github.com/RDX-Rajat-Savdekar/
- LinkedIn: https://www.linkedin.com/in/rajatsavdekar/
- Devpost: https://devpost.com/rajatsavdekar
- Resume TeX reference: living in the `paper-trail` repo (`resume_versions/`)
- Projected resume: living in the `paper-trail` repo — **roadmap only, not for applying until projects are real**

---

## North-star strategy (do not violate)

1. **Visible + defendable** beats deep-but-invisible *and* shallow-but-pretty.
2. Resume claims must survive a 20-minute cold deep-dive (“hardest bug,” schema, tradeoffs, failure modes).
3. **Never fabricate shipped projects** for applications. Projected specs = build plans, not application materials.
4. AI is OK for wording, scaffolding, and speed — **not** for inventing biography or empty “perfect JD” projects on the resume.
5. Portfolio site can show animation craft; **projects themselves must read as product/systems engineering.**
6. Thesis: **free / self-hostable alternatives to paid developer tools** — one deep cut per field, not six thin clones.

### Depth bar (pass/fail)

A project is “deep enough” only if it matches **DevStack / MockPad** class:

| Cleared the bar (keep) | Below the bar (do not add / demote) |
|---|---|
| **DevStack** — multi-service infra, event-sourced workflows, Redis flags, Compose | **SplitIt** — JWT + CRUD expense tracker |
| **MockPad** — Yjs CRDTs, custom WS, session replay | Thin OpenAI chat wrappers, architecture “recommenders,” dashboard+charts toys |

**Interview-proof means:** live demo, Loom, write-up (problem → design → tradeoffs → metrics), ≥1 measured number, runnable Compose/cloud deploy, 3–4 resume bullets of form *verb + system + constraint + result*.

---

## What exists today (as of Aug 2026 briefing)

### Strong (resume-ready when bullets stay honest)

| Asset | Stack / nature | Notes for agents |
|---|---|---|
| **DevStack** | Next.js, Node, Postgres, Redis, Docker | Self-hostable flags/workflows/auth/analytics. Prefer **new separate projects** unless user explicitly asks to deepen DevStack. Do not duplicate DevStack’s workflow engine as Flowpad without clear separation. |
| **MockPad** | React, Node, Yjs, MongoDB | Real-time collab editor + Whisper timelines. Covers real-time/CRDT field — **do not build another SyncBoard unless MockPad is thin.** |
| **Pfizer AI/ML Extern** | OCR + RAG (LlamaIndex), eval-ish reporting | Work experience; overlaps thematically with **Citebase** — Citebase must go *deeper on evals/productization*, not redo OCR demos. |
| **Jalgaon SWE** | JS API, MySQL, Docker CI/CD | Real production metrics (query −40%, deploy −30%). |
| **Hackathons** | Aura (2nd, visionOS); **Stitch** (OpenAI Build Week CI agent); **EmojiCode** (Reddit Devvit) | Mention in Honors / Devpost, not as full deep project slots unless Stitch is expanded. |

### Weak / clutter (avoid repeating patterns)

- SplitIt-class CRUD
- Keyword stuffing without repo/demo proof
- Crowding resume with 5+ medium projects

### Resume variants to support (when shipping)

| Variant | Lead with | Emphasize |
|---|---|---|
| Full-stack / product | DevStack, MockPad, Framepad or Seekpad | E2E ownership, Postgres, auth, deploy |
| Backend / platform | Sightline or Tidegate + DevStack | ingest, Redis, load numbers, Docker |
| Frontend / UI | MockPad, Flowpad, Framepad progress UX | real-time UI, CWV, motion with CLS budget |
| AI-product | Pfizer + Citebase | evals, RAG tradeoffs — not chatbot UI |

---

## Field map (one deep project each)

Shared context → then open **only** the field file you are building.

| Field | Project name | Brief file | Status |
|---|---|---|---|
| Observability | **Sightline** | `sightline.md` | Not started — recommended first *new* project |
| API / edge | **Tidegate** | `tidegate.md` | Not started |
| Webhooks / async | **Hookline** | `hookline.md` | Not started |
| Files / media | **Framepad** | `framepad.md` | Not started |
| Search | **Seekpad** | `seekpad.md` | Not started |
| Auth / identity | **Keynest** | `keynest.md` | Not started |
| AI product | **Citebase** | `citebase.md` | Not started |
| Visual systems | **Flowpad** | `flowpad.md` | Not started (standalone; not DevStack unless asked) |

**Skipped on purpose (already covered):**
- Real-time / CRDT → MockPad
- Feature flags platform → DevStack
- CI repair agent → Stitch (hackathon)

**Suggested build order for job goal:** Sightline → Tidegate *or* Framepad → Citebase (if AI roles) → others as needed. Prefer finishing one interview-proof slice over parallel half-builds.

---

## Non-negotiable engineering expectations

Every field project should eventually include:

- TypeScript and/or Python as appropriate; **strict types** where TS is used
- Real persistence (Postgres and/or purpose-built store) — not JSON files as the “database”
- Auth or multi-tenant boundary where it matters for the domain
- Failure modes: retries, idempotency, or crash recovery — pick what the domain demands
- Observability of the system itself (structured logs at minimum)
- `docker compose up` path documented in README
- Tests for the hard core (not only UI smoke)
- Public README: architecture diagram, tradeoffs, how to run, demo link
- Honest metrics only (load test, latency, clients, eval scores) — no invented numbers on resumes

### Explicitly out of scope for “deep” work

- Tutorial clones with renamed UI
- “JWT login + CRUD dashboard” as the main achievement
- Pure animation/marketing sites as the project
- Wrapping one vendor API with no system design
- Claiming six paid-tool clones when only one subsystem is real

---

## How agents should work in this repo / new repos

1. Read `SHARED_CONTEXT.md` + the **one** field brief.
2. Confirm scope with the user only if the brief conflicts with a new instruction.
3. Prefer a **vertical slice that is demoable** before boiling the ocean.
4. Name repos clearly (`sightline`, `gateway`, etc.); keep code quality resume-defendable.
5. When a milestone ships: draft resume bullets + update the field brief **Status** section.
6. Do not edit application resumes to claim unfinished work.
7. Animation/visual polish: allowed as craft on UI surfaces; never substitute for the systems core.

### Deliverables checklist before “resume ready”

- [ ] Live demo URL
- [ ] 2–4 min Loom (architecture + one hard failure)
- [ ] Short write-up: problem → design → tradeoffs → metrics
- [ ] ≥1 measured number
- [ ] Docker Compose (or clear cloud deploy)
- [ ] 3–4 resume bullets drafted
- [ ] Field brief status updated

---

## Related files in paper-trail

- `deep-projects.md` — short roadmap index (points here)
- `docs/PROJECT_BRIEF.md` — per-field agent briefs
- `resume_versions/` — TeX/PDF resumes
- `journal.md` — owner’s day-to-day notes (may be sparse)

---

## Tone for planning docs and READMEs

Direct, technical, tradeoff-aware. No hype. No fake users/metrics. Sound like an engineer who operated the system, not a landing-page generator.
