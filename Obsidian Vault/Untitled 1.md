# Wrote Documents/git_repos/my_i3wm-ricing/README.md
# Complete i3wm Rice Setup Documentation

## Overview
This document provides a comprehensive, step-by-step account of building a fully-featured, dynamic i3wm desktop environment on Parrot Security 7.3 (Debian 13 based). The result is a cohesive, themeable workspace where the wallpaper drives the entire color palette across the bar, terminal, launcher, and notifications.

---

## 1. System Base

**Distribution:** Parrot Security 7.3 (echo) — Debian 13 (trixie) base  
**Display Server:** X11 (i3 requires X11)  
**Current DE:** KDE (SDDM login manager) — we switch to i3 at the login screen

---

## 2. Package Installation

### Core i3 & Utilities
```bash
i3-wm i3lock picom rofi feh dunst flameshot polybar
alacritty lxappearance polkitd pkexec network-manager network-manager-gnome
brightnessctl xclip maim xdotool wmctrl arandr pavucontrol playerctl thunar
papirus-icon-theme pamixer copyq xss-lock redshift numlockx unclutter-xfixes unzip
fonts-font-awesome bluez git wget tar xz-utils python3-venv python3-pip imagemagick
```

**Key choices:**
- `polybar` over i3status for richer modules
- `rofi` over dmenu for themed launcher
- `alacritty` as GPU-accelerated terminal
- `flameshot` for screenshots (GUI + CLI)
- `copyq` for clipboard history
- `pywal` (via pip venv) for dynamic theming
- `imagemagick` for JPG→PNG lock screen conversion

---

## 3. Font Installation

**Font:** JetBrainsMono Nerd Font (patched with 3000+ glyphs)  
**Method:** Downloaded from ryanoasis/nerd-fonts releases, extracted to `~/.local/share/fonts/JetBrainsMono/`, `fc-cache -fv`

**Why Nerd Font?** Required for polybar icons (    ), rofi icons, i3 workspace glyphs.

---

## 4. GTK Theme (Catppuccin Mocha)

**Theme:** Catppuccin Mocha Blue (standard variant)  
**Source:** Prebuilt release zip from catppuccin/gtk v1.0.3  
**Install:** Extracted to `~/.themes/` (both standard + xhdpi variants)  
**Apply:** `lxappearance` → select "catppuccin-mocha-blue-standard+default" + Papirus icons

---

## 5. pywal (Dynamic Color Engine)

**Installation:** Isolated Python venv at `~/.local/pywal-venv/`, symlinked `wal` to `~/.local/bin/wal`  
**Templates Created:**
- `~/.config/wal/templates/colors-polybar` → generates `[colors]` section for polybar
- `~/.config/wal/templates/colors-alacritty.toml` → `[colors.primary] + [colors.normal]`
- `~/.config/wal/templates/colors-rofi.rasi` → `* { base, mantle, surface0... }`
- `~/.config/wal/templates/dunstrc` → full dunst config with urgency sections

**Wallpaper Script** (`~/.config/i3/scripts/wallpaper.sh`):
- Picks random image from `~/Pictures/wallpapers/`
- Runs `wal -i "$img"` → generates `~/.cache/wal/colors-*`
- Restarts dunst with generated `dunstrc`

**Keybind:** `Super+Ctrl+d` → rerolls wallpaper + full retheme

---

## 6. i3 Configuration (`~/.config/i3/config`)

### Core Settings
```ini
set $mod Mod4
font pango:JetBrainsMono Nerd Font 10
floating_modifier $mod
workspace_auto_back_and_forth yes
gaps inner 12 / outer 8
smart_borders on
for_window [class=".*"] border pixel 4
```

### Catppuccin Mocha Palette (static fallback)
```ini
set $bg    #1e1e2e   # base
set $fg    #cdd6f4   # text
set $acc   #89b4fa   # blue (accent)
set $red   #f38ba8   # red
set $gray  #45475a   # surface1
```

### Window Borders
```ini
client.focused          $acc $bg $fg $acc $acc
client.focused_inactive $gray $bg $fg $gray $gray
client.unfocused        $bg  $bg $fg $bg  $bg
```

### Workspaces (1-10)
```ini
set $ws1 "1" ... $ws10 "10"
bindsym $mod+1 workspace $ws1 ... bindsym $mod+0 workspace $ws10
bindsym $mod+Shift+1 move container to workspace $ws1 ...
bindsym $mod+Tab workspace back_and_forth
```

### Scratchpad
```ini
bindsym $mod+Shift+minus move scratchpad
bindsym $mod+minus scratchpad show
```

