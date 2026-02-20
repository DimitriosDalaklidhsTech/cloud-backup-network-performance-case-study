 # Cloud File Backup & TCP Behaviour
**Translating Network Protocol Design into Business-Relevant Infrastructure Decisions**

> Academic research presentation — University of Western Macedonia, Computer Science  
> Author: Dimitrios Dalaklidis | Course: Network Design

---

## The Business Problem

Every organisation that relies on cloud storage — from a 10-person startup using Dropbox to an enterprise running nightly database backups to AWS — depends on TCP to move that data reliably. But TCP is not a monolith. The protocol makes dozens of design decisions underneath the surface, and those decisions have direct, measurable consequences on backup reliability, network costs, and user experience.

**This project answers a question that matters to any technology-dependent business:**

> *When your cloud backup runs, what is TCP actually doing — and is it making the right trade-offs for your use case?*

---

## What This Project Covers

The analysis is structured across three interconnected themes, progressing from fundamentals to applied decision-making.

### 1. TCP Reliability as a Design Choice

TCP operates on top of an unreliable network layer (IP). Reliability is not free — it is engineered through acknowledgements (ACKs), retransmissions, and congestion control. This section examines the cost-benefit of that engineering: what reliability buys you, what it costs in latency and overhead, and why the right level of aggressiveness depends entirely on the network environment.

**Key insight for business:** A system tuned too aggressively retransmits unnecessarily and amplifies network congestion. A system tuned too conservatively leaves bandwidth on the table and slows backup completion. Neither is a configuration error — both are the result of choosing the wrong protocol defaults for the environment.

### 2. Cloud Backup as a Specific Use Case

Cloud file backup is not generic data transfer. It has a distinct profile: large volumes, background operation, tolerance for latency, and zero tolerance for data loss. This section maps those characteristics onto TCP's behaviour — identifying where TCP's design aligns well with backup workloads and where it creates friction.

Topics covered:
- Why bulk transfers favour long-lived TCP connections over short-lived ones
- How congestion events during backup can degrade unrelated network activity — browsing, video calls, interactive applications running simultaneously
- The role of AIMD (Additive Increase, Multiplicative Decrease) in ensuring fair bandwidth sharing across simultaneous users on the same network
- How timeout sensitivity (RTO) behaves unpredictably in cloud environments with variable RTT — a direct consequence of geographic distribution, multi-tenant infrastructure, and dynamic routing

**Key insight for business:** The variability in cloud RTT means that TCP timeout defaults designed for stable networks can trigger unnecessary retransmissions, inflate backup windows, and increase total transfer time — without any actual data loss having occurred. This is an invisible cost that organisations rarely attribute to protocol misconfiguration.

### 3. Protocol Selection: TCP CUBIC vs. TCP BBR for Cloud Backup

The analysis culminates in a structured comparison of two congestion control algorithms, evaluated specifically against the cloud backup use case.

| | TCP CUBIC | TCP BBR |
|---|---|---|
| **Approach** | Loss-based | Model-based |
| **Congestion signal** | Packet loss | Bandwidth + RTT estimation |
| **Default on** | Most Linux systems | Google infrastructure, opt-in Linux |
| **Strengths** | Proven, widely deployed, fair with peers | Lower latency, resists bufferbloat, adapts to variable capacity |
| **Weaknesses** | Causes bufferbloat on high-bandwidth links | Unfair to CUBIC peers; higher RTT for small files |
| **Cloud backup verdict** | ❌ Weaker choice | ✅ Stronger choice |

**Why CUBIC underperforms for cloud backup:** On LTE and shared wireless links — common in cloud environments — CUBIC aggressively grows its congestion window until it triggers packet loss. This causes bufferbloat, inflated RTT, and self-induced delays that slow backup performance and degrade other network activity simultaneously. In testing, CUBIC produced 5–10% duplicate ACKs and consistently lower throughput for variable-capacity scenarios.

**Why BBR is the stronger choice — with caveats:** BBR models the network bottleneck in real time rather than waiting for loss events. This produces more stable throughput and lower latency for bulk transfers. However, BBR carries meaningful trade-offs: it can dominate bandwidth when sharing a connection with CUBIC flows, and it performs worse than CUBIC for small file transfers (~1 MB) due to its aggressive startup phase introducing temporary RTT inflation before the flow stabilises.

**Conclusion:** There is no universally optimal protocol. The right choice depends on workload size, network type, peer traffic composition, and latency tolerance. This is precisely the kind of contextual, evidence-based decision-making that separates well-architected cloud infrastructure from default deployments.

---

## Skills Demonstrated

This project was deliberately framed to bridge technical depth and business communication — a combination directly relevant to technology consulting, cloud architecture advisory, and digital transformation roles.

**Technical foundations applied:**
- Network protocol analysis (TCP/IP stack, transport layer behaviour)
- Congestion control algorithm comparison (CUBIC, BBR, AIMD, Slow Start)
- Cloud infrastructure reasoning (RTT variability, multi-tenant environments, LTE link behaviour)
- Academic research synthesis across 6 peer-reviewed sources including IETF RFCs and ACM SOSP papers

**Consulting-relevant capabilities demonstrated:**
- Translating low-level protocol mechanics into business-language trade-off analysis
- Structuring a technical recommendation with explicit conditions, caveats, and context-dependence
- Communicating infrastructure risk in terms of user experience and operational cost — not just technical metrics
- Applying a use-case lens to protocol evaluation rather than treating benchmark performance as absolute

---

## References

1. J. Postel, *Transmission Control Protocol*, RFC 793, IETF, 1981
2. M. Mathis et al., *TCP Selective Acknowledgment Options*, RFC 2018, IETF, 1996
3. A. Muthitacharoen et al., *A Low-Bandwidth Network File System*, ACM SOSP, 2001
4. S. Ghemawat et al., *The Google File System*, ACM SOSP, 2003
5. J.-Y. Le Boudec & P. Thiran, *Network Calculus*, Springer LNCS, 2001
6. F. Li et al., *TCP CUBIC versus BBR on the Highway*, PAM 2017, Springer LNCS

---

## About

**Dimitrios Dalaklidis** is a Computer Science student at the University of Western Macedonia (expected graduation July 2026) with hands-on experience in backend systems, TCP/IP networking, Linux internals, and containerisation. This presentation was produced as part of a Network Design course and represents an applied exercise in evaluating infrastructure trade-offs for real-world business scenarios.

📧 dalaklidesdemetres@gmail.com  
🔗 [LinkedIn](https://linkedin.com/in/dimitris-dalaklidis)  
💻 [GitHub](https://github.com/DimitriosDalaklidhs)
