# Module 22: Quality of Service (QoS)

## Advanced Communication Networks

---

## 1. Why QoS?

### The Problem: Not All Traffic Is Equal

Different applications have fundamentally different network requirements:

| Service | Delay Tolerance | Loss Tolerance | Bandwidth | Pattern |
|---------|----------------|----------------|-----------|---------|
| Voice (VoLTE) | Very Low (<150ms) | Low (<1%) | Low (64 kbps) | Constant |
| Video Call | Low (<300ms) | Low (<1%) | Medium (1-5 Mbps) | Variable |
| Video Streaming | Medium (<5s buffer) | Very Low | High (5-25 Mbps) | Bursty |
| IoT Sensor | High (seconds) | Very Low | Very Low (kbps) | Periodic |
| Web Browsing | Medium (<1s) | None (TCP retx) | Variable | Bursty |
| Email | High (seconds) | None (TCP retx) | Low | Bursty |
| Autonomous Driving | Ultra-Low (<10ms) | Ultra-Low | Medium | Constant |

### Without QoS:
- Voice calls get choppy during congestion
- Video freezes while background downloads continue
- Emergency services compete with social media
- Industrial control packets delayed by bulk transfers

### QoS Goal:
**Differentiate traffic and provide appropriate treatment to each service based on its requirements.**

---

## 2. QoS Evolution Across Generations

```
┌─────────────────────────────────────────────────────────────────────┐
│                    QoS EVOLUTION TIMELINE                            │
├──────────┬──────────────┬───────────────────┬───────────────────────┤
│   2G     │     3G       │      LTE (4G)     │       5G NR           │
│          │              │                   │                       │
│ No QoS   │ 4 Traffic    │ Bearer-Based QoS  │ Flow-Based QoS        │
│ (all     │ Classes      │ (QCI + Bearer)    │ (5QI + QoS Flow)      │
│ circuit- │              │                   │                       │
│ switched)│ Conversational│ 9 Standard QCIs  │ 90 Standard 5QIs      │
│          │ Streaming    │ GBR/Non-GBR      │ GBR/Non-GBR/DC-GBR   │
│ Same     │ Interactive  │ Default+Dedicated │ Reflective QoS        │
│ quality  │ Background   │ bearers          │ Fine granularity       │
│ for all  │              │                   │                       │
└──────────┴──────────────┴───────────────────┴───────────────────────┘
```

### 2G (GSM): No Real QoS
- All services are **circuit-switched**
- Each call gets a dedicated timeslot → same quality for everyone
- No differentiation needed (only voice + SMS)

### 3G (UMTS): Four Traffic Classes
| Class | Delay | Example |
|-------|-------|---------|
| Conversational | Strict (< 150ms) | Voice, video call |
| Streaming | Moderate (< 1s) | Video streaming |
| Interactive | Variable | Web browsing |
| Background | Best-effort | Email, downloads |

### LTE (4G): Bearer-Based QoS
- Introduced **EPS bearers** with specific QoS parameters
- **QCI** (QoS Class Identifier): standardized classes
- **ARP** (Allocation and Retention Priority)
- **GBR** (Guaranteed Bit Rate) vs **Non-GBR** bearers

### 5G NR: Flow-Based QoS
- **QoS Flows** identified by **QFI** (QoS Flow Identifier)
- **5QI** replaces QCI with more values
- **Reflective QoS** for simplified UL treatment
- Finer granularity: multiple flows per PDU session

---

## 3. LTE Bearer-Based QoS

### Bearer Concept

```
┌────────────────────────────────────────────────────────────┐
│                    EPS BEARER ARCHITECTURE                   │
│                                                            │
│  ┌─────┐          ┌─────┐          ┌─────┐    ┌───────┐  │
│  │ UE  │──────────│eNB  │──────────│S-GW │────│P-GW   │  │
│  └─────┘  Radio   └─────┘   S1-U   └─────┘S5/8└───────┘  │
│           Bearer            Bearer          Bearer         │
│                                                            │
│  ◄──────── Radio Bearer ────────►                          │
│  ◄──────────── S1 Bearer ──────────────►                   │
│  ◄──────────────── S5/S8 Bearer ────────────────────►      │
│  ◄═══════════════ EPS Bearer (end-to-end) ══════════════►  │
│                                                            │
│  Each EPS Bearer = ONE QCI + ONE ARP                       │
└────────────────────────────────────────────────────────────┘
```

