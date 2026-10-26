# Changelog

## Unreleased

- **Fedora Atomic / Bazzite support**: image-based hosts (rpm-ostree/bootc) are detected via
  `/run/ostree-booted`, or chosen with the new `--host auto|atomic|classic` (`--atomic`,
  `--classic`). Atomic mode never runs `dnf install` (missing tools stop with an
  `rpm-ostree install` hint) and never self-updates an RPM in place.
- **Fixed**: user folders and `--share` dirs are now mounted at their logical path too
  (`/home/you/…`), not only the resolved one (`/var/home/you/…`). On hosts where `/home` is a
  symlink, drag-and-drop and *Attach file* from the file manager silently failed. Existing
  containers are offered a `--recreate`.
- **Fixed**: an unset XDG folder (which falls back to `$HOME`) could mount the whole home when
  `$HOME` is a symlink — the guard now compares resolved paths.
- **Fixed**: the launcher could start itself inside the container and crash with
  `exec: distrobox: not found` when the real home is mounted in the box (atomic hosts,
  `--full-home`): `command -v claude-desktop` found the host launcher in `~/.local/bin`. The
  app binary is now looked up from the package (`dpkg -L`). Fix it with *Repair app-menu entry*
  (`--refresh-launcher`).
- **NVIDIA**: distrobox `--nvidia` is passed automatically when `nvidia-smi -L` works; override
  with `--nvidia` / `--no-nvidia`.
- Explicit `--host` / `--nvidia` choices carry over to GUI actions, the launcher's right-click
  actions and the auto-update timer.
- On atomic hosts the installer warns that distrobox always mounts the real home
  (`/var/home/$USER`, and over `/home/$USER` at every start), so home isolation is off there
  (documented in the README).
- **Fixed** (atomic hosts): that `/home/$USER` bind made the read-only Documents/Desktop/Pictures
  mounts writable again. An `--init-hooks` step now re-binds them read-only at every container
  start. Containers created earlier need `--recreate`.

## 1.3.1 — 2026-09-11

- **Fixed**: the Claude window showed only a close button — no minimize/maximize — because
  the isolated container home has no dconf database, so GTK fell back to GNOME's default
  `appmenu:close` layout. The launcher now mirrors the host's
  `org.gnome.desktop.wm.preferences button-layout` into a one-key GSettings keyfile in the
  container home and starts the app with `GSETTINGS_BACKEND=keyfile`. Applies on the next
  launch after running *Repair app-menu entry* (or any update), which rewrites the launcher.

## 1.3.0 — 2026-08-30

- **Renamed**: the package is now `claude-desktop-fedora` (was `claude-desktop-distrobox`)
  and the script is `claude-desktop-setup.sh` (was `install-claude-desktop-distrobox.sh`),
  matching the repo's new name and the installed `claude-desktop-setup` command. Upgrading
  replaces the old package automatically (`Obsoletes:`) — nothing to uninstall first, and
  self-update from 1.1.1+ handles it on its own. Container, data, login and timers are untouched.

## 1.2.0 — 2026-08-30

- **Native setup window**: Claude Desktop Setup is now a real GTK4/libadwaita window
  (python3-gobject, preinstalled on Fedora Workstation) instead of a zenity button dialog —
  action rows for *Update now* and *Check for tool update*, an **Auto-update** schedule
  selector (*Off / Daily / Weekly / Monthly*, one click applies), and maintenance/removal
  tucked into an *Advanced* expander. Settings apply in place with a toast — the window no
  longer closes and reopens for them. Without the GTK bindings it falls back to the same
  actions as stacked zenity buttons.
- **No more Settings… sub-menu**: auto-update and the setup-tool update check live right in
  the main window; only repair / rebuild / remove stay under *Advanced*.
- **Opt-in scheduled tool-update check**: an *Also check for setup-tool updates* switch in
  the Auto-update section (CLI: `--install-timer --tool-check`) makes the update timer also ask
  GitHub whether a newer setup tool exists and send a desktop notification. Strictly
  notify-only and off by default — without it the tool still never contacts GitHub in the
  background.

## 1.1.1 — 2026-08-30

- **The setup tool can update itself**: *Advanced… → Check for a newer setup tool* (or
  `--check-self-update`) now offers to download the new release RPM, verify it against the
  release's `SHA256SUMS` and install it through polkit. New `--self-update` flag does it without
  asking. Git checkouts still just get the download page.
- Claude Desktop Setup now ships its own icon (the generic `system-software-install`
  theme icon is missing in many icon themes, leaving a placeholder in the dock).

## 1.1.0 — 2026-08-30

- **Auto-update frequency is configurable**: `--install-timer --every daily|weekly|monthly`
  (weekly stays the default).
- **Claude Desktop Setup redesigned**: a plain button window (*Update now / Settings… /
  Advanced… / Close*) instead of nested radio lists. *Settings…* has an **Auto-update**
  drop-down (Off / Daily / Weekly / Monthly); maintenance and removal live under *Advanced…*.
- **Right-click the Claude icon → "Claude Desktop Settings"** opens the setup window directly
  (new desktop action next to *Check for updates*). Existing installs get it after
  *Advanced… → Repair app-menu entry* (or `--refresh-launcher`) and a re-login.
- README: "How is this different?" comparison, setup-menu screenshot; added `CONTRIBUTING.md`
  and `SECURITY.md`.

## 1.0.0 — 2026-08-30

First packaged release.

- **RPM package** (`claude-desktop-distrobox`) — download, double-click, done. Pulls in
  distrobox, podman and zenity as dependencies. Installs `claude-desktop-setup` and a
  **Claude Desktop Setup** app-menu entry.
- **GUI setup app** (`--gui`): install (optionally sharing one folder), update, turn the
  weekly auto-update on/off, repair the launcher, rebuild the container, remove — all with
  progress dialogs.
- **"Attach file" and drag-and-drop work out of the box.** The app is handed real host paths
  (`/home/you/Documents/…`), so the standard user folders are bind-mounted at their real paths —
  `~/Documents`, `~/Desktop`, `~/Pictures` read-only, `~/Downloads` read-write — and linked into
  the dedicated home / GTK sidebar so the file picker lands on them. `--no-user-dirs` opts out.
- `--recreate` action (GUI: **Rebuild the container**) to rebuild the container with the
  current options while keeping login and data; containers made by pre-release builds need it
  once to get the folder mounts.
- **Suspend/resume safe.** After waking on a different network the launcher notices the
  container's stale IP address (pasta copies it once at start) and restarts the container; the
  weekly auto-update timer waits 30 s after a wake-triggered start so it doesn't race the network.
  The app is also given the host's system D-Bus (`--no-system-bus` to opt out) so Electron gets
  logind's real suspend/resume events instead of inferring them from a late timer.
- **Check for updates** action on the Claude launcher (right-click the dock/app icon).
- **Desktop notification** after the weekly auto-update runs (`--notify`).
- **Self-update check** against GitHub Releases (`--check-self-update`), only on request.
- `--version`, `--install` flags.
- `DEBIAN_FRONTEND` reaches apt through sudo; stale icon files are pruned on update; update
  output is logged to a file so timer runs no longer die on SIGPIPE.
