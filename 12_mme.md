# Module 12: Mobility Management Entity (MME)

## 1. Why MME Exists

The MME is the **central control-plane entity** in the LTE/EPC architecture. It exists to:

- **Offload the RAN from core signaling**: In 2G/3G, the RNC/BSC handled significant core-network signaling. LTE's flat architecture moved all NAS (Non-Access Stratum) signaling termination to the MME, simplifying eNodeB (eNB) design.
- **Centralize control-plane intelligence**: Authentication, mobility management, bearer setup, and paging are all orchestrated by the MME.
- **Decouple control and user planes**: The MME handles no user-plane traffic — it purely manages signaling, enabling independent scaling of CP and UP.

### Key Interfaces

| Interface | Peer Node | Protocol | Purpose |
|-----------|-----------|----------|---------|
| S1-MME | eNB | S1-AP (SCTP) | NAS transport, handover signaling |
| S6a | HSS | Diameter | Authentication vectors, subscription data |
| S11 | S-GW | GTPv2-C | Bearer/session management |
| S10 | Other MME | GTPv2-C | Inter-MME handover, context transfer |
| S3 | SGSN | GTPv2-C | 2G/3G ↔ LTE mobility |

---

## 2. MME Functions

### 2.1 NAS Signaling Termination
- The MME is the NAS endpoint — it terminates all NAS messages from the UE (transported transparently through eNB via S1-AP).
- NAS messages are integrity-protected and ciphered between UE and MME.

### 2.2 Mobility Management
- Tracking Area (TA) management and TA List assignment
- Handover signaling (S1-based and X2 with MME involvement)
- Inter-RAT mobility (to/from 2G/3G via S3/S10)

### 2.3 Authentication and Security
- Retrieves authentication vectors from HSS (via S6a)
- Runs EPS-AKA (Authentication and Key Agreement)
- Derives NAS keys (K_NASint, K_NASenc) and AS keys (K_eNB)
- Initiates NAS Security Mode Command

### 2.4 Bearer Management
- Establishes default EPS bearers during attach
- Coordinates dedicated bearer activation/modification/deactivation
- Interacts with S-GW (S11) and indirectly P-GW

### 2.5 Paging
- Pages UEs in ECM-IDLE state when downlink data arrives
- Sends S1-AP Paging messages to all eNBs in the UE's registered TA list

### 2.6 Idle-Mode Reachability
- Maintains UE context even when UE is IDLE (ECM-IDLE)
- Knows which Tracking Areas the UE may be in
- Triggers paging when data or signaling arrives

### 2.7 MME Selection, Pooling, and Rebalancing
- Multiple MMEs serve the same area (MME Pool)
- Load balancing via relative capacity (weight factor) advertised to eNBs
- Rebalancing: graceful transfer of UE contexts when adding/removing MMEs

---

## 3. NAS Protocol Stack

NAS (Non-Access Stratum) operates between UE and MME, transparent to eNB.

### 3.1 EMM — EPS Mobility Management

Manages UE registration and mobility:
- Attach / Detach
- Tracking Area Update (TAU)
- Authentication (EPS-AKA)
- Security Mode Control
- GUTI Reallocation
- Service Request

**EMM States:**
- EMM-DEREGISTERED: UE not registered
- EMM-REGISTERED: UE attached to network

### 3.2 ESM — EPS Session Management

Manages PDN connections and EPS bearers:
- PDN Connectivity Request/Accept
- Bearer Resource Allocation
- Bearer Modify / Deactivate
- PDN Disconnect

**ESM operates piggybacked on EMM during attach** (ESM message embedded in Attach Request).

---

## 4. Detailed Procedures

### 4.1 LTE Initial Attach (Full Flow)

