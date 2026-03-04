---
title: "01 summarization – Ip_plan (CML-bitsize-labs)"
categories: ["01-summarization"]
source_repo: "CML-bitsize-labs"
source_path: "labs/ospf/01-summarization/ip_plan.md"
---

## IP Address Plan

### Router R1
| Interface | IP / Mask     | OSPF Area | Comment |
|----------|----------------|-----------|-----------|
| Eth0/0   | 10.10.0.2/30   | 10        | Transit to R2 |
| Lo1      | 10.10.1.1/30   | 10        | Loopback1 |
| Lo2      | 10.10.2.1/30   | 10        | Loopback2 |
| Lo3      | 10.10.3.1/30   | 10        | Loopback3 |
| Lo4      | 10.11.1.1/30   | –         | Extern |
| Lo5      | 10.11.2.1/30   | –         | Extern |
| Lo6      | 10.11.3.1/30   | –         | Extern |
| **RID**  | **1.1.1.1**    | –         | – |

---

### Router R2
| Interface | IP / Mask     | OSPF Area | Comment |
|----------|----------------|-----------|-----------|
| Eth0/0   | 10.10.0.1/30   | 10        | Transit to R1 |
| Eth0/1   | 10.0.0.1/30    | 0         | Transit to R3 |
| **RID**  | **2.2.2.2**    | –         | – |

---

### Router R3
| Interface | IP / Mask     | OSPF Area | Comment |
|----------|----------------|-----------|-----------|
| Eth0/0   | 10.0.0.2/30    | 0         | Transit to R2 |
| Eth0/1   | 10.20.0.2/30   | 20        | Transit to R4 |
| **RID**  | **3.3.3.3**    | –         | – |

---

### Router R4
| Interface | IP / Mask     | OSPF Area | Comment |
|----------|----------------|-----------|-----------|
| Eth0/0   | 10.20.0.1/30   | 20        | Transit to R3 |
| **RID**  | **4.4.4.4**    | –         | – |

---