### Focus & Movement (vim-style + arrows)
```ini
# Focus
bindsym $mod+j focus left;  $mod+k focus down
bindsym $mod+l focus up;    $mod+semicolon focus right
bindsym $mod+Left/Down/Up/Right focus left/down/up/right

# Move
bindsym $mod+Shift+j move left ... + arrows

# Split
bindsym $mod+h split h;  $mod+v split v
```

### Layouts
```ini
$mod+f fullscreen toggle
$mod+s stacking;  $mod+w tabbed;  $mod+e toggle split
$mod+Shift+space floating toggle
$mod+space focus mode_toggle
$mod+a focus parent
```

### Resize Mode
```ini
bindsym $mod+r mode "resize"
mode "resize" {
    j/k/l/;/arrows → shrink/grow width/height
    Return/Escape/$mod+r → exit
}
```

### System Controls

**Lock/Exit:**
```ini
$mod+Ctrl+l → ~/.config/i3/scripts/lock.sh (random wallpaper lock)
$mod+Shift+x → powermenu.sh
$mod+Shift+c reload;  $mod+Shift+r restart
$mod+Shift+e → i3-nagbar exit confirmation
```

**Applications:**
```ini
$mod+Return → alacritty
$mod+d → rofi drun (with -theme-str pywal colors)
$mod+p → rofi run (with -theme-str)
$mod+b → firefox
$mod+Ctrl+w → nm-connection-editor
$mod+Ctrl+v → copyq toggle
$mod+Ctrl+n → redshift toggle
$mod+Ctrl+d → wallpaper.sh (retheme)
```

**Screenshots (flameshot):**
```ini
Print → flameshot gui (area select)
Shift+Print → flameshot full -p ~/Pictures/Screenshots -c (save + copy)
Ctrl+Print → flameshot full -c (copy only)
$mod+Print → flameshot launcher
```

**Brightness (brightnessctl):**
```ini
XF86MonBrightnessDown/Up → 5% steps
$mod+F5/F6 → fallback keys
```

**Volume (pactl):**
```ini
XF86AudioLowerVolume/RaiseVolume → ±5%
XF86AudioMute → toggle
XF86AudioMicMute → toggle mic
$mod+F11/F12/equal/Shift+m → fallback keys
```

**Media (playerctl):**
```ini
XF86AudioPlay/Next/Prev → play-pause/next/previous
```

**Power Profiles (powerprofilesctl):**
```ini
$mod+F1 performance;  $mod+F2 balanced;  $mod+F3 power-saver
$mod+F4 → notify-send current profile
```

### Autostart
```ini
exec_always picom -b
exec dunst
exec numlockx
exec copyq --start-server
exec unclutter
exec nm-applet
exec flameshot
exec xset s 600 10;  xset dpms 900 1200 1800
exec xss-lock -- i3lock -c 1e1e2e  (fallback solid color)
exec_always ~/.config/i3/scripts/wallpaper.sh
exec_always $HOME/.config/polybar/launch.sh
```

---

## 7. i3 Helper Scripts

### `~/.config/i3/scripts/powermenu.sh`
Rofi dmenu with 5 options: Lock / Logout / Suspend / Reboot / Shutdown  
Uses pywal-themed rofi via `-theme-str "$(cat ~/.cache/wal/colors-rofi.rasi)"`

### `~/.config/i3/scripts/wallpaper.sh`
- Finds random image in `~/Pictures/wallpapers/`
- Runs `wal -i "$img"`
- Restarts dunst with `~/.cache/wal/dunstrc`

### `~/.config/i3/scripts/lock.sh`
- Finds random PNG (preferred) or JPG→PNG converted wallpaper
- Runs `i3lock -i "$img"` with that image
- Used by `$mod+Ctrl+l` and powermenu Lock option

---

## 8. Polybar (`~/.config/polybar/`)

### Launch Script (`launch.sh`)
- Kills existing bars, waits for cleanup
- Iterates `xrandr` connected monitors
- Launches `polybar --reload --config=config.ini main` per monitor
- Logs to `$XDG_RUNTIME_DIR/polybar-<monitor>.log`

### Config (`config.ini`)

**Structure:**
```ini
include-file = ~/.cache/wal/colors-polybar   # MUST BE FIRST

[bar/main]
modules-left = i3
modules-right = wlan eth volume battery cpu memory fs bluetooth date
```

**Modules:**

