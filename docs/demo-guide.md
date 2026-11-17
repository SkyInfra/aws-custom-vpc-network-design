# Instructor Demonstration Guide

A short demonstration can follow this sequence.

## 1. Show the network design

1. Open the VPC console in `us-east-1`.
2. Show `IT-SIMPLERA-VPC` and CIDR `10.20.0.0/16`.
3. Show all four subnets and their AZs:
   - Public-1: `10.20.1.0/24`, `us-east-1a`
   - Private-1: `10.20.2.0/24`, `us-east-1a`
   - Public-2: `10.20.3.0/24`, `us-east-1b`
   - Private-2: `10.20.4.0/24`, `us-east-1b`

## 2. Explain routing

- Show the Internet Gateway attached to the VPC.
- Show the public route table: local route and `0.0.0.0/0` to the Internet Gateway, associated with both public subnets.
- Show the private route table: local route only, associated with both private subnets.
- Explain that no NAT Gateway was created because it was optional and chargeable.

## 3. Explain security

- Show the public security group and point out SSH restricted to the operator IP `/32`.
- Show the private security group and its ICMP/TCP 8080 sources restricted to the public security group.
- Show the private security group's outbound tab separately; confirm the live rule set before the demo.

## 4. Demonstrate the instances and tests

- Open the public Nginx page: [http://100.31.120.30](http://100.31.120.30).
- From the public instance, run `ping -c 4 10.20.2.209` and show successful replies.
- Run `curl --connect-timeout 5 http://10.20.2.209:80` and explain the timeout as the intentionally restricted TCP 80 path.
- Explain that TCP 8080 returned `Connection refused` because no service was listening; do not present it as a security-group denial.
- Show `getent hosts example.com` and `curl -I https://example.com` from the public instance.

## 5. Close with cost and security

Explain that the private subnets have no direct internet route, no NAT Gateway was deployed, SSH is source-restricted, and test resources should be cleaned up after assessment approval.
