# OpenCode Postmortem Plugin

This repository is a standalone OpenCode plugin distribution (not the OpenCode source tree).

It adds a postmortem workflow to OpenCode so you can inspect failed runs, store failure memory, generate prevention rules, and retry with bounded guardrails.

## What this plugin adds

- `postmortem_inspect`: view the latest redacted last-run snapshot
- `postmortem_record_failure`: persist a durable failure record
- `postmortem_why_failed`: deterministic analysis + 1-3 prevention rules
- `postmortem_rules`: list/show/enable/disable/edit/rate/import rules
- `postmortem_failures`: list/show/forget/delete/prune/purge failures
- `postmortem_retry`: generate a retry prompt with relevant guardrails (`--explain` supported)
- `postmortem_disable_lessons`: disable/enable injection for the current session
- `postmortem_config`: show/set storage mode (`user` or `repo`)
- `postmortem_eval`: local-only repeat-failure evaluation metrics

## Repository layout

- `src/` — plugin implementation (TypeScript source)
- `test/` — test suite
- `scripts/` — init script and utilities
- `examples/` — minimal config and usage example
- `dist/` — built output (npm ESM + bundled fallback)
- `INSTALL.md` — install and usage guide

## Install

See [INSTALL.md](INSTALL.md) for full instructions.

**Quick start**: add `"opencode-postmortem-plugin"` to your `opencode.json` plugin array, then run `npx postmortem-init`.

## Quick usage

In OpenCode:

- `/inspect`
- `/record-failure --yes --json`
- `/why-failed --latest --json`
- `/rules --action list --json`
- `/retry --explain`
- `/retry --yes`

## Storage and safety

- Default storage is user-data scoped by project ID.
- Optional repo-local storage can be enabled with `.opencode/postmortem.json` and `{ "storage": "repo" }`.
- Redaction and caps are applied before persistence and before display/injection.

## References

- Plugins docs: https://opencode.ai/docs/plugins
- Commands docs: https://opencode.ai/docs/commands
- Skills docs: https://opencode.ai/docs/skills
