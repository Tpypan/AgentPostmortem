# OpenCode Plugin: Agent Postmortems & Failure Memory
This plan outlines the architecture, UX, and implementation strategy for an OpenCode plugin that captures failure context, generates prevention rules, and injects guardrails into future agent retries.

## 1. Phases & Deliverables

### Phase 1: Core Data Model & Context Inspector (v0.1)
- **Deliverables:** Local storage JSONL manager, basic session context extraction, `/inspect` command.
- **Acceptance Criteria:** Can run `/inspect` to see files, diffs, tool commands, and errors from the last agent attempt without crashing on large diffs or missing repos.

### Phase 2: Failure Memory & Analysis (v0.2)
- **Deliverables:** `/record-failure`, `/forget`, `/why-failed` commands. LLM prompt templates for analysis.
- **Acceptance Criteria:** Users can record failures. The system successfully classifies errors or takes user input. `/why-failed` outputs 1-3 prevention rules under 160 chars each.

### Phase 3: Guardrail Injection & Retry (v0.3)
- **Deliverables:** `/retry` and `/disable-lessons` commands. Retrieval strategy v1 (keyword/BM25 based).
- **Acceptance Criteria:** `/retry` injects max 300-400 tokens of relevant rules. The user sees a 1-line preview of applied lessons.

### Phase 4: Hardening & Evaluation (v1.0)
- **Deliverables:** Secret redaction, concurrent write safety, evaluation telemetry (local-only by default).
- **Acceptance Criteria:** Pass all edge-case tests (binary files, corrupted JSONL). Telemetry scripts to measure repeat failure reduction.

## 2. Module Breakdown & Data Model

### Modules
- `cli/` - Command wrappers for OpenCode (`/inspect`, `/record-failure`, etc.)
- `context/` - Extractors for diffs, tool outputs, session boundaries, and file states.
- `memory/` - JSONL storage engine, caching, concurrent write locks.
- `analysis/` - LLM interaction for `why-failed` and prevention rule generation.
- `retrieval/` - v1 (TF-IDF/BM25) and v2 (embeddings) logic for matching rules to tasks.
- `utils/` - Secret redaction, text truncation (hard caps), git status checks.

### Data Model Schema (JSONL)
```json
{
  "id": "uuid-v4",
  "timestamp": "iso-8601",
  "task_prompt": "string",
  "failure_type": "enum: [objective, subjective]",
  "classification": "string (e.g., 'test_failure', 'wrong_file')",
  "user_reason": "string (optional)",
  "context": {
    "files_touched": ["string"],
    "commands_run": [{"cmd": "string", "exit_code": "int", "output_snippet": "string"}],
    "diff_summary": "string"
  },
  "analysis": {
    "hypotheses": [{"cause": "string", "confidence": "float"}],
    "prevention_rules": ["string (max 160 chars)"]
  },
  "status": "enum: [active, forgotten]"
}
```

## 3. Command UX Spec

- `/inspect`
  - **Output:** A structured terminal UI or markdown table showing: files modified, diff summary, shell commands with exit codes, truncated error logs, and context bloat estimate.
- `/record-failure [reason]`
  - **Interaction:** If `reason` is omitted and failure isn't objectively clear (no non-zero exit codes), prompts the user: "What went wrong? (e.g., edited wrong file)".
  - **Output:** "Saved failure record `[ID]`. Run `/why-failed` for analysis."
- `/why-failed`
  - **Output:** Displays ranked hypotheses with evidence pointers, followed by 1-3 concise prevention rules.
- `/retry`
  - **Output:** "Retrying last task. Applying lessons: *'Never use X in Y'* (skip? y/N)" -> Proceeds with task.
- `/forget <id>`
  - **Output:** "Record `[ID]` marked as forgotten. It will not be injected in future retries."
- `/disable-lessons`
  - **Output:** Sets a session flag. "Lessons disabled for the next run."

## 4. Retrieval Strategy

### v1: Heuristic & Keyword (Current)
- **Mechanism:** Extract keywords from the new task prompt and currently active filenames. Match against `task_prompt`, `classification`, and `files_touched` in the JSONL store.
- **Limits:** Max 3 records retrieved. Strict token cap of 400 tokens total for injected rules. Rank by recency + keyword overlap score.

### v2: Embedding-based (Future/Optional)
- **Mechanism:** Generate local embeddings (e.g., via `SentenceTransformers` or OpenCode's built-in embedding API) for the `task_prompt` and `prevention_rules`.
- **Limits:** Cosine similarity threshold to ensure relevance. Still capped at 3 records / 400 tokens to prevent prompt bloat.

## 5. Integration Strategy with OpenCode

- **Primary (Hooks/APIs):** If OpenCode provides middleware hooks (e.g., `pre_run`, `post_run`), the plugin will automatically snapshot context `post_run` and inject guardrails `pre_run`.
- **Fallback (Wrappers):** If hooks are unavailable, the plugin acts as a CLI wrapper (e.g., `opencode-pm retry`) which reads OpenCode's `.opencode/history` or bash history, builds the augmented prompt, and executes `opencode "augmented prompt"`.

## 6. Test Plan

- **Unit Tests:** 
  - Token truncation and text summarization logic.
  - JSONL read/write with mock corrupted files.
  - Secret redaction regexes.
- **Integration Tests:**
  - Full flow: mock a failing OpenCode session -> `/record-failure` -> `/why-failed` -> `/retry`.
  - Validate retrieval v1 returns the correct rules for a mock task.
- **Failure-Mode Tests:**
  - Running `/inspect` outside a git repo.
  - Diffing a 10MB file or binary file (ensure graceful fallback).
  - Concurrent writes to `.agent/failures.jsonl` from multiple terminal tabs.

## 7. Threat Model, Privacy & Risks

- **Threat:** Secret leakage in logs.
  - **Mitigation:** Run regex-based redaction on tool outputs and diffs before saving to `.agent/`. Avoid sending logs to external APIs for analysis unless explicitly using the user's configured LLM provider.
- **Threat:** Memory bloat / Prompt inflation.
  - **Mitigation:** Hard cap of 3 rules / 400 tokens per `/retry`. JSONL rotation if the file exceeds 1MB.
- **Risk:** Multi-session concurrency.
  - **Mitigation:** Use file locking (e.g., `fcntl` or `lockf`) when appending to JSONL.
- **Privacy:** Defaults to local-only (`.agent/` inside the repo, added to `.gitignore`). Optional global cache path (`~/.opencode/postmortems`) can be enabled via config.

## 8. Evaluation Plan

- **Metrics:** 
  1. *Repeat Failure Rate:* Frequency of the same tool error (e.g., identical stack trace) occurring within 5 subsequent prompts.
  2. *Success Rate:* Percentage of `/retry` commands that result in a zero exit code and user acceptance.
- **Telemetry:** A local script `evaluate_postmortems.py` that parses the JSONL and OpenCode history to calculate these metrics locally, providing the user a dashboard of how much time the plugin has saved.
