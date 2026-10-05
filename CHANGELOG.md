# Changelog

## 2026.10.05

### What Changed
- Keybindings now follow the active keyboard layout: `resolve_binds_by_sym = true` in the `input` block. With `us,be`
  (or `be,us`), Hyprland used to read every bind as if the first layout were active, so after Alt+Shift you typed
  AZERTY but Super+letter binds stayed on their QWERTY key positions. Now Super+A is the A printed on the key in
  whichever layout is active. Workspace binds use `code:` keys (physical positions) and are unchanged. Tested on a
  QEMU install of kiro-hyprland-dms.
- `hyprland.lua` now ends with an **appearance loader**: `pcall(require, "appearance")` loads `appearance.lua` from
  the same folder when it exists. The upcoming Kirotux Hyprland Premium app writes the user's look there (only the
  values they changed), so it never edits `hyprland.lua` itself. No file = this edition's own look, as before.
- **The keyboard layout now follows the installer.** `kb_layout` used to be hardcoded, so a user who picked German
  in Calamares still typed US/Belgian in Hyprland. `hyprland.lua` now reads `XKBLAYOUT` / `XKBVARIANT` from
  `/etc/vconsole.conf` (written by `kiro_final` from the installer's choice); without it (live ISO) the old value stays.
  Kirotux Hyprland Premium can still override it per user in `appearance.lua`.
- **Ctrl+Alt+H opens Kirotux Hyprland Premium** (theme, icons, cursor, font, window look, wallpaper, keyboard,
  presets). The key used to start `hyprland-tweak-tool`, which no ISO ships any more. This edition is used with and
  without the app: where it isn't installed, the key shows a notification "Get Kirotux Hyprland Premium … Comes with
  the KIROTUX ISOs." instead of doing nothing (only on the key press, never by itself).
- `import-gsettings.sh` (runs at every login) now keeps a **Light** choice: it went `prefer-dark` unconditionally, so
  choosing Light in the app was undone at the next login. It also copies the cursor size into gsettings.

### Technical Details
- Added after `kb_options`. Hyprland's default (`false`) resolves symbol binds against the first layout in `kb_layout`.
- Proven on a QEMU kiro-hyprland-dms install (Hyprland 0.56.2): `package.path` starts with the config folder, a
  second `hl.config` merges, every reload re-reads the file, a missing file is silent, and a broken one is reported
  while the rest of the config still loads (`pcall`). The loader must stay the last lines so the user's choices win.
- `installer_keyboard(fallback)` (Lua `io`) above `hl.config`; `kb_layout = kb_layout`, `kb_variant = kb_variant`.
  Checked against `XKBLAYOUT=be`, quoted multi-layout values with variants, an empty value and a missing file.
  On the QEMU dms install (installer: Belgian) Hyprland went from `us,be` to `be`, no config errors. Fallback here: `us,be`.
- `import-gsettings.sh`: `color-scheme` follows `gtk-application-prefer-dark-theme` in `gtk-4.0/settings.ini`
  (`false`/`0` = light, anything else or no file = dark, as before); `gtk-cursor-theme-size` → `cursor-size`.
  Both new reads are `|| true`-guarded for the script's `set -euo pipefail`. Tested on the QEMU dms install.
- Bind added after Ctrl+Alt+E (replacing the `hyprland-tweak-tool` line where it still existed): `sh -c 'command -v
  kirotux-hyprland-premium && exec kirotux-hyprland-premium; exec notify-send …'`. Tested on QEMU both ways (window
  opens; DMS shows the notification, text fits). `keybindings.txt` says it comes with the KIROTUX ISOs.

### Files Modified
- `etc/skel/.config/kiro-hyprland-noctalia/hyprland.lua`
- `etc/skel/.config/kiro-hyprland-noctalia/scripts/import-gsettings.sh`
- `etc/skel/.config/kiro-hyprland-noctalia/keybindings.txt`

## 2026.09.05

### What Changed
- **Print now leaves a file behind.** The screenshot binds ran `grim … | wl-copy`, which put the
  image on the clipboard and nowhere else — no file, no notification, so the key looked dead next to
  chadwm's scrot bind that writes into `~/Pictures`. Both binds now call the shared
  `kiro-screenshot region` / `kiro-screenshot screen`, which saves a timestamped PNG in
  `~/Pictures/Screenshots`, still copies to the clipboard, and notifies with a thumbnail.

### Technical Details
- The helper is `/usr/bin/kiro-screenshot`, shipped by `kiro-wayland-dotfiles` — already a dependency
  of this package. The same clipboard-only line had been copy-pasted into twelve editions, so it now
  lives in exactly one place instead of being fixed twelve times.
- `hyprland.lua`: `run("kiro-screenshot region")`. noctalia has no popout-hiding IPC of its own, so
  the helper's DMS handshake is a no-op here (guarded by `command -v dms`) and the grab is plain.
- **Build `kiro-wayland-dotfiles` first**: it provides the binary this edition's binds call.

### Files Modified
- `etc/skel/.config/kiro-hyprland-noctalia/hyprland.lua`
- `etc/skel/.config/kiro-hyprland-noctalia/keybindings.txt`

## 2026.07.07

### Keyboard: US default + Alt+Shift layout toggle

**What Changed**
- Flipped the layout order `be,us` → **`us,be`**: US QWERTY is now the default at login, Belgian AZERTY the secondary layout.
- Added **`grp:alt_shift_toggle`** to the xkb options (now `grp:alt_shift_toggle,compose:caps`) so **Alt+Shift** switches `us`↔`be`. Before, the second layout was loaded but had no switch key — unreachable.

**Technical Details**
- `grp:alt_shift_toggle` matches the CachyOS Calamares reference (`keyboard/Config.cpp` defaults the group switcher to it when a second layout exists). `compose:caps` (Caps = Compose) unchanged. `input.kb_layout` / `kb_options` in `hyprland.lua`.

**Files Modified**
- `etc/skel/.config/kiro-hyprland-noctalia/hyprland.lua`

## 2026.07.04

### Fix app-launcher icons rendering as magenta/black placeholders

**What Changed.** Added `hl.env("QT_QPA_PLATFORMTHEME", "gtk3")` to `hyprland.lua`.
noctalia-shell 4.7.7 is a Qt6/Quickshell app and resolves app icons through Qt's
platform theme. The session inherited `QT_QPA_PLATFORMTHEME=qt5ct` from
`/etc/environment`, but qt5ct is **Qt5** (and isn't even installed), so Qt6 could
not load it and fell back to the `hicolor` default. Result: every app whose icon
comes from the *theme* rather than its own package (ARandR→`display`, the Avahi
tools→`network-wired`, nm-connection-editor, ATT, archlinux-logout) rendered
noctalia's magenta/black "missing" placeholder; apps shipping their own hicolor
icon were fine. Pointing Qt at the `gtk3` platform theme (plugin `libqgtk3.so`,
already present) makes Qt6 read the GTK icon theme (Surfn, via gsettings) — same
source GTK apps use. **Verified live** on the VM: restarting noctalia with this
env resolved the launcher grid to real Surfn icons.

**Note.** This is the Qt half of the fix; the GTK/gsettings half was
`kiro-wayland-dotfiles` compiling its dconf defaults (same-day). The sibling
`kiro-niri-noctalia` already set `QT_QPA_PLATFORMTHEME=gtk3` in its `misc.kdl` —
this brings the Hyprland edition in line. The systemic `/etc/environment=qt5ct`
still mis-themes Qt6 apps on the other Hyprland/wlroots editions (kiro-hyprland
waybar, wayfire, …) — worth fixing at the ISO level separately.

**Files Modified.** `etc/skel/.config/kiro-hyprland-noctalia/hyprland.lua`.

### Silence the new "started without start-hyprland" banner

**What Changed.** Added `misc.disable_watchdog_warning = true` to
`hyprland.lua`. Current Hyprland ships a `start-hyprland` wrapper that launches
the compositor with a crash-recovery watchdog fd (`--watchdog-fd`); when
Hyprland is exec'd without one it logs a persistent on-screen banner. This is a
*different, newer* check than the older "started without UWSM" nag — launching
via `uwsm start -- Hyprland --config` (see below) does **not** pass a watchdog
fd, so the banner still fired. Since our whole edition-coexistence model relies
on `Hyprland --config <namespaced>`, the config knob is the intended escape
hatch — no launch-path change needed.

**Files Modified.** `etc/skel/.config/kiro-hyprland-noctalia/hyprland.lua`.

### Launch through uwsm (fixes the "started without UWSM" banner)

**What Changed.** The session wrapper now launches Hyprland via
`uwsm start -- Hyprland --config …` instead of a bare `Hyprland --config`.
Hyprland (0.49+) shows a persistent on-screen banner nagging that it was started
without uwsm; uwsm is also the upstream-recommended launch method (systemd user
session + environment import). Added `uwsm` to `depends=()`.

**Note.** The two `on_start` env-propagation lines in `hyprland.lua`
(`dbus-update-activation-environment` + `systemctl --user import-environment`)
are now redundant under uwsm (it owns environment import) but left in place —
harmless. Revisit if a cleaner `uwsm finalize` is wanted.

**Files Modified.** `usr/bin/kiro-hyprland-noctalia-session`;
`../KIROTUX-PKG-BUILD/kiro-hyprland-noctalia/PKGBUILD` (depends).

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
