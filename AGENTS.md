# Working rules for coding agents

These rules apply to any AI coding agent working in this repository.

## Workflow

- Spec first. Do not write code without an issue that links to a section of the design doc or an ADR.
- Use TDD for the engine. Write the failing test or fixture first, then the code.
- One concern per pull request. Keep a PR under about 400 changed lines, except for generated code.
- Use Conventional Commits (`feat:`, `fix:`, `docs:`, `test:`, `chore:`, `refactor:`).
- Do not add `Co-Authored-By` trailers or tool attribution to commits or PRs.

## Git and GitHub approval

- Never commit, push, tag, or rewrite history without explicit approval from the
  maintainer for that action. Show the diff and the commit message first, then wait.
- Never create, edit, or delete anything on GitHub without explicit approval for
  that action. This includes repositories, branches, pull requests, issues,
  comments, reviews, labels, releases, workflows, and settings.
- Approval for one action does not cover the next one.
- Read-only operations are allowed without approval. Examples: `git status`,
  `git log`, `git diff`, `git show`, `git fetch`, cloning a public repository to a
  scratch folder, `gh repo view`, `gh pr view`, `gh issue list`, and `gh api` GET
  requests.

## Engine invariants

- `packages/engine` is pure. No `fs`, network, `Date.now()`, or `Math.random()`.
- The same input gives the same output, byte for byte. Sort every collection before output.
- Every AWS networking rule in the code cites its AWS documentation URL in a comment.
- When the engine cannot decide a path, return `unknown` with a reason. Never guess allow or deny.

## Fixtures

- Never edit `fixtures/**/expected.json` to make a test pass.
- A new or changed `expected.json` needs an oracle note: the AWS Reachability Analyzer result, or a written proof.

## Boundaries

- No new runtime dependency without an ADR.
- No secrets, real AWS account IDs, or real plans in the repository. Use synthetic fixtures only.
- The web viewer never sends plan data off the device.

## Definition of done

- `pnpm lint`, `pnpm typecheck`, and `pnpm test` pass.
- The design doc or an ADR is updated when behavior or design changes.
- The support matrix in `README.md` is updated when resource coverage changes.

## Needs human review

- Any change to engine semantics, fixtures, or ADRs.