**Bearer = GTP-U tunnel with specific QoS parameters (QCI + ARP + bit rates)**

### Default vs Dedicated Bearers

| Aspect | Default Bearer | Dedicated Bearer |
|--------|---------------|-----------------|
| Creation | At PDU connectivity (attach) | On-demand (triggered) |
| Lifetime | Always on (until detach) | Temporary (can be released) |
| QoS | Best-effort (QCI 9) | Specific QoS (any QCI) |
| Type | Always Non-GBR | GBR or Non-GBR |
| IP Address | Assigned with bearer | Shares IP of default |
| Purpose | Internet access | VoLTE, video, gaming |

### QCI Table (Standardized Values)

| QCI | Resource Type | Priority | Packet Delay Budget | Packet Error Loss Rate | Example Services |
|-----|:------------:|:--------:|:------------------:|:---------------------:|-----------------|
| 1 | GBR | 2 | 100 ms | 10⁻² | Conversational Voice |
| 2 | GBR | 4 | 150 ms | 10⁻³ | Conversational Video |
| 3 | GBR | 3 | 50 ms | 10⁻³ | Real-Time Gaming |
| 4 | GBR | 5 | 300 ms | 10⁻⁶ | Non-Conv. Video (Buffered) |
| 5 | Non-GBR | 1 | 100 ms | 10⁻⁶ | IMS Signaling |
| 6 | Non-GBR | 6 | 300 ms | 10⁻⁶ | Video (Buffered), TCP web |
| 7 | Non-GBR | 7 | 100 ms | 10⁻³ | Voice, Live Video, Gaming |
| 8 | Non-GBR | 8 | 300 ms | 10⁻⁶ | Video (Buffered), TCP web |
| 9 | Non-GBR | 9 | 300 ms | 10⁻⁶ | Video, TCP, Email, Chat |

#### Mission-Critical (MCX) QCI Values (65-82)

| QCI | Resource Type | Priority | Delay | Example |
|-----|:------------:|:--------:|:-----:|---------|
| 65 | GBR | 0.7 | 75 ms | MC-PTT Voice |
| 66 | GBR | 2 | 100 ms | MC-Video (non-real-time) |
| 67 | GBR | 1.5 | 100 ms | MC-PTT Video |
| 69 | Non-GBR | 0.5 | 60 ms | MC-PTT Signaling |
| 70 | Non-GBR | 5.5 | 200 ms | MC Data |
| 75 | GBR | 2.5 | 50 ms | V2X Messages |
| 79 | Non-GBR | 6.5 | 50 ms | V2X Messages |
| 82 | GBR | 1.9 | 10 ms | Discrete Automation |
| 83 | GBR | 2.2 | 10 ms | Discrete Automation |



### ARP (Allocation and Retention Priority)

ARP determines whether a bearer can be established during congestion:

| Parameter | Values | Purpose |
|-----------|--------|---------|
| Priority Level | 1-15 (1=highest) | Relative importance |
| Pre-emption Capability | Yes/No | Can this bearer preempt others? |
| Pre-emption Vulnerability | Yes/No | Can this bearer be preempted? |

**Example Scenario:**
```
Congestion situation - resources are full:

Bearer A: Priority=5, Pre-emption Capability=Yes
Bearer B: Priority=10, Pre-emption Vulnerability=Yes

→ Bearer A can preempt (remove) Bearer B to get resources
→ Emergency calls (Priority=1) can preempt regular calls
```

### GBR vs Non-GBR Bearers

