---
name: gwt
description: Use the local gwt helper whenever Git worktrees must be created, listed, moved, removed, or accessed.
---

# gwt worktrees

Use the `gwt` helper for all Git worktree operations.
Do not substitute raw `git worktree` commands when this skill applies.

## Invocation

First check whether the current shell already provides `gwt`:

```bash
type -a gwt
```

If it does not, source `_gwt` from the GWT checkout that contains this skill.
Resolve this `SKILL.md` to its real path; `_gwt` is two directories above its
directory. Source the helper in the same shell command as the operation because
each command may start a fresh shell:

```bash
gwt_skill_file=/absolute/path/to/gwt/skills/gwt/SKILL.md
source "$(dirname "$(realpath "$gwt_skill_file")")/../../_gwt"
gwt() { _gwt_main "$@"; }
gwt list
```

Use the skill path provided by the runtime for `gwt_skill_file`; do not search
the whole filesystem. If `_gwt` is not bundled with the skill and `gwt` is not
installed, report that setup problem instead of using raw `git worktree`.

If the configured profile is missing, select a valid one with `gwt profile
PROFILE` or report the configuration problem. Do not bypass the profile by
operating on the bare repository directly.

## Worktree paths

The active profile's `RELTO` value is the worktree root. `gwt` resolves every
relative `NAME` or `WT` operand from `RELTO`, not from the current directory.
It leaves absolute paths unchanged. The first column of `gwt list` contains
paths relative to `RELTO`; copy those values when referring to existing
worktrees.

For `gwt new NAME [BRANCH]`, `NAME` chooses the path. `BRANCH` chooses the
branch. For example, `gwt new review-123 feature/payments` creates
`$RELTO/review-123` with branch `feature/payments`. Without `BRANCH`, the local
branch name is the basename of `NAME`.

## Commands

- Inspect with `gwt list` or `gwt ls`; use `gwt list -t` for newest-first order.
- Create with `gwt new NAME [BRANCH]`.
- Enter with `gwt cd WT`, or run a command without changing the current shell
  directory with `gwt exec WT CMD...` or `gwt x WT CMD...`.
- Rename with `gwt move WT_SRC WT_DST` or `gwt mv`.
- Remove with `gwt remove WT...` or `gwt rm WT...`.

Before moving or removing a worktree, list it first and copy its path from the
first column. Check for uncommitted work when practical. Pass `-f` to remove
only when the user explicitly authorizes forcing removal. Preserve unrelated
worktrees and report what changed.
