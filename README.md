<p align="center">
  <img src="https://img.shields.io/badge/Threat-CRITICAL-red?style=for-the-badge" alt="Threat Level: Critical"/>
  <img src="https://img.shields.io/badge/Family-Qilin%20(Agenda)-darkred?style=for-the-badge" alt="Family: Qilin"/>
  <img src="https://img.shields.io/badge/Language-Rust-orange?style=for-the-badge&logo=rust" alt="Language: Rust"/>
  <img src="https://img.shields.io/badge/Model-RaaS-purple?style=for-the-badge" alt="Model: RaaS"/>
  <img src="https://img.shields.io/badge/Analysis-Static%20Only-blue?style=for-the-badge" alt="Analysis: Static Only"/>
</p>

<h1 align="center">Qilin Ransomware — Complete Threat Analysis Report</h1>

<p align="center">
  <strong>In-depth static analysis and reverse engineering of the Qilin (Agenda) Ransomware-as-a-Service operator build</strong>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Tool-IDA%20Pro%209.2-informational?style=flat-square&logo=hexrays" alt="IDA Pro"/>
  <img src="https://img.shields.io/badge/Tool-Hex--Rays%20Decompiler-informational?style=flat-square" alt="Hex-Rays"/>
  <img src="https://img.shields.io/badge/Platform-Fedora%2044%20x86__64-informational?style=flat-square&logo=fedora" alt="Platform"/>
  <img src="https://img.shields.io/badge/MITRE%20ATT%26CK-22%20Techniques-yellow?style=flat-square" alt="MITRE ATT&CK"/>
  <img src="https://img.shields.io/github/license/KONDORDEVSECURITYCORP/Qilin-Ransomware-Threat-Analysis?style=flat-square" alt="License"/>
  <img src="https://img.shields.io/github/stars/KONDORDEVSECURITYCORP/Qilin-Ransomware-Threat-Analysis?style=flat-square" alt="Stars"/>
</p>

---

## Overview

This repository contains a **comprehensive static analysis report** of an operational Qilin ransomware sample (`Target.exe`), a Rust-compiled enterprise encryptor belonging to the **Qilin/Agenda Ransomware-as-a-Service (RaaS)** operation.

The analysis was performed using **IDA Pro 9.2 with Hex-Rays Decompiler** on an isolated Fedora 44 x86_64 environment. **No malware was executed at any point** — this is purely static/reverse engineering research for defensive purposes.

---

## Key Findings

| Attribute | Detail |
|-----------|--------|
| **Family** | Qilin (formerly "Agenda") |
| **Variant** | v3+ (Rust rewrite, operator build) |
| **Compilation** | Rust → i686-pc-windows-gnu (MinGW cross-compiled) |
| **Encryption** | RSA-4096 + AES-CTR (with AES-NI) / ChaCha20 (fallback) |
| **Propagation** | PsExec (embedded) + PowerShell (vCenter/ESXi) |
| **Evasion** | Safe Mode boot, sandbox detection, event log wiping |
| **Anti-Forensics** | Free space zeroing, self-deletion, log purging |
| **MITRE ATT&CK** | 22 techniques across 8 tactics mapped |

---

## Report Contents

The full analysis report ([`ANALISIS_RANSOMWARE_QILIN.md`](./ANALISIS_RANSOMWARE_QILIN.md)) covers:

1. **Executive Summary** — High-level threat assessment
2. **Binary Information** — PE headers, sections, hashes, compilation metadata
3. **Attribution & Family** — Qilin/Agenda group identification and history
4. **Embedded Configuration** — Campaign IDs, RSA public key, encryption parameters
5. **Malware Architecture** — Full Rust source tree reconstruction, crate dependencies
6. **Technical Capabilities** (14 subsections):
   - Hybrid encryption (RSA-4096 + AES-CTR/ChaCha20)
   - Privilege escalation to SYSTEM
   - Lateral movement (PsExec + vCenter/ESXi)
   - Shadow copy elimination
   - Safe Mode persistence
   - Process/service termination (65+ processes, 50+ services)
   - File unlocker (Restart Manager API)
   - Network enumeration & share discovery
   - Storage manipulation (offline disks, read-only removal)
   - Registry persistence (all-user autostart)
   - Anti-forensics (event logs, zeroing, self-destruct)
   - Wallpaper/lockscreen intimidation
   - Sandbox detection
   - Mutex-based single execution
