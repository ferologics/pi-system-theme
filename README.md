# pi-system-theme

A Pi extension that automatically syncs Pi's theme with your system appearance (dark/light mode) on macOS, Linux, and Windows.

## How it works

On session start the extension tries to detect and follow your OS/terminal theme. It uses two mechanisms, preferring the more responsive one:

### 1. Terminal push notifications (preferred)

When your terminal supports it, the extension opts in to *push* notifications via mode 2031. The terminal then emits a `CSI ?997;{1|2}n` sequence whenever its theme changes — the extension reacts immediately without any polling.

As part of startup it also sends an **OSC 11** background-colour query so it can set the correct theme right away, before any theme-change event arrives.

### 2. OS API polling (fallback)

When push notifications aren't available, the extension polls the OS on a configurable interval:

```bash
# macOS
/usr/bin/defaults read -g AppleInterfaceStyle

# Linux (GNOME-compatible)
gsettings get org.gnome.desktop.interface color-scheme
# fallback: gsettings get org.gnome.desktop.interface gtk-theme

# Windows
reg query "HKCU\Software\Microsoft\Windows\CurrentVersion\Themes\Personalize" /v AppsUseLightTheme
```

Before spawning any subprocess, the extension first checks the `COLORFGBG` environment variable exported by many terminal emulators — if present, it skips the OS call entirely.

If detection fails or returns an unknown value, the extension keeps the current Pi theme unchanged.

## Defaults (works out of the box)

No config is required.

- `darkTheme`: `dark`
- `lightTheme`: `light`
- `pollMs`: `2000`

## Configuration file (global only)

Path:

- `~/.pi/agent/system-theme.json`

The extension stores only **overrides** in this file. If all values match defaults, the file is removed.

Example:

```json
{
    "darkTheme": "rose-pine",
    "lightTheme": "rose-pine-dawn"
}
```

## Interactive command

Use `/system-theme` to open a small settings menu and edit:

1. dark theme name
2. light theme name
3. poll interval (ms)

Choose **Save and apply** to persist overrides and apply immediately.

## Notes

- When terminal push notifications are confirmed, the polling loop is stopped entirely for the session.
- Linux support currently depends on GNOME-compatible `gsettings` keys (`color-scheme`, with `gtk-theme` fallback).
- Windows support reads `AppsUseLightTheme` from `HKCU\Software\Microsoft\Windows\CurrentVersion\Themes\Personalize`.
- In headless modes without theme support (for example `-p` print mode), the extension stays idle.
- If your current theme is custom and `darkTheme`/`lightTheme` are still default (`dark`/`light`), the extension does nothing to avoid overriding your setup. Configure `/system-theme` to opt into syncing.
- If a configured theme name does not exist, Pi keeps the current theme and logs a warning.

## Install

From npm (standalone package):

```bash
pi install npm:pi-system-theme
```

From git:

```bash
pi install git:github.com/ferologics/pi-system-theme
```

Or from local source while developing:

```bash
pi -e /path/to/pi-system-theme/index.ts
```
