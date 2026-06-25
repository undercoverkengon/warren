# OB1 Project Brain — Architecture & Data-Layer Build Plan

> Companion to `OB1_OPEN_BRAIN_ANALYSIS.md`. This is the "how I'd actually deploy it" note,
> scoped to the decision to run OB1 as a **team-owned project brain** kept hard-separated from a
> personal brain — and starting, deliberately, from the **data layer**.

## 0. Scope of this note

Two separations are in play, and they get **different mechanisms**:

- **Personal ↔ Project** — a wall that must never leak, even by accident. → **hard, structural** (separate deployment / database).
- **Member ↔ Member** (inside the project) — a flexible, evolving line. → **policy-based** (Row-Level Security + scoped MCP).

The governing rule: *match the strength of the boundary to the strength of the mechanism.* Don't put the highest-stakes boundary (personal vs business) on the most fallible mechanism (a policy you have to get right every time).

## 1. Target topology — two brains

```
Personal Brain (OB1 instance A)          Project Brain (OB1 instance B)  ← we build this
- single-player (you)                    - team-owned from day one
- private capture channel                - shared team channel (Slack/Discord)
- never exposed to members               - shared MCP server, RLS per-member
        │                                            ▲
        └────────── one-way "promote" ───────────────┘
                   (deliberate copy, never live sync)
```

- Separate Postgres databases (recommended: separate Supabase projects). The wall is physical — no query can cross it.
- The only bridge is **one-way promotion**: curated personal records *copied* into the project brain as deliberate publishes. Never a live sync. The project brain owns its copy; the personal brain stays the private source.

## 2. The ownership inversion (the one thing to change about OB1's defaults)

OB1 upstream is framed single-owner — *"give others scoped access to parts of **your** brain."* For the project brain, **invert the tenancy root**: the anchor is the **workspace/project**, and members are **peers with roles**, not guests on your account.

- No personal brain underneath it. The project brain is self-contained — a teammate sees everything the project needs without reaching into your space.
- This is the same shape as Warren's **plot** ("humans and agents are peer nodes on a shared event log") and **seeds** (shared issue queue). The project brain is the *memory* plane that sits beside those *coordination* planes.

## 3. Why start with the data layer (the payoff)

Starting here is correct, and the benefits compound:

1. **It IS the hard boundary.** The personal/project wall lives at the database level. Getting the data layer right *is* getting the separation right — everything above it inherits the boundary for free.
2. **Tenancy & RLS are cheap to decide now, expensive to retrofit.** Adding `workspace_id` + author provenance + default-deny RLS to an empty schema is trivial; adding it after data and members exist is a migration nightmare and a leak risk.
3. **It unblocks every consumer at once.** Capture bots, MCP server, dashboards, recipes, and agents are all just clients of the same schema. A correct data layer means each of those is additive, not architectural.
4. **It gives you a shared substrate immediately.** Even before the chat-capture and MCP polish, a well-shaped Postgres + pgvector store is queryable by any AI client and any teammate — the "contribute from the start, no per-person tool" goal is a property of the *shared database*, not of the UI on top.
5. **It forces the identity decision up front**, which is the thing teams most often bolt on too late.

## 4. Data-layer build order

> Sync the fork with upstream first — `undercoverkengon/OB1` is a clean 2026-04-17 snapshot and upstream has moved on. Read OB1's `schemas/` to align names/shapes with what follows rather than inventing parallel tables.

**Step 1 — Substrate.** Stand up a dedicated **Supabase project** for the Project Brain (separate from any personal one). Supabase gives Postgres + pgvector + auth + edge functions in one place, which is exactly OB1's assumed stack. (Self-hosted Postgres/k8s is the alternative if you want zero third-party dependency — heavier to operate; pick per your infra appetite. This is **Decision A** below.)

**Step 2 — Tenancy & identity.**
- Real per-member identities via Supabase Auth from day one. **No shared logins.**
- A `workspace_id` (or `project_id`) as the tenancy root on every memory-bearing table.
- A membership/role table (`workspace_members`: user_id, workspace_id, role) — roles drive RLS and, later, MCP scoping.

**Step 3 — Core schema.** Align to OB1's `schemas/`, but the load-bearing requirements are:
- Memory/record tables carry `workspace_id`, `author_id` (provenance), `source` (which channel/recipe captured it), and `created_at`.
- pgvector embedding column(s) for semantic search, with the right index (HNSW/IVFFlat) once volume justifies it.
- Provenance is **not optional** with a team — "who captured this, from where" matters far more than solo.

**Step 4 — RLS as security, not convenience.**
- **Default-deny** on every table; explicit policies grant access scoped to `workspace_id` and role.
- Write **policy tests** (a row from workspace X must be invisible to a member of workspace Y; a restricted-role member can't read finance/CRM rows). Review these like auth code — a policy bug here is a data leak.
- This is OB1's "Row-Level Security" primitive doing its real job; the rigor is on you, not the framework.

**Step 5 — Promotion path (the only personal→project bridge).** A small, explicit "promote" recipe/function that copies a chosen personal record into the project workspace, stamping it as promoted (and dropping any personal-only fields). One-way, manual, auditable. The KB Builder output (brand voice, avatar, frameworks) is the natural *founding* dataset to promote first — it's business-facing by design.

**Step 6 — Expose (after the schema is sound).** The shared MCP server and the shared Slack/Discord channel are clients of the above. The schema must already support scoping (workspace + role) and provenance (author + source) before these go live, so they slot in without reshaping the store.

## 5. Decisions to lock before writing schema

- **A. Substrate:** Supabase cloud project (recommended) vs self-hosted Postgres/k8s.
- **B. Tenancy grain:** single project-brain workspace to start, or multi-workspace (per-project) from day one? (Schema cost is similar; decide whether one "Project Brain" or several.)
- **C. Role model:** the initial roles (e.g. owner / contributor / restricted) and which record categories are role-gated vs workspace-wide.
- **D. Capture channel:** Slack vs Discord as the shared ingress (affects the integration you wire in Step 6, not the schema).

## 6. What comes after the data layer

In dependency order, each is just another client of the schema:
1. **MCP server** — scoped read/write so any member's AI client connects to the shared brain.
2. **Capture** — the shared team channel as the contribution surface (no per-person onboarding).
3. **Recipes** — importers (ChatGPT/Gmail/Twitter) + workflows, all writing through the same RLS.
4. **Dashboards** — SvelteKit/Next.js views, last because they're the thinnest layer.

## 7. Caveats carried forward

- **RLS is security-critical now.** Default-deny, tested policies, reviewed like auth.
- **Governance/exit:** once others contribute, it's not unilaterally your data. Decide ownership, retention, and offboarding *before* the first teammate joins.
- **Duplication is the price of the hard wall.** Promoted overlap lives in two places and can drift — that's the correct trade vs. re-coupling the brains.
- **Licensing (FSL-1.1-MIT):** internal/team/business use is fine; the compete clause only bites if you'd resell OB1-as-a-product.

---

### Open decisions for you

The four items in §5 (substrate, tenancy grain, role model, capture channel) are the gating choices for writing the first migration. **A** and **C** most shape the schema. Tell me where you land on those and I can draft the concrete table + RLS-policy migration (aligned to OB1's `schemas/` once the fork is synced) as the next step.