| Module | Type | Details |
|--------|------|---------|
| i3 | internal/i3 | Pin workspaces, strip numbers, scroll, colored labels (focused=blue bg) |
| wlan | internal/network | Wireless, shows ESSID + signal% |
| eth | internal/network | Wired, shows local IP |
| volume | custom/script | `~/.config/polybar/scripts/volume.sh` (pamixer), 2s interval |
| battery | internal/battery | BAT0/AC auto-detected, shows % + (H:MM), ramp icons (▁▂▃▄▅▆▇█), CHG animation, red <20% |
| cpu | internal/cpu | Total % |
| memory | internal/memory | % used |
| fs | internal/fs | Root mount, % free |
| bluetooth | custom/script | `bluetoothctl devices Connected`, shows device name when connected |
| date | internal/date | "Mon 27/08  14:30" |

**Static Fallback Colors** (used before wal runs):
```ini
bg = #b81e1e2e  (slightly transparent base)
fg = #cdd6f4
blue = #89b4fa, red = #f38ba8, green = #a6e3a1, yellow = #f9e2af, gray = #45475a
```

---

## 9. Rofi (`~/.config/rofi/config.rasi`)

### Features
- Modes: `drun, run, window` (switch with `Ctrl+Tab` in rofi)
- Icons: Papirus theme, JetBrainsMono Nerd Font 11
- Centered window, 12px rounded, translucent Mocha base, blue accent border
- Pill-style mode switcher (Apps / Cmd / Windows)
- Search bar: placeholder "Type to search…", blinking cursor, blue prompt
- Listview: 9 lines, fixed height, slim scrollbar (surface1 handle)
- Row styling: transparent normal, surface0 (40%) alternate, surface1 selected, green/red for active/urgent
- **Critical fix:** `element-text { text-color: @text }` explicit — otherwise inherits dark default

### Pywal Integration
**NOT** via `@theme` in config (causes instant exit). Instead, launch commands inject colors:
```bash
rofi -show drun -theme-str "$(cat ~/.cache/wal/colors-rofi.rasi 2>/dev/null)"
```
Applied to `$mod+d`, `$mod+p`, and powermenu.

---

## 10. Alacritty (`~/.config/alacritty/alacritty.toml`)

```toml
[general]
import = ["~/.cache/wal/colors-alacritty.toml"]

[font]
size = 11
family = "JetBrainsMono Nerd Font"

[window]
opacity = 0.92
padding = { x = 8, y = 8 }
```
**No static [colors] section** — wal import provides everything. New terminals pick up wallpaper palette automatically.

---

## 11. Picom (`~/.config/picom/picom.conf`)

```ini
backend = "glx"
vsync = true
blur: { method = "dual_kawase", strength = 4 }
rounding = 8
shadow = true (radius 12, offset -8/-8, opacity 0.5)
opacity-rule = [ "95:class_g = 'Alacritty'" ]
blur-background-exclude = [ dock, desktop ]
```
Dual-kawase blur + rounded corners + subtle shadows.

---

## 12. Dunst (`~/.config/dunst/dunstrc`)

**Static fallback** (used until wallpaper.sh restarts it):
```ini
font = JetBrainsMono Nerd Font 10
background = "#1e1e2e"
foreground = "#cdd6f4"
frame_color = "#89b4fa"
corner_radius = 8
```

**Urgency levels** with timeouts.  
**Wal template** (`~/.config/wal/templates/dunstrc`) generates fully themed version with `{background}`, `{foreground}`, `{color1}`, `{color4}`, `{color8}` placeholders for low/normal/critical.

---

## 11. Keybinding Summary

| Key | Action |
|-----|--------|
| `Super+Return` | Terminal (alacritty) |
| `Super+d` | App launcher (rofi drun) |
| `Super+p` | Run command (rofi run) |
| `Super+b` | Firefox |
| `Super+Shift+q` | Kill window |
| `Super+q` | Kill window (alt) |
| `Super+f` | Fullscreen toggle |
| `Super+s/w/e` | Stacking / Tabbed / Toggle split |
| `Super+Shift+space` | Float toggle |
| `Super+Shift+c/r/e` | Reload / Restart / Exit i3 |
| `Super+Tab` | Previous workspace |
| `Super+1..0` | Switch workspace |
| `Super+Shift+1..0` | Move window to workspace |
| `Super+- / Shift+-` | Scratchpad show / move |
| `Super+r` | Resize mode |
| `Super+h/v` | Split horizontal/vertical |
| `Super+Ctrl+l` | Lock (random wallpaper) |
| `Super+Shift+x` | Power menu |
| `Super+Ctrl+d` | Reroll wallpaper + retheme |
| `Super+F5/F6` | Brightness down/up |
| `Super+F11/F12` | Volume down/up |
| `Super+equal/Shift+m` | Volume up / mute |
| `Super+F1/F2/F3` | Power profiles |
| `Super+F4` | Show current power profile |
| `Print / Shift+Print / Ctrl+Print` | Screenshots |
| `Super+Ctrl+w` | WiFi connections |
| `Super+Ctrl+v` | Clipboard history (copyq) |
| `Super+Ctrl+n` | Redshift toggle |

