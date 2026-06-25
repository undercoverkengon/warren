# Relocation Handoff — planning docs → two dedicated repos

> Durable execution plan for moving the planning work off the `warren` fork branch
> into two focused repos. Written so a correctly-scoped session can execute in one pass.
> Source branch: `undercoverkengon/warren` @ `claude/determined-shannon-jxqp04`.

## Decisions (locked)

- **Two repos**, split by lifecycle:
  - `agentic-stack-planning` — the planning hub (living docs, keeps expanding). **Canonical home** for the architecture + decisions docs.
  - `project-brain` — the OB1 implementation (code/schema/infra). **References** the planning hub for rationale; does not duplicate it.
- Both **private**, initialized with a README.
- Pattern going forward: one planning hub indexes many implementation repos; a new repo only per *substantial* implementation.

## Scope requirement for the executing session (critical)

The session that performs the move must have **all three repos in scope at once**:
`undercoverkengon/warren` (source) **+** `agentic-stack-planning` **+** `project-brain` (destinations).
A session scoped only to the new repos **cannot read the source docs** in warren. Note also that
this environment's GitHub integration **cannot create repositories** (`403 Resource not accessible by
integration`) — the repos must be created by the user first.

## Step 1 — Populate `agentic-stack-planning`

Target layout and file mapping (source path in warren → destination path in planning hub):

| Source (warren root) | Destination (`agentic-stack-planning`) |
|---|---|
| `WARREN_ANALYSIS.md` | `analyses/warren.md` |
| `SAFE_AGENTIC_WORKFLOW_ANALYSIS.md` | `analyses/safe-agentic-workflow.md` |
| `AI_SOLOPRENEUR_KB_BUILDER_ANALYSIS.md` | `analyses/ai-solopreneur-kb-builder.md` |
| `INSTALL_AND_MAINTAIN_ANALYSIS.md` | `analyses/install-and-maintain.md` |
| `OB1_OPEN_BRAIN_ANALYSIS.md` | `analyses/ob1-open-brain.md` |
| `OB1_PROJECT_BRAIN_ARCHITECTURE.md` | `architecture/ob1-project-brain.md` |
| `DATA_LAYER_DECISIONS.md` | `decisions/data-layer-decisions.md` |

Content is copied verbatim (rename only). Then add `README.md`:

```markdown
# Agentic Stack — Planning

Living planning hub for an agentic development stack and the systems built from it.
The thinking lives here; each substantial build spins out into its own implementation repo.

## Analyses
Repo-by-repo evaluations, one layer of the stack each:
- [Warren](analyses/warren.md) — control plane / runtime that spawns & sandboxes agents
- [SAFe Agentic Workflow (SAW)](analyses/safe-agentic-workflow.md) — multi-provider role-based process harness
- [install-and-maintain](analyses/install-and-maintain.md) — deterministic-script + agentic-oversight pattern
- [AI Solopreneur KB Builder](analyses/ai-solopreneur-kb-builder.md) — content/prompt curriculum (inputs)
- [OB1 / Open Brain](analyses/ob1-open-brain.md) — self-hosted AI-agnostic memory / data plane

## Architecture
- [OB1 Project Brain](architecture/ob1-project-brain.md) — two-brain topology + data-layer build plan

## Decisions
- [Data-layer decisions](decisions/data-layer-decisions.md) — locked choices for the project-brain build

## Implementations
- **project-brain** — the OB1 project-brain build (separate repo). Rationale lives here; code lives there.

## Layering (how the pieces compose)
Runtime (Warren) runs the agent · Harness (SAW / install-and-maintain) defines the process ·
Memory plane (OB1 / mulch) holds the context · Content (KB Builder) seeds it.
```

## Step 2 — Scaffold `project-brain`

Minimal skeleton; real work (first migration) follows per `decisions/data-layer-decisions.md`:

```
project-brain/
  README.md
  docs/.gitkeep
  db/migrations/.gitkeep
```

`README.md`:

```markdown
# Project Brain

Implementation of a team-owned, self-hosted OB1 "project brain" — a multi-workspace,
RLS-scoped memory/data plane kept hard-separated from any personal brain.

> **Rationale and design intent live in the planning hub**, not here:
> - Architecture: `agentic-stack-planning/architecture/ob1-project-brain.md`
> - Locked decisions: `agentic-stack-planning/decisions/data-layer-decisions.md`
> This repo holds the as-built (schema, migrations, services) and ADRs for where it diverges from the plan.

## Locked data-layer decisions (summary)
- **Substrate:** self-hosted Postgres + pgvector (not Supabase cloud)
- **Tenancy:** multi-workspace from day one (`workspace_id` everywhere)
- **Roles:** owner / contributor / restricted (category-gated), default-deny RLS
- **Capture:** Telegram first, Signal as engagement, ≥4 sources via a channel-agnostic ingest pipeline

## Status
Bootstrapping the data layer. First migration: workspaces, identity, workspace_members (3-role enum),
core memory table (workspace_id + author_id + open `source` + category/sensitivity + pgvector embedding),
default-deny RLS + policy tests.

## Build order
data layer → auth/identity → capture adapters → MCP server → dashboards.
```

## Step 3 — Close out `warren`

Once both repos above are populated and verified:
1. `git rm` the 7 planning docs and this `RELOCATION_HANDOFF.md` from warren.
2. Add `PROJECT_BRAIN_PLANNING_POINTER.md`:

```markdown
# Planning relocated

The agentic-stack planning and OB1 project-brain design work that briefly lived on this
fork branch has moved to dedicated repos:

- **Planning hub:** `undercoverkengon/agentic-stack-planning`
- **Implementation:** `undercoverkengon/project-brain`

This fork (`undercoverkengon/warren`) is upstream `jayminwest/warren`; it was only a scratch
surface for that planning and carries no project-brain work of its own.
```
3. Commit + push.

## Notes carried forward (for the implementation repo)

- Self-hosting replaces Supabase's bundled glue: **auth/identity** (the `auth.uid()` RLS keys off),
  **edge functions**, backups/pooling are now ours. DDL stays portable; auth is its own workstream.
- **Telegram + Signal are not OB1 stock** (it ships Slack/Discord). Design a channel-agnostic ingest
  pipeline now; budget Signal as real work (`signal-cli`/bridge, no bot API).
