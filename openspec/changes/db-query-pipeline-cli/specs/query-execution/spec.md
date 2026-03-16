## ADDED Requirements

### Requirement: Execute one-shot queries through a profile-driven pipeline
The system SHALL allow an operator to run a one-shot MySQL query by selecting a connection profile, choosing an explicit execution mode of `remote` or `local`, resolving the required connection settings, executing the query, and emitting structured results through a single CLI invocation.

#### Scenario: Execute an ad-hoc query successfully
- **WHEN** an operator runs a query command with a valid profile, a valid execution mode, and a valid SQL statement
- **THEN** the system establishes any required connectivity, executes the query once, and emits the query result through the configured output mode

### Requirement: Limit ad-hoc query execution to SELECT statements in v1
The system SHALL reject ad-hoc query statements that are not `SELECT` statements before opening the execution phase.

#### Scenario: Reject a non-SELECT ad-hoc query
- **WHEN** an operator submits an ad-hoc query statement that is not a `SELECT`
- **THEN** the system fails the invocation before executing the statement and reports that only `SELECT` statements are supported in the first version

### Requirement: Emit machine-readable data only on stdout
The system SHALL reserve `stdout` for query result data and SHALL emit operational logs, warnings, progress updates, and tunnel lifecycle messages on `stderr`.

#### Scenario: Capture structured output in a pipeline
- **WHEN** an operator redirects `stdout` to another CLI consumer while a query is executing
- **THEN** only structured result data is written to `stdout` and all operational messaging is written to `stderr`

### Requirement: Use row objects as the stable result model
The system SHALL represent each result row as an object keyed by output column name, and all output modes SHALL derive from that same row-object model.

#### Scenario: Render the same result set in multiple output modes
- **WHEN** the same query result is emitted as NDJSON and as JSON
- **THEN** each row in both outputs reflects the same row-object field mapping

### Requirement: Default to NDJSON output for query results
The system SHALL default one-shot query output to NDJSON and SHALL treat `jsonl` as an alias for the same serialized format.

#### Scenario: Run a query without an explicit format option
- **WHEN** an operator executes a query command without specifying an output format
- **THEN** the system emits one JSON object per line representing each result row

### Requirement: Provide packaged JSON output as an envelope
The system SHALL provide a JSON output mode that emits an envelope containing `columns` and `rows`, where each entry in `columns` is an object with a `name` field representing the final output column name and `rows` contains the same row objects produced by the core result model.

#### Scenario: Request packaged JSON output
- **WHEN** an operator requests JSON output for a query result
- **THEN** the system emits a single JSON document with a `columns` array of `{name}` objects and a `rows` array

### Requirement: Resolve duplicate output column names deterministically
The system SHALL keep the first occurrence of a duplicate output column name unchanged and SHALL rename subsequent duplicates by appending `_1`, `_2`, `_3`, and so on in output order.

#### Scenario: Query returns duplicate column names
- **WHEN** a query result contains multiple selected columns with the same name
- **THEN** the emitted row objects use deterministic renamed keys so that every output field name is unique and stable

#### Scenario: JSON columns reflect normalized duplicate names
- **WHEN** JSON output is requested for a query result that contains duplicate selected column names
- **THEN** the `columns` array lists the final normalized output names in result-column order and the `rows` objects use those same names as keys

### Requirement: Provide table output as a presentation mode
The system SHALL provide a table output mode that renders the normalized row stream for human reading without changing record semantics.

#### Scenario: Render query results as a table
- **WHEN** an operator requests table output for a query result
- **THEN** the system formats the row objects for terminal display using presentation rules such as width, alignment, truncation, and TTY fitting only
