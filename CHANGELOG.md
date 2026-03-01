# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.1] - 2026-03-01

### Added
- OpenCode plugin bundle (`dist/postmortem.plugin.js`) for direct plugin loading
- Install smoke test for npm package verification

### Fixed
- Plugin loading issue where OpenCode expected a function export from the bundle
- Updated documentation to use correct init command (`npx --package agentpostmortem postmortem-init`)

### Changed
- Package renamed to `agentpostmortem` (unscoped)
- Updated repository URLs to new GitHub location

## [0.1.0] - 2026-02-26

### Added
- Initial release
- Core postmortem workflow:
  - `/inspect` — View last run context
  - `/record-failure` — Save failure records
  - `/why-failed` — Analyze failures and generate rules
  - `/rules` — Manage prevention rules
  - `/failures` — Browse failure history
  - `/retry` — Retry with guardrails
  - `/disable-lessons` — Toggle guardrail injection
  - `/postmortem-config` — Configure storage
  - `/postmortem-eval` — View metrics
- JSONL-based failure storage with automatic redaction
- Guardrail injection with token caps (max 3 rules, ~400 tokens)
- User-scoped and repo-local storage options
- Secret redaction for API keys, tokens, passwords, and private keys
- File locking for concurrent access safety
- Symlink protection for repo-local storage
- Test suite (72 tests)

[0.1.1]: https://github.com/Tpypan/AgentPostmortem/compare/v0.1.0...v0.1.1
[0.1.0]: https://github.com/Tpypan/AgentPostmortem/releases/tag/v0.1.0
