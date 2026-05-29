# AGENTS.md Anti-Patterns

Concrete examples of content that should be cut, rewritten, or restructured, with rewrites where useful. Use these to recognize problems quickly during audit.

## 1. Restating what the build files show

**Bad**:
```
## Tech Stack
- React 18.2.0
- TypeScript 5.1.6
- Vite 4.4.5
- Jest 29.6.2
```

**Why it's bad**: All four versions are pinned in `package.json`. They will drift the moment someone bumps a dep. The agent can `cat package.json` faster than it can decide whether to trust this list.

**Rewrite**: Drop it. If a major version constraint actually changes how code should be written (e.g., React 18's `useTransition`, TypeScript 5's decorators), document the *constraint*, not the version.

```
## React conventions
Server Components are not in use here — every component is client-side. Don't add `"use client"` directives; they're noise in this repo.
```

## 2. Generic best practices the model already knows

**Bad**:
```
- Write meaningful variable names
- Keep functions small
- Add comments to explain complex logic
- Handle errors appropriately
- Write tests for new features
```

**Why it's bad**: The model already does these by default. Listing them dilutes the signal of the project-specific rules around them. Worse: the agent may decide if these need to be repeated, *everything* in the file is at this level of generality and stop reading carefully.

**Rewrite**: Delete the section. Replace only with rules that override defaults:

```
## Comments
Default to no comments. Only add a comment when the WHY is non-obvious (a workaround for an upstream bug, an invariant that isn't visible locally). Don't restate what the code does.
```

## 3. ASCII trees that duplicate `ls`

**Bad**:
```
src/
├── components/    # React components
├── hooks/         # Custom React hooks
├── utils/         # Utility functions
├── pages/         # Page components
└── api/           # API client code
```

**Why it's bad**: Every label is obvious from the directory name. The tree adds nothing the agent doesn't see when listing the directory.

**Rewrite**: Document only the non-obvious. If everything in a given tree is obvious, omit the tree entirely.

```
## Module boundaries
- `src/engine/` is pure logic — never import from `src/api/` here. The engine module is reused by a CLI that has no HTTP client.
- `src/internal/` is for experimental code. Anything imported from `src/internal/` outside its own directory is a bug.
```

## 4. Rules without rationale

**Bad**:
```
- Always use the repository pattern for data access
- Don't import directly from `node_modules` in component files
- Prefer composition over inheritance
```

**Why it's bad**: With no "why," an agent will either ignore the rule under edge cases (because it can't tell when the rule is load-bearing) or follow it cargo-cult style into places where it actively hurts. Rules with rationale travel; rules without rationale rot.

**Rewrite**: Add the *why* in one line.

```
- Always go through `src/data/` — direct ORM access from components broke our switch to GraphQL in 2024 because the call sites were everywhere. The data layer is the seam.
```

## 5. Stale references to migration / fix docs

**Bad**:
```
> See `MIGRATION_PLAN_V2.md` for details on the auth refactor.
> See `BLWS_BALANCE_PREFERENCES_FIX_PROMPT.md` for the pattern.
```

**Why it's bad**: These docs were written during a migration and almost always go stale within months. Worse, the file may have been deleted and the reference now points to nothing.

**How to fix**: For each reference, check whether (a) the file still exists, (b) it's still current, (c) the content it points to has been integrated into the codebase. If the migration is done, delete the reference and either inline the key takeaway or drop it.

## 6. "Notes for AI Assistants" full of harness-level advice

**Bad**:
```
## Notes for AI Assistants
1. Think step by step
2. Be careful with destructive commands
3. Always read files before editing them
4. Use the right tools for the job
5. Test your changes before declaring done
```

**Why it's bad**: This is what the agent harness already enforces. The user doesn't need to remind the agent to think. Worse, it crowds out the project-specific notes that should be in this section.

**Rewrite**: Keep only items where this project's behavior diverges from defaults.

```
## Notes for AI Assistants
1. This is Java 8 — `var`, `Optional.isEmpty()`, text blocks, and records are unavailable. Use explicit types.
2. Money is always `BigDecimal`. Never `double` or `float`, even in tests.
3. When tests fail with `UnnecessaryStubbingException`, prefer removing the unused stub over adding `lenient()` unless the stub is genuinely conditional.
```

## 7. Buried critical warnings

**Bad** (warning is in section 9 of a 12-section doc):
```
## 9.3 Build Notes
...
By the way, this project only builds on Windows due to dependencies on PowerShell scripts in the build pipeline.
```

**Why it's bad**: The single most important fact about the project is buried where an agent in a hurry won't see it. The agent will spend a turn trying to `make build` on a Mac and fail confusingly.

**Rewrite**: Promote it to the top, visually distinct.

```
> **This project runs on Windows only.** Build commands use PowerShell; `make` and Unix shell idioms will not work. See "Commands" section for the right invocations.
```

## 8. Long file with no scannable structure

**Symptom**: 800-line AGENTS.md with one `##` header and the rest in flowing prose. Sentences run on. Code blocks rare.

**Why it's bad**: Agents skim. So do humans. If the structure doesn't surface the load-bearing parts, they get missed.

**Fix**: Convert to bullets and code blocks. Promote critical warnings to callouts. If the doc is genuinely long because the project is genuinely complex, add a 5–10 line TL;DR at the top.

## 9. Self-praising prose

**Bad**:
```
Our codebase follows industry best practices and emphasizes clean, maintainable code. We pride ourselves on a robust testing culture and a developer experience that prioritizes...
```

**Why it's bad**: Zero behavioral signal. The agent learns nothing. Every line of prose that doesn't tell the agent something it would otherwise get wrong is bloat.

**Rewrite**: Delete.

## 10. Architecture overview as prose

**Bad** (4 paragraphs describing how the auth flow works):
```
When a user logs in, the request hits the gateway, which validates the API key against...
```

**Why it's bad**: If the description is accurate, the agent could get it from reading 3 files. If it's *in*accurate (likely, after a few refactors), it's actively misleading. Architecture docs in AGENTS.md decay faster than almost anything else.

**Rewrite**: Either (a) point to a `docs/architecture.md` that's kept current as part of the deploy process, or (b) reduce to the load-bearing constraint:

```
## Auth
- Auth state lives in `src/auth/session.ts`. Never read cookies directly elsewhere; always go through `getSession()`.
- Tokens are rotated on every request — don't cache them.
```

## 11. Decisions documented without the alternative considered

**Bad**:
```
- We use Zustand for state management.
```

**Why it's bad**: The agent might add Redux or Jotai or context providers without realizing this is a "we tried that and it didn't work" decision. The rule looks like a preference; it's actually a constraint.

**Rewrite**:
```
- State management is Zustand. We tried Redux Toolkit (too much boilerplate for our team) and Jotai (atom proliferation). Don't reach for context providers either — that was the pre-Zustand pattern and we migrated off it.
```

## 12. Treating AGENTS.md as a changelog

**Bad**:
```
Last updated: 2024-08-14 by Sarah
Recent changes:
- Added auth section
- Updated build commands
- Removed deprecated linting notes
```

**Why it's bad**: Git history already has this. The "last updated" date will rot the moment someone tweaks a typo without updating it, eroding trust in the rest of the file.

**Rewrite**: Delete. The git log is the changelog.