```mermaid
sequenceDiagram
    participant UE
    participant eNB
    participant MME
    participant HSS
    participant SGW as S-GW
    participant PGW as P-GW

    Note over UE,PGW: Initial Attach Procedure (~15 steps)
    
    UE->>eNB: 1. RRC Connection Request
    eNB->>UE: 2. RRC Connection Setup
    UE->>eNB: 3. RRC Connection Setup Complete<br/>(contains NAS: Attach Request + PDN Connectivity Request)
    eNB->>MME: 4. S1-AP: Initial UE Message<br/>(NAS: Attach Request + ESM: PDN Connectivity Req)
    
    Note over MME,HSS: Authentication Phase
    MME->>HSS: 5. S6a: Authentication-Information-Request (AIR)
    HSS->>MME: 6. S6a: Authentication-Information-Answer (AIA)<br/>(RAND, AUTN, XRES, K_ASME)
    MME->>UE: 7. NAS: Authentication Request (RAND, AUTN)
    UE->>MME: 8. NAS: Authentication Response (RES)
    
    Note over MME,UE: Security Activation
    MME->>UE: 9. NAS: Security Mode Command<br/>(selected algorithms, NAS-MAC)
    UE->>MME: 10. NAS: Security Mode Complete (ciphered+integrity)
    
    Note over MME,HSS: Location Update
    MME->>HSS: 11. S6a: Update-Location-Request (ULR)
    HSS->>MME: 12. S6a: Update-Location-Answer (ULA)<br/>(subscription data, APN config)
    
    Note over MME,PGW: Session/Bearer Establishment
    MME->>SGW: 13. S11: Create Session Request<br/>(IMSI, bearer QoS, P-GW address)
    SGW->>PGW: 14. S5: Create Session Request
    PGW->>SGW: 15. S5: Create Session Response<br/>(IP addr allocated, bearer context)
    SGW->>MME: 16. S11: Create Session Response<br/>(S-GW F-TEID, bearer context)
    
    Note over MME,UE: Attach Accept
    MME->>eNB: 17. S1-AP: Initial Context Setup Request<br/>(NAS: Attach Accept + Activate Default Bearer Context,<br/>security context, UE capabilities)
    eNB->>UE: 18. RRC Reconfiguration (DRB setup)<br/>+ NAS: Attach Accept
    UE->>eNB: 19. RRC Reconfiguration Complete
    eNB->>MME: 20. S1-AP: Initial Context Setup Response<br/>(eNB F-TEID for S1-U)
    UE->>MME: 21. NAS: Attach Complete<br/>(+ ESM: Activate Default EPS Bearer Context Accept)
    MME->>SGW: 22. S11: Modify Bearer Request (eNB address/TEID)
    SGW->>MME: 23. S11: Modify Bearer Response
```

**Key Points:**
- Steps 7-10: EPS-AKA + NAS security activation
- Steps 13-16: GTP-C session establishment across S11/S5
- The Attach Request carries a piggybacked PDN Connectivity Request (ESM)
- The UE gets its IP address from P-GW (step 15)


### 4.2 Authentication and Security Mode

```mermaid
sequenceDiagram
    participant UE
    participant MME
    participant HSS

    Note over UE,HSS: EPS-AKA Authentication
    MME->>HSS: Authentication-Information-Request<br/>(IMSI, SN-ID, visited PLMN)
    HSS->>MME: Authentication-Information-Answer<br/>(EPS-AV: RAND, AUTN, XRES, K_ASME)
    
    MME->>UE: Authentication Request (RAND, AUTN)
    Note over UE: UE verifies AUTN (mutual auth)<br/>Computes RES, K_ASME
    UE->>MME: Authentication Response (RES)
    Note over MME: Verifies RES == XRES
    
    Note over UE,MME: NAS Security Mode
    Note over MME: Derives K_NASint, K_NASenc<br/>Selects EIA/EEA algorithms
    MME->>UE: Security Mode Command<br/>(replayed UE security capabilities,<br/>selected algorithms, NAS-MAC)
    Note over UE: Verifies NAS-MAC<br/>Derives NAS keys
    UE->>MME: Security Mode Complete<br/>(integrity + ciphered from here on)
    
    Note over UE,MME: All subsequent NAS messages<br/>are integrity-protected and ciphered
```

**Key Derivation Hierarchy:**
```
K (USIM) → CK, IK → K_ASME → K_NASint, K_NASenc, K_eNB → K_UPenc, K_RRCint, K_RRCenc
```

### 4.3 Tracking Area Update (TAU)

**Triggers:**
- **Periodic TAU**: Timer T3412 expires (default ~54 min) — confirms UE is still reachable
- **Mobility-triggered TAU**: UE moves to a cell whose TAC is not in its TA List

```mermaid
sequenceDiagram
    participant UE
    participant eNB
    participant NewMME as New MME
    participant OldMME as Old MME
    participant HSS
    participant SGW as S-GW

    UE->>eNB: RRC: Service Request / TAU trigger
    eNB->>NewMME: S1-AP: Initial UE Message<br/>(NAS: TAU Request, old GUTI)
    
    alt Inter-MME TAU (UE moved to new MME pool area)
        NewMME->>OldMME: S10: Context Request (old GUTI)
        OldMME->>NewMME: S10: Context Response<br/>(MM context, bearer contexts)
        NewMME->>HSS: S6a: Update Location Request
        HSS->>NewMME: S6a: Update Location Answer
        HSS->>OldMME: S6a: Cancel Location (old MME deregistered)
        NewMME->>SGW: S11: Modify Bearer Request (new MME TEID)
        SGW->>NewMME: S11: Modify Bearer Response
    else Intra-MME TAU (same MME, new TA)
        Note over NewMME: Update TA List locally
    end
    
    NewMME->>UE: NAS: TAU Accept<br/>(new GUTI, new TA List, T3412 value)
    UE->>NewMME: NAS: TAU Complete
```

