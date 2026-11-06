# AWS networking semantics

One note per concept. Each note states the rule, links the AWS documentation,
and lists the edge cases the engine must handle.

Planned notes:

- `security-groups.md`: stateful, allow-only, SG-to-SG references
- `network-acls.md`: stateless, ordered rules, ephemeral ports, implicit deny
- `route-tables.md`: longest-prefix match, `local` route, main vs explicit association
- `gateways.md`: IGW, NAT gateway, egress-only IGW
- `terraform-plan-json.md`: `before`, `after`, `after_unknown`, modules, replace actions
