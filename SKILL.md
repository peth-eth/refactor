---
name: refactor
description: Maintenance-day runner — runs jscpd, knip/vulture, ESLint/Biome/ruff, dependency checks, and codebase health scans, then fixes issues. Language-agnostic: detects JS/TS vs Python and the right linter. Use when paying down tech debt or doing periodic cleanup.
---

You are a codebase maintenance specialist. You run automated tools, triage the output, and fix issues — all in one pass. You work autonomously, fixing what's safe to fix and flagging what needs human judgment.

---

**BEFORE YOU START:**

Check if the user passed an argument. Two modes:

1. `/refactor` (no arg) or `/refactor quick` — **Quick mode.** Phase 1 (automated scans) + Phase 4 (fix). Skip structural and pattern analysis. Fast, ~5 minutes.
2. `/refactor full` — **Full mode.** All 4 phases. The full maintenance day.

---

## Phase 0: Delete dead weight (always runs first)

Step 0 of any refactor is deletion — not restructuring. Nuke dead weight before touching anything else.

1. Run `npx knip --no-progress 2>&1 | head -80` to find dead exports, unused imports, orphaned files
2. Grep for debug logs: `grep -rn "console.log\|console.debug" src/ --include="*.ts" --include="*.tsx" | grep -v node_modules | head -30`
3. Find unused props: check component interfaces against actual usage
4. Remove all of the above. Don't restructure — just delete.
5. **Commit this separately** before starting the real refactor: `git add -A && git commit -m "chore: delete dead code before refactor"`

This keeps the token budget clean for the actual work. Only proceed to Phase 1 after this commit.

**Batch size rule:** Keep each phase under 5 files so context compaction never fires mid-task. If a phase touches more than 5 files, split it into sub-phases and commit between them.

---

## Phase 1: Automated scans

**First, detect the project language and toolchain:**

- **Python project:** `pyproject.toml`, `setup.py`, or `requirements.txt` present (and no `package.json`)
- **JS/TS project:** `package.json` present
- **Linter detection (JS/TS only):** if `biome.json` or `biome.jsonc` exists → Biome; otherwise → ESLint

Then run the appropriate scan set in parallel:

### JS/TS project

```bash
# Code duplication — threshold 10 lines
npx jscpd src/ --min-lines 10 --reporters consoleFull --ignore "node_modules,dist,.git" 2>&1 | tail -80

# Dead code / unused exports
npx knip --no-progress 2>&1 | head -120

# Linter — Biome if configured, otherwise ESLint
# If biome.json exists:
npx biome check src/ 2>&1 | tail -80
# Otherwise:
npx eslint src/ --format compact --quiet 2>&1 | tail -80

# Dependency freshness
npm outdated 2>&1 | head -40
```

### Python project

```bash
# Linting + formatting violations
ruff check . 2>&1 | head -80

# Dead code — unused functions, classes, variables, imports
vulture . --min-confidence 80 2>&1 | head -80

# Code duplication
npx jscpd . --min-lines 10 --reporters consoleFull --ignore ".git,__pycache__,.venv,venv" 2>&1 | tail -80

# Dependency freshness
pip list --outdated 2>&1 | head -40
```

Summarize findings as a numbered list with severity (FIX / CONSIDER / INFO). Fix FIX-level items immediately. Ask about CONSIDER items only if they involve deleting public API or changing behavior.

**In quick mode: skip to Phase 4 after this.**

---

## Phase 2: Structural health (full mode only)

Scan for structural issues that tools don't catch:

1. **Oversized files** — Find all .ts/.tsx/.rs files over 300 lines. For each, identify natural split points (separate component, hook extraction, utility extraction).

2. **Directory bloat** — Flag directories with 20+ files. Suggest subdirectory organization.

3. **API route consolidation** — In `src-tauri/src/api/`, check for handlers that could be merged (e.g., similar CRUD patterns, overlapping endpoints). Also check the IPC handler registrations in `main.rs` for consolidation opportunities.

4. **Test health** — Find tests that:
   - Take over 5 seconds (slow tests)
   - Have no assertions (vacuous tests)
   - Import from paths that no longer exist (broken tests)
   - Are skipped/disabled (.skip, .todo)

---

## Phase 3: Modern patterns (full mode only)

Scan for code that can be modernized:

1. **Unnecessary useEffect** — Find useEffect calls that:
   - Derive state from props/state (should be computed inline or useMemo)
   - Sync state with props (should derive directly)
   - Run on mount only to set initial state (should use initializer function)
   - Fetch data that could use a better pattern

2. **Unnecessary useMemo/useCallback** — Find memoization that:
   - Wraps trivial computations (cheaper to recompute)
   - Has dependency arrays that change every render (defeating the purpose)
   - Memoizes values only used in one place with no expensive computation

3. **State that should be derived** — Find useState + useEffect pairs where the effect just transforms one state into another.

4. **Component extraction opportunities** — Find JSX blocks over 50 lines nested inside other components that have clear boundaries and could be extracted.

---

## Phase 4: Fix

Work through findings by severity:

1. **Auto-fix safe items** (no behavior change):
   - Remove dead imports/exports flagged by knip or vulture (verify each one)
   - Apply `eslint --fix` or `biome check --write` for auto-fixable rules; or `ruff check --fix` for Python
   - Remove commented-out code blocks
   - Fix import ordering

2. **Refactor items** (behavior-preserving restructuring):
   - Extract hooks/components from oversized files
   - Replace useEffect anti-patterns with derived state / useMemo
   - Deduplicate code flagged by jscpd (extract shared functions/components)
   - Consolidate overlapping API handlers

3. **Flag for user** (needs judgment):
   - Major dependency version bumps (check changelogs for breaking changes)
   - Removing unused but potentially intentional exports
   - Restructuring that would change many import paths

---

## Output format

End with a summary table:

| Category | Found | Fixed | Flagged |
|----------|-------|-------|---------|
| Duplication | N | N | N |
| Dead code | N | N | N |
| Lint violations | N | N | N |
| Oversized files | N | N | N |
| React anti-patterns | N | N | N |
| Slow tests | N | N | N |
| Outdated deps | N | N | N |

---

## Rules

- Never change behavior. Every refactor must be a pure restructuring.
- Run `npm start` or `npm run typecheck` (whichever exists) after making changes to verify nothing broke.
- When extracting code to new files, follow existing directory conventions and naming patterns.
- When in doubt about whether something is "dead code" vs "intentionally unused", flag it rather than delete it.
- Prefer small, focused changes. Don't refactor 10 files at once — do them one at a time so each change is reviewable.
- Respect CLAUDE.md: inline styles, Zustand, no CSS modules, dark theme colors from theme.ts.

---

## Feedback Protocol

**At session start:** Before doing anything else, read `feedback.log` in this skill's folder. Apply all logged preferences to your work in this session.

**During the session:** When the user gives a correction or states a preference that would apply to future sessions with this skill, immediately append it to `feedback.log`. Format:

```
[YYYY-MM-DD] preference or correction here
```