```
┌─────────────────────────────────────────────────────┐
│              GBR BEARER                              │
│  ┌───────────────────────────────────┐              │
│  │  Guaranteed Bit Rate (GBR)        │              │
│  │  ═══════════════════ ← guaranteed │              │
│  │  Maximum Bit Rate (MBR)           │              │
│  │  ─────────────────── ← cap        │              │
│  │                                   │              │
│  │  Resources RESERVED at admission  │              │
│  │  Used for: Voice, Video calls     │              │
│  └───────────────────────────────────┘              │
├─────────────────────────────────────────────────────┤
│              NON-GBR BEARER                         │
│  ┌───────────────────────────────────┐              │
│  │  No guaranteed rate               │              │
│  │  ~~~~~~~~~~~~~~~~~~~~~~~~ ← best  │              │
│  │                            effort │              │
│  │  Limited by APN-AMBR / UE-AMBR    │              │
│  │                                   │              │
│  │  No resource reservation          │              │
│  │  Used for: Web, Email, Apps       │              │
│  └───────────────────────────────────┘              │
└─────────────────────────────────────────────────────┘
```

### LTE QoS Limitations

1. **Coarse granularity**: One QCI per bearer → one bearer per service type
2. **Bearer overhead**: Each new QCI requires a new GTP tunnel
3. **Signaling heavy**: Dedicated bearer setup requires full NAS/S1AP signaling
4. **Inflexible**: Cannot mix traffic with different QoS in one bearer
5. **Limited QCIs**: Only 9 standard values (extended to ~20 with MCX)

---

## 4. 5G Flow-Based QoS

### Key Concept: QoS Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                   5G QoS ARCHITECTURE                            │
│                                                                 │
│  ┌──────── PDU Session ────────────────────────────────┐        │
│  │                                                     │        │
│  │  ┌─── QoS Flow 1 (QFI=1, 5QI=1) ──── Voice ─┐    │        │
│  │  │  SDF: src=UE, dst=IMS, proto=RTP           │    │        │
│  │  └────────────────────────────────────────────┘    │        │
│  │                                                     │        │
│  │  ┌─── QoS Flow 2 (QFI=2, 5QI=2) ──── Video ─┐    │        │
│  │  │  SDF: src=UE, dst=IMS, proto=RTP/video     │    │        │
│  │  └────────────────────────────────────────────┘    │        │
│  │                                                     │        │
│  │  ┌─── QoS Flow 3 (QFI=3, 5QI=9) ──── Web ───┐    │        │
│  │  │  SDF: src=UE, dst=*, proto=TCP             │    │        │
│  │  └────────────────────────────────────────────┘    │        │
│  │                                                     │        │
│  │  All flows share ONE N3/N9 GTP-U tunnel            │        │
│  │  (QFI marked in GTP-U extension header)            │        │
│  └─────────────────────────────────────────────────────┘        │
│                                                                 │
│  UE ──── gNB ──── UPF ──── DN                                  │
│      N3(GTP-U)  N9(GTP-U)                                      │
│      QFI in hdr  QFI in hdr                                    │
└─────────────────────────────────────────────────────────────────┘
```

**Key differences from LTE:**
- Multiple QoS Flows share ONE PDU session (one GTP tunnel)
- QFI (6-bit) identifies the flow within the tunnel
- No need to create/delete bearers → less signaling
- Finer granularity with less overhead

### 5QI Table (Key Standardized Values)

| 5QI | Resource Type | Priority | Packet Delay | Packet Error Rate | Example Services |
|-----|:------------:|:--------:|:------------:|:-----------------:|-----------------|
| 1 | GBR | 20 | 100 ms | 10⁻² | Conversational Voice |
| 2 | GBR | 40 | 150 ms | 10⁻³ | Conversational Video |
| 3 | GBR | 30 | 50 ms | 10⁻³ | Real-Time Gaming |
| 4 | GBR | 50 | 300 ms | 10⁻⁶ | Non-Conv. Video (Buffered) |
| 5 | Non-GBR | 10 | 100 ms | 10⁻⁶ | IMS Signaling |
| 6 | Non-GBR | 60 | 300 ms | 10⁻⁶ | Video, TCP (default) |
| 7 | Non-GBR | 70 | 100 ms | 10⁻³ | Voice, Video, Gaming |
| 8 | Non-GBR | 80 | 300 ms | 10⁻⁶ | Video, TCP web |
| 9 | Non-GBR | 90 | 300 ms | 10⁻⁶ | Video, TCP, Email (default) |
| 65 | GBR | 7 | 75 ms | 10⁻² | MC-PTT Voice |
| 66 | GBR | 20 | 100 ms | 10⁻² | Non-MC-PTT Voice |
| 67 | GBR | 15 | 100 ms | 10⁻³ | MC Video |
| 69 | Non-GBR | 5 | 60 ms | 10⁻⁶ | MC-PTT Signaling |
| 70 | Non-GBR | 55 | 200 ms | 10⁻⁶ | MC Data |
| 79 | Non-GBR | 65 | 50 ms | 10⁻² | V2X Messages |
| 80 | Non-GBR | 68 | 10 ms | 10⁻⁶ | Low-Latency eMBB |
| 82 | DC-GBR | 19 | 10 ms | 10⁻⁴ | Discrete Automation (small) |
| 83 | DC-GBR | 22 | 10 ms | 10⁻⁴ | Discrete Automation (large) |
| 84 | DC-GBR | 24 | 30 ms | 10⁻⁵ | Intelligent Transport |
| 85 | DC-GBR | 21 | 5 ms | 10⁻⁵ | Electricity Distribution |
| 86 | GBR | 18 | 5 ms | 10⁻⁴ | V2X (platooning) |
| 87 | DC-GBR | 25 | 5 ms | 10⁻³ | Interactive Service (motion) |
| 88 | DC-GBR | 23 | 10 ms | 10⁻⁴ | Interactive Service (steering)|
| 89 | DC-GBR | 26 | 15 ms | 10⁻⁴ | Visual Content (cloud gaming) |
| 90 | DC-GBR | 27 | 20 ms | 10⁻⁴ | Visual Content (AR/cloud XR) |

> **DC-GBR** = Delay-Critical GBR (new in 5G)

### QoS Rules

QoS rules tell the UE how to classify uplink traffic:

```
QoS Rule Structure:
┌────────────────────────────────────┐
│  Rule ID                           │
│  Rule Precedence (priority)        │
│  QFI (target QoS flow)            │
│  Packet Filter(s):                │
│    - Source IP/Port               │
│    - Destination IP/Port          │
│    - Protocol                     │
│    - Type of Service              │
│    - Flow Label                   │
└────────────────────────────────────┘

