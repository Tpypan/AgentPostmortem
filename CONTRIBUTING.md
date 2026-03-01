# Contributing to AgentPostmortem

Thank you for your interest in contributing! This document provides guidelines and instructions.

## Development Setup

### Prerequisites

- [Bun](https://bun.sh) (recommended) or Node.js 18+
- Git

### Getting Started

```bash
# Clone the repository
git clone https://github.com/Tpypan/AgentPostmortem.git
cd AgentPostmortem

# Install dependencies
bun install

# Run tests
bun test

# Build
bun run build

# Lint
bun run lint
```

## Project Structure

```
src/
├── index.ts              # Plugin entry point & tool definitions
├── postmortem.plugin.ts  # OpenCode plugin loader
├── analysis.ts           # Failure classification logic
├── injection.ts          # Guardrail injection system
├── inspect.ts            # Context inspector
├── record-failure.ts     # Failure recording
├── retry.ts              # Retry with guardrails
├── why-failed.ts         # Failure analysis
├── manage-failures.ts    # Failure record management
├── manage-rules.ts       # Prevention rule management
├── redaction.ts          # Secret redaction
├── selection.ts          # Rule selection algorithm
├── eval.ts               # Metrics evaluation
├── model.ts              # Zod schemas & types
├── snapshot/             # Last-run context capture
├── storage/              # File storage & locking
├── store/                # Failure store operations
└── templates/            # Command/skill templates
```

## Making Changes

### Code Style

- TypeScript with strict mode
- ES modules (ESM)
- Zod for validation
- Functional style preferred

### Commit Messages

Follow conventional commits:

- `feat:` — New feature
- `fix:` — Bug fix
- `docs:` — Documentation only
- `test:` — Adding/updating tests
- `refactor:` — Code refactoring
- `chore:` — Maintenance tasks

### Testing

- All new features need tests
- All bug fixes need regression tests
- Run `bun test` before submitting

### Pull Request Process

1. Fork the repository
2. Create a feature branch (`git checkout -b feat/my-feature`)
3. Make your changes
4. Add/update tests
5. Run tests: `bun test`
6. Run lint: `bun run lint`
7. Commit with conventional commit message
8. Push and open a pull request

## Adding New Commands

1. Create the tool in `src/` (e.g., `src/my-command.ts`)
2. Add Zod schema in `src/model.ts`
3. Register the tool in `src/index.ts`
4. Create template files:
   - `src/templates/commands/my-command.md`
   - `src/templates/skills/my-command/SKILL.md`
5. Add tests in `test/my-command.test.ts`
6. Update README.md command table

## Security Considerations

- Never log or persist unredacted secrets
- All user input must be validated with Zod
- File operations must handle edge cases (symlinks, locks, corruption)

## Questions?

Open an issue for bugs, feature requests, or questions.

## License

By contributing, you agree that your contributions will be licensed under the MIT License.
