## ADDED Requirements

### Requirement: Execute named diagnostic queries
The system SHALL allow operators to invoke predefined diagnostic queries by name instead of supplying raw SQL on every run.

#### Scenario: Run a named diagnostic query
- **WHEN** an operator invokes a diagnostic query that exists for the selected target context
- **THEN** the system resolves the diagnostic definition and executes its associated query through the standard execution pipeline

### Requirement: Support bounded parameters for diagnostics
The system SHALL allow diagnostic queries to declare named parameters with explicit input boundaries so repeatable debugging workflows can accept limited runtime input.

#### Scenario: Supply parameters to a diagnostic query
- **WHEN** an operator runs a diagnostic query that requires parameters and provides values that satisfy the declared constraints
- **THEN** the system executes the diagnostic query with those parameter values applied

### Requirement: Reject invalid diagnostic inputs before execution
The system SHALL validate required diagnostic parameters before opening the query execution phase and SHALL fail the invocation when provided inputs do not satisfy the diagnostic definition.

#### Scenario: Diagnostic parameter validation fails
- **WHEN** an operator omits a required diagnostic parameter or provides a disallowed value
- **THEN** the system reports the validation failure without executing the diagnostic query

### Requirement: Reuse the standard result pipeline for diagnostics
The system SHALL emit diagnostic query results through the same row-object pipeline and output formats used for ad-hoc queries.

#### Scenario: Request NDJSON output from a diagnostic query
- **WHEN** an operator runs a diagnostic query with NDJSON output
- **THEN** the system emits one row object per line using the same output contract as direct query execution

### Requirement: Preserve diagnostic intent in operator-visible output
The system SHALL identify the selected diagnostic query in operator-visible execution context without contaminating structured result data.

#### Scenario: Observe diagnostic execution context
- **WHEN** an operator runs a named diagnostic query successfully
- **THEN** the system may report the diagnostic name and execution phase on `stderr` while keeping `stdout` limited to structured data