Example:
  Rule 1: dst=10.0.0.5, proto=RTP, port=5060 → QFI=1 (voice)
  Rule 2: dst=10.0.0.5, proto=RTP, port=8000 → QFI=2 (video)
  Rule 3: default (match all)                 → QFI=3 (best effort)
```

### Reflective QoS

```
┌──────────────────────────────────────────────────────────┐
│              REFLECTIVE QoS MECHANISM                      │
│                                                          │
│  1. Network sends DL packet with:                        │
│     - QFI = X                                            │
│     - RQI = 1 (Reflective QoS Indication)               │
│                                                          │
│  2. UE observes DL packet headers:                       │
│     - Source IP/Port → becomes UL Destination            │
│     - Destination IP/Port → becomes UL Source            │
│     - QFI = X → UE uses same QFI for reply              │
│                                                          │
│  3. UE creates "derived QoS rule" automatically:         │
│     - No explicit QoS rule signaling needed!             │
│     - Timer-based: rule valid for RQ Timer duration      │
│                                                          │
│  Benefit: Reduces signaling for establishing QoS rules   │
│  Use case: When UE doesn't know server IP in advance     │
└──────────────────────────────────────────────────────────┘
```



---

## 5. QoS Mapping and Enforcement

### End-to-End QoS Mapping Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    5G QoS MAPPING (End-to-End)                           │
│                                                                         │
│  APPLICATION LAYER                                                      │
│  ┌──────────────────────────────────────────────────┐                   │
│  │  IP Packets (SDF = Service Data Flow)            │                   │
│  │  Voice pkt | Video pkt | Web pkt | IoT pkt      │                   │
│  └──────┬───────────┬──────────┬──────────┬─────────┘                   │
│         │           │          │          │                             │
│  QoS CLASSIFICATION (SDF → QoS Flow)                                   │
│  ┌──────▼───────────▼──────────▼──────────▼─────────┐                   │
│  │  QoS Rules / SDF Templates                       │                   │
│  │  Match: 5-tuple (src/dst IP, src/dst port, proto)│                   │
│  │  Action: Assign QFI                              │                   │
│  └──────┬───────────┬──────────┬──────────┬─────────┘                   │
│         │QFI=1      │QFI=2     │QFI=3     │QFI=4                        │
│         ▼           ▼          ▼          ▼                             │
│  QoS FLOW LAYER                                                         │
│  ┌─────────────┬────────────┬───────────┬───────────┐                   │
│  │ Flow(5QI=1) │Flow(5QI=2) │Flow(5QI=9)│Flow(5QI=82)│                  │
│  │ Voice       │ Video      │ Web       │ URLLC     │                   │
│  └──────┬──────┴─────┬──────┴─────┬─────┴─────┬─────┘                   │
│         │            │            │            │                         │
│  DRB MAPPING (gNB decides)                                              │
│  ┌──────▼────────────▼────────────▼────────────▼─────┐                   │
│  │  DRB 1 (GBR)  │  DRB 2 (GBR)  │  DRB 3 (Non-GBR)│                   │
│  │  QFI=1         │  QFI=2         │  QFI=3, QFI=4   │                   │
│  └────────────────┴────────────────┴─────────────────┘                   │
│                                                                         │
│  NOTE: Multiple Non-GBR flows CAN share one DRB                         │
│        GBR flows typically get dedicated DRBs                            │
└─────────────────────────────────────────────────────────────────────────┘
```

