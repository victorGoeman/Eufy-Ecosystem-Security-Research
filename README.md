# Eufy Ecosystem: Security Research

## Research Publication
This repository contains the proof-of-concept details and vulnerabilities for the research presented at **USENIX WOOT '24**. 

**Paper Title:** Reverse Engineering the Eufy Ecosystem: A Deep Dive into Security Vulnerabilities and Proprietary Protocols  
**Authors:** Victor Goeman, Dario de Ruck, Tom Cordemans, Jorn Lapon, and Vincent Naessens (DistriNet, KU Leuven)  
**Conference:** 18th USENIX WOOT Conference on Offensive Technologies (WOOT 24)  

📥 **[Read the Full Paper on USENIX](https://www.usenix.org/conference/woot24/presentation/goeman)**

---

## 1. Overview
This research explores the systematic failures in the proprietary cryptographic implementations within the Eufy smart home ecosystem. Our audit revealed that "Security by Obscurity" was heavily relied upon, leading to deterministic key generation based on non-secret hardware identifiers.

### Key Research Areas
* **WPA2-PSK Generation:** Analysis of the low-entropy derivation of network keys for the Homebase 2 hidden Wi-Fi network.
* **Media Encryption:** Reverse engineering the AES key derivation used to protect p2p communication, stored video and images.

---

## 2. Vulnerability Index

| CVE ID | Title | Severity | Impact |
| :--- | :--- | :--- | :--- |
| **[CVE-2023-37822](./CVE-2023-37822.md)** | Low Entropy WPA2-PSK | 8.2 (High) | Proximity-based LAN intrusion (20s crack time). |
| **[CVE-CVE-2024-51346](./CVE-2024-51346.md)** | Deterministic AES Keying | 7.7 (High) | Bulk decryption of stored user media via CPUID. |

---

## 3. Methodology
The analysis was performed using a combination of:
1. **Hardware Hacking:** UART shell acquisition of the Homebase 2 to extract the firmware.
2. **Binary Analysis:** Reverse engineering the `homesecurity` binary using Ghidra.
3. **Traffic Analysis:** Intercepting and decrypting proprietary P2P packets via Wireshark.

---