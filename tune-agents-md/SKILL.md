---
name: tune-agents-md
description: Audit and improve a project's AGENTS.md (or CLAUDE.md, .cursorrules) so it's a high-leverage instructions file for AI assistants. Use when the user wants to improve, audit, tune up, review, rewrite, or evaluate their AGENTS.md / CLAUDE.md / AI instructions file, or asks how to make their project guidance more effective for agents.
---

# Tune AGENTS.md

Audit a project's agent instructions file and propose concrete edits. The goal is **leverage per line** — every sentence should change agent behavior in a way the agent couldn't have figured out from the codebase itself.

## Mental model

AGENTS.md is for a smart engineer who can read code, grep, and run commands — but doesn't know the war stories. Good content lives in this gap:

- **The "why" behind unusual patterns** — why a rule exists, what broke before
- **Project-specific overrides to common defaults** — "we use X, not Y"
- **Load-bearing constraints** — platform, version, dependency quirks that bite silently
- **Mistakes agents repeatedly make** — what looks right but isn't

Bad content lives outside it: restating `package.json`, generic best practices, structure visible from `ls`, vague rules without rationale.

The bar for every line: **if I deleted this, would a competent agent get something wrong?** If no, delete.

## Process

### 1. Locate and consolidate

Check for every AI instruction file in the repo: `AGENTS.md`, `CLAUDE.md`, `.cursorrules`, `.cursor/rules/*`, `.github/copilot-instructions.md`, and any other tool-specific files (Aider, Continue, etc.).

`AGENTS.md` is the canonical file. Before auditing, consolidate so all content lives there:

- **Move all instructions from non-`AGENTS.md` files into `AGENTS.md`**, de-duplicating against what's already there. If `AGENTS.md` doesn't exist, promote the most complete file by renaming it to `AGENTS.md`.
- **`CLAUDE.md` must end up containing only `@AGENTS.md`** — nothing else. The include directive pulls in the canonical file so Claude Code still sees the same instructions.
- **Other tool files** — once content has moved, decide per tool: symlink to `AGENTS.md`, delete the file, or leave a short "See AGENTS.md" pointer. Don't leave duplicate content; it will drift.

Show the user a brief summary of what was moved and from where before proceeding to audit.

**If no instruction file exists anywhere, stop and tell the user no instructions file was found.** This skill optimizes existing instructions; it doesn't create new ones from scratch.

### 2. Read it fully

Read every line of the current file. Don't skim. The whole point is to evaluate each line against the bar.

### 3. Audit each section

For each section, paragraph, or rule, categorize it as one of:

- **Bloat** — true but derivable from the code in under a minute (project structure, tech stack, generic best practices)
- **Vague** — a rule without a concrete example or rationale ("write clean tests", "be careful with X")
- **Mixed-signal** — critical warnings buried alongside nice-to-haves with no visual hierarchy
- **Extract** — genuinely useful but not globally useful. It pays a context tax on every agent turn even though it's only relevant in one corner of the project. Move it somewhere it'll be loaded only when needed.
- **Keep** — load-bearing for *every* agent turn in this project (accuracy verified in the next step)

The bar for Keep is high: would a competent agent be worse off on a typical task without this line? If the content only matters when touching a specific file, subsystem, or workflow, it's an Extract candidate.

#### Where to extract

Pick the destination by scope:

- **In-code comment** — rule applies to one file or function. Put the rule at the point of use; the agent sees it when editing that location. Example: an invariant about a specific function lives in a comment on that function.
- **Nested `AGENTS.md`** in a subdirectory — rule applies to a subtree (e.g., `tests/AGENTS.md`, `migrations/AGENTS.md`). Tools that read `AGENTS.md` pick it up automatically when working in that subtree.
- **Separate doc referenced from `AGENTS.md`** — content is procedural and the agent will look it up by name. Example: `docs/release-process.md` with a one-line pointer "See `docs/release-process.md` when cutting a release." Lazy-loaded — only paid for when followed.
- **A new skill** — content is a multi-step procedure with its own decision tree and would benefit from being triggered by intent phrasing rather than file location. Example: "How to add a new promotion rule" — a workflow with branching choices is a better fit for a skill than a doc. Use `/skill-creator` to scaffold it.

Cut the bloat and queue the extracts before verifying. There's no point verifying a claim that's getting deleted or moved out.

See `references/checklist.md` for the full audit checklist with specific patterns to watch for, and `references/anti-patterns.md` for examples of content that should be cut.

### 4. Verify surviving claims against the codebase

This is the step most agents skip. Spot-check only what survived audit — anything **Wrong** (factually inaccurate or out of date) gets re-categorized here:

- **Versions** — do declared versions match `package.json` / `pom.xml` / `Cargo.toml` / lockfiles? Stale version numbers signal stale everything.
- **Paths and module names** — does the directory tree it describes match `ls`? Have files moved or been renamed?
- **Commands** — do the build/test/lint commands actually exist as scripts/targets? Try one or two if reasonable.
- **External file references** — files it tells the agent to "see X.md for details" — do those files still exist? Are they current?
- **Tech stack claims** — Java version, Node version, framework version. Confirm against build files.

Flag every drift. Stale facts in AGENTS.md are worse than missing facts — they get cited confidently.

### 5. Discover what's missing

Audit is half the job; the harder half is finding gaps. Read for what *isn't* there:

- **Recent gotchas** — scan `git log` for the last 30–60 days. Bug fixes whose commit messages describe non-obvious traps are candidates. ("Fixed X because Y didn't realize Z" → Z belongs in AGENTS.md.)
- **Repeated PR feedback** — patterns that humans keep correcting are patterns an agent will get wrong.
- **Platform/environment surprises** — Windows vs Unix, container vs host, specific shell. If running commands requires non-default flags or quoting, document it.
- **Forbidden patterns** — what should an agent *not* do? Negative rules are often more valuable than positive ones and almost always under-documented.
- **The "wish I'd known" list** — ask the user: "What do you find yourself correcting agents on repeatedly?" That's the highest-signal input.

### 6. Propose edits

Present findings as a categorized list. For each item:

- **Where** — section or line range
- **Verdict** — Wrong / Bloat / Vague / Missing / Mixed-signal / Extract (with destination)
- **Why** — one-line rationale tied to the mental model above
- **Proposed change** — concrete edit, not abstract advice

Ask the user to confirm or veto each category, then apply. Don't silently restructure — the user has calibration about their team that the skill doesn't.

### 7. Apply and stop

Make the edits with `Edit`. Don't add a changelog comment, "last updated" date, or meta-commentary inside the file — those rot. The file is the artifact; the git history is the changelog.

## Output style for the file itself

- **Concrete > abstract.** "Use `Select-Object -First 100`" beats "be aware of PowerShell quirks."
- **Examples with ✓/✗ pairs** for rules that have a tempting wrong form.
- **Negative rules deserve a "Why"** so future-agents can judge edge cases instead of cargo-culting.
- **Stable references over pinned specifics.** "Java 8" (constraint) is stable; "1.8.0_452" (patch version) drifts.
- **No prose where a code block would do.** Commands, paths, and signatures go in code blocks.
