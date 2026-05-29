# AGENTS.md Audit Checklist

Section-by-section guide for evaluating an AGENTS.md. Use against the "leverage per line" bar from SKILL.md: every line should change agent behavior in a way the agent couldn't have figured out from the code itself.

## Project overview / intro

**Keep**: One or two sentences naming what the project is and what it does — useful for orienting the agent if the repo's purpose isn't obvious from the README.

**Cut / collapse**:
- Marketing copy ("a world-class, scalable system...")
- Long history of how the project evolved
- Stakeholder lists, team org charts
- Anything that reads like a pitch deck

**Verify**: Is the description still accurate? Has the project's scope changed?

## Tech stack and versions

**Keep**:
- Major-version constraints that affect code choices: "Java 8 (no `var`, no `Optional.isEmpty()`)", "Python 3.9 (no `match`)"
- Framework choices that aren't obvious from the build file
- Unusual or surprising dependencies and why they're used

**Cut**:
- Patch-version pins ("Spring Boot 2.7.18") — these drift and the build file is authoritative
- Restating every dependency that's obvious from `package.json` / `pom.xml`
- "We use TypeScript" in a `*.ts` repo

**Verify**: Run `cat package.json | grep version` or equivalent. Flag any version that disagrees.

## Directory / module structure

**Keep**:
- Non-obvious responsibility boundaries ("the `engine/` module is pure logic — never put HTTP code here")
- Modules whose names don't describe their job
- Cross-cutting concerns that aren't visible from the tree (e.g., "anything in `internal/` is unsupported")

**Cut**:
- ASCII tree diagrams that duplicate `ls` output
- One-line-per-directory descriptions when the names are self-explanatory
- Any structure that's been refactored since the diagram was written

**Verify**: Run `ls` at the top level. Compare against any directory tree in the file. Flag drift.

## Build, test, and tooling commands

**Keep**:
- The exact incantations that work, especially when shell-quirky ("PowerShell needs `-D` args quoted: `\"-DskipTests=true\"`")
- Non-default flags that are load-bearing
- Selectors for running a single test (people forget these)
- Coverage / lint / format commands if the names aren't obvious

**Cut**:
- "Run `npm test` to run tests" — discoverable from `package.json`
- Long lists of every available script
- Commands that no longer exist

**Verify**: Try at least one command from each category if reasonable. Flag any that fail or are missing.

## Code conventions and style

**Keep**:
- Conventions that override common defaults ("we don't use Optional; return null and check")
- Project-specific naming patterns
- Required helpers or constants (e.g., "all magic strings live in `TapeConstants.java`")
- Anti-patterns with concrete ✓/✗ examples and a one-line why

**Cut**:
- Generic best practices ("use meaningful names", "keep functions small", "write tests")
- Restating what a linter / formatter already enforces (if Prettier owns the indentation, don't write rules about indentation)
- Style preferences without rationale or examples

**Verify**: Spot-check the codebase. Do existing files actually follow the stated conventions? If not, either the convention is aspirational (note that) or the doc is wrong.

## Gotchas and war stories

**This is usually the highest-value section.** Most files under-invest here.

**Keep**:
- Specific bugs that bit the team and how to avoid them
- Patterns that look right but break in production
- Workarounds for upstream library quirks
- Anything with "the reason this exists is…"

**Cut**:
- Gotchas about features that have since been removed
- War stories without an actionable takeaway

**Add if missing**: Scan `git log --oneline -n 100` and PR titles for fix commits. Anything described as "fixed a subtle issue with X" is a candidate for a gotcha entry.

## Platform / environment notes

**Keep**:
- OS-specific quirks (Windows shell behavior, macOS-only flags, container vs host paths)
- Required env vars that aren't in `.env.example`
- Tooling versions that matter (Node 18 vs 20, Python venv vs system)

**Cut**:
- Generic "make sure you have Git installed" advice
- Setup steps that are in a `Makefile` or `setup.sh` already

## "Notes for AI Assistants" / meta sections

These are often a mess. Audit hard.

**Keep**:
- Things agents specifically and repeatedly get wrong on this codebase
- Constraints the agent can't infer (e.g., "this Java codebase targets Java 8, even though the local JDK may be newer")

**Cut**:
- Generic agent advice ("think step by step", "be careful with destructive commands") — the harness already handles this
- Reminders about general programming practices

## File references to other docs

**Verify** every `see X.md for details` reference:
- Does the file exist?
- Is it still current?
- If it's a "fix prompt" or "migration doc" — has the migration happened? If yes, the reference is outdated.

**Common drift**: Files like `BLWS_BALANCE_PREFERENCES_FIX_PROMPT.md`, `MIGRATION_PLAN.md`, `OLD_README.md` often hang around long after their relevance ended.

## Visual hierarchy and length

After content audit, look at the shape:

- **Are critical warnings visible?** "This project runs on WINDOWS" deserves to be at the top, in caps, not buried in section 7.
- **Is the file over ~300 lines?** Most teams over-document. Consider whether half the content carries the weight.
- **Are sections in priority order?** First screen should be the highest-leverage content.
- **Is there a TL;DR?** For long files, a 10-line summary at the top helps.

## Quick sanity questions

Before finalizing, ask:

1. If a brand-new agent read only the first 50 lines, would they avoid the top 3 mistakes?
2. Is there anything in here that the model would already do correctly by default?
3. Is there anything in the team's recent PR review feedback that *isn't* in here?
4. If I deleted this file entirely, what's the first thing that would go wrong?
