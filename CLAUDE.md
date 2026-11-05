# CLAUDE.md — proxmox-recipes-experimental

Read this before doing anything in this repo. These rules are not suggestions.

## What this repo is

Experimental recipes that create a Debian LXC container on a Proxmox VE host
and install one self-hosted app into it **natively, in a way the app's
upstream does not support**: typically an app that only ships Docker images,
installed by doing by hand what its Dockerfile does.

It is the companion of
[proxmox-recipes](https://github.com/farmnerds-net/proxmox-recipes), which
only takes apps with an official native install. Apps that qualify there go
there, not here.

## Engine

This repo has **no engine of its own**. Every `ct/<app>.sh` sets
`RECIPES_APP_REPO="farmnerds-net/proxmox-recipes-experimental"` and loads
`misc/build.func` from proxmox-recipes. The engine then takes
`install/<app>.sh` from this repo, and the container's `recipe update`
follows this repo too. The end of every `ct/<app>.sh`:

```bash
if [[ -n "${RECIPES_SRC:-}" ]]; then
  # shellcheck source=/dev/null
  source "${RECIPES_SRC%/}/misc/build.func"
else
  engine=$(curl -fsSL --retry 2 --max-time 30 \
    "https://raw.githubusercontent.com/farmnerds-net/proxmox-recipes/${RECIPES_REF:-main}/misc/build.func") || {
    printf 'proxmox-recipes: could not download misc/build.func\n' >&2
    exit 1
  }
  # shellcheck source=/dev/null
  source /dev/stdin <<<"$engine"
fi

recipe_main
```

`RECIPES_APP_REPO` goes in the metadata block above it, with the `APP_*`
variables.

- Never copy `misc/` into this repo. Engine changes go to proxmox-recipes,
  through a PR there.
- An `install/<app>.sh` follows proxmox-recipes' app-script contract
  (`app_install`, `app_update`, optional `app_version`, `app_web`,
  `APP_PORTS`, `APP_CONFIG`) and its Bash rules: `set -Eeuo pipefail` from
  the engine, quote everything, ShellCheck clean.
- Local testing uses both checkouts:
  `RECIPES_SRC=/root/proxmox-recipes RECIPES_APP_SRC=/root/proxmox-recipes-experimental bash ct/<app>.sh`

## Scope rules (hard)

- **Debian LXC only, unprivileged.** No Ubuntu, Alpine, or other templates.
- **Native installs only.** No Docker, no Podman, no container-in-container.
- **Derive from upstream's own files, and cite them.** The install may
  follow upstream's Dockerfiles, compose files, entrypoint scripts and
  release artifacts. The top of `install/<app>.sh` lists each upstream file
  it follows, with the release tag it was written against.
- **Pin and check versions.** Install a specific upstream release, never a
  moving branch. `app_update` refuses (with a clear message) to move to a
  release it was not written for when upstream's build changed in a way the
  script can't handle; it must not half-upgrade.
- **Clean-room.** Write it yourself from upstream's files. Do not copy code,
  names or flow from community-scripts/ProxmoxVE, other helper-script
  projects, or other people's native ports of the same app (e.g.
  "immich-native"), and don't read their source while writing ours.
- **Label it.** Every app doc starts with the experimental warning (see
  `docs/apps/`), saying upstream doesn't support this install and where to
  get help (not upstream's issue tracker).
- **No telemetry.** Turn off any the app or its build tools have.

## Testing rules (hard)

The same as proxmox-recipes, because Claude Code runs as root on a Proxmox
host:

- **Test on farmlab2 only**, never the production host.
- **Only create, modify, or destroy containers with IDs 900–999.** Never
  touch any other CT or VM.
- **Never change host config** (`/etc/network/`, `/etc/pve/` beyond the 9xx
  CTs `pct` manages, storage, firewall, cluster). If a recipe seems to need
  that, stop and ask.
- Before `pct destroy`, confirm the ID is in 900–999. Destroy test
  containers once a test passes.
- A recipe is done only when ShellCheck passes, a real install succeeds end
  to end, and `recipe update` has been run on it.

## Git

- Branch per recipe, push it, open a PR to `main` for the user to review.
  Don't push straight to `main`.
- Commit messages: short imperative subject, body explains why.
