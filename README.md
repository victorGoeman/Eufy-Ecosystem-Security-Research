# CVE-2024-51346: Cryptographic Bypass and Media Decryption in Eufy Ecosystem

## Vulnerability Metadata

| Field | Details |
| :--- | :--- |
| **Vendor** | Eufy (Anker) |
| **Product** | Homebase 2 |
| **Affected Version** | Up to and including v3.3.4.1h |
| **Component** | P2P Communication Protocol / Encrypted Media |
| **Attack Type** | Network & Physical |
| **CWE ID** | CWE-330: Use of Insufficiently Random Values, CWE-321: Use of Hard-coded Cryptographic Key |
| **CVSS 3.1 Vector** | `CVSS:3.1/AV:L/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:N` |
| **Base Score** | 7.7 (High) |
| **Impact** | Full loss of confidentiality for stored media and live P2P streams. |

---

## 1. Vulnerability Overview
* **Vulnerability Type:** Use of Insufficiently Entropic Keys (CWE-331) / Insufficient Encryption (CWE-311)
* **Severity:** Critical
* **Status:** Confirmed & Patched (Coordinated Disclosure)
* **Affected Components:** P2P Communication Protocol, Media Storage (AES)
* **Impact:** Full loss of confidentiality for images and stored media.

---

## 2. Executive Summary
The encryption mechanism used to secure Peer-to-Peer (P2P) streams and stored media on Eufy devices is cryptographically unsound. The encryption keys are derived entirely from information that is either broadcast over the air (OTA) or stored in plaintext metadata headers. This allows an attacker to passively reconstruct keys and decrypt sensitive user media without authentication. 

---

## 3. Technical Analysis: Key Derivation Failures

The core of the vulnerability lies in the `create_pic_code_v1` algorithm, which relies on predictable, non-secret parameters to generate AES keys.

### 3.1 Information Leakage (The "Eufy Header")
The parameters required to generate encryption keys are exposed in plaintext or easily intercepted:
* **Serial Number:** Broadcast OTA and included in the "Eufy header" of stored images.
* **PPCS_ID:** Intercepted OTA during P2P connection establishment.
* **Random Number:** Included in the image metadata header.

**Header Format:**
`eufysecurity :<serialNumber>: 01 <randomNumber>:`

### 3.2 Vulnerability 1: P2P Stream Encryption
The P2P encryption key is a simple concatenation of the `PPCS_ID` and the `SerialNumber`. Because both are transmitted during the handshake, an eavesdropper can decrypt the stream in real-time.

* **Key Construction:** `Key = PPCS_ID[0:15] + serialNumber[9:15]`

### 3.3 Vulnerability 2: Media (AES) Encryption
Stored media (images/video) suffers from a two-stage failure:
1.  **Partial Encryption:** Only the header of the image file is encrypted. This is insufficient to protect the visual data.
2.  **Low Entropy Brute-Force:** The AES key is derived from the Serial Number and only **4 characters** of the `PPCS_ID` (treated as an integer). Making it computationally feasible to brute-force the key even if the full `PPCS_ID` is not captured.

---

## 4. Reverse Engineered Algorithm
The following logic defines the `create_pic_code_v1` key generation process:

```python
# Pseudocode for Vulnerable Key Generation logic
def GET_HOMEBASE_CODE(serial, PPCS_ID):
    # Extracts a predictable suffix from the ID
    sfx = get_PPCS_Suffix(PPCS_ID)
    # Concatenates serial substring with the suffix
    baseCode = serial[0:l] + str(sfx)
    return baseCode

def GET_RAND_SEED(PPCS_ID):
    sfx = get_PPCS_Suffix(PPCS_ID)
    # The 'random' seed is derived from a weak random string and the ID
    rndStr = "01" + str(random()) + str(1000 - sfx)
    seed = Obfuscate1(MD5(rndStr))
    return seed

def CREATE_IMAGE_KEY(baseCode, seed):
    # Final Key Derivation using SHA256
    h = SHA256("01" + baseCode + seed)
    encKey = Obfuscate2(h)
    return encKey
```

**Note:** Video stream uses a similar algorithm, this algorithm is for image encryption.

# 5. Proof of Concept (Steps to Reproduce)
1.  **Capture Traffic**: Monitor the local network or intercept the image file transfer to extract the eufysecurity header.

2.  **Identify Parameters**: Parse the Serial Number and Random Number from the header strings.

3.  **Key Reconstruction**: 
    * For P2P: Concatenate the known PPCS_ID and Serial Number.

    * For Media: Utilize the reverse-engineered logic to reconstruct the key. If the PPCS_ID is unknown, run a brute-force attack on the 4-character integer suffix.

4.  **Decryption**: Use the reconstructed key to decrypt the P2P stream or the image header, granting access to the raw media.

# 6. Impact
-   **Total Confidentiality Loss**: An attacker with access to intercepted traffic can view private photos/videos.

-   **Passive Exploitation**: No active interaction with the device is required; decryption is possible purely through passive observation.

## Research Publication
This repository contains the proof-of-concept details for the research presented at **USENIX WOOT '24**. 

**Paper Title:** Reverse Engineering the Eufy Ecosystem: A Deep Dive into Security Vulnerabilities and Proprietary Protocols  
**Authors:** Victor Goeman, Dairo de Ruck, Tom Cordemans, Jorn Lapon, and Vincent Naessens, DistriNet, KU Leuven  
**Conference:** 18th USENIX WOOT Conference on Offensive Technoloies (WOOT 24)  

**[Read the Full Paper on USENIX](https://www.usenix.org/conference/woot24/presentation/goeman)**
