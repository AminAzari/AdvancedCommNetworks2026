# Prerequisites Review — Advanced Communication Networks

This document reviews foundational concepts assumed as prior knowledge. Use it as a refresher before diving into the course modules.

---

## 1. OSI Model Review (Telecom Focus)

The OSI 7-layer model provides the reference framework. In mobile/telecom networks, the most relevant layers are:

| Layer | Name | Telecom Relevance |
|-------|------|-------------------|
| 1 | **Physical** | RF transmission, modulation, antenna design, channel coding |
| 2 | **Data Link** | MAC scheduling, ARQ/HARQ, radio resource management |
| 3 | **Network** | IP routing, GTP tunneling, mobility management |
| 4 | **Transport** | TCP/UDP for end-to-end delivery, SCTP in signaling |
| 7 | **Application** | IMS/SIP for voice, HTTP/2 in 5G SBA interfaces |

**Key points to remember:**
- Layers 1–2 are where most wireless-specific innovation happens (PHY/MAC).
- The "user plane" carries data (Layer 3+), while the "control plane" manages signaling.
- In 5G, the protocol stack is split: CU (PDCP and above) vs. DU (RLC/MAC) vs. RU (PHY).
- GTP-U tunnels encapsulate user IP packets between base station and core.

---

## 2. Basic RF Concepts

### Frequency & Bandwidth
- **Frequency (f):** Number of oscillations per second (Hz). Mobile bands range from 700 MHz to 100 GHz.
- **Wavelength:** λ = c / f. Higher frequency → shorter wavelength → smaller antennas but higher path loss.
- **Bandwidth (BW):** Range of frequencies occupied by a signal. More bandwidth → higher data rate (Shannon).

### Modulation
- **Purpose:** Map digital bits onto analog carrier signals.
- **Common schemes:** BPSK, QPSK, 16-QAM, 64-QAM, 256-QAM.
- Higher-order modulation = more bits/symbol but requires better SNR.

### Signal-to-Noise Ratio (SNR)
- SNR = P_signal / P_noise (linear) or 10·log₁₀(P_signal/P_noise) in dB.
- Shannon capacity: C = BW · log₂(1 + SNR) [bits/s].
- Determines which modulation and coding scheme (MCS) can be used.

### Quick Reference
```
QPSK  → 2 bits/symbol → robust, used at cell edge
64-QAM → 6 bits/symbol → high throughput, needs high SNR
256-QAM → 8 bits/symbol → peak rates, very close to base station
```

---

## 3. Digital Communications Basics

### Bit Error Rate (BER)
- Probability that a received bit differs from the transmitted bit.
- Depends on SNR, modulation order, and coding.
- Typical targets: 10⁻³ (voice), 10⁻⁶ (data before ARQ).

### Channel Coding
- **Purpose:** Add redundancy to detect/correct errors.
- **Turbo codes:** Used in 3G/4G for data channels.
- **LDPC codes:** Used in 5G NR for data (eMBB).
- **Polar codes:** Used in 5G NR for control channels.
- **Code rate (R):** Ratio of information bits to total bits. R = k/n (e.g., R = 1/2 means 50% redundancy).

### OFDM Concept
- **Problem:** Wideband channels suffer frequency-selective fading.
- **Solution:** Divide wideband channel into many narrow subcarriers, each experiencing flat fading.
- **Key parameters:**
  - Subcarrier spacing (Δf): 15 kHz (LTE), 15/30/60/120/240 kHz (5G NR).
  - FFT size: determines number of subcarriers.
  - Cyclic prefix (CP): guard interval to combat inter-symbol interference (ISI).
- **Advantages:** Simple equalization (1-tap per subcarrier), flexible resource allocation.
- **OFDMA:** Multiple users share subcarriers → basis of LTE & 5G downlink.

---

## 4. IP Networking Fundamentals

### Addressing
- **IPv4:** 32-bit addresses (e.g., 192.168.1.1), NAT common in mobile networks.
- **IPv6:** 128-bit addresses, increasingly used in 5G (avoids NAT).
- **Subnetting:** CIDR notation (e.g., 10.0.0.0/24 = 256 addresses).

