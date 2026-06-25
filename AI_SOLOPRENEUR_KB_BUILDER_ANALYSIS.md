# Repo Analysis — `AI-Captains-Academy/ai-solopreneur-kb-builder`

> Companion analysis to the Warren and SAW tours, in the same format.
> Source: https://github.com/AI-Captains-Academy/ai-solopreneur-kb-builder (public, read via web — the git endpoint is blocked by this session's egress policy).

## A guided prompt sequence for building an "AI-ready" business knowledge base

**The one-liner:** A structured, copy-paste **prompt curriculum** that walks a solopreneur through interviewing themselves with an AI assistant and compiling the answers into a reusable business knowledge base — brand voice, customer avatar, positioning, story, design system, and operating frameworks — as a set of markdown files any future AI tool can reference.

The defining fact, stated plainly: **this is not software.** There is no code, no runtime, no build, no tests. It's documentation — 35 markdown prompts and a README — licensed **CC BY 4.0** (a content license, not a software license like the MIT used by Warren and SAW). It's also brand new and unproven: **1 commit, no releases, 0 stars/forks.** Treat it as a *content artifact / methodology*, not a tool.

## What it's made of (the "how it's built" analog)

- **Format:** pure Markdown. Each workflow step is a folder containing a single `prompt.md` you paste into a chat-capable AI.
- **Intended runtime:** any conversational LLM — "optimized for Claude, functions with other capable LLMs."
- **Prerequisites for the user:** a conversation-capable AI tool, a folder to store outputs, and "candid self-reflection." Estimated completion: **1–2 weeks**.
- **No dependencies, no install.** You clone/download it and read it; the "execution" happens inside whatever AI chat you use.
- **A standard prompt-file anatomy** (from the sample `01-kb-structure-setup/1.1-create-kb-files/prompt.md`): a **title/goal**, a **role/expertise declaration** ("you are a specialist in…"), a **task definition** (the concrete deliverable), an **initial action gate** (e.g. "ask for the full directory path before proceeding"), and **file specifications** (named outputs with numbered-prefix conventions). Not every prompt carries an explicit expected-output / save / test block — the sample reads more as a directive than a rigid template.

## The architecture, by directory

The repo *is* its directory structure — nine numbered modules, executed in order, 35 prompts total:

```
01-kb-structure-setup/        Scaffold the KB files/folders (e.g. 1.1-create-kb-files/prompt.md)
02-avatar-customer-research/  Customer avatar + emotional mapping        (8 prompts)
03-competitive-analysis/      Rival research, positioning gaps           (6 prompts)
04-brand-voice-messaging/     Tone patterns + communication strategy     (7 prompts)
05-design-system/             Visual standards: color, type, spacing      (5 prompts)
06-brand-story/               Transformation narrative + values          (4 prompts)
07-business-frameworks/       Decision-making + operating patterns        (3 prompts)
08-integration-testing/       Verify the AI can actually use the KB       (3 prompts)
09-application-prompts/       Put the finished KB to work                  (5 prompts)
README.md  LICENSE (CC BY 4.0)
```

Each numbered module contains sub-step folders (e.g. `1.1-create-kb-files/`), each holding a `prompt.md`. The numbering enforces sequence — this is a linear curriculum, not a menu.

## The core model — a four-beat pattern repeated per module

This is the closest thing to an "engine." Every section follows the same loop:

> **Discovery** (conversational Q&A with the AI) → **Documentation** (structured output) → **Save** (compile into a KB file) → **Test** (verify the AI can use the saved file)

The product of running the whole sequence is a small set of durable markdown artifacts (six–seven, depending on how you count):

- `avatar-profile.md` — customer demographics + emotional mapping
- `brand-guide.md` — market positioning + core identity
- `competitive-analysis.md` — rival research + opportunity gaps
- `brand-voice-and-messaging.md` — tone + communication strategy
- `brand-design-system.md` — colors, typography, spacing
- `brand-story.md` — transformation narrative + values
- `business-frameworks.md` — decision-making + operations

Module 08 ("Integration & Testing") is the conceptual mirror of a quality gate: it exists to confirm the assembled KB is actually consumable by an AI before module 09 starts applying it.

## What's notable about its discipline

- **Sequence + naming as the only "enforcement."** There are no hooks, no CI, no validation scripts (it's content) — discipline comes entirely from the numbered ordering and consistent file-naming conventions. Compared to Warren's 13-gate `check:all` or SAW's role-based gates, governance here is conventional, not mechanical.
- **A built-in verification step.** The Discovery→Documentation→Save→**Test** loop and the dedicated module 08 show an intent toward "don't just generate, confirm it's usable" — a lightweight echo of the evidence/verification ethos in the other two repos.
- **Output-first design.** It's organized around the six artifacts it produces, which makes the deliverable unambiguous.
- **Maturity caveat.** One commit, no releases, no stars, no version history. There's no track record, changelog, or community signal — the opposite end of the spectrum from SAW's 16 releases / 382 tests or Warren's v0.9.4 maturity.

## State right now

A single-commit, CC BY 4.0 content repo with a complete-looking nine-module / 35-prompt curriculum and a README that explains the what/why/how. No releases, no issues, no activity beyond the initial publish.

## How it relates to Warren and SAW

These three are now clearly three different *layers* of the AI-development stack:

| | **Warren** | **SAW** | **kb-builder** |
|---|---|---|---|
| **Category** | Runtime / control plane | Process harness (config + prompts + scripts) | Content / prompt curriculum |
| **Artifact** | TS service + UI + SQLite | Markdown commands/skills + shell | Markdown prompts only |
| **License** | MIT (software) | MIT (software) | CC BY 4.0 (content) |
| **Executes** | Sandboxed agents it spawns | Reshapes an existing AI CLI | Pasted into any chat by a human |
| **Maturity** | v0.9.4, heavy CI | v2.10.0, 16 releases, 382 tests | 1 commit, no releases |
| **Who runs it** | The platform, autonomously | A team of role-agents | One solopreneur, by hand |
| **Output** | A pushed git branch | A merged, gated PR | 6–7 business KB markdown files |

It's the least technical of the three and the easiest to evaluate — there are no prereqs to satisfy and nothing to run. If anything, its natural relationship to the others is as **input**: the KB files it produces (brand voice, avatar, frameworks) are exactly the kind of project context you'd drop into a `CLAUDE.md` / `.mulch/` so a Warren- or SAW-driven agent works with your business voice baked in.

---

### A caveat on sourcing

Assembled from the repo's web-visible README, the directory tree, and one sample `prompt.md` — the git-clone endpoint is blocked by this session's egress policy, so I did not read all 35 prompts. Prompt counts per module are as the README reports them. If you fork it into your own profile and add it to a session's scope (or paste files), I can do a prompt-by-prompt review of the actual content quality. Given it's a content repo with no code, though, there are no Warren host-prereq concerns here at all.