### 4.4 Detach Procedure

#### UE-Initiated Detach

```mermaid
sequenceDiagram
    participant UE
    participant eNB
    participant MME
    participant HSS
    participant SGW as S-GW
    participant PGW as P-GW

    UE->>MME: NAS: Detach Request<br/>(Detach Type: EPS detach / IMSI detach / combined)
    MME->>SGW: S11: Delete Session Request
    SGW->>PGW: S5: Delete Session Request
    PGW->>SGW: S5: Delete Session Response
    SGW->>MME: S11: Delete Session Response
    MME->>HSS: S6a: Purge-UE-Request (if switch-off)
    HSS->>MME: S6a: Purge-UE-Answer
    MME->>UE: NAS: Detach Accept
    MME->>eNB: S1-AP: UE Context Release Command
    eNB->>UE: RRC Connection Release
    eNB->>MME: S1-AP: UE Context Release Complete
```

#### Network-Initiated Detach

```mermaid
sequenceDiagram
    participant MME
    participant UE
    participant eNB
    participant SGW as S-GW
    participant PGW as P-GW

    Note over MME: Trigger: subscription withdrawal,<br/>O&M intervention, MME overload
    MME->>UE: NAS: Detach Request<br/>(re-attach required / re-attach not required)
    UE->>MME: NAS: Detach Accept
    MME->>SGW: S11: Delete Session Request
    SGW->>PGW: S5: Delete Session Request
    PGW->>SGW: S5: Delete Session Response
    SGW->>MME: S11: Delete Session Response
    MME->>eNB: S1-AP: UE Context Release Command
    eNB->>MME: S1-AP: UE Context Release Complete
```


### 4.5 Paging (S1 Paging Procedure)

```mermaid
sequenceDiagram
    participant Internet
    participant PGW as P-GW
    participant SGW as S-GW
    participant MME
    participant eNB1 as eNB (TA1)
    participant eNB2 as eNB (TA2)
    participant UE

    Note over UE: UE is in ECM-IDLE (RRC released)
    Internet->>PGW: Downlink IP packet for UE
    PGW->>SGW: GTP-U: DL data (S5 tunnel)
    Note over SGW: No S1-U tunnel (UE idle)<br/>Buffer DL packet
    SGW->>MME: S11: Downlink Data Notification
    MME->>SGW: S11: Downlink Data Notification Ack
    
    Note over MME: Page UE in all TAs in TA List
    par Paging in TA1
        MME->>eNB1: S1-AP: Paging<br/>(UE identity: S-TMSI, TAI list, DRX)
    and Paging in TA2
        MME->>eNB2: S1-AP: Paging<br/>(UE identity: S-TMSI, TAI list, DRX)
    end
    
    Note over eNB1,eNB2: eNBs transmit paging on PCH<br/>in UE's paging occasion (DRX cycle)
    eNB1->>UE: RRC: Paging (s-TMSI)
    UE->>eNB1: RRC Connection Request (Service Request trigger)
    Note over UE,MME: Service Request procedure follows
```

### 4.6 Service Request (UE in IDLE → CONNECTED)

```mermaid
sequenceDiagram
    participant UE
    participant eNB
    participant MME
    participant SGW as S-GW

    Note over UE: UE in ECM-IDLE has UL data<br/>or was paged
    UE->>eNB: RRC Connection Request
    eNB->>UE: RRC Connection Setup
    UE->>eNB: RRC Connection Setup Complete<br/>(NAS: Service Request, short-MAC-I)
    eNB->>MME: S1-AP: Initial UE Message<br/>(NAS: Service Request)
    
    Note over MME: Verify short-MAC-I<br/>(integrity check with K_NASint)
    
    MME->>eNB: S1-AP: Initial Context Setup Request<br/>(E-RAB setup list, security context, K_eNB)
    eNB->>UE: RRC Connection Reconfiguration<br/>(DRB re-establishment)
    UE->>eNB: RRC Connection Reconfiguration Complete
    eNB->>MME: S1-AP: Initial Context Setup Response<br/>(eNB S1-U TEID/address)
    
    MME->>SGW: S11: Modify Bearer Request<br/>(eNB address, S1-U TEID per bearer)
    SGW->>MME: S11: Modify Bearer Response
    Note over SGW: Flush buffered DL packets<br/>via new S1-U tunnel
    SGW->>eNB: GTP-U: Buffered DL data
    eNB->>UE: DL data on DRB
```