### Routing
- Mobile core networks use IP routing between network functions.
- **Static routes** for simple topologies; **BGP/OSPF** in carrier networks.
- **GTP tunneling:** Encapsulates subscriber IP packets inside GTP/UDP/IP between gNB and UPF.

### Transport Protocols
| Protocol | Properties | Use in Telecom |
|----------|-----------|----------------|
| **TCP** | Reliable, ordered, congestion-controlled | HTTP/2 (5G SBA), web traffic |
| **UDP** | Unreliable, low-latency | VoIP/RTP, GTP-U user plane |
| **SCTP** | Multi-homing, message-oriented | S1AP/NGAP signaling (4G/5G) |

### DNS & NAT
- UEs obtain IP via PDN/PDU session establishment (not DHCP directly).
- DNS resolution for service discovery in 5G SBA (NRF).

---

## 5. Basic Probability

### Why Probability in Telecom?
- Random access (RACH): collision probability with multiple users.
- Scheduling: modeling traffic arrivals (Poisson process).
- Fading: Rayleigh/Rician distributions model multipath.
- BER analysis: Q-function, error probability calculations.

### Key Concepts
- **Random variable (X):** Outcome of a random experiment.
- **PDF / PMF:** Probability density/mass function.
- **Expected value:** E[X] = Σ xᵢ · P(xᵢ) — average outcome.
- **Variance:** Var(X) = E[(X - μ)²] — spread around the mean.

### Distributions You'll Encounter
| Distribution | Use Case |
|-------------|----------|
| **Uniform** | Random backoff timers |
| **Exponential** | Inter-arrival times (memoryless) |
| **Poisson** | Number of arrivals in a time window |
| **Rayleigh** | Envelope of multipath fading (NLOS) |
| **Gaussian (Normal)** | Noise modeling (AWGN) |

### Useful Formula
- Collision probability in slotted ALOHA with n users, 1 slot: P(collision) = 1 - (1 - 1/K)^n ≈ 1 - e^(-n/K) for large K.

---

## 6. Decibel Math

### Why Decibels?
Wireless signals span enormous dynamic ranges (pW to W). Decibels compress this into manageable numbers and turn multiplications into additions.

### Definitions

| Unit | Formula | Reference |
|------|---------|-----------|
| **dB** | 10·log₁₀(P₁/P₂) | Ratio (dimensionless) |
| **dBm** | 10·log₁₀(P / 1 mW) | Absolute power ref. to 1 mW |
| **dBW** | 10·log₁₀(P / 1 W) | Absolute power ref. to 1 W |

### Common Conversions

| Linear | dB |
|--------|-----|
| ×2 | +3 dB |
| ×10 | +10 dB |
| ×100 | +20 dB |
| ×1000 | +30 dB |
| ×0.5 | −3 dB |
| ×0.1 | −10 dB |

### Quick Reference Table

| Power | dBm |
|-------|------|
| 1 mW | 0 dBm |
| 10 mW | 10 dBm |
| 100 mW | 20 dBm |
| 1 W | 30 dBm |
| 20 W (macro BS) | 43 dBm |

### Link Budget (Preview)
```
Received Power (dBm) = Tx Power (dBm) + Tx Gain (dBi) - Path Loss (dB) + Rx Gain (dBi)
```

### Practice Problems
1. A transmitter outputs 2 W. Express in dBm. → 10·log₁₀(2000) = 33 dBm.
2. Path loss is 130 dB, Tx = 43 dBm, antenna gains total 18 dBi. Rx power? → 43 + 18 − 130 = −69 dBm.
3. SNR = 20 dB. What is the linear ratio? → 10^(20/10) = 100.

---

## Summary Checklist

Before starting Module 2, verify you can:

- [ ] Identify which OSI layers handle PHY, MAC, and IP functions
- [ ] Convert between linear power and dBm
- [ ] Explain why OFDM combats frequency-selective fading
- [ ] Calculate Shannon capacity given BW and SNR
- [ ] Describe the difference between TCP and UDP and where each is used
- [ ] Compute simple collision probabilities for random access

If any topic feels unfamiliar, review the referenced sections or consult the recommended textbooks in the syllabus.

---

*Last updated: August 2026*
