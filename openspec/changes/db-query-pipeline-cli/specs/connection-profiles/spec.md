## ADDED Requirements

### Requirement: Resolve reusable connection profiles
The system SHALL allow operators to select a reusable profile that defines the SSH connectivity settings, database connection settings, and execution defaults needed for a target environment.

#### Scenario: Select a profile for query execution
- **WHEN** an operator invokes the CLI with a valid profile identifier
- **THEN** the system loads the profile and uses its resolved settings for the execution pipeline

### Requirement: Support customer-site and local access patterns
The system SHALL support profiles that describe local development targets as well as environments reachable only through an operator-managed support machine and SSH connectivity.

#### Scenario: Run against a customer-reachable environment
- **WHEN** an operator selects `remote` mode for a profile that requires SSH-mediated connectivity to a remote database
- **THEN** the system uses the profile's connection settings to establish that path before query execution

#### Scenario: Run against a local environment
- **WHEN** an operator selects `local` mode for a profile that connects directly to a reachable database
- **THEN** the system executes the query without starting an SSH tunnel

### Requirement: Expose execution defaults through profiles
The system SHALL allow profiles to define execution defaults that the CLI can apply when the operator does not override them at invocation time.

#### Scenario: Use a profile's default execution settings
- **WHEN** an operator runs a query with a profile that defines execution defaults and omits those options on the command line
- **THEN** the system applies the profile's default values during execution

### Requirement: Fail clearly when profile resolution is incomplete
The system SHALL reject execution when the selected profile is missing required connectivity or database settings and SHALL report the failure before attempting the query.

#### Scenario: Profile is missing required fields
- **WHEN** an operator selects a profile that lacks required connection information
- **THEN** the system fails before query execution and reports which required profile data is missing

### Requirement: Keep operational connection messaging off stdout
The system SHALL emit tunnel setup, connection progress, and related operational messages on `stderr` so profile-based execution remains safe to compose in shell pipelines.

#### Scenario: Observe SSH tunnel progress during execution
- **WHEN** a profile requires SSH connectivity and the operator runs a query
- **THEN** the system reports tunnel lifecycle information on `stderr` without writing those messages into structured result output

### Requirement: Validate required external tools before execution
The system SHALL detect whether the external tools required by the selected execution mode are available before query execution begins.

#### Scenario: Remote mode is missing required tools
- **WHEN** an operator selects `remote` mode and one or more required external tools are unavailable
- **THEN** the system fails before execution and reports that `ssh` and `mysql` are required for remote mode, including which required tools are missing

#### Scenario: Local mode is missing mysql
- **WHEN** an operator selects `local` mode and `mysql` is unavailable
- **THEN** the system fails before execution and reports that `mysql` is required for local mode
