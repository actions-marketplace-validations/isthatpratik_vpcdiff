# 0001: TypeScript for one engine across CLI, Action, and browser

- **Status:** Accepted
- **Date:** 2026-11-06

## Context

The tool has three interfaces: a CLI, a GitHub Action, and a web viewer that must
run fully in the browser so plan data never leaves the device.

## Decision

Write the engine in TypeScript and ship it as one package used by all three interfaces.

## Alternatives considered

- **Go or Rust CLI plus a separate JS viewer:** two engines that can disagree.
- **Rust compiled to WebAssembly:** one engine, but slower iteration and a harder
  build for a v0 project. Can be revisited if performance needs it.
- **Python:** does not run in the browser without Pyodide.

## Consequences

- One source of truth for semantics.
- GitHub Actions run JavaScript natively.
- Performance on very large plans must be measured.
