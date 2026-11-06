# Contributing

Thanks for your interest. The project is in early design, so the best way to help
right now is to open an issue about AWS networking behavior that the design misses.

## Before you write code

1. Open or find an issue.
2. For a change in engine behavior, link the AWS documentation that describes the behavior.
3. For a design change, propose an ADR in `docs/adr/` (copy `0000-template.md`).

## Pull requests

- Keep each PR to one concern.
- Add or update a fixture for every change in engine behavior.
- Use Conventional Commits for commit messages.
- Make sure `pnpm lint`, `pnpm typecheck`, and `pnpm test` pass.

## Fixtures

Fixtures live in `fixtures/<name>/` and contain:

- `main.tf`: the Terraform source
- `plan.json`: output of `terraform show -json`
- `expected.json`: the expected reachability result
- `ORACLE.md`: how the expected result was verified

Use synthetic resources only. Never commit a plan from a real account.
