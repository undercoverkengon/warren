# OB1 Project Brain — Data-Layer Decisions (locked)

> Records the gating decisions from the data-layer kickoff, with their build implications.
> Pairs with `OB1_PROJECT_BRAIN_ARCHITECTURE.md`.

## The four decisions

| # | Decision | Choice |
|---|----------|--------|
| A | Substrate | **Self-hosted Postgres + pgvector** (not Supabase cloud) |
| B | Tenancy grain | **Multi-workspace from day one** |
| C | Role model | **owner / contributor / restricted** |
| D | Capture channels | **Telegram first**, then others; **Signal** as an engagement channel; **≥4 sources** total |

## A. Substrate — self-hosted Postgres + pgvector

**Implication — this is the biggest divergence from OB1's defaults.** OB1 assumes Supabase, which bundles three things we now have to provide ourselves:

- **Auth/identity.** Supabase Auth issues the `auth.uid()` that RLS policies key off. Self-hosting means standing up identity separately (an auth service, or app-level auth that sets a Postgres session variable / role per request). RLS itself is native Postgres and stays — but the *identity feeding it* is now ours to build.
- **Edge Functions.** OB1's serverless logic (capture handlers, MCP glue) runs as Supabase Edge Functions. Self-hosted, these become an app service / our own serverless or long-running endpoints.
- **Operational surface.** Backups, connection pooling (pgBouncer), migrations, and pgvector index management are now our responsibility. OB1's Kubernetes deployment path is the relevant reference here. Fits an existing infra posture (cf. `Prod-Box-Builder`).

**Net:** more control and zero third-party dependency, at the cost of rebuilding the Supabase-provided glue. Plan the first migration as plain Postgres DDL (portable), and treat auth + the edge-function equivalents as their own workstreams.

## B. Tenancy — multi-workspace from day one

- `workspace_id` (FK to a `workspaces` table) on **every** memory-bearing row.
- A `workspace_members` table (`user_id`, `workspace_id`, `role`) — the join that drives RLS and later MCP scoping.
- Lets distinct projects (e.g. MyVinylCatalog, MyExpenseHelper) live as separate workspaces in one brain without a later tenancy retrofit.

## C. Roles — owner / contributor / restricted

- **owner** — full read/write across the workspace; manages membership.
- **contributor** — read/write on shared records; cannot manage membership.
- **restricted** — scoped read/write; **category-gated** (cannot see records tagged as sensitive, e.g. finance/CRM-type categories).
- RLS enforces this: policies join `workspace_members.role` and a record `category`/`sensitivity` column. **Default-deny**, with explicit grants. This is real enforcement on day one, so the policies have teeth to test against.

## D. Capture channels — Telegram → Signal → others (≥4 sources)

**Implication — Telegram and Signal are NOT OB1 stock integrations.** OB1 ships **Slack/Discord** capture. So:

- **Custom capture adapters are required** for Telegram (Bot API — straightforward) and Signal (hard: needs `signal-cli` or a bridge; no first-class bot API). Budget Signal as a real piece of work, not a config toggle.
- **Architectural consequence (decide now, in the schema):** do **not** couple capture to any one provider. Design a **channel-agnostic ingest pipeline** — each adapter normalizes its provider's message into a common record shape, then writes through the same RLS. The schema's `source` field is an open text/enum (not a Slack/Discord-only enum), and provenance carries `source` + external message id + author. Adding the 3rd/4th channel is then additive, never a reshape.
- Target source set: Telegram (first), Signal (engagement), plus two more (likely Slack/Discord, where OB1's stock adapters give a head start).

## Net effect on the first migration

The first DDL should establish: `workspaces`, `users`/identity, `workspace_members` (with the 3-role enum), the core memory table(s) carrying `workspace_id` + `author_id` + open `source` + `category`/`sensitivity` + `created_at` + pgvector embedding, and **default-deny RLS** with workspace + role policies plus a policy-test suite. Auth, capture adapters, and the MCP server are downstream workstreams that all write through this schema.
