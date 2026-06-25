# Repo Analysis — `bybren-llc/safe-agentic-workflow` (SAW)

> Companion analysis to the Warren tour, in the same format.
> Source: https://github.com/bybren-llc/safe-agentic-workflow (public, read via web — the git endpoint is blocked by this session's egress policy).
> Per the user's note, this deliberately does **not** dwell on explaining SAFe itself — only on how the repo is built and operates.

## SAW — a multi-provider, process-as-service harness for team-based AI development

**The one-liner:** SAW ("SAFe Agentic Workflow") is a production-tested AI agent *harness* that imposes a structured, role-based delivery process on top of off-the-shelf AI coding CLIs. Instead of a single autonomous agent, you get a coordinated "team" of specialized agent profiles moving a ticket through explicit gates from requirements to merge. Its stated philosophy is **"process as service, not control"** — guardrails and workflow automation rather than an autonomous actor operating unsupervised. It's MIT-licensed, currently **v2.10.0** ("Multi-Domain Sync Engine", March 2026), ~111 stars / 28 forks, 16 releases, and advertises 5 months of use / ~2,193 commits / 382-of-382 tests passing.

The crucial distinction from Warren: **SAW is not a runtime.** It doesn't spawn or sandbox anything. It is a *bundle of configuration, prompts, and scripts* that you copy into your own project to reshape how an existing AI CLI (Claude Code, Gemini, Codex, Cursor) behaves. Warren orchestrates and isolates agents; SAW choreographs the process those agents follow.

## How it's built

- **Primary "language" is prompts-and-scripts, not application code.** GitHub measures it as **Shell 97.9% / Python 1.4% / JavaScript 0.7%**. The actual product is markdown (commands, skills, agent profiles with YAML frontmatter) plus shell automation — "prompt-as-code." There is no compiled server, no UI, no database.
- **Distribution model:** copy a provider directory into your repo (`cp -r .claude/ /your-project/.claude/`), fill in placeholders in a `SETUP.md`, then drive it with slash commands (`/start-work TICKET-123`).
- **Multi-provider by design**, with one harness per platform:
  - **Claude Code** (Anthropic) — deepest integration, the production-validated path
  - **Gemini CLI** (Google) — uses shell/file-injection features
  - **Codex CLI** (OpenAI) — natural-language interaction
  - **Cursor IDE** (Anysphere) — `.mdc` rule files, glob-activated background agents
- **Integration substrate:** **MCP** (Model Context Protocol) for tool access, and **Linear** as the system-of-record for tickets and delivery evidence.
- **Toolchain for the harness's own tests/CI** is JS/TS-based: `yarn ci:validate` runs TypeScript type-checking, ESLint (flat config), Prettier, and unit tests — even though the shipped artifacts are mostly shell + markdown.

## The architecture, by directory

```
.claude/        Claude Code harness — commands, skills, agent profiles, configs
.gemini/        Gemini CLI harness — TOML commands + settings
.codex/         Codex CLI harness — TOML config + README
.cursor/        Cursor IDE rules — .mdc files, glob-based activation
.agents/        Shared, cross-provider skills
dark-factory/   tmux-based persistent autonomous agent teams (remote headless)
docs/           Whitepapers, onboarding, the workflow SOP, methodology
examples/       Reference implementations
patterns/       Reusable workflow patterns
scripts/        Setup + sync automation (multi-domain sync engine)
templates/      Specification templates
tests/          Test suite (382 tests)
```

The provider directories (`.claude/`, `.gemini/`, `.codex/`, `.cursor/`, `.agents/`) are the heart of the repo — each is a parallel rendering of the same process for a different CLI. `scripts/` houses the **sync engine** that keeps those parallel harnesses (and downstream projects) aligned — the headline feature of the current v2.10.0 release.

## The core model — three layers + an 11-role team + a gated contract

This is SAW's analog to Warren's feature set. Three pieces compose it:

**1. Three architectural layers** (increasing autonomy, decreasing human friction):
- **Layer 1 — Hooks:** automatic guardrails; format checks and quality blockers that fire without being asked.
- **Layer 2 — Commands:** ~**24 slash commands** the user invokes (`/start-work`, `/pre-pr`, …) covering startup, validation, deployment, coordination.
- **Layer 3 — Skills:** ~**18 model-invoked skills** that load automatically (via "Skills 2.0" frontmatter triggers) — pattern discovery, API validation, doc generation, etc.

**2. Eleven specialized agent roles** with explicit ownership boundaries and handoff "exit states." The non-negotiable ones are independence gates:
- **BSA** (requirements/AC/DoD) — *not collapsible*; owns the upstream "stop-the-line" gate.
- **System Architect** — pattern validation + Stage-1 PR review.
- **BE / FE / Data Engineers** — implementation; exit state "Ready for QAS"; they write code but **never create PRs or merge**.
- **QAS** (Quality Assurance) — the **gate owner** with unlimited iteration authority; **not collapsible** (must run as an independent subagent to prevent self-review).
- **Security Engineer** — **not collapsible**; independent security/RLS audit.
- **RTE** (Release Train Engineer) — **PR shepherd**: opens PRs and watches CI but writes no product code and does not merge; this is the one role that **may be collapsed** into the implementer.
- Plus **TDM** (delivery manager / evidence tracking), **TW** (docs), **DPE** (test data).

