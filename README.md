# proxmox-recipes-experimental

Experimental recipes that install self-hosted apps natively into Proxmox
LXC containers, **in ways the apps' developers do not support.**

> **Warning:** These recipes are for apps whose developers only support
> Docker. Each recipe installs the app without Docker by doing by hand what
> the app's own Docker files do. That can break when the app changes, and
> the app's developers won't help with it: please don't report problems with
> these installs to them. If you want something that just keeps working,
> run the app the way its developers support.

For apps with an official native install, use
[proxmox-recipes](https://github.com/farmnerds-net/proxmox-recipes). This
repo uses its engine, so containers look and behave the same: Default and
Advanced setup, container notes, logs, and the `recipe` command inside.

## Apps

| App | Recipe | Notes |
| --- | ------ | ----- |

None yet.

## Usage

Run this on the Proxmox host shell, replacing `<app>` with the recipe you want:

```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/farmnerds-net/proxmox-recipes-experimental/main/ct/<app>.sh)"
```

> **Warning:** Recipes run as **root on your Proxmox host**. Read a recipe
> before you run it.

Inside the container, `recipe update` upgrades the OS and then the app,
`recipe info` shows the version, IP, ports and config, and `recipe help`
lists the commands. See
[proxmox-recipes](https://github.com/farmnerds-net/proxmox-recipes#usage)
for setup modes, presets, logs and failures.

### Testing a branch or local checkouts

```bash
RECIPES_APP_REF=<branch> bash ct/<app>.sh   # app scripts from that branch of this repo
RECIPES_REF=<branch> bash ct/<app>.sh       # engine from that branch of proxmox-recipes
RECIPES_SRC=/path/to/proxmox-recipes RECIPES_APP_SRC=/path/to/proxmox-recipes-experimental bash ct/<app>.sh
```

## Layout

- `ct/` — host-side recipe, one per app. Loads the engine from proxmox-recipes.
- `install/` — the install script for each app. Runs inside the container.
- `docs/apps/` — one page per app, starting with what is unsupported about it.

## Support

If these recipes save you time, you can
[support farmnerds on Ko-fi](https://ko-fi.com/farmnerds).

## License

[MIT](LICENSE)
