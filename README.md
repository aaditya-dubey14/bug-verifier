# bug-verifier

`bug-verifier` is a Go-based CLI tool for reproducing bugs, verifying fixes, and generating verification reports using repeatable test environments.

## Goal

The goal of this project is to help engineers verify whether a reported bug is fixed by running reproducible test cases against affected and fixed versions.

## Initial Scope

Phase 1 focuses on SQL-based database bug verification.

The tool will:

- Run a SQL reproducer against an affected version
- Run the same reproducer against a fixed version
- Capture output from both runs
- Compare results
- Generate a verification report
- Generate a Jira/GitHub-ready verification comment

## Example

```bash
bug-verifier verify-sql \
  --ticket PS-12345 \
  --affected-image percona/percona-server:8.0.36 \
  --fixed-image percona/percona-server:8.0.42 \
  --sql reproducer.sql
