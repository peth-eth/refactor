# /refactor — Claude Code Skill

A maintenance-day skill for Claude Code. Runs automated tools, triages output, and fixes issues in one pass. Works autonomously — fixes what's safe, flags what needs judgment.

**Language-agnostic:** detects JS/TS vs Python and picks the right tool for each.

## What it does

**Quick mode** (`/refactor`) — Phase 1 scans + fixes. Fast, ~5 minutes.

**Full mode** (`/refactor full`) — All 4 phases. The full maintenance day.

### Phase 0 — Delete dead weight first
Nukes dead code before touching anything else. Separate commit, clean baseline.

### Phase 1 — Automated scans

| Project type | Duplication | Dead code | Linting |
|---|---|---|---|
| JS/TS + ESLint | jscpd | knip | eslint |
| JS/TS + Biome | jscpd | knip | biome check |
| Python | jscpd | vulture | ruff |

### Phase 2 — Structural health *(full mode)*
Oversized files, bloated directories, API route consolidation, broken/slow tests.

### Phase 3 — Modern patterns *(full mode)*
Unnecessary useEffect/useMemo, state that should be derived, component extraction.

### Phase 4 — Fix
Auto-fixes safe items. Refactors behavior-preserving issues. Flags anything that needs human judgment.

## Installation

```bash
mkdir -p ~/.claude/skills/refactor
curl -o ~/.claude/skills/refactor/SKILL.md \
  https://raw.githubusercontent.com/peth-eth/refactor/main/SKILL.md
```

Then invoke with `/refactor` or `/refactor full` in any Claude Code session.

## Output

Ends with a summary table:

| Category | Found | Fixed | Flagged |
|----------|-------|-------|---------|
| Duplication | N | N | N |
| Dead code | N | N | N |
| Lint violations | N | N | N |
| Oversized files | N | N | N |
| React anti-patterns | N | N | N |

## Feedback protocol

The skill reads `feedback.log` in its directory at session start and applies logged preferences automatically. Corrections you give during a session are appended for future runs.
