# 0003: `unknown` is a first-class result

- **Status:** Accepted
- **Date:** 2026-11-06

## Context

Plans contain values that are only known after apply, and the engine will not
support every AWS resource. A wrong "not reachable" can hide real exposure.

## Decision

Every reachability query returns `allowed`, `blocked`, or `unknown`.
`unknown` always carries a reason, for example `after_unknown: aws_subnet.app.cidr_block`
or `unsupported: aws_vpc_peering_connection`.

## Consequences

- Users can trust `allowed` and `blocked`.
- The UI and CLI must show `unknown` clearly, not hide it.
- Coverage can grow over time without silent errors.
