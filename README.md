# dotfiles

Personal config sync for [Omarchy](https://omarchy.org) (Arch + Hyprland).

![Omarchy setup](screenshots/omarchy-setup.png)

## Branches

| Branch | Contents |
|--------|----------|
| `omarchy` | Active Omarchy visual setup (this branch) |
| `mint` | Archived Linux Mint + i3 setup |

## What's synced

This repo stores **only your changes on top of a stock Omarchy install** — not
a copy of Omarchy itself. A fresh Omarchy install seeds `~/.config` with all its
defaults from `/etc/skel`; this repo carries just the files you modified or
added, so pulling it reproduces this device's setup exactly.

Tracked (genuine deviations from stock):

- `hypr/input.lua`, `hypr/looknfeel.lua` — the two window-manager files you
  customized (the rest of `hypr/` is stock and left alone)
- `omarchy/shell.json` — clock format, weather unit, custom network plugin, power %
- `omarchy/firefox/` — square + frosted-glass static theme
- `omarchy/packages/` — app add/remove manifest
- `omarchy/scripts/` — `sync-apps`, `sync-plymouth`, `vesktop-theme-sync`
- `omarchy/hooks/` — custom hooks (voxtype, agent, fingerprint, vesktop)
- `omarchy/plugins/zephyr.network/` — custom bar network module
- `omarchy/sddm/`, `omarchy/plymouth/` — boot/login theme backups
- `omarchy/branding/`, `omarchy/defaults/agent`
- `omarchy/themes/vantablack-pastel/` — custom theme (fork of stock `vantablack`)
- `README.md`, `.gitignore`, `screenshots/`

Explicitly excluded by `.gitignore` (stock-identical, re-seeded by a fresh
install — not "your changes"):

- Stock terminal configs: `alacritty/`, `foot/`, `kitty/`, `ghostty/`
- Stock Hypr files: `bindings.lua`, `hyprland.lua`, `autostart.lua`,
  `hyprsunset.conf`, `.luarc.json`, `xdph.conf`
- Stock Omarchy files: `extensions/omarchy-menu.jsonc`,
  `themed/alacritty.toml.tpl.sample`, the sample hooks under `hooks/*/`
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

## Firefox (static theme)

Firefox uses a **static** square + frosted-glass theme that is deliberately
**not** synced to Omarchy themes — switching the Omarchy theme does not recolor
or restart Firefox. The frozen templates live in `omarchy/firefox/`:

- `base.userChrome.css` — the glass UI (square corners, frosted surfaces)
- `base.userContent.css` — square corners on `about:*` pages

To re-apply this look to the active profile after a fresh install:

```bash
cp ~/.config/omarchy/firefox/base.userChrome.css ~/.config/mozilla/firefox/*.default-release/chrome/userChrome.css
cp ~/.config/omarchy/firefox/base.userContent.css ~/.config/mozilla/firefox/*.default-release/chrome/userContent.css
```

Requires Firefox's `toolkit.legacyUserProfileCustomizations.stylesheets=true`
(user.js) plus `widget.wayland.opaque-region.enabled=false` for the glass to render.

**Important — Firefox must use the System theme, not the built-in Dark theme.**
Set this in Firefox (Settings → General → Appearance → Theme → System) or set the
pref in the active profile:

```js
// about:config / prefs.js
user_pref("extensions.activeThemeID", "default-theme@mozilla.org");
```

The built-in Dark theme (`firefox-compact-dark@mozilla.org`) repaints the whole
chrome over `userChrome.css` and keeps the UI in default grey — only this
System-theme setting lets the glass colors in `base.userChrome.css` take effect.
The stylesheet also forces `color-scheme: dark` so text/icons stay light on the
black glass.

## Boot / login screen (plymouth + SDDM)

The customized boot splash and login theme live in `/usr/share` (not `~/.config`),
so a stock `omarchy update` would overwrite them. They're backed up here:

- `omarchy/plymouth/` — boot splash theme
- `omarchy/sddm/` — login (post-logout) theme

Restore them (e.g. after an update reverts them, or on a fresh install):

```bash
~/.config/omarchy/scripts/sync-plymouth     # copies both into /usr/share + rebuilds initramfs
```

## Restore on a fresh Omarchy install

A fresh install seeds `~/.config` with stock defaults (from `/etc/skel`) and
this repo carries only your changes on top of those defaults. So graft the repo
onto the existing `~/.config` rather than cloning into it:

```bash
cd ~/.config
git init -b omarchy
git remote add origin https://github.com/Zephyr73/dotfiles.git
git fetch origin
git reset --hard origin/omarchy   # overlays your tracked changes; stock files elsewhere stay as stock
git branch -u origin/omarchy      # so future updates are just `git pull`
```

> `git reset --hard` only restores the tracked files. Because the repo tracks
> only your deviations, all stock files keep their fresh-install defaults — that
> is correct. Your two customized Hypr files (`input.lua`, `looknfeel.lua`) and
> `omarchy/shell.json` are restored over their stock versions by the reset.

Then regenerate the machine-specific monitor config if missing, apply apps, and
reload:

```bash
omarchy refresh config hypr/monitors.lua   # only needed if the file is missing
~/.config/omarchy/scripts/sync-apps        # install/remove apps per manifest
omarchy restart shell                      # reload bar/shell (Hyprland auto-reloads WM config)
```

Firefox theming and the boot/login theme need manual re-apply after a fresh
install (see the Firefox and plymouth/SDDM sections above).

## Keeping machines in sync

```bash
cd ~/.config
git add -A && git commit -m "update" && git push   # save changes
git pull                                            # pull changes on another machine
```
