# Installation Guide

## Requirements

- [OpenCode](https://opencode.ai) CLI installed
- Node.js 16+ (for init command)

## Quick Install

### Step 1: Add to OpenCode config

Create or edit `opencode.json` in your project root:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "plugin": ["agentpostmortem"]
}
```

OpenCode automatically installs npm plugins on startup.

### Step 2: Initialize templates

Run the init command to set up slash commands and skill definitions:

```bash
npx --package agentpostmortem postmortem-init
```

This creates:

- `.opencode/commands/*.md` — Slash command wrappers
- `.opencode/skills/*/SKILL.md` — Skill definitions
- `.opencode/postmortem.json` — Config stub

The init command is idempotent—safe to re-run.

### Step 3: Restart OpenCode

Restart OpenCode to load the plugin.

### Step 4: Verify

```
/postmortem-config --action show
```

You should see output with your project ID and storage path.

---

## Alternative: Local File Install

If you can't use npm, download the bundled plugin directly:

```bash
# Download the bundle
curl -fsSL https://unpkg.com/agentpostmortem@latest/dist/postmortem.plugin.js \
  -o .opencode/plugins/postmortem.plugin.js

# Initialize templates
npx --package agentpostmortem postmortem-init
```

OpenCode auto-loads `.js` files from `.opencode/plugins/`.

---

## Configuration

### Storage Location

**Default (user-scoped):**

```
~/Library/Application Support/opencode/postmortems/<project-id>/
```

**Repo-local storage** (for sharing with team):

```json
// .opencode/postmortem.json
{
  "storage": "repo"
}
```

Data stored in `.opencode/postmortems/` within your project.

> **Note**: Repo-local storage requires `.opencode/` to be a real directory, not a symlink.

### Options

| Option     | Type               | Default  | Description                                  |
| ---------- | ------------------ | -------- | -------------------------------------------- |
| `storage`  | `"user" \| "repo"` | `"user"` | Where to store postmortem data               |
| `storeRaw` | `boolean`          | `false`  | Save unredacted snapshots (use with caution) |

---

## Troubleshooting

### Plugin not loading

1. Check `opencode.json` syntax is valid JSON
2. Ensure `"agentpostmortem"` is in the `plugin` array
3. Restart OpenCode completely

### Init command fails

1. Ensure Node.js 16+ is installed: `node --version`
2. Try with npx: `npx --package agentpostmortem postmortem-init`

### Commands not found

The init command must run to create `.opencode/commands/*.md` files. Re-run it if commands are missing.
