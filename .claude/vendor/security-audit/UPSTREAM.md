# Upstream

Vendored copy of the Cloudflare security-audit skill.

- Source: https://github.com/cloudflare/security-audit-skill
- Vendored from: `skills/security-audit/` plus the repo root `LICENSE`
- Commit: `c1c8a8c1471069fb0e188eeaff69b8e8db6564a8`
- Date vendored: 2026-11-24
- License: MIT (see `LICENSE`)

Every other file in this directory is an unmodified copy of that upstream
commit. Nothing upstream was edited. `UPSTREAM.md` is the only local file.

## Known upstream issues

- [#128](https://github.com/cloudflare/security-audit-skill/issues/128) — an
  agent cleared a real Windows user profile while chasing a "scratch-local
  HOME". `/audit-workflow` item 2 never sets, clears or deletes HOME.
- [#160](https://github.com/cloudflare/security-audit-skill/issues/160) —
  without the OS sandbox nothing reaches `confirmed`. `/audit-workflow` item 2
  runs source-only unless a real OS sandbox is verified, and admits a
  source-traced `needs_validation` record into the fix queue only after an
  independent security review ("source-proven (no sandbox)").

## How to bump

Run from the repo root. `NEW` is the upstream commit to vendor.

```bash
NEW=<commit>
SB=$(mktemp -d)

# 1. Clone to a sandbox dir and pin the commit.
git clone https://github.com/cloudflare/security-audit-skill "$SB/cf"
git -C "$SB/cf" checkout "$NEW"

# 2. Vet before copying anything.
git -C "$SB/cf" log --oneline c1c8a8c1471069fb0e188eeaff69b8e8db6564a8.."$NEW"
diff -r --exclude=UPSTREAM.md --exclude=LICENSE \
  "$SB/cf/skills/security-audit" .claude/vendor/security-audit
# Read every changed .cjs file in full. Grep for network calls, child_process,
# and fs writes outside the output dir. Re-check upstream status of #128/#160.

# 3. Copy.
cp -p "$SB/cf/skills/security-audit/"* .claude/vendor/security-audit/
cp -p "$SB/cf/LICENSE" .claude/vendor/security-audit/LICENSE

# 4. Verify: first two commands print nothing; the third must show `# fail 0`.
diff -r --exclude=UPSTREAM.md --exclude=LICENSE \
  "$SB/cf/skills/security-audit" .claude/vendor/security-audit
cmp "$SB/cf/LICENSE" .claude/vendor/security-audit/LICENSE
(cd /tmp && node --test "$OLDPWD/.claude/vendor/security-audit/validate-findings.test.cjs" "$OLDPWD/.claude/vendor/security-audit/validate-coverage-ledger.test.cjs") 2>&1 | grep -E '^# (pass|fail)'

# 5. Update the commit and date at the top of this file.

# 6. Clean up the sandbox clone.
python3 -c 'import shutil,sys; shutil.rmtree(sys.argv[1])' "$SB"
```

After the bump is committed, refresh the installed copy the same way:
`cp -p .claude/vendor/security-audit/* ~/.claude/vendor/security-audit/`
