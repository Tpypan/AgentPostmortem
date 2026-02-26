# Draft: OpenCode Postmortem Plugin

## Requirements (confirmed)
- Build an OpenCode plugin that makes agent runs debuggable and progressively more reliable.
- Provide a context + execution “postmortem” view of the last attempt: files involved, diffs produced, commands/tools run, errors, and likely missing context.
- Add a failure memory workflow so the agent does not repeat the same mistake twice on the same repo.
- Add `/record-failure` to save a failure point with evidence and a short, actionable prevention rule.
- Memory is project-scoped, stored locally, with structured JSON as source of truth plus a readable markdown summary.
- Safe by default: redaction + tight size limits.
- Add `/retry` to rerun the same task with a small set of relevant guardrails injected.
- Retrieval injects only the most relevant past failures, caps prompt injection to avoid bloat, and allows skipping/deleting bad memories.
- Input artifact: Gemini plan in `opencode-postmortem-plugin-afbf93.md` to review, validate, and extend.

## Technical Decisions (pending)
- Plugin integration surface with OpenCode (native plugin hooks vs CLI wrapper) depends on actual OpenCode extension APIs.
- Storage format details (JSON vs JSONL) and on-disk locations (per-repo vs global) to align with OpenCode conventions.
- Redaction policy (regex-only vs additional heuristics) and evidence retention/rotation policies.
- Retrieval strategy (heuristic/BM25 vs embeddings) and whether embeddings are allowed offline.

## Research Findings (pending)
- Need authoritative info on OpenCode plugin APIs, slash command registration, and where OpenCode stores run history/tool logs.

## Session Notes
- Source plan under review: `opencode-postmortem-plugin-afbf93.md` (114 lines; phases v0.1→v1.0; modules; JSONL schema; command UX; retrieval; integration; tests; threat model; evaluation).
- The source plan text includes prefix artifacts like `#KK|` / `#TB|` on each line; likely an export/formatting artifact that should not carry into the final plan.
- Background research launched:
  - Explore: critique plan for gaps/errors.
  - Librarian: verify OpenCode plugin APIs, slash command registration, hooks, and history storage paths.
  - Librarian: prior art for failure-memory tooling + redaction/retention/retrieval patterns.
  - Oracle: architecture risk review + recommended minimal viable design.

## Plan Review (from explore critique)
- Critical: Integration assumptions are unverified (source plan relies on hooks or parsing `.opencode/history`/shell history).
- Critical: Non-interactive environments not handled (prompts/terminal UI assumptions).
- Major: Failure classification is underspecified ("objective vs subjective" undefined; inconsistent records likely).
- Major: Retrieval v1 (keyword/BM25) needs acceptance criteria + robustness (synonyms, typos, error-code exactness) and user feedback loop.
- Major: No lesson feedback loop (rate/edit/mark-effective) beyond `/forget`.
- Minor: Retention is size-only; add time-based retention and prioritization.
- Minor: Redaction regex-only; add multi-pass + high-entropy detection; consider secret scanners.
- Minor: Partial/incremental failures not represented; add per-step/tool events.
- Minor: Evaluation metrics need operational definitions; correlation across retries needs IDs.

## Prior Art Patterns (from librarian)
- Distilled postmortem: store compact, human-readable "lessons" alongside structured store; avoid raw token-heavy logs as the primary memory.
- Hybrid retrieval: BM25/FTS for exact error codes + optional vectors for semantics.
- Redaction pipeline: path-based + regex + high-entropy + sentinel placeholders; redact before disk.
- Compaction over truncation: summarize older artifacts into a compact record when hitting limits.
- Typed failure hierarchy: categorize failures to drive different retry behaviors and guardrail templates.

## Open Questions
- What OpenCode version/implementation is the target (and does it expose hooks like pre/post-run)?
- Should the plugin capture *all* runs automatically, or only when explicitly invoked (opt-in) to minimize privacy risk?
- Should failure memories be committed (team-shared) or always local-only?

## Scope Boundaries
- INCLUDE: postmortem inspector UI/format, failure record + management commands, guardrail injection on retry, safe storage + redaction.
- EXCLUDE (for now): cloud sync, team-wide sharing, automatic remote telemetry by default.