### Role of Each Network Element

| Element | QoS Role | Actions |
|---------|----------|---------|
| **PCF** | Policy Decision | Creates PCC rules, determines 5QI/ARP, sends to SMF |
| **SMF** | QoS Control | Derives QoS profiles, installs QoS rules, configures UPF |
| **UPF** | DL Enforcement | SDF detection, QFI marking, gating, policing, rate limiting |
| **gNB** | Radio Scheduling | Maps QoS flows→DRBs, schedules based on 5QI, enforces AMBR |
| **UE** | UL Classification | Applies QoS rules, maps UL packets to QoS flows |

### QoS Enforcement Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│              QoS POLICY & ENFORCEMENT ARCHITECTURE                │
│                                                                  │
│  ┌─────┐  PCC Rules   ┌─────┐  QoS Profile  ┌─────┐           │
│  │ PCF │──────────────►│ SMF │───────────────►│ gNB │           │
│  │     │               │     │               │     │           │
│  └──┬──┘               └──┬──┘               └─────┘           │
│     │                     │                                      │
│     │  Policy decision    │  PDR/FAR/QER                        │
│     │  (from AF/NEF)      │  (PFCP rules)                      │
│     │                     ▼                                      │
│     │               ┌─────────┐                                 │
│     │               │   UPF   │  Enforces:                      │
│     │               │         │  - SDF detection                │
│     │               │         │  - QFI marking                  │
│     │               │         │  - Gate control (open/close)    │
│     │               │         │  - Rate policing (GBR/MBR)      │
│     │               │         │  - Traffic measurement          │
│     │               └─────────┘                                 │
│     │                                                            │
│     │  ┌─────┐  Service Request  ┌─────┐                       │
│     └──│ AF  │◄─────────────────│ App │                        │
│        │(IMS)│                   │     │                        │
│        └─────┘                   └─────┘                        │
│                                                                  │
│  Flow: AF requests QoS → PCF creates policy → SMF enforces      │
│        → UPF marks/polices → gNB schedules → UE classifies      │
└──────────────────────────────────────────────────────────────────┘
```

---

## 6. Bearer vs Flow Comparison

### Side-by-Side Architecture

```
┌────────────────────────────────┬────────────────────────────────────┐
│         LTE (Bearer-Based)     │         5G (Flow-Based)            │
├────────────────────────────────┼────────────────────────────────────┤
│                                │                                    │
│  SDF (IP flow)                 │  SDF (IP flow)                     │
│      │                         │      │                             │
│      ▼                         │      ▼                             │
│  TFT (Traffic Flow Template)   │  QoS Rule (packet filters)        │
│      │                         │      │                             │
│      ▼                         │      ▼                             │
│  EPS Bearer                    │  QoS Flow (QFI)                    │
│  (1 QCI per bearer)           │  (1 5QI per flow)                  │
│      │                         │      │                             │
│      ▼                         │      ▼                             │
│  GTP-U Tunnel                  │  PDU Session                       │
│  (1 tunnel per bearer)        │  (1 tunnel, many flows)            │
│      │                         │      │                             │
│      ▼                         │      ▼                             │
│  Radio Bearer (DRB)            │  DRB (may carry multiple flows)    │
│  (1:1 with EPS bearer)        │  (N:1 mapping possible)            │
│                                │                                    │
├────────────────────────────────┼────────────────────────────────────┤
│  Mapping: 1:1:1:1              │  Mapping: N:M (flexible)           │
│  (SDF→Bearer→Tunnel→DRB)     │  (SDFs→Flows→Session→DRBs)        │
└────────────────────────────────┴────────────────────────────────────┘
```

### Detailed Comparison Table

| Aspect | LTE (Bearer-Based) | 5G (Flow-Based) |
|--------|-------------------|-----------------|
| QoS Unit | EPS Bearer | QoS Flow |
| Identifier | EBI (EPS Bearer ID) | QFI (QoS Flow ID, 6-bit) |
| QoS Class | QCI (1-9 + MCX) | 5QI (1-90+) |
| Granularity | Coarse (1 QCI/bearer) | Fine (per-flow) |
| Tunnel | 1 GTP-U per bearer | 1 GTP-U per PDU session |
| Setup | NAS signaling per bearer | QoS rule install (lighter) |
| UL Classification | TFT (uplink packet filter) | QoS Rules |
| DL Classification | TFT at P-GW | SDF template at UPF |
| New QoS Class | New bearer (heavy signaling) | New flow (lightweight) |
| Sharing | No sharing between bearers | Flows share tunnel |
| Reflective QoS | Not supported | Supported (RQI) |
| Resource Types | GBR, Non-GBR | GBR, Non-GBR, DC-GBR |

---

## 7. GBR vs Non-GBR vs Delay-Critical GBR

### Three Resource Types in 5G

```
┌─────────────────────────────────────────────────────────────────────┐
│                    5G QoS RESOURCE TYPES                             │
│                                                                     │
│  ┌─────────────────────┐  ┌─────────────────────┐                  │
│  │       GBR            │  │     Non-GBR          │                  │
│  │                     │  │                     │                  │
│  │  ▓▓▓▓▓▓▓▓▓▓ GFBR   │  │  No rate guarantee  │                  │
│  │  ░░░░░░░░░░ (up to  │  │  ~~~~~~~~~~~~~~~~   │                  │
│  │             MFBR)   │  │  Limited by AMBR    │                  │
│  │                     │  │                     │                  │
│  │  Resources reserved │  │  Best effort        │                  │
│  │  Admission control  │  │  No admission ctrl  │                  │
│  │                     │  │                     │                  │
│  │  Voice, Video call  │  │  Web, Email, Apps   │                  │
│  └─────────────────────┘  └─────────────────────┘                  │
│                                                                     │
│  ┌─────────────────────────────────────────────┐                    │
│  │       Delay-Critical GBR (DC-GBR)           │                    │
│  │                                             │                    │
│  │  ▓▓▓▓▓▓▓▓▓▓ GFBR + Bounded Delay           │                    │
│  │                                             │                    │
│  │  Additional parameter:                      │                    │
│  │    Maximum Data Burst Volume (MDBV)         │                    │
│  │    → Guarantees delay bound for burst       │                    │
│  │                                             │                    │
│  │  URLLC: Industrial automation, V2X,         │                    │
│  │         Remote surgery, AR/VR               │                    │
│  └─────────────────────────────────────────────┘                    │
└─────────────────────────────────────────────────────────────────────┘
```

### Parameters by Resource Type

| Parameter | GBR | Non-GBR | DC-GBR |
|-----------|:---:|:-------:|:------:|
| 5QI | ✓ | ✓ | ✓ |
| ARP | ✓ | ✓ | ✓ |
| GFBR (Guaranteed Flow Bit Rate) | ✓ | ✗ | ✓ |
| MFBR (Maximum Flow Bit Rate) | ✓ | ✗ | ✓ |
| Session-AMBR | ✗ | ✓ | ✗ |
| Maximum Data Burst Volume | ✗ | ✗ | ✓ |
| Packet Delay Budget | ✓ | ✓ | ✓ (strict) |
| Admission Control | Required | Not needed | Required |
| Resource Reservation | Yes | No | Yes |

### Rate Parameters Explained

```
GFBR = Guaranteed Flow Bit Rate
  → Network MUST provide at least this rate
  → If congestion, GBR flows protected up to GFBR

