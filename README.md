# bug-verifier

`bug-verifier` is a Go-based CLI tool for reproducing bugs, verifying fixes, and generating verification reports using repeatable test environments.

The project is intended to help engineers validate whether a reported issue is reproducible, whether a fix works, and produce clear verification evidence for issue trackers such as Jira and GitHub.

## Current Status

Early development.

The initial focus is SQL-based database bug verification using containerized test environments.

## Project Goals

`bug-verifier` aims to:

- Reproduce reported bugs in a repeatable environment
- Run SQL/scripts against specific product versions
- Compare behavior between affected and fixed versions
- Capture logs, command output, and environment details
- Generate Markdown verification reports
- Generate Jira/GitHub-ready verification comments
- Keep final verification decisions human-reviewed

## Initial Scope

The first implementation focuses on SQL reproducer verification for database bugs.

Initial target workflows:

- Reproduce a bug on a single database version
- Compare behavior between an affected version and a fixed version
- Generate verification artifacts for review

## Installation

Installation instructions will be added once the first release is available.

During development, build from source:

```bash
git clone git@github.com:aaditya-dubey14/bug-verifier.git
cd bug-verifier
go build -o bug-verifier ./cmd/bug-verifier
```

Run:

```bash
./bug-verifier
```

## Usage

### Reproduce a bug on a single version

Use this mode when you only want to confirm whether a bug is reproducible on one database image.

```bash
bug-verifier verify-sql \
  --ticket PS-12345 \
  --image percona/percona-server:8.0.36 \
  --sql reproducer.sql
```

In this mode, `bug-verifier` should:

- Start the requested database container
- Run the SQL reproducer
- Capture output
- Save logs
- Generate a reproduction report

### Verify a fix by comparing affected and fixed versions

Use this mode when you have both the affected version and a fixed version.

```bash
bug-verifier verify-sql \
  --ticket PS-12345 \
  --affected-image percona/percona-server:8.0.36 \
  --fixed-image percona/percona-server:8.0.42 \
  --sql reproducer.sql
```

In this mode, `bug-verifier` should:

- Start the affected-version container
- Run the SQL reproducer
- Capture affected-version output
- Start the fixed-version container
- Run the same SQL reproducer
- Capture fixed-version output
- Compare results
- Generate a verification report

### Optional fixed version

`--fixed-image` is optional.

When `--fixed-image` is not provided, `bug-verifier` runs in reproduction-only mode.

Valid examples:

```bash
bug-verifier verify-sql \
  --ticket PS-12345 \
  --image percona/percona-server:8.0.36 \
  --sql reproducer.sql
```

```bash
bug-verifier verify-sql \
  --ticket PS-12345 \
  --affected-image percona/percona-server:8.0.36 \
  --sql reproducer.sql
```

```bash
bug-verifier verify-sql \
  --ticket PS-12345 \
  --affected-image percona/percona-server:8.0.36 \
  --fixed-image percona/percona-server:8.0.42 \
  --sql reproducer.sql
```

Invalid examples:

```bash
bug-verifier verify-sql \
  --ticket PS-12345 \
  --fixed-image percona/percona-server:8.0.42 \
  --sql reproducer.sql
```

```bash
bug-verifier verify-sql \
  --ticket PS-12345 \
  --sql reproducer.sql
```

## CLI Design

Planned command structure:

```bash
bug-verifier verify-sql
bug-verifier verify-script
bug-verifier report
bug-verifier version
```

Future commands may include:

```bash
bug-verifier jira fetch PS-12345
bug-verifier jira comment PS-12345 --file verification-comment.md
bug-verifier github fetch owner/repo#123
bug-verifier proxysql verify
bug-verifier pxc verify
bug-verifier pmm verify-api
bug-verifier pmm verify-ui
```

## Planned Output

For each run, `bug-verifier` should create a report directory.

Example:

```text
reports/PS-12345/
  environment.txt
  commands.log
  affected-output.txt
  fixed-output.txt
  diff.txt
  verification-report.md
  verification-comment.md
```

For reproduction-only mode:

```text
reports/PS-12345/
  environment.txt
  commands.log
  output.txt
  reproduction-report.md
  verification-comment.md
```

## Result Categories

The tool should classify results using clear, reviewable states:

