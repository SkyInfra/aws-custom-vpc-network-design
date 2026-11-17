# Network Design and Routing

## 1. Region selection

The deployment uses **US East (N. Virginia), `us-east-1`**. The region was selected for this hands-on exercise and all listed resources were created there. The same design principles apply in other AWS Regions; resources in different regions are separate deployments.

## 2. VPC

| Property | Value |
|---|---|
| Name | `IT-SIMPLERA-VPC` |
| VPC ID | `vpc-0b28a4a82969d27b2` |
| IPv4 CIDR | `10.20.0.0/16` |
| DNS resolution | Enabled |
| DNS hostnames | Enabled |

A `/16` network provides a private address space for the four smaller subnet ranges used in this project.

## 3. Subnet plan

| Subnet name | CIDR | Availability Zone | Purpose |
|---|---|---|---|
| Public-1 | `10.20.1.0/24` | `us-east-1a` | Public-facing workloads |
| Private-1 | `10.20.2.0/24` | `us-east-1a` | Internal workloads |
| Public-2 | `10.20.3.0/24` | `us-east-1b` | Public-facing workloads / AZ coverage |
| Private-2 | `10.20.4.0/24` | `us-east-1b` | Internal workloads / AZ coverage |

Each subnet has a unique, non-overlapping CIDR. Both public and private tiers are represented in each of two Availability Zones.

## 4. Internet Gateway

- Name: `IT-SIMPLERA-IGW`
- Attached to: `IT-SIMPLERA-VPC`
- Gateway ID: `igw-07f3f47f685d3a994`

An Internet Gateway is the VPC component that enables internet routing for resources that also have the required public addressing, route-table entry, and security/network permissions. Merely attaching an Internet Gateway does not make every subnet public.

## 5. Route tables

### Public route table

- Name: `IT-SIMPLERA-Public-RT`
- ID: `rtb-08d830fb6621c83f4`
- Explicit associations: Public-1 and Public-2

| Destination | Target | State |
|---|---|---|
| `10.20.0.0/16` | local | Active |
| `0.0.0.0/0` | `igw-07f3f47f685d3a994` | Active |

**Traffic path:** Public instance → public subnet route table → Internet Gateway → internet. Return traffic is subject to security group and network ACL rules.

### Private route table

- Name: `IT-SIMPLERA-Private-RT`
- ID: `rtb-074c035e5e60201ed`
- Explicit associations: Private-1 and Private-2

| Destination | Target | State |
|---|---|---|
| `10.20.0.0/16` | local | Active |

There is no `0.0.0.0/0` route to an Internet Gateway or NAT Gateway.

**Traffic path:** Private instance → private subnet route table → VPC local route for destinations inside the VPC. Internet destinations have no route through this table and are not reachable through an internet gateway.

### If private instances later need outbound internet

With approval for the additional cost, a NAT Gateway can be placed in a public subnet. The private route table can then receive `0.0.0.0/0 → NAT Gateway`. The NAT Gateway uses the public subnet's route to the Internet Gateway. This optional component was **not deployed** in this assignment.

## 6. EC2 test instances

| Property | Public EC2 | Private EC2 |
|---|---|---|
| Name | `IT-SIMPLERA-Public-EC2` | `IT-SIMPLERA-Private-EC2` |
| Instance ID | `i-0d77544cb0be27dad` | `i-03ba248a17c0e63de` |
| Private IPv4 | `10.20.1.249` | `10.20.2.209` |
| Public IPv4 | `100.31.120.30` | None |
| Subnet | Public-1 | Private-1 |
| Instance type | `t3.micro` | `t3.micro` |
| OS | Ubuntu 26.04.1 LTS | Ubuntu 26.04.1 LTS |

The public instance runs Nginx and serves a project landing page over HTTP. The private instance has no public IPv4 address and is not directly addressable from the public internet.

## 7. Key networking concepts demonstrated

- **VPC:** logically isolated network in AWS.
- **Subnet:** a range of IP addresses within one Availability Zone.
- **Public subnet:** subnet whose associated route table has a route to an Internet Gateway.
- **Private subnet:** subnet without a direct route to an Internet Gateway.
- **Route table:** determines where subnet traffic is directed.
- **Internet Gateway:** provides an internet route for eligible VPC resources.
- **Local route:** enables communication between addresses covered by the VPC CIDR, subject to security controls.
