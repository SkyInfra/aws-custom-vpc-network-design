# AWS Custom VPC Network Design

A practical AWS networking implementation in **US East (N. Virginia), `us-east-1`**. This project documents a custom VPC, four subnets across two Availability Zones, public and private routing, security groups, EC2 connectivity, and validation tests.

## Architecture

![AWS custom VPC network architecture](diagrams/aws-vpc-architecture.svg)

The diagram is intentionally project-focused: it shows the AWS resources and traffic paths without organization or assignment-provider branding. The editable SVG is available at [`diagrams/aws-vpc-architecture.svg`](diagrams/aws-vpc-architecture.svg).

## Implemented network

| Component | Configuration |
|---|---|
| VPC | `10.20.0.0/16` (`vpc-0b28a4a82969d27b2`) |
| Availability Zones | `us-east-1a`, `us-east-1b` |
| Public subnet A | `10.20.1.0/24` — `us-east-1a` |
| Private subnet A | `10.20.2.0/24` — `us-east-1a` |
| Public subnet B | `10.20.3.0/24` — `us-east-1b` |
| Private subnet B | `10.20.4.0/24` — `us-east-1b` |
| Internet Gateway | Attached to the VPC |
| Public route table | VPC local route + `0.0.0.0/0` to the Internet Gateway; associated with both public subnets |
| Private route table | VPC local route only; associated with both private subnets |
| Public EC2 | `10.20.1.249`, public IPv4 `100.31.120.30`, Nginx |
| Private EC2 | `10.20.2.209`, no public IPv4 address |

A NAT Gateway was not deployed. The private subnets therefore have no default route to the internet.

## Validation results

- SSH access to the public EC2 instance succeeded from the permitted administrative source.
- Ping from the public EC2 instance to the private EC2 instance succeeded (4 packets received).
- TCP 8080 returned `Connection refused`: the host was reachable, but no service was listening on that port.
- TCP 80 from the public EC2 instance to the private EC2 instance timed out, consistent with the absence of a matching private security-group ingress rule.
- DNS resolution and outbound HTTPS from the public EC2 instance succeeded; `curl -I https://example.com` returned HTTP 200.
- The public Nginx landing page loaded successfully.

See [`docs/connectivity-tests.md`](docs/connectivity-tests.md) for commands and interpretation, and [`docs/security-groups.md`](docs/security-groups.md) for the configured access rules.

## Repository guide

- [`docs/network-design.md`](docs/network-design.md) — VPC, subnet plan, and routing.
- [`docs/security-groups.md`](docs/security-groups.md) — security-group rules and rationale.
- [`docs/connectivity-tests.md`](docs/connectivity-tests.md) — terminal tests and results.
- [`docs/cost-and-cleanup.md`](docs/cost-and-cleanup.md) — cost decisions and cleanup steps.
- [`docs/demo-guide.md`](docs/demo-guide.md) — instructor demonstration sequence.
- [`diagrams/aws-vpc-architecture.svg`](diagrams/aws-vpc-architecture.svg) — editable architecture diagram.
- [`evidence/README.md`](evidence/README.md) — evidence index and screenshot gallery.

## Evidence

Evidence was supplied in an 11-page document. The repository's evidence index identifies the console and terminal captures and their relevance. These are historical screenshots captured during implementation; the AWS resources may be deleted after evidence collection. Confirm the private security group's outbound rules in the live console if your instructor requires a clear screenshot of that specific tab.

Do not commit private keys, passwords, access tokens, or other credentials.
