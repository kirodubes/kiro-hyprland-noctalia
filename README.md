# kiro-hyprland-noctalia

The **Kiro Hyprland (noctalia) edition** — the Hyprland member of the KIROTUX
Wayland line, paired with the **noctalia-shell** desktop shell. Public,
open-core, shipped through the Kiro package repo (`nemesis_repo`).

Sibling to [kiro-niri-noctalia](../kiro-niri-noctalia/) (niri + noctalia) and the
waybar-based `kiro-hyprland` edition.

## What it is

A ready-to-run Hyprland desktop configured the Kiro way: SUPER-based keybinds,
Kiro app defaults, Tokyo Night accents, and the whole shell (bar, launcher, lock
screen, notifications, wallpaper, control center, session menu, polkit agent)
provided by **noctalia-shell** (Quickshell v4). Hyprland is only the compositor;
everything else is noctalia, driven over `qs -c noctalia-shell ipc call`.

The edition is fully namespaced: its config lives in
`~/.config/kiro-hyprland-noctalia/` and it ships its own display-manager session
entry, so it installs and runs **next to** the other Kiro editions without
clobbering `~/.config/hypr/`.

## What it ships

- `etc/skel/.config/kiro-hyprland-noctalia/hyprland.lua` — the Hyprland config (Lua, 0.55+)
- `etc/skel/.config/kiro-hyprland-noctalia/scripts/import-gsettings.sh` — mirrors GTK theme into gsettings
- `etc/skel/.config/kiro-hyprland-noctalia/bg/kiro.jpg` — Kiro wallpaper
- `etc/skel/.config/kiro-hyprland-noctalia/keybindings.txt` — keybinding reference
- `etc/skel/.config/noctalia/{settings,colors,plugins}.json` — noctalia-shell config
- `usr/bin/kiro-hyprland-noctalia-session` — session wrapper (`Hyprland --config …`)
- `usr/share/wayland-sessions/kiro-hyprland-noctalia.desktop` — "Kiro Hyprland Noctalia" login entry

## How to install

On a Kiro system (the package repo is already wired into `pacman.conf`):

```bash
sudo pacman -S kiro-hyprland-noctalia
```

Then log out and pick **Kiro Hyprland Noctalia** from your display manager.

## Keybindings

`SUPER` is the mod key. `SUPER + CTRL + S` opens the searchable keybinding
cheatsheet (`kiro-keybindings`); the full list is in
[`keybindings.txt`](etc/skel/.config/kiro-hyprland-noctalia/keybindings.txt).