| Result | Meaning |
|---|---|
| `verified-fixed` | The issue was reproduced on the affected version and is no longer reproducible on the fixed version |
| `still-reproducible` | The issue is still reproducible on the fixed version |
| `reproduced` | The issue was reproduced on a single tested version |
| `not-reproduced` | The issue could not be reproduced using the provided steps |
| `blocked` | Verification could not continue due to an environment, dependency, or execution problem |
| `inconclusive` | The output was insufficient to make a reliable conclusion |

## Planned Roadmap

### Phase 1: SQL Reproducer Verification

Support local SQL reproducer files.

Planned capabilities:

- Run SQL against a single database image
- Capture stdout and stderr
- Save command logs
- Save database output
- Generate a basic reproduction report

### Phase 2: Affected vs Fixed Comparison

Compare behavior between affected and fixed versions.

Planned capabilities:

- Run the same reproducer against two images
- Capture affected-version output
- Capture fixed-version output
- Generate output diff
- Classify result as verified, still reproducible, blocked, or inconclusive

### Phase 3: Report Generation

Generate reusable verification artifacts.

Planned capabilities:

- Markdown verification report
- Jira-ready verification comment
- GitHub-ready verification comment
- Environment summary
- Command log
- Output files

### Phase 4: Jira and GitHub Integration

Integrate with issue trackers.

Planned capabilities:

- Fetch issue details
- Read reproduction steps
- Download attachments
- Draft comments
- Post comments only after user confirmation

### Phase 5: Product-Specific Runners

Add support for more products and workflows.

Planned targets:

- MySQL
- Percona Server for MySQL
- ProxySQL
- Percona XtraDB Cluster
- PMM API checks
- PMM UI checks

### Phase 6: Community Extensibility

Make the tool extensible for wider use.

Planned capabilities:

- Plugin-style runners
- Configurable verification templates
- Custom test environments
- CI integration
- Reusable verification profiles
- Support for non-Percona database projects

## Non-Goals for the Initial Version

The first version will not:

- Automatically close issues
- Automatically change Jira or GitHub issue status
- Make final verification decisions without human review
- Support every product type immediately
- Replace engineer judgment
- Handle complex distributed systems automatically

## Design Philosophy

`bug-verifier` should be:

- Deterministic
- Transparent
- Reviewable
- Easy to run locally
- Safe by default
- Useful before it is fully automated

The tool should help engineers move faster, but it should not hide what it is doing.

Every verification run should produce enough evidence for another engineer to review, reproduce, and trust the result.

## Development

### Requirements

Initial development requirements:

- Go
- Git
- Docker
- MySQL client or database client tooling, depending on runner implementation

Check Go version:

```bash
go version
```

Build:

```bash
go build -o bug-verifier ./cmd/bug-verifier
```

Run:

```bash
./bug-verifier
```

Format code:

```bash
go fmt ./...
```

Run tests:

```bash
go test ./...
```

## Suggested Repository Structure

```text
bug-verifier/
  cmd/
    bug-verifier/
      main.go

  internal/
    cli/
      root.go
      verify_sql.go

    runner/
      docker.go
      mysql.go
      command.go

    verifier/
      sql.go
      result.go
      compare.go

    report/
      markdown.go
      templates.go

    config/
      config.go

  templates/
    verification-report.md.tmpl
    verification-comment.md.tmpl

  examples/
    mysql/
      reproducer.sql

  reports/
    .gitkeep

  go.mod
  README.md
  LICENSE
```

## Example Verification Comment

```text
Verification completed.

Ticket:
PS-12345

Tested environment:
Docker-based local environment

Affected version:
percona/percona-server:8.0.36

Fixed version:
percona/percona-server:8.0.42

Steps performed:
1. Started the affected-version container.
2. Ran the provided SQL reproducer.
3. Captured the affected-version output.
4. Started the fixed-version container.
5. Ran the same SQL reproducer.
6. Compared the outputs.

Result:
The issue is no longer reproducible on the fixed version.

Conclusion:
Verified as fixed.
```

## Contributing

Contribution guidelines will be added as the project matures.

For now, suggested contribution areas include:

- CLI structure
- SQL runner
- Docker runner
- Report templates
- Result classification
- Documentation
- Test examples

## License

BugVerifier is free software: you can redistribute it and/or modify it under the terms of the
GNU General Public License version 3 as published by the Free Software Foundation.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY;
without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
See the [GNU General Public License](https://www.gnu.org/licenses/gpl-3.0) for more details..
