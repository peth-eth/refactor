# /refactor — Claude Code Skill

A maintenance-day skill for Claude Code. Runs automated tools, triages output, and fixes issues in one pass. Works autonomously — fixes what's safe, flags what needs judgment.

## What it does

**Quick mode** (`/refactor`) — Phase 1 scans + fixes. Fast, ~5 minutes.
- Dead code / unused exports (knip)
- Code duplication (jscpd)
- Lint violations (ESLint)
- Stale dependencies

**Full mode** (`/refactor full`) — All 4 phases. The full maintenance day.
- Everything in quick mode, plus:
- Structural health (oversized files, bloated directories)
- React anti-patterns (unnecessary useEffect, useMemo, derived state)
- Component extraction opportunities

## Installation

Copy `SKILL.md` into your Claude Code skills directory:

```bash
mkdir -p ~/.claude/skills/refactor
curl -o ~/.claude/skills/refactor/SKILL.md \
  https://raw.githubusercontent.com/peth-eth/claude-skill-refactor/main/SKILL.md
```

Then invoke with `/refactor` or `/refactor full` in any Claude Code session.

## Requirements

- Claude Code CLI
- Node.js project with a `src/` directory
- Optional but recommended: `knip`, `jscpd`, ESLint configured in the project

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

The skill reads `feedback.log` in its directory at session start. Any corrections or preferences you give during a session are appended there and applied in future sessions.

```bash
# feedback.log lives alongside SKILL.md
~/.claude/skills/refactor/feedback.log
```
