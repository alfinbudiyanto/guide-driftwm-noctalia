# driftwm-noctalia
user guide to install driftwm with noctalia for Fedora Linux

## Source Code
- [driftwm](https://github.com/malbiruk/driftwm)
- [driftwm-noctalia](https://github.com/youssefvdel/driftwm-noctalia) — noctalia shell fork adapted for driftwm
---

# *driftwm*

## Install

### Build from source

Requires Rust 1.88+ (edition 2024).

Install build dependencies:

**Fedora:**

```bash
sudo dnf install libseat-devel libdisplay-info-devel libinput-devel mesa-libgbm-devel libxkbcommon-devel
```

Then build and install:

```bash
git clone https://github.com/malbiruk/driftwm.git
cd driftwm
make build
sudo make install
```

To uninstall, run `sudo make uninstall` from the repository.

Then build and install:

```bash
git clone https://github.com/malbiruk/driftwm.git
cd driftwm
make build
sudo make install
```

To uninstall, run `sudo make uninstall` from the repository.

### Optional runtime dependencies

driftwm runs standalone — none of these are required — but each enables or
improves a feature:

- `xwayland-satellite` (≥ 0.7) — X11 app support (see below).
- `xdg-desktop-portal` + `xdg-desktop-portal-wlr` (≥ 0.8.0) or `xdg-desktop-portal-cosmic` — screencasting, and screenshot apps that go through the portal (e.g. Flameshot). wlr needs a dmenu-style picker in `$PATH` (`wmenu`/`wofi`/`rofi`/`bemenu`/`mew`/`fuzzel`) to choose what to share.
- `grim` + `slurp` — screenshots (+ cropping to region). driftwm also has a built-in canvas/DPI capture: see [IPC › Screenshots](docs/ipc.md#screenshots).
- `adwaita-fonts` — renders SSD title bars in `Adwaita Sans` to match GTK apps; without it a generic sans-serif is substituted. Font, size, weight, and alignment are configurable under `[decorations]`.
- A cursor theme — most desktops set one up already; on a bare install driftwm falls back to a basic built-in arrow.

**X11 apps** run through [xwayland-satellite](https://github.com/Supreeeme/xwayland-satellite),
which driftwm spawns at startup, exporting `DISPLAY=:N` so X11 clients connect
transparently — no extra config beyond having the binary in `$PATH`.

- **Arch:** `sudo pacman -S xwayland-satellite`
- **Fedora:** `sudo dnf install xwayland-satellite`
- **NixOS:** `pkgs.xwayland-satellite`
- **Debian/Ubuntu:** not yet packaged — `cargo install --locked xwayland-satellite`

If satellite isn't found at startup, driftwm logs a warning and continues without
X11 support. You can override the binary path or disable the integration in
[`config.reference.toml`](config.reference.toml) under `[xwayland]`.

### Running

driftwm auto-detects whether it's running nested (inside an existing Wayland
session) or on real hardware (from a TTY). Just run `driftwm`. For display
manager integration, select "driftwm" from the session menu.

> [!TIP]
> When launched by a display manager, driftwm runs as a systemd user service — view logs with `journalctl --user -u driftwm.service` (add `-f` to follow). Run directly and logs go to stderr.

## Quick start

`mod` is Super by default. Terminal and launcher are auto-detected (foot/alacritty/kitty, fuzzel/wofi/bemenu); override in config.

| Shortcut           | Action        |
| ------------------ | ------------- |
| `mod+return`       | Open terminal |
| `mod+d`            | Open launcher |
| `mod+q`            | Close window  |
| `mod+l`            | Lock screen   |
| `mod+ctrl+shift+q` | Quit          |

Feature-specific bindings (navigation, zoom, snap) are in their respective sections above.

## Configuration

Config file: `~/.config/driftwm/config.toml` (respects `XDG_CONFIG_HOME`).

```bash
mkdir -p ~/.config/driftwm
cp /etc/driftwm/config.reference.toml ~/.config/driftwm/config.toml
```

Missing file uses built-in defaults. Partial configs merge with defaults —
only specify what you want to change. Use `"none"` to unbind a default binding.
Validate without starting: `driftwm --check-config`.

```toml
# Launch programs at startup
autostart = ["waybar", "swaync", "swayosd-server"]
```

Every option is documented in **[docs/config.md](docs/config.md)** (generated
from [`config.reference.toml`](config.reference.toml)): input settings,
scroll/momentum tuning, snap behavior, decorations, effects, per-output config,
gesture bindings, mouse bindings, touch bindings, and window rules.


## License

GPL-3.0-or-later

---
# *Noctalia*

## Installation

### Prerequisites

- **driftwm** installed and running (`curl -fsSL https://raw.githubusercontent.com/malbiruk/driftwm/main/install.sh | sudo sh`)
- **Build tools**: cmake, ninja, gcc/clang
- **Qt6 development**: qt6-qtbase-devel, qt6-qtdeclarative-devel, qt6-qtwayland-devel
- **Runtime deps**: pipewire, wireplumber (for audio), upower (for battery)

### Fedora

```bash
# Install build dependencies
sudo dnf install cmake ninja-build gcc-c++ \
  qt6-qtbase-devel qt6-qtdeclarative-devel qt6-qtwayland-devel \
  qt6-qtbase-private-devel wayland-devel wayland-protocols-devel \
  pipewire-devel libxkbcommon-devel

# Install runtime dependencies
sudo dnf install wlr-randr wl-clipboard brightnessctl
```

---

### Step 1: Install noctalia-qs

Build the Quickshell fork from source (required — standard quickshell lacks noctalia's QML APIs):

```bash
git clone --depth 1 https://github.com/noctalia-dev/noctalia-qs.git ~/builds/noctalia-qs
cd ~/builds/noctalia-qs
cmake -GNinja -B build \
  -DCMAKE_BUILD_TYPE=RelWithDebInfo \
  -DCMAKE_INSTALL_PREFIX=/usr/local
cmake --build build
sudo cmake --install build
```

Verify:
```bash
qs --version
# Expected: noctalia-qs 0.0.12 (or newer)
```

### Step 2: Install driftwm-noctalia

```bash
mkdir -p ~/.config/quickshell
git clone https://github.com/youssefvdel/driftwm-noctalia.git \
  ~/.config/quickshell/noctalia-shell
```

### Step 3: Configure driftwm

Add to `~/.config/driftwm/config.toml`:

```toml
# Launch the shell on startup
autostart = ["qs -c noctalia-shell -d"]

# Keybindings for noctalia panels
[keybindings]
"mod+d"   = "spawn qs ipc -c noctalia-shell call launcher toggle"
"mod+n"   = "spawn qs ipc -c noctalia-shell call notifications toggleHistory"
"mod+v"   = "spawn qs ipc -c noctalia-shell call launcher clipboard"
"mod+tab" = "spawn qs ipc -c noctalia-shell call sessionMenu toggle"
```

> **Important:** Use `-d` (daemonize) in the autostart — without it, the shell may not launch properly inside driftwm's autostart.

Apply config:
```bash
touch ~/.config/driftwm/config.toml   # driftwm auto-reloads on config change
```

Or restart driftwm.

### Step 4: Verify

After restarting driftwm, you should see the noctalia bar at the top of each monitor.

```bash
# Check the shell is running
ps aux | grep quickshell

# Test launcher via keybinding (Mod+d) or command line
qs ipc -c noctalia-shell call launcher toggle

# Test control center
qs ipc -c noctalia-shell call controlCenter toggle

# Check for errors
ls -t /run/user/1000/quickshell/by-id/ | head -1 | xargs -I{} \
  strings /run/user/1000/quickshell/by-id/{}/log.qslog | grep ERROR
```

---

## Troubleshooting

### Shell not launching from autostart

Make sure `-d` (daemonize) is in the autostart command:
```toml
autostart = ["qs -c noctalia-shell -d"]
```

### Bar not visible / not clickable

The bar uses `WlrLayer.Overlay` on driftwm. If you changed the bar position or monitor config, the bar might be off-screen or hidden. Check:
```bash
cat /run/user/1000/driftwm/state | grep layers
```
You should see entries like `noctalia-bar-content-<output-name>`.

### IPC commands fail with "No running instances"

This means the shell process isn't registered properly. Kill any stale instances and restart:
```bash
pkill -f "qs -c noctalia-shell"
qs -c noctalia-shell -d
```

### High CPU usage

During first startup, the shell loads icons, plugins, and fonts (~1 minute at high CPU). After settling, expect ~10-15% CPU. If it stays above 30%, check for plugin issues:
```bash
# Check which plugins are loaded
cat ~/.config/noctalia/plugins.json
```

### Settings not persisting

Settings are stored in `~/.config/noctalia/settings.json`. If driftwm-specific settings (wallpaper=enabled, blur=enabled) keep reverting, the DriftwmService.initialize() auto-applies them at every startup. This is expected behavior.

---

## Architecture

```
shell.qml (entry point)
├── Services/Compositor/CompositorService.qml  (detection router)
│   └── DriftwmService.qml  ★ this fork's backend
├── Modules/
│   ├── Bar/            (taskbar + widgets including CanvasPosition)
│   ├── Panels/         (launcher, control center, settings, etc.)
│   ├── MainScreen/     (per-output panel host + bar)
│   ├── Dock/           (macOS-style dock)
│   ├── Background/     (wallpaper — disabled on driftwm)
│   ├── Notification/   (notification popups)
│   ├── OSD/            (on-screen display)
│   └── LockScreen/     (session lock)
├── Services/           (backend logic)
└── Commons/            (shared utilities, settings, i18n)
```

---

## License

MIT License — see [LICENSE](./LICENSE) for details.
