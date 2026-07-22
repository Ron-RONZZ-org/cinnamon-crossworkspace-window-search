# Cross-Workspace Window Search

> Full-screen global window search across all workspaces for Cinnamon Desktop.

Press `<Super>+<Ctrl>+<Alt>+Up` → type to filter → press Enter to jump. Never lose a window across your workspaces again.

![Screenshot placeholder]()

## Features

- **Search all windows** across every workspace in real-time
- **Filter by title or application** — substring match, case-insensitive
- **Keyboard-driven** — arrow keys navigate, Enter opens, Escape closes
- **Full-screen overlay** — plenty of room for results, stays out of your way when dismissed
- **Workspace badges** — see at a glance which workspace each window belongs to
- **Zero panel clutter** — no dock icon, no applet. Invoked only on demand

## Requirements

- **Cinnamon Desktop** 5.x or later (Linux Mint 21.x / 22.x)
- Linux Mint 22.1 (Cinnamon 6.4) recommended

## Installation

```bash
# 1. Clone the repo
git clone https://github.com/Ron-RONZZ-org/cinnamon-crossworkspace-window-search.git ~/.local/share/cinnamon/extensions/


# 2. Symlink the src directory into Cinnamon's extension folder
ln -sf "$(pwd)/src" ~/.local/share/cinnamon/extensions/cinnamon-crossworkspace-window-search@ron-ronzz-org.github.com

# 3. Enable the extension
#    System Settings → Extensions → Cross-Workspace Window Search → toggle ON
#    (or restart Cinnamon with Ctrl+Alt+Escape)
```

## Usage

| Key | Action |
|-----|--------|
| `<Super>+<Ctrl>+<Alt>+Up` | Open search overlay |
| Type anything | Filter windows by title or app class |
| `↑` / `↓` | Navigate through results |
| `PgUp` / `PgDn` | Page through results |
| `Home` / `End` | Jump to first / last result |
| `Enter` | Switch to selected window |
| `Escape` | Close search overlay |

## How it works

The extension hooks into Cinnamon's window manager APIs (`Meta.Display`, `Meta.WorkspaceManager`) and Cinnamon's app tracker (`Cinnamon.WindowTracker`) to enumerate all open windows, fetch their icons and titles, and present them in a searchable full-screen overlay — all without any panel widgets or permanent UI.

## Development

```bash
# Edit the source
#   src/extension.js    — main logic
#   src/stylesheet.css  — visual styles
#   src/metadata.json   — extension metadata

# Reload after changes: Ctrl+Alt+Escape
# (restarts the Cinnamon shell, preserves all open apps)

# View JS errors
journalctl -f -o cat /usr/bin/cinnamon
```

**Note**: The Looking Glass D-Bus `ReloadExtension` method exists but Cjs's module cache does not always invalidate, so hot-reload often serves stale code. `Ctrl+Alt+Escape` is the only reliable reload method.

## License

AGPL-3.0 — see [LICENSE](LICENSE).
