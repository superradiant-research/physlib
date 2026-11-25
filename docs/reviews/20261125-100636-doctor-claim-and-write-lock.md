# Doctor claim and write-lock pattern review

**Date:** 2026-11-25T10:06
**Scope:** `.beads/.gitignore`, `docs/session-summaries/2026-11-25-100500-lean-toolchain-and-beads-setup.md`
**Commits:** `9837e2c4` (parent `a055c157`, matches `origin/beads-setup`). `upstream/master` is still `7910f51f`.

---

## Critical

None.

## Warning

### 1. CLAUDE.md triage paths are unchanged

**File:** `CLAUDE.md` (lines 8, 71, 83-84, 92)

`9837e2c4` does not touch `CLAUDE.md`. The issues file is still `.beads/issues.jsonl`, and `bv --robot-triage` still keeps `quick_ref` and `recommendations` under `.triage`. This is already **physlib-tov**. The session-summary half of that issue is in this commit. The `CLAUDE.md` half is not.

## Suggestion

### 1. The summary still counts four warnings after the pattern warning cleared

**File:** `docs/session-summaries/2026-11-25-100500-lean-toolchain-and-beads-setup.md` (line 15)

The new sentence says doctor reports four warnings, then says `.gitignore` now names `.write.lock`. A `br doctor` run after `9837e2c4` prints three warnings and no longer prints `gitignore.beads_inner_present`:

- `write_lock`: `.beads/.write.lock` was about 645 seconds old and looked orphaned. It was left in place.
- `db.sidecars`: WAL without a SHM sidecar, which doctor calls expected for frankensqlite.
- `base_jsonl.missing_post_flush`: `.beads/beads.base.jsonl` is still missing. `last_export_time` is now `2026-11-25T18:03:49.459905374+00:00`. The file stays uncommitted.

The rest of the sentence matches that run. Health is still `HEALTH workspace: healthy`.

## Positive

- The `.gitignore` comment is accurate: `*.lock` already ignored `.write.lock`, and doctor looks for the name. Adding `.write.lock` cleared that warning.
- The commit is those two files only. It does not contain `.claude/learning-seeds.md`, `beads.base.jsonl`, or a master reset. The message has no Co-authored-by trailer.
- The summary no longer says doctor is free of warnings.
