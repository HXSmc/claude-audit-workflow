# audit-workflow

Claude Code slash command: `/audit-workflow` — a full codebase audit queue (bugs, security,
API auth, dependency CVEs, WCAG, minimap, UI anti-slop, production readiness) run in strict
order via a GLM hub-spoke orchestrator pattern. Claude acts as the hub (plans, writes specs,
reviews diffs, runs gates); volume work (find/fix/test) is delegated to spokes, and external
fact lookups (CVEs, versions, licenses) go through a research step — keeping the hub's own
usage for judgment calls only.

## Install

Copy the command into your Claude Code commands directory:

```bash
mkdir -p ~/.claude/commands
cp .claude/commands/audit-workflow.md ~/.claude/commands/audit-workflow.md
```

Then run `/audit-workflow` in any repo.

## Requirements

This command assumes you have some hub-spoke orchestration setup (e.g. a `glm-code`-style
spoke runner and a `/research` command) already configured in your Claude Code environment.
Adjust the spoke/research commands referenced in the file to match your own setup if different.

## Queue

1. Bugs
2. Security
3. API auth
4. Dependency CVEs
5. WCAG
6. Minimap
7. UI anti-slop
8. Production readiness (public-facing apps only)
9. Update bookkeeping (`audit.md`)

Each step's findings are written to a gitignored `.md` file before any fix is applied, then
verified with real test/gate runs before being marked done.
