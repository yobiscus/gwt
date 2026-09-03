# Profile examples

These examples show complete profile files for two repositories. They are more
specific than the quick-start profile in the README, so they live here instead
of in the main setup flow.

## app

The `app` profile keeps worktrees under `$HOME/code/app` and starts new
worktrees from `main`. Its custom column marks worktrees that contain a build
directory.

```bash
_gwt_vars[RELTO]=$HOME/code/app
_gwt_vars[REMOTE]=https://github.com/example/app.git
_gwt_vars[REPO]=$HOME/.local/share/gwt/app
_gwt_vars[DEFAULT_BRANCH]=main
_gwt_vars[CUSTOM]='[[ -d "$WORKSPACE/build" ]] && echo 📦 build'
```

## docs

The `docs` profile uses the same layout with a different repository and
worktree directory.

```bash
_gwt_vars[RELTO]=$HOME/code/docs
_gwt_vars[REMOTE]=https://github.com/example/docs.git
_gwt_vars[REPO]=$HOME/.local/share/gwt/docs
_gwt_vars[DEFAULT_BRANCH]=main
_gwt_vars[CUSTOM]='[[ -d "$WORKSPACE/site" ]] && echo 📦 site'
```

## Default profile

If `app` is the usual profile, make it the default with a symlink:

```bash
ln -s app.profile ~/.config/gwt/default.profile
```
