# Security Groups and Access Control

Security groups act as stateful virtual firewalls attached to network interfaces/instances. This project uses separate groups for public and private EC2 roles.

## Public security group

| Property | Value |
|---|---|
| Name | `IT-SIMPLERA-Public-SG` |
| ID | `sg-090d40809d2c1166b` |

### Inbound rules

| Type | Protocol | Port | Source | Purpose |
|---|---|---:|---|---|
| HTTPS | TCP | 443 | `0.0.0.0/0` | HTTPS ingress rule configured |
| HTTP | TCP | 80 | `0.0.0.0/0` | Public Nginx demo page |
| SSH | TCP | 22 | Operator public IP `/32` at configuration time | Restricted administration |

SSH is not open to all IPv4 addresses. The source is limited to a single public IPv4 address using a `/32` CIDR. If the operator's public IP changes, update this rule deliberately.

### Outbound rules

The public security group retained the default outbound allow-all rule to `0.0.0.0/0`. This permits the public instance to perform DNS lookups and outbound HTTPS requests, subject to routing.

## Private security group

| Property | Value |
|---|---|
| Name | `IT-SIMPLERA-Private-SG` |
| ID | `sg-04c3b4919fe34f9f5` |

### Inbound rules

| Type | Protocol | Port | Source | Purpose |
|---|---|---:|---|---|
| Custom TCP | TCP | 8080 | `IT-SIMPLERA-Public-SG` | Allow the designated application port only from the public tier |
| All ICMP - IPv4 | ICMP | All | `IT-SIMPLERA-Public-SG` | Permit the controlled ping validation |

No inbound HTTP (TCP 80) rule from the public security group was configured. The test from the public instance to private TCP 80 timed out, which is consistent with the intended restriction.

### Outbound rules — verification note

The submitted screenshot set did not clearly show the private security group's outbound tab. **Verify the live outbound rules in the AWS console and capture a readable screenshot before final submission.** Do not treat the public security group's outbound rule as evidence of the private group's configuration.

## Least-privilege rationale

1. SSH is limited to the operator's IP address rather than exposed to the entire internet.
2. The private instance accepts only the specifically required test/application traffic from the public security group.
3. The private tier has no public IP and no direct Internet Gateway route.
4. Security-group references are used for private inbound sources, so the rule follows the identity of the approved public tier rather than a broad subnet CIDR.

## Important distinction: routing vs security group

A connection can fail for different reasons. A route may be missing, a security group may deny traffic, a network ACL may deny traffic, or the destination host may have no service listening. For this reason, the TCP 8080 result (`Connection refused`) is not described as a security-group block. The TCP 80 timeout is the intentional blocked-traffic demonstration documented in the test log.
