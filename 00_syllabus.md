# Advanced Communication Networks — Course Syllabus

## Course Information

| Field | Details |
|-------|---------|
| **Course Title** | Advanced Communication Networks |
| **Credits** | 3 |
| **Total Contact Hours** | 48 hours |
| **Duration** | 16 weeks × 3 hours/week |
| **Format** | Lectures + Labs + Project |
| **Target Students** | Senior undergraduate / graduate EE, Telecom Engineering |

---

## Course Objectives

By the end of this course, students will be able to:

1. Trace the complete evolution of mobile networks from GSM (2G) through UMTS (3G), LTE (4G), 5G NR/SA, to emerging 6G concepts.
2. Explain WHY each generation was developed, WHAT problem it solved, and HOW traffic flows through each architecture.
3. Analyze core network architectures: circuit-switched, packet-switched, all-IP EPC, and service-based 5G Core.
4. Describe mobility management, handover mechanisms, and their evolution across generations.
5. Understand network slicing, QoS frameworks, SDN, and NFV in modern networks.
6. Explain 5G Core procedures (registration, authentication, PDU session) at the signaling level.
7. Critically assess 6G research directions (ISAC, RIS, AI-native, cell-free MIMO, sub-THz).

---

## Module List (26 Modules)

| # | Module Title | File |
|---|-------------|------|
| 1 | GSM, GPRS, and EDGE | `01_gsm_gprs_edge.md` |
| 2 | 2G Circuit Switching | `02_circuit_switching.md` |
| 3 | 2G/3G Core Networks | `03_2g_3g_core.md` |
| 4 | Mobility and Handover | `04_mobility_handover.md` |
| 5 | 4G LTE Architecture | `05_4g_lte.md` |
| 6 | E-UTRAN (LTE Radio Access) | `06_eutran.md` |
| 7 | LTE Scheduling | `07_lte_scheduling.md` |
| 8 | Random Access Channel (RACH) | `08_random_access.md` |
| 9 | Evolved Packet Core (EPC) | `09_epc.md` |
| 10 | Diameter Protocol | `10_diameter.md` |
| 11 | Home Subscriber Server (HSS) | `11_hss.md` |
| 12 | Mobility Management Entity (MME) | `12_mme.md` |
| 13 | S-GW and P-GW | `13_sgw_pgw.md` |
| 14 | 5G Networks Introduction | `14_5g_networks.md` |
| 15 | Software Defined Networking (SDN) | `15_sdn.md` |
| 16 | Network Function Virtualization (NFV) | `16_nfv.md` |
| 17 | 3GPP Standardization | `17_3gpp.md` |
| 18 | 5G Service-Based Architecture (SBA) | `18_sba.md` |
| 19 | 5G Core Architecture | `19_5g_core.md` |
| 20 | 5G Procedures | `20_5g_procedures.md` |
| 21 | Network Slicing | `21_network_slicing.md` |
| 22 | Quality of Service (QoS) | `22_qos.md` |
| 23 | User Plane | `23_user_plane.md` |
| 24 | Control Plane | `24_control_plane.md` |
| 25 | Network Evolution — Grand Synthesis | `25_network_evolution.md` |
| 26 | Future 6G Technologies | `26_future_6g.md` |

**Supporting files:** `00_prerequisites.md` (background review), `00_syllabus.md` (this file)

---

## Grading Breakdown

| Component | Weight | Details |
|-----------|--------|---------|
| **Exams** | 50% | Midterm (20%) + Final (30%) |
| **Labs** | 30% | 6 lab assignments (5% each) |
| **Project** | 20% | Proposal (5%) + Report & Demo (15%) |

---

## Recommended Textbooks

1. **Sesia, S., Toufik, I., & Baker, M.** — *LTE – The UMTS Long Term Evolution: From Theory to Practice*, 2nd Ed., Wiley.
2. **Dahlman, E., Parkvall, S., & Sköld, J.** — *5G NR: The Next Generation Wireless Access Technology*, Academic Press.
3. **Holma, H. & Toskala, A.** — *LTE for UMTS*, Wiley.
4. **3GPP Specifications** — TS 23.501 (5GS), TS 23.502 (procedures), TS 24.501 (NAS), TS 38.300 (NR) — at [3gpp.org](https://www.3gpp.org).
5. **Rappaport, T. S.** — *Wireless Communications: Principles and Practice*, 2nd Ed., Prentice Hall.

---

## Week-by-Week Schedule

| Week | Modules | Topic Focus |
|------|---------|-------------|
| 1 | 1–2 | GSM architecture, GPRS/EDGE, circuit switching fundamentals |
| 2 | 3 | 2G/3G core networks, UMTS architecture, CS/PS domains |
| 3 | 4 | Mobility management, handover types (2G→5G evolution) |
| 4 | 5–6 | LTE architecture, E-UTRAN, protocol stack; **Lab 1 assigned** |
| 5 | 7–8 | LTE scheduling, RACH procedure |
| 6 | 9 | Evolved Packet Core (deep dive); **Lab 2 assigned** |
| 7 | 10–11 | Diameter protocol, HSS, authentication |
| 8 | 12–13 | MME procedures, S-GW/P-GW, GTP; **Midterm Review** |
| 9 | — | **MIDTERM EXAM** |
| 10 | 14 | 5G introduction, NR, NSA vs SA; **Lab 3 assigned** |
| 11 | 15–16 | SDN, NFV, cloud-native networks |
| 12 | 17–18 | 3GPP standardization, Service-Based Architecture; **Lab 4 assigned** |
| 13 | 19–20 | 5G Core architecture, 5G procedures (registration, PDU session) |
| 14 | 21–22 | Network slicing, QoS evolution (QCI→5QI); **Lab 5 assigned** |
| 15 | 23–25 | User plane, control plane, network evolution synthesis; **Lab 6 assigned** |
| 16 | 26 | Future 6G technologies, **project presentations & final review** |
| 17 | — | **FINAL EXAM** |

---

## Lab Assignments

| Lab | Week | Topic | Related Modules |
|-----|------|-------|-----------------|
| 1 | 4 | LTE Link Budget & Path Loss | 5, 6 |
| 2 | 6 | EPC Signaling (Attach/Detach trace) | 9, 12 |
| 3 | 10 | 5G NR Scheduling & Resource Allocation | 7, 14 |
| 4 | 12 | 5G Core SBA Service Discovery | 18, 19 |
| 5 | 14 | Network Slicing Configuration | 21, 22 |
| 6 | 15 | Traffic Flow Tracing (User Plane) | 23, 24 |

---

## Project

Students work individually or in pairs (max 2) on a semester project. Topics may include:
- Comparative analysis of LTE vs 5G for a specific use case
- Network slicing design for an enterprise scenario
- Implementation of a scheduling algorithm (using the wireless_lab simulator)
- 6G research topic survey and feasibility analysis
- Signaling analysis using real-world packet captures

**Deliverables:** Proposal (Week 13), Final Report + Demo/Presentation (Week 16)

---

## Academic Integrity

All submitted work must be original. Collaboration on labs is encouraged for discussion but each student submits individually. Plagiarism will result in a zero for the assignment and referral to the academic integrity board.

---

*Last updated: August 2026*