---

## 12. Directory Structure Created

```
~/.config/
├── i3/
│   ├── config
│   └── scripts/
│       ├── lock.sh
│       ├── powermenu.sh
│       └── wallpaper.sh
├── polybar/
│   ├── launch.sh
│   ├── config.ini
│   └── scripts/
│       ├── volume.sh
│       └── bluetooth.sh
├── rofi/
│   └── config.rasi
├── alacritty/
│   └── alacritty.toml
├── picom/
│   └── picom.conf
├── dunst/
│   └── dunstrc
├── wal/
│   └── templates/
│       ├── colors-polybar
│       ├── colors-alacritty.toml
│       ├── colors-rofi.rasi
│       └── dunstrc
```

---

## 13. Installation Script

**Location:** `/home/user/Documents/git_repos/my_i3wm-ricing/setup.sh`

**Features:**
- Idempotent (backs up existing configs to `~/.config/rice-backup-<timestamp>/`)
- Auto-detects battery (`BAT0`, `BAT1`...) and adapter (`AC`, `ADP`...)
- Installs all packages, font, GTK theme, pywal venv
- Writes every config file from heredocs
- Validates i3 config with `i3 -C`
- Generates initial wallpaper palette if `~/Pictures/wallpapers/` has images
- Prints complete keybinding cheatsheet on completion

**Run on fresh Debian/Ubuntu/Parrot:**
```bash
cd /home/user/Documents/git_repos/my_i3wm-ricing && ./setup.sh
# log out → select "i3" at SDDM → log in
```

---

## 14. Dynamic Theming Flow

```
User presses Super+Ctrl+d
        ↓
wallpaper.sh picks random image from ~/Pictures/wallpapers/
        ↓
wal -i "$img" generates ~/.cache/wal/colors-*
        ↓
polybar reads include-file = ~/.cache/wal/colors-polybar (on next reload/start)
alacritty reads import = ["~/.cache/wal/colors-alacritty.toml"] (new windows)
rofi receives -theme-str "$(cat ~/.cache/wal/colors-rofi.rasi)" (each launch)
dunst restarted with ~/.cache/wal/dunstrc (immediate)
        ↓
Entire desktop recolors to match wallpaper
```

---

## 15. Known Limitations & Notes

1. **i3 borders** stay static blue (`$acc`) — i3 cannot read dynamic palettes without config rewrite
2. **Alacritty** needs new window after retheme (import only read at startup)
3. **PNG-only** for i3lock — JPGs auto-converted via ImageMagick to `/tmp/i3lock-bg.png`
4. **Polybar tray race** — harmless "Failed to put tray above..." log from nm-applet startup order
5. **Rofi `@theme`** inside config.rasi crashes rofi — must use `-theme-str` at launch
6. **Wayland** — this setup is X11-only; won't work on pure Wayland sessions
7. **Backup** — every run backs up `~/.config/{i3,polybar,rofi,alacritty,picom,dunst}` to timestamped directory

---

## 16. First Boot Checklist

After running `setup.sh` and logging into i3:

1. `Super+Return` → verify alacritty opens with Mocha colors
2. `Super+d` → rofi centered, themed, icons visible
3. `Super+Shift+x` → power menu appears
4. `Super+Ctrl+d` → wallpaper changes, bar/rofi/dunst retheme
5. `Print` → flameshot GUI works
6. Volume/brightness/media keys function
7. `Super+1..0` workspace switching works
8. Battery/time/CPU/RAM/FS/BT visible on polybar
9. Add wallpapers to `~/Pictures/wallpapers/` for variety

---

## 17. Maintenance

**Update packages:** `sudo apt update && sudo apt upgrade`  
**Refresh pywal:** `~/.local/pywal-venv/bin/pip install -U pywal`  
**Add wallpapers:** Drop PNG/JPG files into `~/Pictures/wallpapers/`  
**Edit configs:** All in `~/.config/` — changes persist across reboots  
**Reinstall script:** Re-run `setup.sh` — it backs up before overwriting

---

## 18. Credits & Inspiration

- **Catppuccin** — beautiful pastel palette
- **pywal** — wallpaper-driven theming engine
- **polybar** — extensible status bar
- **rofi** — versatile launcher
- **i3-gaps** (merged upstream) — gaps + smart borders
- **JetBrainsMono Nerd Font** — programming font with icons
- **flameshot** — screenshot tool
- **copyq** — clipboard manager

---

*Generated from live configuration on Parrot Security 7.3 (Debian 13), August 2025*