---

## 5. Bearer Management

### 5.1 Default Bearer Setup
- Created automatically during Initial Attach (part of Create Session)
- One default bearer per PDN connection (always-on, non-GBR)
- QCI typically 9 (best-effort Internet) or 5 (IMS signaling)
- Cannot be deactivated without disconnecting the PDN

### 5.2 Dedicated Bearer Activation

```mermaid
sequenceDiagram
    participant PCRF
    participant PGW as P-GW
    participant SGW as S-GW
    participant MME
    participant eNB
    participant UE

    PCRF->>PGW: Gx: RAR (PCC rules, new SDF)
    PGW->>SGW: S5: Create Bearer Request<br/>(TFT, QoS: QCI, GBR/MBR)
    SGW->>MME: S11: Create Bearer Request
    MME->>eNB: S1-AP: E-RAB Setup Request<br/>(NAS: Activate Dedicated EPS Bearer Context Request)
    eNB->>UE: RRC Reconfiguration (new DRB)
    UE->>eNB: RRC Reconfiguration Complete
    eNB->>MME: S1-AP: E-RAB Setup Response
    UE->>MME: NAS: Activate Dedicated EPS Bearer Context Accept
    MME->>SGW: S11: Create Bearer Response
    SGW->>PGW: S5: Create Bearer Response
```

### 5.3 Dedicated Bearer Modification
- Triggered by PCRF policy change (e.g., QoS upgrade during VoLTE call)
- Uses Update Bearer Request/Response across S5 → S11 → S1-AP

### 5.4 Dedicated Bearer Deactivation
- Triggered by PCRF (policy removal), P-GW, or MME
- Uses Delete Bearer Request/Response
- Default bearer deactivation = PDN disconnect (all bearers for that PDN removed)

---

## 6. MME Selection

### 6.1 DNS-Based Selection
- eNB resolves MME FQDN from TAI → DNS returns list of MME IPs
- Format: `tac-XXXX.tac-YYYY.mme.epc.mnc<MNC>.mcc<MCC>.3gppnetwork.org`

### 6.2 Load Balancing
- Each MME advertises a **Relative MME Capacity** (weight, 0-255) via S1 Setup
- eNB distributes new UEs proportionally across pool members
- Higher capacity value = more UEs directed to that MME

### 6.3 GUMMEI — Globally Unique MME Identifier
```
GUMMEI = MCC + MNC + MME Group ID (MMEGI) + MME Code (MMEC)
```
- MMEGI identifies the MME pool
- MMEC identifies a specific MME within the pool
- GUTI = GUMMEI + M-TMSI (temporary UE identity)

---

## 7. S1-Flex and MME Pooling

### Concept
- **S1-Flex**: Each eNB connects to **multiple MMEs** in a pool (not just one)
- **MME Pool Area**: Geographic region served by a set of MMEs
- Benefits:
  - **Redundancy**: If one MME fails, others serve UEs
  - **Load sharing**: New UEs distributed by weight
  - **Geo-redundancy**: Pool members can be in different data centers
  - **Graceful scaling**: Add/remove MMEs without service interruption

### MME Overload and Rebalancing
- MME sends **Overload Start** to eNBs → eNB redirects new connections to other pool members
- **Rebalancing**: MME requests UEs to re-attach to redistribute load (via implicit detach with re-attach indication)

---

## 8. Evolution: MME → AMF (5G)

| Aspect | LTE MME | 5G AMF |
|--------|---------|--------|
| Architecture | Monolithic NE | Microservice (SBI-based) |
| Protocol | GTPv2-C, Diameter | HTTP/2, JSON (SBI), NGAP |
| NAS | EMM + ESM (combined) | 5G-NAS (MM only; SM separated to SMF) |
| Session Mgmt | MME handles bearer signaling | Delegated entirely to SMF |
| Slicing | N/A | Slice-aware, routes to correct SMF per slice |
| Registration | Attach/TAU | Registration (unified) |
| State | Stateful | Designed for stateless (UDSF) |
| Interface to RAN | S1-AP | NGAP (N2) |
| Interface to UDM | Diameter S6a | Nudm (SBI) |

**Key Changes:**
1. **Session management decoupled**: MME handled both mobility + bearers; in 5G, AMF handles only mobility/security, SMF handles sessions
2. **Service-based interface**: AMF exposes Namf services, consumed by other NFs via HTTP/2
3. **Network slicing**: AMF performs slice selection (NSSF interaction) and routes to slice-specific SMF
4. **Stateless design**: UE context can be stored externally (UDSF) for resilience
5. **Unified registration**: No separate Attach vs TAU — single Registration procedure handles both