7. **Imports by DLL** — Complete API mapping (226 imports)
8. **Indicators of Compromise (IOCs)** — Hashes, infrastructure, artifacts, registry keys
9. **Execution Flow** — Full operational diagram
10. **MITRE ATT&CK Mapping** — 22 techniques with IDs
11. **Detection Recommendations** — YARA rules, network indicators, endpoint telemetry
12. **Conclusions** — Risk assessment and operational status

---

## Indicators of Compromise (IOCs)

### File Hashes

| Algorithm | Hash |
|-----------|------|
| **SHA-256** | `227ecf1a779cf7c19c0db869f66abf4b43d61b3700628d1d58b6eaeafe5496ba` |
| **SHA-1** | `fc3cab5aeea32162a6294cfc87fd16a81c068cfe` |
| **MD5** | `1c3da2e8855da39b0259b0515407b1b9` |

### Network Infrastructure

| Purpose | Domain |
|---------|--------|
| Negotiation panel | `spzq5vlkhohit3gddz2vu4gxw23m7czqx3hqx63jqpprf3cgdly5tkid.onion` |
| Leak blog #1 | `kbsqoivihgdmwczmxkbovk7ss2dcynitwhhfu5yw725dboqo5kthfaad.onion` |
| Leak blog #2 | `ijzn3sicrcy7guixkzjkib4ukbiilwc3xhnmby4mcbccnsd7j2rekvqd.onion` |

### YARA Detection Rule

A complete YARA rule for detecting this Qilin variant is included in the full report.

---

## MITRE ATT&CK Coverage

```
┌─────────────────────┬──────────────────────────────────────────────────┐
│ Tactic              │ Techniques                                       │
├─────────────────────┼──────────────────────────────────────────────────┤
│ Execution           │ T1059.001, T1059.003, T1569.002                  │
│ Persistence         │ T1547.001                                        │
│ Privilege Escalation│ T1134                                            │
│ Defense Evasion     │ T1070.001, T1070.004, T1497, T1562.009           │
│ Credential Access   │ T1555                                            │
│ Discovery           │ T1057, T1007, T1135, T1018, T1082                │
│ Lateral Movement    │ T1021.002, T1210                                 │
│ Impact              │ T1486, T1490, T1489, T1529, T1491.001            │
└─────────────────────┴──────────────────────────────────────────────────┘
```

---

## Usage

This report is intended for:

- **Threat Intelligence Teams** — IOCs, TTPs, and attribution data for threat tracking
- **Incident Responders** — Detection indicators and behavioral patterns for active incidents
- **SOC Analysts** — YARA rules, endpoint telemetry, and network signatures
- **Malware Researchers** — Technical deep-dive into Qilin's Rust architecture
- **Security Engineers** — Defense recommendations and detection engineering

---

## Disclaimer

> **This repository contains NO malware code, binaries, or executable components.**
>
> All content is strictly for **educational, research, and defensive cybersecurity purposes**. The analysis documents threat actor techniques to improve organizational defenses. Use this information responsibly and in accordance with applicable laws and regulations.

---

## Author

**Claudio Cortez** — Security Researcher @ [KONDORDEV SECURITY CORP](https://github.com/KONDORDEVSECURITYCORP)

---

## Related Research

| Repository | Description |
|------------|-------------|
| [Babuk Ransomware Threat Analysis](https://github.com/KONDORDEVSECURITYCORP/Babuk-Ransomware-Threat-Analysis) | Complete analysis of Babuk v1 (ECDH + ChaCha20) |
| [VanHelsing RaaS Panel](https://github.com/KONDORDEVSECURITYCORP/VanHelsing-RaaS-Panel) | Leaked RaaS panel source code analysis |
| [Lumma Stealer Analysis](https://github.com/KONDORDEVSECURITYCORP/lumma-stealer-tactics-impact-and-defense-strategies) | LummaC2 forensic analysis and defense strategies |
| [InfoStealer Threat Intelligence Compendium](https://github.com/KONDORDEVSECURITYCORP/InfoStealer-Threat-Intelligence-Compendium) | 10 critical threats of 2024-2025 documented |

---

<p align="center">
  <sub>If this research helps your security posture, consider giving it a ⭐</sub>
</p>
