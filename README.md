# AgentPostmortem

[![npm version](https://img.shields.io/npm/v/agentpostmortem.svg)](https://www.npmjs.com/package/agentpostmortem)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![OpenCode Plugin](https://img.shields.io/badge/OpenCode-Plugin-blue.svg)](https://opencode.ai)

A postmortem workflow plugin for [OpenCode](https://opencode.ai) that captures failure context, generates prevention rules, and injects guardrails into future retries—helping AI agents learn from mistakes instead of repeating them.

## Why AgentPostmortem?

AI coding agents often make the same mistakes repeatedly because they lack persistent memory across sessions. AgentPostmortem solves this by:

- **Capturing failure evidence** — Files touched, commands run, errors encountered
- **Generating prevention rules** — Short, actionable constraints derived from failures
- **Injecting guardrails** — Relevant rules are automatically included in retry prompts
- **Tracking metrics** — Measure if failures are actually decreasing over time

All data stays local. No telemetry, no cloud sync.

## Installation

### Quick Start

1. Add the plugin to your `opencode.json`:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "plugin": ["agentpostmortem"]
}
```

2. Initialize templates in your project:

```bash
npx --package agentpostmortem postmortem-init
```

3. Restart OpenCode

### Verify Installation

In an OpenCode session, run:

```
/postmortem-config --action show
```

You should see the config output with your project ID and storage path.

## Commands

| Command              | Description                                                 |
| -------------------- | ----------------------------------------------------------- |
| `/inspect`           | View the last run's context: files, diffs, commands, errors |
| `/record-failure`    | Save a structured failure record for the last attempt       |
| `/why-failed`        | Analyze a failure and generate prevention rules             |
| `/rules`             | Manage prevention rules (list, edit, disable, rate)         |
| `/failures`          | Browse and manage failure history                           |
| `/retry`             | Retry the last task with relevant guardrails injected       |
| `/disable-lessons`   | Temporarily disable guardrail injection                     |
| `/postmortem-config` | Configure storage location (user or repo)                   |
| `/postmortem-eval`   | View repeat-failure metrics                                 |

## Usage Examples

### Capture and analyze a failure

When an agent attempt fails:

```
/inspect --errors
/record-failure --yes
/why-failed --latest
```

This captures what happened, saves a durable record, and generates 1-3 prevention rules.

### Retry with guardrails

Before retrying a failed task:

```
/rules --action list
/retry --explain
```

The `/retry` command injects the most relevant prevention rules (max 3, ~400 tokens) into the prompt. Use `--explain` to see which rules were selected and why.

### Manage failure memory

View and clean up old records:

```
/failures --action list
/forget <failure-id>
```

Forgotten records won't be used for guardrail injection but remain in history.

### Evaluate improvement

Track whether failures are repeating:

```
/postmortem-eval --window 20
```

Returns repeat-failure rate and other metrics computed locally.

## Configuration

### Storage Location

By default, data is stored in your user directory, scoped by project ID:

```
~/Library/Application Support/opencode/postmortems/<project-id>/
```

To store data in your repository (for sharing across machines or team members):

```json
// .opencode/postmortem.json
{
  "storage": "repo"
}
```

Data will be stored in `.opencode/postmortems/` within your project.

### Privacy & Redaction

All text is redacted before persistence and injection:

- API keys, tokens, passwords
- Private keys (PEM format)
- Database connection strings

## Project Structure

```
agentpostmortem/
├── src/
│   ├── index.ts              # Plugin entry point
│   ├── postmortem.plugin.ts  # OpenCode plugin loader
│   ├── analysis.ts           # Failure classification
│   ├── injection.ts          # Guardrail injection logic
│   ├── snapshot/             # Last-run context capture
│   ├── storage/              # File storage and locking
│   ├── store/                # Failure record management
│   └── templates/            # Command/skill templates
├── test/                     # Test suite
├── examples/                 # Usage examples
├── dist/                     # Build output
└── scripts/                  # Init script
```

## Development

```bash
# Install dependencies
bun install

# Build
bun run build

# Test
bun test

# Lint
bun run lint
```

## Documentation

- [Installation Guide](INSTALL.md) — Detailed setup instructions
- [Examples](examples/minimal/) — Full command reference and playbooks
- [OpenCode Plugins](https://opencode.ai/docs/plugins) — Plugin documentation

## Contributing

Contributions are welcome! Please read [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for version history.

## License

MIT © tpypan