**3. The vNext workflow contract (v1.4)** — a linear, gated state machine from ticket to merge:
- **Stop-the-Line gate:** an implementer cannot start until acceptance criteria + definition-of-done exist; missing requirements halt work until BSA supplies them.
- **QAS quality gate:** can bounce work back to specialists indefinitely; nothing proceeds without explicit approval.
- **Three-stage PR review:** (1) System Architect validates patterns/security → (2) "ARCHitect-in-CLI" comprehensive architectural review → (3) Human (HITL) final review + merge.
- **Complex-code review loop (Method 4):** when a change exceeds size thresholds (bash >100 lines, TS >200 lines), a mandatory Architect review loop activates before a PR is allowed.
- **Chain of custody via exit states:** `Ready for QAS` → `Approved for RTE` → `Ready for HITL Review` → `MERGED`, with evidence posted to Linear at each hop.

## What's notable about the project's discipline

- **Evidence-based delivery:** every gate demands verifiable artifacts logged in Linear rather than trust — the same "gates are terminal, not advisory" ethos Warren's agents carry, but enforced organizationally across roles.
- **Independence as a hard constraint:** QAS and Security never review their own work; the harness structurally forbids the collapse that would let an implementer mark its own homework.
- **Real CI rigor for a "prompt" repo:** contributors must pass `yarn ci:validate` (tsc + ESLint flat + Prettier + unit tests), and the CI pipeline adds branch/PR-name structure validation, rebase-only linear-history enforcement, unit/integration/E2E tests, secret scanning, a production build, and conflict detection on sensitive files. Commit/branch naming is SAFe-ticket-formatted.
- **Versioned, changelog-tracked harness:** 16 releases, a dedicated `HARNESS_CHANGELOG.yml`, and a 382/382 test badge — it's maintained like a product, not a snippet collection.
- **Explicitly research-grounded:** the README claims direct implementation of patterns from six Anthropic engineering papers on agent harnesses, skills, and code-execution-with-MCP.

## Dark Factory (the autonomous tier)

`dark-factory/` is a self-contained module for running **persistent, autonomous agent teams on a remote headless machine using tmux**. Each role (TDM lead, BE/FE devs, QAS, RTE) runs its own AI CLI instance in an isolated tmux pane, with the TDM coordinating per SAFe; the operator observes over SSH (via Cursor). It supports Claude Code or Codex runtimes, sizes teams by story/feature/epic, keeps per-agent logs + git worktrees on the remote, and integrates with GitHub merge-queue enforcement. The docs frame it as mature/operational rather than a toy — this is the closest SAW gets to Warren's "always-on, server-side agents" territory, but built from tmux + SSH rather than a sandbox runtime.

## State right now

**v2.10.0**, MIT-licensed, actively released (16 versions), with a green 382-test badge and multi-provider support badges. The current release centers on the **multi-domain sync engine** in `scripts/` — keeping the four provider harnesses and downstream consumer projects in lockstep, which is the maintenance problem a four-target prompt harness inevitably runs into.

## How it relates to Warren (positioning)

They solve adjacent but different problems and could even compose:

| | **Warren** | **SAW** |
|---|---|---|
| **What it is** | A runtime/control plane | A process harness (config + prompts + scripts) |
| **Core artifact** | A TypeScript service + UI + SQLite | Markdown commands/skills + shell, copied into your repo |
| **Isolation** | burrow sandbox (bwrap), per-run | None — runs in your normal CLI/checkout |
| **Concurrency model** | One ephemeral agent per run, steerable | A *team* of role-agents passing a ticket through gates |
| **Provider scope** | Anthropic-tier defaults (pi/claude-code/sapling) | Claude Code, Gemini, Codex, Cursor |
| **Source of truth** | Its own DB + Plot/Seeds event logs | Linear tickets + evidence |
| **Autonomy ceiling** | Server-spawned runs that push a branch | Dark Factory: tmux teams on a remote box |

In principle SAW's *process* (roles, gates, exit states) could sit **inside** a Warren-dispatched agent's instructions, while Warren provides the sandbox + branch lifecycle SAW lacks. Worth flagging as a follow-up if you're evaluating whether to adopt one, the other, or both.

---

### A caveat on sourcing

This was assembled from the repo's web-visible README, `AGENTS.md`, the workflow SOP, `CONTRIBUTING.md`, and the GitHub repo page — the git-clone endpoint is blocked by this session's egress policy, so I did not read the full file tree or run anything. Figures (commit count, test totals, stars) are as the project reports them. If you want a file-level deep dive (actual hook scripts, skill frontmatter, the sync engine internals), the cleanest path is to start a session scoped to this repo so I can read the tree directly.
