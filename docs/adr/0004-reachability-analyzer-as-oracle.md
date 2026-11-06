# 0004: AWS Reachability Analyzer is the test oracle

- **Status:** Accepted
- **Date:** 2026-11-06

## Context

Expected results written by hand can repeat the same misunderstanding as the code.

## Decision

Each fixture is applied once to a throwaway AWS account. Its expected results are
checked with AWS VPC Reachability Analyzer, then the infrastructure is destroyed.
The result and the analysis IDs are recorded in the fixture's `ORACLE.md`.
Account IDs and other real identifiers are removed before commit.

## Consequences

- Strong evidence that results match AWS behavior.
- Small cost per analysis; keep the corpus focused.
- Cases the analyzer does not support need a written proof instead.
