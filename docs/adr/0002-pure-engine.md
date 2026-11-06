# 0002: The engine is pure

- **Status:** Accepted
- **Date:** 2026-11-06

## Context

Results must be reproducible, testable with golden files, and safe to run in a browser.

## Decision

`packages/engine` has no I/O: no file system, network, clock, or randomness.
Input is a parsed plan object. Output is a plain data structure.
Interfaces (CLI, Action, viewer) do all I/O.

## Consequences

- Same input, same output, byte for byte.
- Easy to test and to run in any environment.
- Callers must load and parse files themselves.
