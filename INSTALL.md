# Installation Guide

## npm (recommended)

### 1. Add the plugin to your OpenCode config

In your project root, add the plugin to `opencode.json` (create the file if it doesn't exist):

```json
{
  "$schema": "https://opencode.ai/config.json",
  "plugin": ["opencode-postmortem-plugin"]
}
```

OpenCode automatically installs npm plugins using Bun. No `npm install` or `bun install` needed in your project.

### 2. Install command and skill templates (optional but recommended)

Run the init command to copy command wrappers and skill definitions into your project:

```bash
npx postmortem-init
```

This copies:
- `.opencode/commands/*.md` — slash command wrappers (`/inspect`, `/retry`, `/failures`, etc.)
- `.opencode/skills/*/SKILL.md` — skill definitions for OpenCode agents
- `.opencode/postmortem.json` — empty per-project config stub

The init command is safe to re-run: it skips files that already exist.

### 3. Restart OpenCode

The plugin loads at OpenCode startup. Restart OpenCode after adding it to the config.

---

## Local file install (fallback)

If you cannot use npm or prefer a self-contained setup, use the bundled plugin file.

### 1. Download the bundled plugin

Get `dist/postmortem.plugin.js` from the [GitHub releases page](https://github.com/tpypan/opencode-postmortem/releases) (or build it yourself: `bun run build`).

### 2. Place it in your project

```bash
mkdir -p .opencode/plugins
cp postmortem.plugin.js .opencode/plugins/
```

OpenCode automatically loads all `.js` and `.ts` files from `.opencode/plugins/`.

### 3. Install templates

Run the init command (requires Node.js):

```bash
npx postmortem-init
```

Or manually copy templates from the [src/templates/](src/templates/) directory.

### 4. Restart OpenCode

---

## Per-project configuration

The plugin works out of the box with no configuration. By default, postmortem data is stored in your user data directory (scoped by project).

To enable repo-local storage (stored in `.opencode/postmortems/` inside your project), create or edit `.opencode/postmortem.json`:

```json
{
  "storage": "repo"
}
```

> **Note**: Repo-local storage is only safe if `.opencode/` is not a symlink. The plugin checks this automatically.

---

## Verifying the install

In an OpenCode session, run:

```
tool: postmortem_config --action show --json
```

You should see the config, storage path, and project ID.