MFBR = Maximum Flow Bit Rate  
  → Network MAY provide up to this rate
  → Excess above MFBR is dropped/shaped

Example (VoLTE):
  GFBR = 40 kbps (AMR-NB codec rate)
  MFBR = 80 kbps (with overhead margin)

Example (Video Call):
  GFBR = 1 Mbps (minimum acceptable quality)
  MFBR = 4 Mbps (HD quality when resources available)
```

---

## 8. AMBR (Aggregate Maximum Bit Rate)

### AMBR Hierarchy

```
┌─────────────────────────────────────────────────────────────────┐
│                      AMBR HIERARCHY                              │
│                                                                 │
│  ┌───────────────────────────────────────────────────────┐      │
│  │                 UE-AMBR (per UE)                      │      │
│  │  Aggregate cap across ALL Non-GBR QoS flows          │      │
│  │  of all PDU sessions                                  │      │
│  │  Example: UE-AMBR = 100 Mbps DL / 50 Mbps UL        │      │
│  │  (stored in subscription data)                        │      │
│  │                                                       │      │
│  │  ┌─────────────────────┐  ┌─────────────────────┐    │      │
│  │  │ Session-AMBR (Sess1)│  │ Session-AMBR (Sess2)│    │      │
│  │  │ 50 Mbps DL/25 UL   │  │ 30 Mbps DL/15 UL   │    │      │
│  │  │                     │  │                     │    │      │
│  │  │  ┌───┐ ┌───┐ ┌───┐│  │  ┌───┐ ┌───┐       │    │      │
│  │  │  │F1 │ │F2 │ │F3 ││  │  │F4 │ │F5 │       │    │      │
│  │  │  │Non│ │Non│ │GBR││  │  │Non│ │GBR│       │    │      │
│  │  │  │GBR│ │GBR│ │   ││  │  │GBR│ │   │       │    │      │
│  │  │  └───┘ └───┘ └───┘│  │  └───┘ └───┘       │    │      │
│  │  │                     │  │                     │    │      │
│  │  │  Session-AMBR caps  │  │  Session-AMBR caps  │    │      │
│  │  │  F1+F2 (not F3)    │  │  F4 (not F5)        │    │      │
│  │  └─────────────────────┘  └─────────────────────┘    │      │
│  │                                                       │      │
│  │  UE-AMBR caps: F1+F2+F4 (all Non-GBR across sess)   │      │
│  │  GBR flows (F3, F5) are NOT limited by AMBR          │      │
│  └───────────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────────┘
```

### AMBR Enforcement Points

| AMBR Type | Scope | Enforced At | Applies To |
|-----------|-------|-------------|-----------|
| UE-AMBR | All sessions | gNB (RAN) | All Non-GBR flows, all sessions |
| Session-AMBR | Per PDU session | UPF | Non-GBR flows in that session |

**Key Rules:**
- AMBR applies ONLY to Non-GBR flows
- GBR flows are limited by their own MFBR, not AMBR
- UE-AMBR ≥ sum of Session-AMBRs (logically)
- Enforced separately for UL and DL

---

## 9. QoS Enforcement Architecture

### Complete End-to-End QoS Signaling

```
┌─────────────────────────────────────────────────────────────────────────┐
│           COMPLETE QoS ENFORCEMENT FLOW                                  │
│                                                                         │
│  ┌─────┐        ┌─────┐        ┌─────┐        ┌─────┐      ┌─────┐   │
│  │ AF  │        │ PCF │        │ SMF │        │ UPF │      │ gNB │   │
│  │(IMS)│        │     │        │     │        │     │      │     │   │
│  └──┬──┘        └──┬──┘        └──┬──┘        └──┬──┘      └──┬──┘   │
│     │               │              │              │             │       │
│     │ 1. Service    │              │              │             │       │
│     │    Request    │              │              │             │       │
│     │──────────────►│              │              │             │       │
│     │  (via NEF)    │              │              │             │       │
│     │               │ 2. PCC Rule  │              │             │       │
│     │               │    (5QI,ARP, │              │             │       │
│     │               │    filters)  │              │             │       │
│     │               │─────────────►│              │             │       │
│     │               │              │              │             │       │
│     │               │              │ 3. N4 Rules  │             │       │
│     │               │              │   (PDR/QER)  │             │       │
│     │               │              │─────────────►│             │       │
│     │               │              │              │             │       │
│     │               │              │ 4. QoS Profile             │       │
│     │               │              │   (5QI, rates)             │       │
│     │               │              │───────────────────────────►│       │
│     │               │              │              │             │       │
│     │               │              │ 5. QoS Rules │             │       │
│     │               │              │   to UE (NAS)│             │       │
│     │               │              │─────────────────────────────►UE   │
│     │               │              │              │             │       │
│  ═══╪═══════════════╪══════════════╪══════════════╪═════════════╪═══   │
│     │           ENFORCEMENT PHASE                  │             │       │
│     │               │              │              │             │       │
│     │               │              │         UL:  │◄────────────│       │
│     │               │              │         UE classifies      │       │
│     │               │              │         (QoS rules→QFI)    │       │
│     │               │              │              │             │       │
│     │               │              │         DL:  │             │       │
│     │               │              │         UPF detects SDF    │       │
│     │               │              │         marks QFI          │       │
│     │               │              │         polices rate       │       │
│     │               │              │              │────────────►│       │
│     │               │              │              │  gNB maps   │       │
│     │               │              │              │  QFI→DRB    │       │
│     │               │              │              │  schedules  │       │
│     │               │              │              │             │──►UE  │
└─────────────────────────────────────────────────────────────────────────┘
```

### Summary of Enforcement Actions

| Node | Uplink Actions | Downlink Actions |
|------|---------------|-----------------|
| **UE** | Classify packets using QoS rules, mark QFI | Receive and process |
| **gNB** | Map QoS flows→DRBs, enforce UE-AMBR | Schedule based on 5QI priority, map DRB→flows |
| **UPF** | Verify QFI, enforce Session-AMBR | Detect SDF, mark QFI, gate/police/shape |
| **SMF** | Configure UPF (N4), signal gNB (N2), signal UE (N1) | Same |
| **PCF** | Generate PCC rules from policy | Same |

---

## Summary: Key Takeaways

| Concept | Key Point |
|---------|-----------|
| QoS Purpose | Different services need different network treatment |
| QCI (LTE) | 9 standard classes, one per bearer |
| 5QI (5G) | 90+ classes, per QoS flow |
| Bearer (LTE) | 1:1 mapping with QCI, separate GTP tunnel |
| QoS Flow (5G) | Many flows share one PDU session, lighter signaling |
| GBR | Guaranteed rate, admission control, resource reserved |
| Non-GBR | Best effort, limited by AMBR |
| DC-GBR | GBR + bounded delay (MDBV), for URLLC |
| AMBR | Aggregate cap on Non-GBR (UE-level and session-level) |
| Reflective QoS | UE derives UL QoS from DL (reduces signaling) |
| PCF | Policy brain, creates PCC rules |
| UPF | DL classifier and enforcer |
| gNB | Radio scheduler, DRB mapping, UE-AMBR |

---

## QCI vs 5QI Quick Reference

| Feature | QCI (LTE) | 5QI (5G) |
|---------|-----------|----------|
| Values | 1-9 standard + 65-82 MCX | 1-90+ standardized |
| Resource Types | GBR, Non-GBR | GBR, Non-GBR, DC-GBR |
| Priority Scale | 1-9 (1=highest) | 1-127 (1=highest) |
| Associated With | EPS Bearer | QoS Flow |
| Delay Range | 50-300 ms | 5-300 ms |
| New Category | — | Delay-Critical GBR |
| URLLC Support | Limited | Native (5ms delay, 10⁻⁵ loss) |

---

*Module 22 — Quality of Service (QoS) — Advanced Communication Networks*
