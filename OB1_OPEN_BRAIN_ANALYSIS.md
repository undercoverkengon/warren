# Repo Analysis — `undercoverkengon/OB1` (Open Brain)

> Companion analysis to the Warren, SAW, install-and-maintain, and kb-builder tours, in the same format.
> Source: https://github.com/undercoverkengon/OB1 (public, read via web — the git endpoint is blocked by this session's egress policy).

## What this repo actually is

`undercoverkengon/OB1` is **your undiverged public fork** of `NateBJones-Projects/OB1` ("Open Brain"), by Nate B Jones — created 2026-04-17 and not modified since, so it mirrors upstream byte-for-byte. Upstream is a sizable, popular project (~3,900 stars, ~750 forks, 241 commits). This analysis describes the upstream content your fork carries.

## Open Brain — a self-hosted, AI-agnostic persistent memory layer for your thinking

**The one-liner:** "The infrastructure layer for your thinking." Open Brain is a **personal knowledge / memory backend** that gives every AI tool you use — Claude, ChatGPT, Cursor, future ones — shared access to the *same* persistent memory, through open protocols and open infrastructure rather than a SaaS chain. Its slogan: **"One database, one AI gateway, one chat channel — any AI plugs in. No middleware, no SaaS."**

This is a different *layer* from the previous four repos. Warren is a runtime, SAW and install-and-maintain are process/harness patterns, kb-builder is a content curriculum — **OB1 is a data plane**: the place your context lives so agents can read and write it. Its closest cousin in your stack is Warren's bundled **mulch** (persistent agent memory), but OB1 is broader and human-facing — a personal "second brain" infrastructure, not just agent expertise files.

## How it's built

- **Languages:** TypeScript 58.9% / Python 21.1% / JavaScript 8.7% / PL/pgSQL 7.5% / Svelte 2.2% / CSS 1.4%. The PL/pgSQL share is meaningful — a lot of the logic lives in Postgres itself (functions + Row-Level-Security policies).
- **Data layer:** **PostgreSQL + pgvector** (vector search over your memories), hosted on **Supabase** or self-hosted on **Kubernetes**.
- **AI access:** **MCP servers** expose scoped slices of your brain to AI clients via a standard protocol; **Supabase Edge Functions** for serverless logic.
- **Capture:** **Slack / Discord bots** as the single "chat channel" ingestion point for thoughts and context.
- **Frontends:** dashboards in both **SvelteKit** and **Next.js**.
- **Server:** a **Deno** TypeScript service (`server/` has `deno.json` + `index.ts`).
- **License:** **FSL-1.1-MIT** (Functional Source License, MIT-after-two-years). Note this is *source-available*, not OSI-open-source on day one — a deliberate contrast to the MIT/CC-BY licensing of the other repos. It's anti-commercial-competition for a window, then converts to MIT.
- **No releases** published; development is commit-stream on `main` (241 commits).

## The architecture, by directory

```
primitives/    Foundational concepts taught once, reused everywhere (RLS, shared MCP)
extensions/    The six-step learning path — progressively richer brains
recipes/       Standalone importers + workflows (ChatGPT/Twitter/Gmail import, auto-capture)
skills/        Reusable prompt packs for Claude Code / AI clients
.claude/skills Claude-Code-native skill definitions
server/        Deno TypeScript service (MCP / gateway entry: deno.json + index.ts)
schemas/       Database schemas (Postgres / pgvector)
integrations/  External service connectors (Slack, Discord, etc.)
dashboards/    SvelteKit + Next.js UIs
resources/     Supporting assets
docs/          Guides (incl. the ~45-min Getting Started)
README.md  CLAUDE.md  CONTRIBUTING.md  SECURITY.md  LICENSE.md  (+ community files)
```

The fact that it ships both a `CLAUDE.md` and `skills/` + `.claude/skills/` means OB1 is itself **agent-tooling-aware** — it's designed to be operated *by* Claude Code, not just queried by it.

## The core model — primitives that compound, taught through extensions

Open Brain is structured as a **curriculum over real infrastructure** — you don't just deploy it, you learn it by building progressively harder "brains":

