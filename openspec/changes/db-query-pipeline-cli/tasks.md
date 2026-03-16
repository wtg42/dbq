## 1. CLI foundation

- [ ] 1.1 Scaffold the standalone Zig CLI module, command entrypoint, and top-level command structure for ad-hoc queries and named diagnostics
- [ ] 1.2 Define the shared execution context and phase lifecycle that carry profile resolution, `remote`/`local` mode selection, connectivity state, query inputs, and output mode selection through one-shot runs
- [ ] 1.3 Add baseline tests that exercise command invocation, argument parsing, and clean separation of `stdout` and `stderr`

## 2. Connection profiles and connectivity

- [ ] 2.1 Implement profile loading and validation for SSH settings, database settings, and execution defaults
- [ ] 2.2 Implement preflight dependency checks so `remote` mode requires `ssh` and `mysql`, while `local` mode requires `mysql`
- [ ] 2.3 Implement connectivity setup that opens required SSH-mediated access paths for `remote` mode, skips tunnels for `local` mode, and reports lifecycle progress on `stderr`
- [ ] 2.4 Add tests for valid profile resolution, missing required profile fields, dependency failures, and `remote` versus `local` execution paths

## 3. Query execution and result contract

- [ ] 3.1 Implement one-shot `SELECT` query execution by invoking the external `mysql` binary and normalizing results into the internal row-object stream
- [ ] 3.2 Reject non-`SELECT` ad-hoc queries before execution begins
- [ ] 3.3 Implement deterministic duplicate-column renaming using the `col`, `col_1`, `col_2` sequence in output order
- [ ] 3.4 Add tests covering successful query execution, non-`SELECT` rejection, duplicate output column names, and row-object consistency across output modes

## 4. Output modes and presentation

- [ ] 4.1 Implement NDJSON as the default serializer and accept `jsonl` as an alias for the same output behavior
- [ ] 4.2 Implement JSON envelope output using the `{columns, rows}` structure where `columns` is an ordered array of `{name}` descriptors derived from the normalized output schema
- [ ] 4.3 Implement table presentation that handles width, alignment, truncation, and TTY fitting without changing record semantics
- [ ] 4.4 Add tests for NDJSON default output, JSON envelope output, table rendering, and strict `stdout`/`stderr` separation

## 5. Named diagnostics

- [ ] 5.1 Define the diagnostic query definition format and loader for named, parameterized diagnostics
- [ ] 5.2 Implement diagnostic parameter validation with bounded inputs before query execution begins
- [ ] 5.3 Route named diagnostics through the same execution pipeline and output contract used for ad-hoc queries
- [ ] 5.4 Add tests for successful diagnostic execution, invalid diagnostic parameters, and operator-visible diagnostic context on `stderr`

## 6. Safety and operator workflow validation

- [ ] 6.1 Add command-level guardrails and defaults that keep the first release biased toward read-focused production debugging workflows
- [ ] 6.2 Document expected operator workflows, including profile usage, output mode selection, duplicate-column behavior, and diagnostic invocation
- [ ] 6.3 Run the affected automated test suite and a representative end-to-end local validation flow before implementation sign-off
