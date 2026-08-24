# dotfiles

Personal config sync for [Omarchy](https://omarchy.org) (Arch + Hyprland).

## Branches

| Branch | Contents |
|--------|----------|
| `omarchy` | Active Omarchy visual setup (this branch) |
| `mint` | Archived Linux Mint + i3 setup |

## What's synced

Whitelist approach via `.gitignore` — only these paths are tracked:

- `hypr/` — window manager config (bindings, look & feel, input, autostart)
- `omarchy/` — shell/bar layout (`shell.json`), themes, theme hooks, custom scripts
- `backgrounds/`
- Terminals: `alacritty/`, `foot/`, `kitty/`, `ghostty/`

Not synced on purpose:

- `hypr/monitors.lua` — machine-specific monitor layout
- Everything else in `~/.config` — browser profiles, app state, secrets
- Backup files (`*.bak.*`)

## Apps (packages)

Deviations from stock Omarchy are tracked as a manifest:

- `omarchy/packages/add.txt` — packages installed on top of stock
- `omarchy/packages/remove.txt` — stock packages debloated away

Apply on this machine after pulling changes:

```bash
~/.config/omarchy/scripts/sync-apps        # add/remove per manifest
~/.config/omarchy/scripts/sync-apps --dry-run   # preview only
```

AUR packages (`vesktop-bin`, `visual-studio-code-bin`) are detected and routed through `omarchy pkg aur add` automatically.

## Restore on a fresh Omarchy install

A fresh install already populates `~/.config` with defaults, so graft the repo
onto it instead of cloning into it:

```bash
cd ~/.config
git init -b omarchy
git remote add origin https://github.com/Zephyr73/dotfiles.git
git fetch origin
git reset --hard origin/omarchy   # replaces whitelisted defaults, touches nothing else
git branch -u origin/omarchy      # so future updates are just `git pull`
```

Then regenerate the machine-specific monitor config if missing, apply apps, and reload:

```bash
omarchy refresh config hypr/monitors.lua   # only needed if the file is missing
~/.config/omarchy/scripts/sync-apps        # install/remove apps per manifest
omarchy restart shell                      # reload bar/shell (Hyprland auto-reloads WM config)
```

## Keeping machines in sync

```bash
cd ~/.config
git add -A && git commit -m "update" && git push   # save changes
git pull                                            # pull changes on another machine
```
