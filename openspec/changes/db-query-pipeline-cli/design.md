## Context

The proposed CLI replaces a fragmented database debugging workflow that currently depends on manual SSH tunnel setup, ad-hoc MySQL commands, and shell post-processing. The tool must work well for human operators doing one-shot investigation, while also establishing a predictable pipeline contract that future automated diagnostics can reuse.

The first implementation targets one-shot MySQL execution through reusable connection profiles. The implementation stack is Zig with the standard library as the primary foundation, while delegating database and tunnel transport to existing `mysql` and `ssh` binaries rather than introducing third-party protocol libraries. The most important architectural constraint is that the CLI behaves like a clean Unix data tool: structured records on `stdout`, operational logs on `stderr`, and deterministic output that downstream tools can safely consume. The initial scope is intentionally narrow: no interactive shell, no write-oriented administration workflow, no embedded `jq`-style query language, and no multi-database abstraction layer.

## Goals / Non-Goals

**Goals:**
- Define a CLI execution pipeline that resolves a profile, establishes SSH connectivity when required, executes a read-focused MySQL query, normalizes rows into a stable record stream, and emits structured output.
- Use Zig std to orchestrate command execution, dependency checks, output parsing, and serializers while relying on existing system tools for SSH and MySQL access.
- Support two operator-facing workflows: direct ad-hoc queries and named diagnostic queries with bounded parameters.
- Make the row object the core data model so `ndjson`, JSON envelope output, and table rendering all derive from the same record stream.
- Separate data output from operational messaging so the tool composes cleanly with `jq` and other CLI consumers.
- Keep profile resolution and execution guardrails explicit enough for support engineers working from customer-reachable machines.

**Non-Goals:**
- Interactive SQL sessions, shell-like REPL behavior, or long-running local agents.
- A generic plugin platform for arbitrary pipeline stages in the first version.
- Full preservation of every database-native type nuance in the initial release.
- An embedded replacement for `jq` or a general-purpose data transformation language.
- Broad database support beyond MySQL-compatible targets.
- Rich presentation features that change record semantics.

## Decisions

### 1. Model execution as a visible pipeline with a row-object stream core
The user-visible workflow is `profile -> connectivity -> query -> normalized row stream -> serializer/presenter -> stdout`. This keeps the product aligned with its main use case: quickly inspecting remote data through a composable CLI chain.

Alternative considered:
- Treat the tool as a thin wrapper around `mysql` and shell piping. Rejected because it keeps connection setup, output cleanliness, and future diagnostics logic fragmented.

### 2. Use row objects as the minimum stable record contract
Each result row is represented as a plain object keyed by the selected column names. This is the contract consumed by all output modes.

Alternative considered:
- Event-style envelopes such as `{type, data, meta}`. Rejected for the first version because they complicate `jq` usage and add ceremony to the dominant inspection workflow.

### 3. Make NDJSON the default serialized output
`ndjson` is the default because it matches streaming result delivery, shell composition, and large-result inspection. `jsonl` is treated as an alias. Structured data is emitted only on `stdout`.

Alternative considered:
- Default to a JSON array. Rejected because it biases the implementation toward collecting full results before emission and weakens pipeline composability.

### 4. Use a JSON envelope for packaged JSON output
When users request JSON output, the CLI emits an envelope shaped as `{ "columns": [{"name": ...}], "rows": [...] }`. The `columns` array declares the final output column order and names after any duplicate-name normalization. This leaves room for future metadata without changing the row model used by NDJSON.

Alternative considered:
- Emit a bare array of row objects. Rejected because it provides no explicit schema channel and makes future metadata additions harder to introduce cleanly.

### 5. Resolve duplicate column names with deterministic renaming
If the query returns duplicate selected column names, the first keeps its original name and subsequent duplicates are renamed with `_1`, `_2`, and so on. This keeps output stable without forcing all callers to alias every query up front.

Alternative considered:
- Fail the query when duplicate column names are returned. Rejected because it creates unnecessary friction for quick debugging, though documentation should still encourage explicit SQL aliases for readability.

