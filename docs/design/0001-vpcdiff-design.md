# 0001: vpcdiff design

- **Status:** Draft
- **Author:** Pratik Desai
- **Created:** 2026-11-06

## 1. Problem

A Terraform pull request that changes security groups, NACLs, or route tables is
hard to review. The diff shows rule changes, not their effect. Reviewers cannot
easily answer:

- Does this change make a database reachable from the internet?
- Does it cut an existing path, for example from the load balancer to the service?
- Does a rename replace a security group and cause a short outage?

Today these questions are answered by hand, or after deploy with
AWS VPC Reachability Analyzer, which needs a live account and checks one path at a time.

## 2. Goals

1. Compute the reachability matrix before and after a plan, offline, from `terraform show -json`.
2. Report added paths, removed paths, and outage risk from replacements.
3. Explain every result hop by hop, with the exact deciding rule.
4. Return `unknown` instead of guessing when information is missing.
5. Run the same engine in a CLI, a GitHub Action, and a browser.

## 3. Non-goals

- Live account scanning.
- Generic policy linting.
- Clouds other than AWS (v0.x).
- Layer 7 behavior (HTTP routing rules, WAF).

## 4. Prior art

| Tool | Approach | Gap for this use case |
|---|---|---|
| AWS VPC Reachability Analyzer | Formal analysis of live config (Tiros) | Live account only, one path per analysis, after deploy |
| Batfish | General network config analysis, supports AWS JSON snapshots | Reads `describe_*` output, not plans; heavy server |
| Checkov, Trivy, Regula | Rules over plan JSON | Each rule judged alone, no paths |
| inframap | Graph of resources and SG links | Draws links, does not evaluate reachability |
| breadth | Behavior graph of a whole repo (code, traces, Terraform); graph diff per PR | Network edges come from SG ingress rules only; NACLs, route tables, IGW/NAT, and egress are not evaluated |
| tf-analyze | 215 rules over HCL or plan JSON, plus an attack graph from internet to "crown jewels" | Exposure is regex and flag based (`0.0.0.0/0`, `publicly_accessible`); no NACL or route evaluation; diff mode narrows files, it does not diff paths |

### Notes from reading the source (2026-11-06)

**breadth** (`theorjiugovictor/breadth`, TypeScript, MIT, commit `20444b7`):

- `collectors/src/infra/terraform.ts` reads `.tf`, state, or plan JSON
  (`planned_values` and `resource_changes`).
- It turns SG ingress rules (`aws_security_group` inline, `aws_security_group_rule`,
  `aws_vpc_security_group_ingress_rule`) into `network` edges from a CIDR node
  or a source SG to the workloads that use the group.
- NACL rule resources are matched as rules but never become edges.
  `aws_route_table*`, `aws_internet_gateway`, and `aws_nat_gateway` are graph
  nodes only. Egress rules are skipped.
- "Reachable from the internet" (`insights/src/rules/infra-posture.ts`) means:
  an SG edge from `0.0.0.0/0` or `::/0` reaches the node. It does not check that
  the subnet routes to an IGW or that a NACL allows the traffic.
- The plan is used for create, update, delete, and replace actions, for example
  "replacing orders-db loses data". It is not used for path outage risk.

**tf-analyze** (`ChrisAdkin8/tf-analyze`, Python, Apache-2.0, commit `2161ce6`):

- `scripts/_network.py` reduces SG, Azure NSG, and GCP firewall resources to
  inbound allow rules `(sources, port range)` and flags world-open ones. Egress
  is ignored. There are no deny rules, because AWS SGs have none.
- `scripts/_attack_graph.py` builds the graph from HCL text with regexes.
  A node is internet-reachable when its body matches a pattern, for example
  `cidr_blocks = [... "0.0.0.0/0"]` or `publicly_accessible = true`.
  Reachability spreads only through `vpc_security_group_ids` references.
- Route tables and NACLs appear only in a module-reuse rule
  (`catalog/MOD-REUSE-AWS-VPC-001.yaml`), not in any reachability logic.
- `--mode plan` runs the rule catalog over `planned_values`. `--mode diff`
  uses `git diff` to choose which files to scan. Neither compares before and
  after reachability.

**Conclusion:** neither tool models the full hop sequence (SG, NACL, route
table, IGW/NAT) or gives a before/after path diff with the deciding rule at each
hop. The vpcdiff scope in section 2 stays as it is. breadth is the closest in
intent (behavior diff on a PR), so re-check it before major decisions.

### Background reading

- Backes et al., "Reachability Analysis for AWS-based Networks", CAV 2019.
- Fogel et al., "A General Approach to Network Configuration Analysis", NSDI 2015.
- Kazemian et al., "Header Space Analysis: Static Checking for Networks", NSDI 2012.

## 5. Model (draft)

**Endpoint:** something traffic starts from or arrives at: an ENI-backed resource
(instance, RDS, ALB, ECS task), the internet, or a CIDR.

**Packet class:** `(protocol, src CIDR set, dst port range)`, represented as interval sets.

**Hop sequence for a VPC-to-internet path:**

```
src ENI → SG egress → subnet NACL outbound → route table → IGW/NAT → internet
```

and the reverse for inbound traffic. Security groups are stateful. NACLs are
stateless, so return traffic must also be allowed (ephemeral ports).

**Result per (src, dst, protocol, port):**

- `allowed` with the hop list and the deciding rule at each hop
- `blocked` with the first hop that denies it
- `unknown` with the reason (unknown value in the plan, unsupported resource)

## 6. Pipeline

```
plan.json
  → parse         (resource_changes → before/after resource sets)
  → normalize     (inline rules, *_rule resources, and new-style SG rule resources → one form)
  → build model   (VPCs, subnets, routes, NACLs, SGs, endpoints)
  → evaluate      (reachability matrix)
  → diff          (before vs after)
  → explain       (hop-by-hop traces)
  → render        (CLI text/JSON, PR comment, viewer)
```

## 7. Open questions

1. How to treat `after_unknown` values (for example, a CIDR computed at apply time)?
2. Which endpoint pairs to evaluate by default, so the matrix stays small?
3. How to show "transient outage during replace" in the diff?
4. Graph layout for the viewer: precomputed or interactive?

## 8. Testing strategy

- Golden fixtures: `main.tf`, `plan.json`, `expected.json`, `ORACLE.md`.
- Each fixture is verified once against AWS Reachability Analyzer (free tier account, then destroyed).
- Property tests, for example: adding an allow rule never removes a path.
