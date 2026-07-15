# driftwm-noctalia
user guide to install driftwm with noctalia for Fedora Linux

---

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

### Arch Linux

```bash
sudo pacman -S cmake ninja gcc qt6-base qt6-declarative qt6-wayland \
  wayland wayland-protocols pipewire libxkbcommon wlr-randr wl-clipboard
```

### Ubuntu/Debian

```bash
sudo apt install cmake ninja-build g++ qt6-base-dev qt6-declarative-dev \
  qt6-wayland-dev libwayland-dev wayland-protocols libpipewire-0.3-dev \
  libxkbcommon-dev wlr-randr wl-clipboard
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