### 6. Treat table output as presentation only
`table` consumes the normalized record stream and handles display concerns such as width, alignment, truncation, and TTY fitting. It must not alter record semantics, redefine schema, or become the primary internal model.

Alternative considered:
- Let table formatting drive output structure. Rejected because it would leak presentation concerns into the core execution and serialization layers.

### 7. Implement the CLI in Zig std while calling external `mysql` and `ssh` binaries directly
The CLI core is implemented in Zig std, including argument parsing, profile loading, dependency checks, process orchestration, output parsing, and serializers. External tools are invoked directly with argument vectors instead of shell-composed command strings so quoting and execution behavior remain explicit and testable.

Alternative considered:
- Use shell scripts as the primary implementation. Rejected because orchestration, error handling, and output contracts become harder to keep deterministic as the tool grows.

### 8. Support explicit `remote` and `local` execution modes
The CLI exposes explicit execution modes so operators can intentionally choose SSH-mediated remote access or direct local database access. `remote` is the primary production-oriented path; `local` exists for development or already-local targets.

Alternative considered:
- Infer transport mode implicitly from whichever profile fields happen to exist. Rejected because connectivity behavior should stay legible and explicit to the operator.

### 9. Keep safety defaults biased toward production debugging and limit v1 queries to `SELECT`
The initial CLI should steer operators toward read-oriented execution, named diagnostics for repeatable workflows, and explicit profile-based connectivity. In v1, ad-hoc query execution is limited to `SELECT` statements so the tool remains aligned with inspection-first usage in sensitive environments.

Alternative considered:
- Ship unrestricted query execution first and add guardrails later. Rejected because the tool is specifically intended for production-adjacent and customer-site debugging contexts.

## Risks / Trade-offs

- [Duplicate-column fallback may hide ambiguous SQL intent] -> Encourage aliases in docs and surface final column names explicitly in JSON envelope output.
- [Relying on external `mysql` and `ssh` binaries increases environment sensitivity] -> Check required tools before execution and fail with a clear dependency report for the selected mode.
- [Type normalization can be inconsistent across drivers for edge cases such as decimal, bigint, datetime, and blob values] -> Keep the first release focused on readable, deterministic output and defer exact edge-case policy until implementation validates real driver behavior.
- [Separating logs to `stderr` makes some users miss execution context in captured output] -> Keep messaging concise and add verbosity controls later if needed.
- [A future request for built-in data shaping could bloat scope toward a `jq` replacement] -> Keep v1 focused on serialization and presentation only; revisit limited transforms in a later change if real usage justifies them.
- [Named diagnostics can drift into a general workflow engine] -> Limit the first design to parameterized query definitions with clear input boundaries.
- [SSH and DB connectivity failures may produce a poor operator experience if phases are opaque] -> Preserve phase-oriented progress messages on `stderr` while keeping `stdout` data-only.

## Migration Plan

No production migration is required because this introduces a new CLI surface rather than replacing an existing deployed service. Implementation can proceed in phases:

1. Scaffold the standalone CLI module and basic command surface.
2. Implement profile loading, dependency checks, and explicit `remote`/`local` execution paths.
3. Implement one-shot `SELECT` query execution with NDJSON output.
4. Add JSON envelope and table presenters over the same row stream contract.
5. Add named diagnostics on top of the same execution pipeline.
6. Validate usage on local development targets first, then constrained support-engineer workflows.

Rollback is straightforward: stop using the new CLI and return to the existing manual workflow until issues are resolved.

## Open Questions

- Which type-normalization rules should be fixed in v1 versus explicitly documented as implementation-defined for now?
- How should named diagnostics be stored and distributed: local files in the repo, packaged assets, or profile-adjacent definitions?
- Beyond `SELECT`-only, which read-only or safety-oriented restrictions, if any, should apply differently to ad-hoc queries versus named diagnostics?
- Should future metadata in JSON output live beside `columns` and `rows`, and if so which fields are worth reserving early?
