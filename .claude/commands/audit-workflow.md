---
description: Full codebase audit queue — bugs, security, API auth, dependency CVEs, WCAG, minimap, UI anti-slop, production readiness (public-facing gated) — each verified, fixed, and tested in strict order via the GLM hub-spoke orchestrator (GLM spokes do the volume, agy/research does external facts, Claude hub only plans + reviews + gates — conserves Claude quota)
---

# Audit Workflow (GLM hub-spoke)

You are the Claude HUB. Do NOT use `ultracode` / Claude-heavy Workflow orchestration for these items — the orchestration now routes VOLUME to GLM `glm-code` spokes and EXTERNAL RESEARCH to agy (`/research`), keeping Claude for judgment only. For each item: the hub decomposes it into a spec, fires GLM spoke(s) to find → verify → fix → test, and the hub reviews the diff + runs the gates itself. Any external-fact lookup (CVE, version, advisory, license) goes to agy via `/research` — the hub does not web-search directly.

Read `~/.claude/orchestrator/PROTOCOL.md` once before starting (adjust the path if your hub-spoke orchestrator lives elsewhere). Queue these IN THIS ORDER and autonomously finish each (findings + fixes + tests, hub-verified) before starting the next. Use `audit.md` at the repo root to speed up (create if missing: what was audited when, known-clean areas, recurring hotspots, commands/gates used).

**Find existing bookkeeping files across the WHOLE repo first, not just root.** Before item 1,
search the entire tree for `audit.md`, `bugs.md`, `secure.md`, `minimap`,
`ui-slop.md`, `prod-readiness.md` — a monorepo (e.g. `apps/web/`, `apps/api/`, `packages/*`) or a
prior run scoped to a subdirectory can leave these anywhere, not only at the repo root:
```bash
find . -maxdepth 4 \( -name "audit.md" -o -name "bugs.md" -o -name "secure.md" \
  -o -name "minimap" -o -name "ui-slop.md" \
  -o -name "prod-readiness.md" \) -not -path "*/node_modules/*" -not -path "*/.git/*"
```
If a match turns up outside root, READ it before starting (it may already cover the area you're
about to re-audit — don't duplicate work) and keep writing to that SAME location for this run,
rather than creating a second copy at root. Only create a fresh file at root if the search
finds nothing anywhere in the tree.

**Public-facing gate (decide before item 8):** determine whether this app is genuinely public-facing (has or will have a real external audience — a live domain, an app-store listing, paying/anonymous users) versus personal/local-only (runs on localhost or the user's own machine, no outside users). If it isn't obvious from the repo (a deployed URL in `package.json`/README, a hosting config, auth aimed at unknown users vs. just the owner), ask the user in one line rather than guessing. Record the answer in `audit.md` so re-runs don't re-ask.

1. **Bugs** — hub specs a bug hunt; GLM spokes hunt across the whole codebase and VERIFY each finding, writing `bugs.md` (gitignored) BEFORE any fix; then spokes fix + test each; hub reviews every diff and runs the gates. (ignore all .md in git)
2. **Security** — GLM spokes audit injection, authn, secrets, and access control; each finding verified into `secure.md` before fixing + testing; hub reviews the diffs itself (secrets / network binds / access control are hub-scrutinized directly).
3. **API auth** — GLM spokes audit every endpoint under `src/routes/` for missing auth checks and fix issues. If the repo has no `src/routes/`, audit its real API/route layer (e.g. `app/api/` in Next.js) and state which path was used.
4. **Dependency CVEs** — send to **agy** via `/research`: CVEs, abandonment, and license risk on the direct dependencies. Hub reviews agy's verified report, then GLM spokes apply the fixes (version bumps / replacements); hub runs the build + tests.
5. **WCAG** — GLM spokes audit every page component for WCAG AA accessibility violations and fix all issues; hub spot-checks.
6. **Minimap** — hub + GLM reader spokes map the codebase (what each subsystem does, how they connect, features and weaknesses) into a gitignored `minimap` file BEFORE fixing; then GLM spokes fix the weaknesses; hub reviews.
7. **UI anti-slop (all apps — public-facing or personal).** GLM spokes audit every user-facing page/component against common AI-SaaS-template tells: "AI-Powered" pill badges, gradient-mesh hero backgrounds, a single gradient-colored keyword inside the headline, boilerplate "Transform your X with cutting-edge Y" subheadlines, the gradient-CTA + ghost-"Watch Demo" pair, fake trust-stat 3-card rows wedged under the hero, and generic icon-in-card feature grids with filler one-liners — plus the root tell, template reuse (this app's design should not be interchangeable with an unrelated product's). Write findings to `ui-slop.md` (gitignored) before fixing; GLM spokes fix, hub spot-checks against the same checklist.
8. **Production readiness (public-facing apps ONLY — skip for personal/local-only, per the gate above; record in `audit.md` why it was skipped or run).** GLM spokes audit and fix, hub reviews each diff, against this checklist — write findings to `prod-readiness.md` (gitignored) before fixing:
   - **Rate limiting** on public endpoints, especially signup/auth and any AI/LLM-calling endpoint, so the app can't be spammed or cost-abused.
   - **Real authentication provider** (e.g. Clerk, Firebase Auth, Auth0 — not a hand-rolled auth system), including re-authentication on sensitive actions.
   - **Row-level / row-scoped access control on the database** (e.g. Postgres RLS keyed on `user_id`) so users can only ever read/write their own rows, not just app-layer checks.
   - **Secrets and API keys live server-side in environment variables**, never hardcoded or shipped to the client.
   - **Caching for repeated/expensive requests** (e.g. Redis) instead of recomputing or re-querying every time.
   - **Heavy work runs as async background jobs** (e.g. PDF parsing, file processing) rather than blocking the request/response cycle.
   - **API load testing before launch**, not just after users start hitting it.
   - A vibe-coded UI is the visible tip of the iceberg — this checklist is the real-software-engineering mass underneath. Do not let item 8 get treated as optional polish once the gate says the app is public-facing.
9. Update `audit.md` accordingly to speed up next time's workflow (including the public-facing gate decision from above, so re-runs don't re-ask).

Rules:
- **Engine = GLM hub-spoke, NOT ultracode.** Volume (find / fix / test) → `glm-code` spokes. External facts → agy `/research`. The Claude hub ONLY plans, writes specs, reviews diffs, and runs gates. This is deliberate: it conserves Claude quota and uses a flat-rate coding plan for the heavy lifting.
- **Quota discipline (GLM-driven):** check your GLM quota/usage before every wave. If usage is high, prefer a cheaper/lighter model tier and halve wave sizes; if usage is near the cap, stop until it resets. Honor any peak-hour rate deferral your provider has.
- Reported findings files (`bugs.md`, `secure.md`, `minimap`) must be written BEFORE their fixes are applied, per each item's wording; keep the minimap and audit bookkeeping files gitignored where the item says so.
- Verify fixes with real test / gate runs the hub executes — evidence before "done" on every item; a spoke's ✅ is a claim, not proof.
- Long run: check your own usage/rate limits periodically throughout and pause/resume around them if your setup requires it.
- Finish with a summary: per-item outcome, findings counts (verified vs refuted), fixes applied, gates status, and spend.
