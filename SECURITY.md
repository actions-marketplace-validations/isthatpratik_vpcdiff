# Security policy

`vpcdiff` reads Terraform plans, which can contain sensitive infrastructure details.
The tool is designed to run fully offline and never sends plan data anywhere.

## Reporting a vulnerability

Please do not open a public issue for security problems.
Use GitHub's private vulnerability reporting on this repository
(**Security → Report a vulnerability**).

I will acknowledge a report within 7 days.

## Scope

In scope:

- Any path where plan data leaves the user's device
- Code execution through a crafted plan file
- Incorrect results that hide real exposure (false "not reachable")
