## Why

The current database debugging workflow is split across SSH tunnel setup, MySQL client commands, shell parsing, and ad-hoc scripts, which makes quick investigation slow and inconsistent. We need a CLI-first tool that turns this into a repeatable pipeline for both direct human use and future AI-assisted diagnostics, including customer environments where only an on-site support engineer's machine can reach the product network.

## What Changes

- Introduce a CLI-first database query tool that resolves a target profile, establishes secure connectivity, executes MySQL queries, and emits structured output for downstream CLI consumers.
- Define two usage modes: ad-hoc query execution for trusted human operators and named diagnostics for safer, repeatable debug workflows that can later be exposed to AI.
- Add a profile model that captures SSH and database connection details so operators do not need to reconstruct tunnel and query parameters for every run.
- Establish safety defaults for production-facing usage, including read-focused workflows, deterministic machine-readable output, and clear execution boundaries suitable for remote support scenarios.
- Keep the initial scope focused on one-shot command execution rather than interactive sessions, GUI management, long-running agents, or multi-database support.

## Capabilities

### New Capabilities
- `query-execution`: Run one-shot MySQL queries through a CLI pipeline that handles connection setup and produces JSON or NDJSON output.
- `diagnostic-queries`: Execute named, parameterized diagnostic queries intended for repeatable debugging and future AI-driven invocation.
- `connection-profiles`: Resolve reusable target profiles that package SSH access, database connection settings, and execution defaults for local and customer-site environments.

### Modified Capabilities
- None.

## Impact

- A new standalone CLI codebase or incubation directory will be needed, rather than placing this work under the existing generic `tools/` folders.
- Future specs, design, and tasks will need to define the CLI surface, profile configuration format, execution lifecycle, output contracts, and safety guardrails.
- This change creates a foundation for human-operated debugging, AI-assisted diagnostics, and constrained execution on support engineer machines that act as the only permitted access point into customer networks.
