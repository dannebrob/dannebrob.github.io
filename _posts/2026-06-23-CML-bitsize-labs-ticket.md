---
title: "Ticket (CML-bitsize-labs)"
categories: ["03-troubleshooting"]
source_repo: "CML-bitsize-labs"
source_path: "labs/ospf/03-Troubleshooting/ticket.md"
---

# TICKET: NET-2847

**Title:** Office Branch Connectivity Degraded — Unreachable Subnets via Core Routers

**Priority:** HIGH  
**Status:** OPEN  
**Assignee:** Network Operations  
**Reporter:** John Martinez (IT Support)  
**Created:** 2024-11-15 09:32 UTC  
**Updated:** 2024-11-15 10:15 UTC

---

## SUMMARY

Users at the East Campus location (192.168.3.0/24) are unable to reach servers on the West Campus network (192.168.1.0/24). Core backbone connectivity appears partial. Three routers (R1, R2, R3) run OSPF across multiple areas, but end-to-end routing is broken despite Layer 2 connectivity being confirmed.

---

## IMPACT

- **Affected Users:** ~45 staff at East Campus (Ubuntu3 subnet)
- **Service:** Inter-office network access, file server connectivity
- **Business Impact:** Departments cannot access shared resources; productivity blocked
- **Workaround:** None currently available

---

## SYMPTOMS

1. **Ping fails end-to-end:**
   ```
   ubuntu3$ ping 192.168.1.10
   PING 192.168.1.10 (192.168.1.10) 56(84) bytes of data.
   (no response — times out after 5 seconds)
   ```

2. **Local network works fine:**
   ```
   ubuntu3$ ping 192.168.3.10
   64 bytes from 192.168.3.10: icmp_seq=1 ttl=64 time=2.1 ms
   ```

3. **Partial OSPF adjacencies:**
   - R1 ↔ R2: Up and stable
   - R2 ↔ R3: **Stuck in EXSTART/EXCHANGE** (never reaching FULL state)
   - Loopback addresses (1.1.1.1, 2.2.2.2, 3.3.3.3) do not ping from remote routers

4. **Routes missing from routing table:**
   ```
   R2# show ip route ospf
   (no routes from Area 2 appear)
   ```

---

## ENVIRONMENT

| Component | Details |
|-----------|---------|
| **Topology** | 3-router OSPF backbone (R1, R2, R3) |
| **OSPF Design** | Multi-area: Area 0 (backbone), Area 1 (West Campus), Area 2 (East Campus) |
| **Router Models** | Cisco IOS-XE (all routers) |
| **Backbone Link** | R1 G0/1 (10.0.12.0/30) ↔ R2 G0/0; R2 G0/1 (10.0.23.0/30) ↔ R3 G0/0 |
| **Access Links** | R1 G0/0 → Ubuntu1 (192.168.1.0/24); R3 G0/1 → Ubuntu3 (192.168.3.0/24) |
| **OSPF Process** | Process ID 1; Router IDs: R1=1.1.1.1, R2=2.2.2.2, R3=3.3.3.3 |

---

## TROUBLESHOOTING STEPS ALREADY ATTEMPTED

- ✅ **Layer 2 verification:** All cables confirmed connected; no down interfaces reported
- ✅ **Basic reachability:** OSPF process running on all routers; hello/dead timers match
- ✅ **IP addressing:** No duplicate IPs; subnets non-overlapping
- ✅ **Interface status:** All interfaces `show as up/up`
- ❌ **OSPF adjacency fix:** Unknown — R2 ↔ R3 link still stuck
- ❌ **Route redistribution:** East Campus subnet not appearing in routing tables
- ❌ **Root cause identified:** No

---

## EXPECTED BEHAVIOR

1. R2 and R3 should form a **FULL OSPF adjacency** on the backbone link
2. All routers should have complete OSPF database (LSA from all areas)
3. Routing tables should contain:
   - West Campus network (192.168.1.0/24) reachable via R1
   - East Campus network (192.168.3.0/24) reachable via R3
4. Users at both campuses can ping and route to each other seamlessly

---

## CURRENT BEHAVIOR

1. R2 ↔ R3 adjacency stuck in **EXSTART/EXCHANGE** state
2. OSPF database incomplete on backbone routers
3. East Campus subnet (192.168.3.0/24) **missing from all routing tables**
4. End-to-end ping fails; routing loop or path unavailable

---

## DIAGNOSTIC COMMANDS TO RUN

```bash
# On each router:
show ip ospf neighbor              # Check adjacency state
show ip ospf interface              # Verify area assignments & network type
show ip ospf database               # Check LSDB completeness
show ip route ospf                  # Verify learned routes
show running-config | include ospf  # Review OSPF config
show ip ospf neighbor detail        # Deep dive on stuck adjacency

# From Ubuntu hosts:
ping 192.168.1.10                   # Test cross-campus reachability
traceroute 192.168.3.10             # Identify where path breaks
route -n                             # Check local routing table
```

---

## NOTES

- **Last known good state:** Not documented; assume this was deployed recently
- **Configuration management:** No backups available; must troubleshoot live config
- **Urgency:** East Campus users escalating; needs resolution within 4 hours
- **Access:** Console access available to all routers; no restrictions

---

## NEXT STEPS (TO BE ASSIGNED)

1. **Network Engineer:** Connect to R2 and R3; run `show ip ospf interface` to check network type on backbone link
2. **Network Engineer:** Verify OSPF `network` statements on all routers; check wildcard masks (especially R3)
3. **Network Engineer:** Once root cause identified, propose fix and test before deploying to production
4. **Ticket Owner:** Document findings and update customer

---

## REFERENCES

- [OSPF Multi-Area Design Guide](internal-wiki/ospf-areas)
- [Network Type Mismatch Troubleshooting](internal-wiki/ospf-network-types)
- [Router Configuration Repo](internal-gitlab/network-configs/main)

---

**Please reply with:**
- Initial findings from diagnostic commands
- Suspected root cause(s)
- Proposed remediation steps
- Estimated time to resolve
