# gwt

`gwt` is a small Bash helper for managing Git worktrees from one bare repository.
It is meant for people who keep several working copies of the same repo around:
one for a feature branch, one for a review branch, one for a quick bug fix, and
so on.

The script keeps the bare repo in one configured location and creates worktrees
under a directory you choose. After that, `gwt new`, `gwt cd`, `gwt ls`, and
`gwt rm` handle most of the daily work.

## Features

- Profile-based setup for one or more repositories.
- Bare-repo initialization with the configured remote and default branch.
- Create a worktree from the default branch or from a named remote branch.
- Jump into a worktree by name.
- Run a command inside a worktree without changing your current shell directory.
- List worktrees with branch, commit, and last-commit date.
- Add worktree notes, displayed before the last-commit date.
- Sort worktrees by latest commit date with `gwt list -t`.
- Rename and remove worktrees through Git's worktree commands.
- Bash completion for commands and known worktree names.
- Optional custom list column, useful for showing local state such as build
  output or project-specific markers.

## Requirements

- Bash
- Git
- GNU-style command-line tools used by the script, including `realpath`,
  `column`, `sort`, `grep`, and `awk`
- Any tools used by your custom profile commands. For example, one of the
  profiles below uses `rg` in `_gwt_vars[CUSTOM]`.

## Install

Clone or copy this repository somewhere stable, then source `_gwt` from your
shell startup file.

Example `~/.bashrc`:

```bash
source /path/to/gwt/_gwt
alias gwt=_gwt_main
complete -F _gwt_completion gwt
```

Create a profile before opening a new shell, or the script will warn that the
default profile is missing.

## Configure A Profile

Profiles live in:

```bash
${XDG_CONFIG_HOME:-$HOME/.config}/gwt
```

Create `~/.config/gwt/default.profile`:

```bash
_gwt_vars[RELTO]=$HOME/code
_gwt_vars[REMOTE]=git@github.com:yourname/yourrepo.git
_gwt_vars[REPO]=$HOME/code/.yourrepo
_gwt_vars[DEFAULT_BRANCH]=main
```

Profile fields:

- `RELTO`: parent directory for named worktrees. `gwt new feature-a` creates
  `$RELTO/feature-a`.
- `REMOTE`: Git remote used by `gwt init` and fetches.
- `REPO`: local bare repository managed by `gwt`.
- `DEFAULT_BRANCH`: branch fetched during initialization and used when `gwt new`
  is called without a base branch.

See [examples/profiles.md](examples/profiles.md) for fuller profile examples,
including `app` and `docs` setups.

Use another profile by setting `_GWT_PROFILE` before sourcing the script:

```bash
export _GWT_PROFILE=work
source /path/to/gwt/_gwt
alias gwt=_gwt_main
complete -F _gwt_completion gwt
```

You can also switch profiles in an existing shell:

```bash
gwt profile work
```

## Initialize

Once the profile exists, initialize the bare repository:

```bash
gwt init
```

That creates `$REPO`, adds `origin`, and fetches the configured default branch
and tags.

## Commands

```text
gwt init
gwt new NAME [BRANCH]
gwt cd WT
gwt exec WT CMD...
gwt x WT CMD...
gwt list
gwt ls
gwt list -t
gwt note WT "NOTE"
gwt move WT_SRC WT_DST
gwt mv WT_SRC WT_DST
gwt remove WT [WT...] [-f]
gwt rm WT [WT...] [-f]
gwt profile PROFILE
gwt p PROFILE
```

Worktree names are relative to `RELTO` unless you pass an absolute path.

## Examples

Create a worktree from the default branch:

```bash
gwt new auth-refresh
```

Create a worktree from a remote branch:

```bash
gwt new review-1234 feature/payment-retry
```

Jump into a worktree:

```bash
gwt cd auth-refresh
```

Run tests in a worktree while staying in the current directory:

```bash
gwt x auth-refresh make test
```

List worktrees:

```bash
gwt ls
```

List worktrees with the most recently committed work first:

```bash
gwt ls -t
```

Add or replace a worktree note:

```bash
gwt note auth-refresh "Waiting for review"
```

Notes appear before the timestamp in `gwt ls` and stay with a worktree when
it is renamed. Clear a note with `gwt note auth-refresh ""`.

Rename a worktree:

```bash
gwt mv auth-refresh auth-refresh-v2
```

Remove one or more worktrees:

```bash
gwt rm auth-refresh review-1234
```

Force removal when Git refuses a normal cleanup:

```bash
gwt rm old-experiment -f
```

## Custom List Column

Set `_gwt_vars[CUSTOM]` in a profile to append a column to `gwt ls`. The command
runs with `WORKSPACE` set to the worktree path.

Example:

```bash
_gwt_vars[CUSTOM]='[[ -d "$WORKSPACE/build" ]] && echo 📦 build'
```

With that in place, any worktree containing a `build` directory gets an extra
`build` marker in the list output.

## Notes

`gwt` uses Git's own worktree commands underneath. If something looks odd,
`git -C "$REPO" worktree list` is the best first check.

The helper changes directories inside your current shell only when you use
`gwt cd` or create a worktree with `gwt new`. `gwt exec` and `gwt x` run in a
subshell and leave your current directory alone.
