# Repo Analysis — `jayminwest/warren` (via `undercoverkengon/warren` fork)

> First in the series; originally delivered in conversation and reconstructed here as a
> document for parity with the other analyses. Same format as the rest.

## Warren — a self-hostable control plane for ephemeral cloud agents

**The one-liner:** You point Warren at a GitHub repo, pick an agent, write a prompt. Warren spawns that agent inside a sandbox, streams its events to a UI live, lets you steer it mid-run, then pushes the resulting workspace branch back. "One container, one volume, one HTTP API, one UI." MIT-licensed, by Jaymin West, ~v0.9.4 (approaching 1.0).

The crucial distinction from the rest of the stack: **Warren is a runtime / control plane.** It spawns and isolates agents; it is not a prompt pack or a process convention.

## How it's built

- **Runtime:** Bun (runs TypeScript directly, no server build step).
- **Language:** strict TypeScript (`noUncheckedIndexedAccess`, no `any`).
- **Storage:** SQLite via `bun:sqlite` (Drizzle ORM; a Postgres path also exists).
- **HTTP:** a single `Bun.serve` process serves both the JSON API and the React SPA.
- **UI:** React + Vite + Tailwind + shadcn-style components, its own `@os-eco/warren-ui` package.
- **Sandboxing:** delegated to **burrow** (a bwrap-based sandbox runtime). Warren orchestrates; burrow isolates. They co-tenant one container and talk over a unix socket; the supervisor spawns `burrow serve` as a sibling process.

## The architecture, by directory

- `src/server/` — HTTP API + SPA host, route handlers, auth, event-streaming bridges.
- `src/supervisor/` — boots burrow, manages git identity/credentials, token budgets.
- `src/burrow-client/` — typed facade over burrow's HTTP client.
- `src/runs/` — run lifecycle: dispatch, cancel, cost caps/analytics, conversation idle/merge.
- `src/registry/` — agent registry incl. the inline built-in agents (`builtins/`).
- `src/cli/` — the `warren` / `wr` CLI.
- `src/plots/`, `src/plan-runs/`, `src/seeds-cli/`, `src/preview/` — opt-in feature subsystems.

## The standalone path vs. the bundled features

A fresh install needs nothing else: the `claude-code` agent ships inline, so a GitHub URL + an Anthropic key dispatches a run end-to-end. On top of that, five os-eco tools are **opt-in built-ins** that light up only when used: **canopy** (prompt libraries), **mulch** (persistent agent memory — `.mulch/`), **seeds** (issue queue — `.seeds/`), **sapling** (alt harness, inline), **plot** (shared human+agent coordination event log — `.plot/` + a `plot_id`). Plus **plan-run**, a serial plan-execution dispatch mode.

## Model selection (recommended, not runtime-auto)

Built-in agents ship with a **pinned model tier** (`src/registry/builtins/model-tiers.ts`): **opus** = `claude-opus-4-8`, **sonnet** = `claude-sonnet-4-6`. Planning/reasoning agents (planner, leveret, nightwatch) get Opus; scoped coding agents (claude-code, pi, sapling, bugwatch, pr-fixer) get Sonnet. Resolution precedence at dispatch: **operator per-run override > `.warren/config.yaml` project defaults > agent frontmatter tier**, with deploy-time env overrides (`WARREN_MODEL_OPUS` / `_SONNET`) for the tier identifiers. So fit is encoded per-agent up front, not auto-selected at runtime.

## Brownfield handling

Warren treats existing repos as the default case: clone untouched, read the project's own `CLAUDE.md`/`AGENTS.md` to resolve the quality gate (`$WARREN_QUALITY_GATE` → documented command → fallback), inherit the project's git pre-commit hooks (`core.hooksPath` arming, opt-out `agent.skipGitHooks`), and only activate bundled features the repo already opted into (`.mulch/`, `.seeds/`, `.plot/`). The one Warren-specific file is an optional `.warren/config.yaml`.

## Host prerequisites (what can't run in the sandbox)

The burrow sandbox runs as an unprivileged UID-1000 user with `/usr`, `/bin`, `/lib`, `/etc`, `/opt` read-only — so the agent **cannot install system packages**. The image ships Bun, Node 22, npm, pnpm, git. Repos that can't run their gate here: non-JS toolchains (Rust/Go/JVM/Python/C/C++), gates needing a live service (Postgres/MySQL/Redis/Docker/testcontainers), hooks shelling out to non-JS tools (Python `pre-commit`, shellcheck), far-off Node version pins, and gates needing network beyond the policy allowlist. Self-contained JS/TS gates run fine.

## What's notable about its discipline

`SPEC.md` is ~179KB (canonical V1 design record); a unified `bun run check:all` (alias `verify`) runs ~13 gates including dup detection, dep checking, file-size/debt/bundle-size/coverage **ratchets**, and a CI-parity check. It dogfoods seeds/mulch/plot for its own development; commit history shows agent-authored work.

## State

~v0.9.4, clean polish toward 1.0 (file decomposition, coverage ratcheting, CLI surface). `undercoverkengon/warren` is a fork; this planning work lived on its `claude/determined-shannon-jxqp04` branch before relocating to the dedicated planning repo.

## Where it sits in the stack

**Control plane / runtime** — the layer that spawns and sandboxes agents. The harnesses (SAW, install-and-maintain) define *how* agents work; the memory plane (OB1, mulch) holds *what* they recall; kb-builder produces *inputs*. Warren is what actually *runs* the agent that reads the memory and follows the process.
