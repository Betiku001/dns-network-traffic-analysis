# Case Study: Network Traffic Analysis & DNS Outage Investigation

## Executive Summary
At 1:24 PM, multiple external clients reported an inability to access the primary web application (`yummyrecipesforme.com`), encountering a **"destination port unreachable"** error. As part of the Tier-1 Security Operations Center (SOC) team, a packet capture investigation was conducted using **tcpdump**. Traffic analysis identified an **ICMP "udp port 53 unreachable"** response from the DNS server, confirming a breakdown in domain name resolution. Current working hypotheses point to either a **Denial of Service (DoS) attack** or a **firewall port-blocking misconfiguration**.

---
## Incident Overview
* **Target Application:** `yummyrecipesforme.com`
* **Protocols Identified:** UDP, DNS, ICMP
* **Primary Symptom:** "Destination port unreachable" / Port 53 Failure
* **Analysis Tool:** `tcpdump`
* **Status:** Escalated to Tier-2 Engineering / Network Operations

---

## Technical Analysis & Log Interpretation

### 1. Packet Capture Data Analysis
Network packet inspection using `tcpdump` revealed a repeating sequence of log lines per connection attempt:

```text
13:24:32.192571 IP client.local.51020 > 203.0.113.2.domain: 35084+ A? yummyrecipesforme.com. (42)
13:24:32.192600 IP 203.0.113.2 > client.local: ICMP 203.0.113.2 udp port 53 unreachable length 70
```
### 2. Log Breakdown & Diagnostic Markers
* **UDP Transport:** The initial outbound request from the client browser used UDP to reach the primary DNS server (203.0.113.2) on Port 53, the standard port for DNS operations.

DNS Request Flags:

* **Query ID (35084+):** The trailing + sign indicates specific operational flags attached to the UDP query.

* **Record Flag (A?):** Indicates an active DNS query requesting an A record mapping yummyrecipesforme.com to an IPv4 address.

* **ICMP Error Response:** Instead of a valid DNS response containing the target IP address, the destination server returned an ICMP error stating udp port 53 unreachable.

### Root Cause Hypotheses
Based on the packet trace, two primary root causes are under investigation:

1. **Denial of Service (DoS) Attack:** An external threat actor may have targeted the DNS server (203.0.113.2) with high-volume flood traffic, crashing the DNS service process or exhausting network sockets.

2. **Firewall Rule Misconfiguration:** An administrative change or automated rule update on network firewalls may have blocked incoming UDP traffic on Port 53, preventing legitimate client requests from reaching the DNS daemon.

## Incident Handling Workflow (NIST SP 800-61 Alignment)
[ 1. Detection & Analysis ] ➔ [ 2. Containment & Escalation ] ➔ [ 3. Eradication & Recovery ]
   - Analyzed tcpdump logs        - Escalated to Tier-2 Engineers     - Audit firewall rules (Port 53)
   - Identified ICMP error        - Identified destination IP        - Restart/Failover DNS daemon
### Remediation & Immediate Action Items
1. **Service Verification:** Inspect system logs (systemctl status / event logs) on 203.0.113.2 to confirm if the DNS service process is active.
2. **Firewall Audit:** Review active Access Control Lists (ACLs) and firewall rules on host and network firewalls for unintended blocks on UDP Port 53.
3. **Failover Execution:** Temporarily update network routing/DHCP options to point clients to a secondary DNS resolver while primary server recovery is underway.
4. **Hardening Controls:** Implement rate-limiting and anti-DoS traffic shaping on the perimeter firewall to protect DNS infrastructure against resource exhaustion.

## Remediation & Immediate Action Items
├── README.md               # Incident Case Study Write-up
└── logs/
    └── tcpdump_dns.log     # Sanitized raw packet capture logs
    ```
