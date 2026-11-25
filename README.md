# audit-workflow

`/audit-workflow` is a Claude Code slash command that runs an ordered 17-item codebase audit
queue over a hub-spoke orchestrator: the hub plans each item and judges findings, cheap spokes
do the volume work (find, fix, test), and a research step handles external fact lookups (CVEs,
versions, licenses). A Step 0 project profile, derived from evidence in the repo, decides which
checks apply.

## Install

From a clone of this repo:

```bash
mkdir -p ~/.claude/commands ~/.claude/vendor
cp .claude/commands/audit-workflow.md ~/.claude/commands/audit-workflow.md
cp -R .claude/vendor/security-audit ~/.claude/vendor/
```

Then run `/audit-workflow` in any repo.

## Requirements

- Claude Code.
- A hub-spoke setup: a `glm-code`-style spoke runner and a `/research` command; adjust the
  names in [the command file](.claude/commands/audit-workflow.md) if yours differ.
- Node. The security skill's validators (`validate-findings.cjs`,
  `validate-coverage-ledger.cjs`) are Node scripts; `node --test` runs their bundled tests.
- Optional local scanners: semgrep, gitleaks, osv-scanner.
- Optional bubblewrap, if you want the security pass inside a real OS sandbox.

## Vendored: Cloudflare security-audit skill

Item 2 (Security) runs Cloudflare's security-audit-skill as its business-logic layer. It is
vendored unmodified under the MIT license
([LICENSE](.claude/vendor/security-audit/LICENSE)).
[UPSTREAM.md](.claude/vendor/security-audit/UPSTREAM.md) records the pinned upstream commit and
how to bump it. Install puts the skill at `~/.claude/vendor/security-audit`, not in your skills folder: that keeps it from auto-loading in other sessions without the item-2 guardrails (issue #128).

How item 2 runs it:

- `quick` profile by default, with a fixed agent budget.
- Source-only, unless a real OS sandbox (bubblewrap or similar) is verified present.
- It never sets, clears or deletes the HOME environment variable. Upstream issue #128: an agent cleared
  a real user profile while chasing a scratch-local HOME.
- No live probing.

## Queue

0. Idea pressure-test
1. Bugs
2. Security — local scanners first, then the Cloudflare security-audit skill pass
3. API auth
4. Dependency CVEs
5. Free-text input charset validation / Unicode hardening
6. Refactoring / code unification
7. Over-engineering
8. WCAG
9. UI anti-slop
10. Policy / ToS compliance check
11. Production readiness
12. Dynamic security testing
13. Minimap
14. AI-assistant discoverability / `llms.txt`
15. Lender-grade financial hygiene
16. Update `audit.md`

Findings go to a gitignored `.md` file before any fix is applied. Every fix is verified with
real test/gate runs; checks the profile marks N/A are excluded from the score.
