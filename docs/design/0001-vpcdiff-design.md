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
| breadth, tf-analyze | Graph and exposure analysis from Terraform | Early stage; NACL and route semantics not documented |

TODO: read the source of `breadth` and `tf-analyze` and update this table.

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