**Primitives** ("concepts that compound" — learn once, reuse across extensions):
- **Row-Level Security** — Postgres policies for multi-user data isolation (used in Extensions 4–6).
- **Shared MCP Server** — giving others *scoped* access to parts of your brain (Extension 4).

**Extensions** — a six-step path from beginner to advanced, each adding a primitive:
1. **Household Knowledge Base** — capture home facts for agent recall *(beginner)*
2. **Home Maintenance Tracker** — scheduling + history *(beginner)*
3. **Family Calendar** — multi-person schedule coordination *(intermediate)*
4. **Meal Planning** — recipes, plans, grocery lists *(intermediate)*
5. **Professional CRM** — contacts tied to captured thoughts *(intermediate)*
6. **Job Hunt Pipeline** — applications + interview tracking *(advanced)*

**Recipes** — standalone capabilities to import external data (ChatGPT history, Twitter, Gmail) or run workflows (auto-capture, meal planning).

**Skills** — reusable prompt packs (competitive analysis, financial review, meeting synthesis) you run in Claude Code against your brain.

So the loop is: **capture** (Slack/Discord) → **store** (Postgres + pgvector) → **expose** (MCP, scoped by RLS) → **use** (any AI client, via skills/recipes).

## What's notable

- **Open-protocol, anti-lock-in by design.** MCP + Postgres + open infra means your cognitive data stays portable across tools — the explicit antithesis of per-app SaaS memory silos.
- **Database-as-application.** A 7.5% PL/pgSQL share and RLS-as-a-primitive show real logic pushed into Postgres; security and multi-tenancy are schema-level, not bolted on.
- **Taught, not just shipped.** The primitives/extensions structure makes it a learning system — closer in spirit to install-and-maintain (a pattern made tangible) than to a drop-in product. There's a ~45-minute Getting Started and a deliberate difficulty ramp.
- **Source-available licensing.** FSL-1.1-MIT is a real differentiator to flag — fine for personal/internal use, with commercial-compete restrictions that lapse to MIT after two years.

## State right now

A healthy, actively-developed upstream (241 commits, ~3.9k stars) with no formal releases. **Your fork is a clean snapshot from 2026-04-17 with no local changes** — if you intend to build on it, you'd want to sync it with upstream before starting, since two months of upstream commits have likely landed since.

## How it relates to your stack

OB1 sits in the same cluster of agentic-infra forks you've collected (warren, mulch, seeds, canopy, pi, install-and-maintain, …) — and it's the **memory/data plane** of that set:

| Layer | Repo(s) | Role |
|-------|---------|------|
| Control plane / runtime | **Warren** | Spawns & sandboxes agents |
| Process / harness | **SAW**, **install-and-maintain** | How agents work (roles, gates, deterministic+agentic) |
| **Memory / data plane** | **OB1**, mulch | Where context lives so agents recall it |
| Content / inputs | **kb-builder** | Material you feed into the above |

The natural synthesis: OB1 (or mulch) holds your persistent context; a harness (SAW / install-and-maintain) defines the process; Warren runs the agent that reads OB1 over MCP and acts. kb-builder produces the kind of brand/business context you'd *seed* OB1 with.

### A Warren-compatibility note (since that thread is open)

If you ever wanted a **Warren agent** to work *on* OB1, it would hit several of the host-prereq walls from the earlier analysis: the `server/` runs on **Deno** (not in Warren's image — only Bun + Node), there's a **Python 21%** share (no Python in the image), and its tests would expect a live **Postgres + pgvector / Supabase** service the sandbox doesn't run. So OB1 is a *great* memory backend to plug into Warren-driven agents, but a *poor* fit as a target repo for them without a custom image. Treat it as infrastructure to consume, not a project to dispatch against.

---

### A caveat on sourcing

Assembled from the web-visible README, the repo tree, and the `server/` listing — the git-clone endpoint is blocked by this session's egress policy, so I did not read the schemas, edge functions, or extension code in full. Stats (commits, stars, language split) are as GitHub reports them for the fork/upstream. Because your fork is undiverged, everything here describes upstream `NateBJones-Projects/OB1`. If you sync the fork and add it to a session's scope, I can do a schema- and server-level deep read.
