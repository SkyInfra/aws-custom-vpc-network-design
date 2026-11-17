# Connectivity Test Evidence

This document records the commands run and the observed results. Tests were performed against the deployed instances in `us-east-1`.

## 1. Workstation → public EC2 SSH

**Command (from the operator workstation):**

```bash
nc -vz 100.31.120.30 22
ssh -i ~/Downloads/it-simplera.pem ubuntu@100.31.120.30
```

**Observed:** TCP port 22 was reachable and SSH login succeeded.

**Meaning:** The public instance was reachable from the configured source IP allowed by the public security group.

## 2. Public EC2 → private EC2 ICMP

**Command (run on the public EC2):**

```bash
ping -c 4 10.20.2.209
```

**Observed:** 4 packets transmitted, 4 received, 0% packet loss.

**Meaning:** The private security group allows ICMP from the public security group. This validates controlled private-address connectivity within the VPC.

## 3. Public EC2 → private EC2 TCP 8080

**Command:**

```bash
nc -zv 10.20.2.209 8080
```

**Observed:** `Connection refused`.

**Meaning:** The destination was reachable enough to reject the TCP connection, but no process was listening on port 8080. This is **not** a successful application test and **not** evidence that the security group blocked the traffic. The inbound rule exists for TCP 8080 from the public security group.

## 4. Public EC2 → private EC2 TCP 80 (intentionally restricted)

**Command:**

```bash
curl --connect-timeout 5 http://10.20.2.209:80
```

**Observed:** Connection timed out after 5 seconds.

**Meaning:** TCP 80 is not allowed by the private security group's inbound rules from the public tier. The timeout is consistent with the intended restriction. A timeout alone does not uniquely identify which network control dropped the packet, but the configured security-group rule provides the expected explanation.

## 5. Public EC2 DNS resolution

**Command (run on Ubuntu EC2):**

```bash
getent hosts example.com
```

**Observed:** DNS resolution returned IPv6 address records.

**Meaning:** The public instance successfully resolved the hostname. `getent` is a Linux command; run it on the EC2 instance rather than macOS if it is not available locally.

## 6. Public EC2 outbound HTTPS

**Command:**

```bash
curl -I https://example.com
```

**Observed:** `HTTP/2 200`.

**Meaning:** The public instance completed an outbound HTTPS request successfully.

## 7. Public web server

The public EC2 instance runs Nginx and serves the custom IT-SIMPLERA landing page.

**URL:** [http://100.31.120.30](http://100.31.120.30)

**Meaning:** The public web endpoint was reachable over HTTP.

## Test summary

| Source | Destination | Protocol / port | Observed result | Status |
|---|---|---|---|---|
| Operator workstation | Public EC2 | TCP 22 | SSH successful | Pass |
| Public EC2 | Private EC2 | ICMP | 4/4 replies | Pass |
| Public EC2 | Private EC2 | TCP 8080 | Connection refused | Host reachable; no listener |
| Public EC2 | Private EC2 | TCP 80 | Timeout | Expected restricted traffic |
| Public EC2 | `example.com` | DNS | Resolved | Pass |
| Public EC2 | `example.com` | HTTPS 443 | HTTP/2 200 | Pass |
| Browser | Public EC2 | HTTP 80 | Nginx page loaded | Pass |

## Evidence to attach

Place original screenshots in the repository's `evidence/` folder. Capture the terminal output and corresponding AWS console rules clearly. Include source, destination, protocol/port, and result in the screenshot captions or this test table.
