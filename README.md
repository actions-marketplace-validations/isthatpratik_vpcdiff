# vpcdiff

See how a Terraform change affects AWS network reachability before you run `apply`.

`vpcdiff` reads a Terraform plan (`terraform show -json`), builds the network
reachability graph before and after the change, and shows the difference:

- paths that become reachable (for example, `0.0.0.0/0 → aws_db_instance.main:5432`)
- paths that are lost (for example, `alb → ecs:8080`)
- security group replacements (`-/+`) that can cause a short outage

Every result comes with a hop-by-hop explanation: which route, NACL rule, and
security group rule allowed or blocked the traffic.

> **Status:** early design. Nothing is usable yet. See [`docs/design`](docs/design) and [`docs/adr`](docs/adr).

## Interfaces

| Interface | Use |
|---|---|
| Web viewer | Load a plan in the browser. Graph diff, path inspector, "can A reach B?" query. The plan never leaves your device. |
| CLI | Text or JSON output. Exit code `1` when the plan adds new exposure. |
| GitHub Action | Posts a reachability summary on the pull request. |

All three use the same engine.

## Goals

- **Correct.** Results are checked against AWS VPC Reachability Analyzer on a fixture corpus.
- **Explainable.** Every allowed or blocked path cites the rule that decided it.
- **Honest.** When the engine cannot decide a path, it returns `unknown` with a reason. It does not guess.
- **Offline.** No AWS credentials. No network calls.

## Non-goals

- Live account scanning. Use AWS Reachability Analyzer or Batfish for that.
- Generic policy rules ("no SSH from 0.0.0.0/0"). Checkov and Trivy already do that well.
- Clouds other than AWS, for now.

## Support matrix

Planned for v0.1:

| Resource | Status |
|---|---|
| `aws_vpc`, `aws_subnet` | planned |
| `aws_route_table`, `aws_route`, `aws_route_table_association` | planned |
| `aws_internet_gateway`, `aws_nat_gateway` | planned |
| `aws_security_group` (inline rules) | planned |
| `aws_security_group_rule`, `aws_vpc_security_group_ingress_rule`, `aws_vpc_security_group_egress_rule` | planned |
| `aws_network_acl`, `aws_network_acl_rule` | planned |
| `aws_instance`, `aws_db_instance`, `aws_lb`, `aws_ecs_service` (as endpoints) | planned |
| VPC peering, Transit Gateway, VPC endpoints, prefix lists | not yet; reported as `unknown` |

## Repository layout

```
packages/engine   pure reachability engine (no I/O)
packages/cli      command-line interface
packages/action   GitHub Action
apps/viewer       client-side web viewer
fixtures          plan.json + expected results, verified against AWS
docs              design doc, ADRs, AWS semantics notes
```

## License

[Apache-2.0](LICENSE)
