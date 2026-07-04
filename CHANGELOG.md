# Changelog

## 2026.07.04

### Shared noctalia config extracted to `kiro-noctalia`

**What Changed.** Removed the `etc/skel/.config/noctalia/` folder from this
package and added `kiro-noctalia` to `depends=()`. noctalia reads the fixed path
`~/.config/noctalia/`, so the identical config shipped here and by
`kiro-niri-noctalia` caused a pacman file conflict on
`/etc/skel/.config/noctalia/settings.json` — the two editions could not
co-install. The shared config now has a single owner, `kiro-noctalia`.

**Technical Details.** The compositor config (`~/.config/kiro-hyprland-noctalia/`),
session and wrapper are unchanged and still shipped here. Only the shared
`noctalia/` skel folder moved out. The golden copy at
`/usr/share/kiro/kiro-hyprland-noctalia/` now contains only this edition's config;
`kiro-noctalia` ships its own noctalia golden copy.

**Files Modified.** removed `etc/skel/.config/noctalia/*`; `../KIROTUX-PKG-BUILD/kiro-hyprland-noctalia/PKGBUILD` (depends + comment).

### Initial config package

**What Changed.** Stood up `kiro-hyprland-noctalia`, the Kiro Hyprland edition
paired with **noctalia-shell**. The Hyprland compositor config is adapted from
the waybar-based `kiro-hyprland` edition; the waybar/mako/swaybg/rofi/hypridle
shell stack is removed and replaced with noctalia-shell (bar, launcher, lock,
notifications, wallpaper, control center, session menu, polkit agent), driven
over `qs -c noctalia-shell ipc call`. noctalia config is shared with the niri
noctalia edition.

**Technical Details.**
- **Namespaced for co-installation.** Config lives in
  `~/.config/kiro-hyprland-noctalia/` (not `~/.config/hypr/`); a session wrapper
  (`kiro-hyprland-noctalia-session`) launches `Hyprland --config …` pointed at
  it, and the edition ships its own `kiro-hyprland-noctalia.desktop`. No
  `conflicts=` — installs next to every other Kiro edition, picked per-login.
  (Hyprland is launched directly, so a plain `--config` flag suffices — no
  `NIRI_CONFIG`-style env dance the niri edition needs.)
- Autostart: noctalia-shell + `xdg-user-dirs-update` + gsettings import + the
  archiso-gated Calamares line. Dropped from the waybar edition: waybar, mako,
  swaybg, hypridle, polkit-gnome, nm-applet, variety (all covered by noctalia).
- Keybinds: launcher / control center / settings / lock / wallpaper routed to
  noctalia IPC; rofi-theme-selector, betterlockscreen and Variety binds removed.
  `swapwithmaster` moved from `SUPER+CTRL+Return` to `SUPER+CTRL+Space` so
  `SUPER+CTRL+Return` can open the noctalia launcher.
- Deps: `hyprland noctalia-shell` + the Wayland utility stack; `polkit` (not
  polkit-gnome) since noctalia provides the agent; `xdg-desktop-portal-hyprland`
  handles screencast natively (no GNOME portal needed).

**Files Modified.**
- New repo — full initial tree (see README.md).

**Known follow-up.** This edition does not hide the upstream `hyprland.desktop`
session entry (the private waybar `kiro-hyprland` edition still launches through
it). On a box where only namespaced Kiro Hyprland editions exist, the upstream
"Hyprland" entry reads an empty `~/.config/hypr/` and is a blank-session trap —
resolve by migrating `kiro-hyprland` to its own namespaced session, then hiding
upstream.
