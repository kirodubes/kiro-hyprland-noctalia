# kiro-hyprland-noctalia — Claude project instructions

## Overview
Config package for the **Kiro Hyprland (noctalia) edition** — the Hyprland member
of the KIROTUX Wayland line paired with noctalia-shell (sibling to
[kiro-niri-noctalia](../kiro-niri-noctalia/CLAUDE.md) and the waybar-based
[kiro-hyprland](../kiro-hyprland/CLAUDE.md)). **Public, open-core**, shipped via
`nemesis_repo`. Full research + decisions live in the internal
`Kiro-HQ/Kirotux/study-of-hyprland-noctalia.md`.

## Edition spec (the WM-variable matrix)
- **Compositor:** Hyprland (wlroots-family, tiling). Config reused from the
  `kiro-hyprland` edition, waybar stack swapped for noctalia.
- **Config language:** Lua (`hl.*` API, Hyprland 0.55+). Single file
  `etc/skel/.config/kiro-hyprland-noctalia/hyprland.lua` (monolithic, like the
  waybar edition — not split into modules).
- **Desktop shell:** **noctalia-shell** (Quickshell v4) — bar, launcher, lock,
  notifications, wallpaper, control center, session menu, polkit agent (via its
  plugin). Driven over `qs -c noctalia-shell ipc call <module> <action>`.
  Hyprland is only the compositor.
- **Autostart:** Hyprland's `on_start(cmd)` (fires on the `hyprland.start`
  event). Launches `qs -c noctalia-shell`, `xdg-user-dirs-update`, the gsettings
  import, dbus/systemd env propagation, and the archiso-gated Calamares line.
- **Coexistence (the key design point):** namespaced. Config in
  `~/.config/kiro-hyprland-noctalia/`; the `kiro-hyprland-noctalia-session`
  wrapper runs `Hyprland --config ~/.config/kiro-hyprland-noctalia/hyprland.lua`;
  own `kiro-hyprland-noctalia.desktop` session entry. **No `conflicts=`** — all
  Kiro editions install together and are picked per-login. Hyprland is launched
  directly (no systemd service), so `--config` is enough; the niri editions need
  the `NIRI_CONFIG` env var only because niri starts via `systemctl`.
- **Theming:** noctalia owns runtime accent colours (matugen internally). Base
  GTK look + cursor shipped via `/etc/dconf/` by `kiro-wayland-dotfiles` (this
  edition is a dconf-only consumer). `import-gsettings.sh` mirrors GTK theme into
  gsettings at session start.
- **Dependency note:** `noctalia-shell` (+ `noctalia-qs`) come from
  **chaotic-aur / cachyos** (both in Kiro's `pacman.conf`) — **do not** repackage
  into `nemesis_repo`.

## Keybindings
- SUPER-based Kiro scheme, reused from `kiro-hyprland`. noctalia actions
  (launcher / control center / settings / lock / wallpaper) routed to
  `qs -c noctalia-shell ipc call`. `kiro-keybindings` on `SUPER+CTRL+S`.
- `keybindings.txt` mirrors `hyprland.lua` — keep them in lockstep; a
  duplicate-chord scan must pass.
- Notable change from the waybar edition: `swapwithmaster` moved
  `SUPER+CTRL+Return` → `SUPER+CTRL+Space` (Return now opens the launcher).

## Patterns / gotchas
- `hl.dsp.exec_cmd(cmd)` (the keybind `run()` helper) goes through `/bin/sh`, so
  the noctalia IPC strings, `$(slurp)` and pipes in keybinds work. `hl.exec_cmd`
  in `on_start` execs **argv directly** (no shell) — the Calamares line is
  wrapped in `sh -c` for its `[ ]`/`&&`.
- noctalia provides lock **and** idle — do not add hypridle/hyprlock/swayidle.
- noctalia provides the polkit agent — depend on `polkit`, not `polkit-gnome`,
  and do not autostart a separate agent.
- Screencast works through `xdg-desktop-portal-hyprland` (hard dep) — no GNOME
  portal needed (unlike niri, which optdepends `xdg-desktop-portal-gnome`).
- **Hyprland `--config` + Lua:** the wrapper points `-c` at `hyprland.lua`;
  Hyprland detects the format by extension. Verify on a real Hyprland boot that
  `--config` loads the `.lua` correctly and that the namespaced `scripts/` path
  resolves.

## Known follow-up
- Does **not** hide upstream `hyprland.desktop` (the private waybar
  `kiro-hyprland` still launches through it). On a box with only namespaced Kiro
  Hyprland editions, upstream "Hyprland" is a blank-config trap — fix by
  migrating `kiro-hyprland` to a namespaced session, then hiding upstream (as the
  niri editions hide `niri.desktop`).

## Build / delivery
- Source-of-truth for the config; delivered as the `kiro-hyprland-noctalia`
  package via `../KIROTUX-PKG-BUILD/kiro-hyprland-noctalia/build.sh` (public
  recipe → `~/EDU/nemesis_repo/`). After editing here: rebuild the package, then
  the ISO to test a fresh install.
- See [../CLAUDE.md](../CLAUDE.md) for the full KIROTUX delivery architecture.
