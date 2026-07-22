# AGENTS.md — Cross-Workspace Window Search

This is the canonical, repo-wide instruction file for AI agents working on this project.

## Hierarchical Context Model

Agents **must** follow this rule:

> When working inside a directory, load the nearest `AGENTS.md` file and merge it with parent `AGENTS.md` files up to root.
> Local rules override global rules.

Context resolution order (highest priority first):
1. `AGENTS.md` in current working directory (if present)
2. Root `AGENTS.md` — global project rules

---

## Project Overview

**Cross-Workspace Window Search** is a Cinnamon Desktop extension that provides full-screen global window search across all workspaces. Press `<Super><Ctrl><Alt>Up` to open a search overlay, type to filter open windows by title or application class, and jump directly to the desired window — regardless of which workspace it lives on.

---

## Language and Naming Conventions

- **Language**: JavaScript (Cjs — Cinnamon's fork of GJS)
- **Style**: ES6 classes, `const`/`let`, arrow functions where appropriate
- **File naming**: lowercase with hyphens (`extension.js`, `stylesheet.css`)
- **CSS classes**: `window-search-` prefix for all custom styles

---

## Tech Stack

| Layer | Technology |
|-------|------------|
| Runtime | Cjs (Cinnamon JavaScript) — no build step |
| UI Toolkit | St (Cinnamon's widget toolkit, based on Clutter) |
| Desktop APIs | Meta (window manager), Cinnamon.WindowTracker |
| Theme | Cinnamon's built-in CSS engine via `stylesheet.css` |

---

## Project Structure

```
cinnamon-crossworkspace-window-search/
├── src/                           # Extension root — symlink target
│   ├── extension.js               # Main logic: hotkey, UI, window listing, filtering
│   ├── metadata.json              # UUID, name, version
│   └── stylesheet.css             # Cinnamon/St styles for the search panel
├── README.md
├── AGENTS.md
├── .gitignore
└── LICENSE
```

---

## Development Workflow

### Install the extension for testing

```bash
# One-time setup: symlink src/ into Cinnamon's extensions directory
ln -sf "$(pwd)/src" ~/.local/share/cinnamon/extensions/cinnamon-crossworkspace-window-search@ron-ronzz-org.github.com
```

### Enable / disable

```bash
# Via GUI: System Settings → Extensions → Cross-Workspace Window Search
# Or via CLI:
gsettings set org.cinnamon enabled-extensions "$(gsettings get org.cinnamon enabled-extensions | sed 's/]$/, \"cinnamon-crossworkspace-window-search@ron-ronzz-org.github.com\"]/')"
```

### Reload after changes

After editing `extension.js` or `stylesheet.css`:

```bash
# Reliable: restart Cinnamon shell (preserves all open windows)
# Press: Ctrl+Alt+Escape
```

**`Ctrl+Alt+Escape` is the only reliable reload method.** The Looking Glass D-Bus `ReloadExtension` exists but Cjs's module cache does not always invalidate correctly, so hot-reload often serves stale bytecode. Always use a full shell restart for deterministic results after code changes.

To view JS errors after restart:

```bash
journalctl -f -o cat /usr/bin/cinnamon
```

### Keybindings

The extension registers `<Super><Ctrl><Alt>Up` via `Main.keybindingManager.addHotKey()`. Unregister on disable via `removeHotKey()`.

---

## Coding Guidelines

1. **Match Cinnamon's Cjs patterns** — use `imports.gi.*` for GObject Introspection, `imports.ui.*` for Cinnamon UI modules. See existing built-in extensions at `/usr/share/cinnamon/extensions/` and `~/.local/share/cinnamon/extensions/`.
2. **Always clean up signals** in `disable()` — Cinnamon extensions persist across enable/disable cycles. Every `connect()` must have a corresponding `disconnect()`.
3. **Winow rows must be reusable** — `_clearRows()` destroys old rows; `_populateList()` creates fresh ones. No stale references.
4. **Style via CSS classes, not inline `set_style()`** — use `window-search-*` classes defined in `stylesheet.css`. Inline styles only for dynamic pseudo-states like `:selected`.
5. **Use `Main.activateWindow()`** to switch workspace and focus — this is the correct Cinnamon API. Do not call `metaWindow.activate()` directly unless you handle workspace switching yourself.

---

## Testing

This is a UI extension — testing is manual in v0.1.0:

1. Symlink → Enable in Extensions UI → Restart Cinnamon
2. Press `<Super><Ctrl><Alt>Up`
3. Type to filter, arrow-keys to navigate, Enter to open, Escape to close
4. Check `Looking Glass` (Ctrl+Alt+L or `Super+L`) for JS errors:
   ```js
   // In Looking Glass, run:
   global.log('test message');
   // View output with:
   journalctl -f -o cat /usr/bin/cinnamon
   ```

---

## What to Avoid

- Do not use `imports.ui.popupMenu` or `imports.ui.applet` — this is an extension, not an applet. Extensions have no panel presence.
- Do not use `setInterval()` or `setTimeout()` without cleaning up in `disable()` — use `GLib.timeout_add()` and store the source ID for removal.
- Do not hardcode screen dimensions — use `Main.layoutManager.primaryMonitor` or `global.screen_width`/`global.screen_height`.
- Do not assume a fixed workspace count — use `global.workspace_manager.get_n_workspaces()`.
