# @vpcdiff/engine

Pure reachability engine. Parses a Terraform plan object, builds the network model,
evaluates reachability before and after, and returns the diff with explanations.
No I/O (see [ADR 0002](../../docs/adr/0002-pure-engine.md)).
