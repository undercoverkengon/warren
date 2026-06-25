# Repo Analysis — `disler/install-and-maintain`

> Companion analysis to the Warren, SAW, and kb-builder tours, in the same format.
> Source: https://github.com/disler/install-and-maintain (public, read via web — the git endpoint is blocked by this session's egress policy).

## A reference implementation of the "deterministic script + agentic oversight" pattern for Claude Code

**The one-liner:** A small, opinionated demo repo (by disler / IndyDevDan) that shows how to automate a project's **install and maintenance** workflows by pairing **deterministic scripts** (run as Claude Code hooks) with **agentic supervision** (an agent that runs the *same* scripts, then analyzes or asks questions). Its thesis, stated up front: **"Agents + code beats agents. Agents + code beats just code."** The README frames the result as "a living document that executes" — the script is the source of truth, and the agent is layered on top for diagnosis and interaction, never for execution.

Unlike the kb-builder (pure content) but like SAW, this is a **harness pattern** — except it's a working, runnable **demonstration**: a real full-stack sample app exists purely so there's something concrete to install and maintain. It's an educational/reference repo (100 stars, 38 forks, **1 commit**, no releases, no license file visible), not a maintained product.

## How it's built

- **Languages:** Python 70.5% / Vue 19.5% / CSS 4.3% / Just 3.6% / HTML 1.2% / TypeScript 0.9%. The Python majority is the hook scripts + the FastAPI backend; Vue/TS is the frontend.
- **The sample app** (the thing being installed/maintained):
  - **Backend:** Python 3.11+ managed by **uv**, FastAPI, SQLite (`starter.db`), run via `uvicorn`.
  - **Frontend:** Vite + Vue 3 + TypeScript, run via `npm`.
- **The harness:** a `.claude/` directory (commands, hooks, agents, `settings.json`) plus a **`justfile`** acting as the command registry.
- **Requirements:** Claude Code, Python 3.11+ with uv, Node.js 18+, and the `just` command runner.
- **Practical note:** the justfile's Claude recipes invoke `claude --model opus --dangerously-skip-permissions …` — it pins **Opus** and **skips the permission prompt**. Fine for a throwaway demo box; worth flagging before reusing the recipes anywhere real.

## The architecture, by directory

```
.claude/
  agents/        Agent configuration(s)
  commands/      install.md, install-hil.md, maintenance.md, prime.md  (slash commands)
  hooks/         session_start.py, setup_init.py, setup_maintenance.py (the deterministic scripts)
  settings.json  Harness config (hook wiring, etc.)
ai_docs/         Reference docs the agent can pull in for context
apps/
  backend/       FastAPI + uv + SQLite
  frontend/      Vite + Vue 3 + TypeScript
images/          README assets
justfile         Command registry (the user-facing entry point)
.env.sample      Env template
```

The conceptual centre is the relationship between **`.claude/hooks/*.py`** (deterministic execution) and **`.claude/commands/*.md`** (agentic wrappers) — both ultimately drive the same Python scripts.

## The core model — three execution modes over one set of scripts

This is the whole idea, and it's elegant. The same underlying scripts (`setup_init.py`, `setup_maintenance.py`) are reachable through a **spectrum** of three modes, differing only in *whether an agent supervises afterward* and *whether it asks questions beforehand*:

| Mode | What it adds | `just` recipe | Underlying action |
|------|--------------|---------------|-------------------|
| **Deterministic** | Nothing — hook runs, exits | `just cldi` (init), `just cldm` (maintain) | Run the Python script, period |
| **Deterministic + Agentic** | Agent runs the script, then **diagnoses / reports** | `just cldii`, `just cldmm` | Same script + analysis |
| **Interactive (HIL)** | Agent **asks clarifying questions** mid-workflow | `just cldit` (`/install true`) | Same script + dialogue |

Plus `just reset` to undo an install (removes `.venv`, `starter.db`, `node_modules`, logs), and `just fe` / `just be` to install-and-run each app directly.

Slash commands map onto the same: `/install`, `/install true` (the human-in-the-loop variant, `install-hil.md`), `/maintenance`, and `/prime` (load context). The hooks (`session_start.py`, `setup_init.py`, `setup_maintenance.py`) are the deterministic floor every mode sits on.

**When to use which** (per the README):
- **CI/CD** → deterministic only (preserves reliability/repeatability).
- **Fresh local setup** → basic deterministic.
- **Troubleshooting / dependency maintenance** → agentic (diagnosis catches issues a bare script wouldn't).
- **Unfamiliar codebase** → interactive (the agent educates by asking).

## What's notable about its discipline

- **"Script is the source of truth."** The deterministic and agentic paths can't diverge because they execute the *identical* script — the agent layer only observes and converses. This is the cleanest articulation of a principle Warren also leans on (hooks/gates are real, the agent doesn't get to reinterpret them).
- **CI-compatibility preserved by construction.** Because the base mode is a plain hook with no agent, the same automation that a human runs interactively also runs unattended in CI — no second implementation to maintain.
- **Agent confined to analysis, never execution.** A deliberate safety boundary: flexibility (questions, diagnosis) is added without surrendering determinism where it matters.
- **Teaching artifact, not a product.** One commit, no license, no releases, no tests. The value is the *pattern*, and the sample app is scaffolding to make it tangible. Judge it as a blog-post-in-repo-form (disler's usual format), not as maintained software.

## State right now

A single-commit reference repo, popular for its size (100★/38 forks), demonstrating one tight idea end-to-end with a runnable full-stack example. No versioning, no license, no ongoing activity beyond the initial publish.

## How it relates to the others

Now the set spans four distinct layers:

| | **Warren** | **SAW** | **install-and-maintain** | **kb-builder** |
|---|---|---|---|---|
| **Category** | Runtime / control plane | Process harness (multi-provider) | Pattern demo / harness (Claude Code) | Content / prompt curriculum |
| **Artifact** | TS service + UI + SQLite | Markdown cmds/skills + shell | Python hooks + sample app + justfile | Markdown prompts only |
| **Core idea** | Spawn & sandbox steerable agents | Role-based gated delivery | Deterministic scripts + agentic oversight | Self-interview → KB files |
| **Scope** | Full platform | Org-scale workflow | One pattern, deeply | One curriculum |
| **Maturity** | v0.9.4, heavy CI | v2.10.0, 16 releases, 382 tests | 1 commit, demo | 1 commit, demo |
| **Run by** | The platform | A team of role-agents | A dev locally / in CI | One solopreneur |

The most direct tie-in is to **Warren**: install-and-maintain's "script is the source of truth, agent supervises but never replaces execution" is *exactly* the contract Warren enforces with its quality gates and git-hook arming. This repo is a clean, minimal illustration of the principle Warren operationalizes at platform scale — and its hook scripts are the kind of thing you'd point a Warren agent's `$WARREN_QUALITY_GATE` or `prepare`/`core.hooksPath` setup at.

---

### A caveat on sourcing

Assembled from the web-visible README, the `justfile`, and the `.claude/` directory tree (commands + hooks filenames) — the git-clone endpoint is blocked by this session's egress policy, so I did not read the hook scripts or command bodies in full. One naming note: `install-hil.md` is almost certainly **H**uman-**i**n-the-**L**oop (the interactive `/install true` mode), not the "hardware-in-the-loop" a naive reading might suggest. If you fork it into your profile and add it to a session's scope, I can do a line-level read of the hooks and commands.
