# AEGIS-ZERO Risk Register

> **Document Type:** Governance — Operational Risk Register  
> **Framework Alignment:** NIST CSF 2.0 GV.RM · NIST SP 800-53 RA-2/RA-3 · NIST AI RMF GV-1.4  
> **Owner:** AetherHorizon  
> **Version:** 1.0.0  
> **Review Cadence:** Quarterly, or after any HIGH/CRITICAL security event

---

## 1. Risk Tolerance Statement

AEGIS-ZERO operates as a personal security research and portfolio deployment. Risk tolerance
is calibrated accordingly:

| Risk Category | Tolerance | Rationale |
|---|---|---|
| Confidentiality of personal data | **Low** | Personal systems contain sensitive data |
| Integrity of security tooling | **Low** | Compromised tools = blind spots |
| Availability of primary workstation | **Medium** | Acceptable brief downtime for updates |
| Availability of security stack | **Medium** | Short gaps tolerated; long gaps unacceptable |
| AI system misbehavior (Guardian Agent) | **Low** | HITL gate required for all CRITICAL actions |
| Supply chain compromise | **Low** | All dependencies pinned; hashes verified |
| IoT device compromise | **Medium** | VLAN isolation limits blast radius |
| Public exposure of this research | **None** | Built in public intentionally |

**Residual Risk Acceptance:** Risks rated MEDIUM and below with documented mitigations are
accepted. All HIGH and CRITICAL risks require active mitigation or documented escalation.

---

## 2. Risk Register

### Risk Rating Key

| Score | Likelihood | Impact | Rating |
|-------|-----------|--------|--------|
| L × I | 1-Low 2-Med 3-High | 1-Low 2-Med 3-High | 1-3 Low · 4-6 Med · 7-9 High |

---

### R-001 — Nation-State or Advanced Persistent Threat

| Field | Value |
|-------|-------|
| **Risk ID** | R-001 |
| **Category** | External Threat — Advanced |
| **Description** | Sophisticated attacker with zero-day exploits and custom tooling targets the network over an extended period. Uses living-off-the-land techniques to avoid detection. |
| **Likelihood** | 1 — Low (not a high-value target by nation-state standards) |
| **Impact** | 3 — High (full network compromise, data exfiltration) |
| **Inherent Risk Score** | 3 — Low-Medium |
| **Primary Controls** | Tier 5 UEBA behavioral analysis over time; Tier 7 LSTM temporal anomaly; Tier 2 VLAN limits blast radius |
| **Residual Risk** | 2 — Low |
| **Status** | Mitigated with accepted residual |
| **Review Date** | Quarterly |

---

### R-002 — AI-Powered Attack Agent

| Field | Value |
|-------|-------|
| **Risk ID** | R-002 |
| **Category** | Emerging Threat — Agentic AI |
| **Description** | An LLM-orchestrated attack framework autonomously scans for and exploits vulnerabilities. May include prompt injection attacks targeting AI workloads on AI_LAB VLAN. |
| **Likelihood** | 3 — High (rapidly proliferating attack tooling) |
| **Impact** | 2 — Medium |
| **Inherent Risk Score** | 6 — Medium |
| **Primary Controls** | Tier 1 Suricata AI-aware rules; Tier 8 Guardian Prompt Firewall; Tier 8 AI Honeypot |
| **Residual Risk** | 3 — Low (after Tier 8 deployment) |
| **Status** | Active — mitigated by Tier 8 (Phase 7) |
| **Review Date** | Monthly during Phase 7 build |

---

### R-003 — Prompt Injection / RAG Poisoning

| Field | Value |
|-------|-------|
| **Risk ID** | R-003 |
| **Category** | AI-Specific Threat — ATLAS AML.T0051 / AML.T0020 |
| **Description** | Attacker injects malicious instructions into AI agent input (direct) or embeds adversarial content in documents retrieved by RAG systems (indirect). Could hijack Guardian Agent actions. |
| **Likelihood** | 3 — High (technique is well-documented and easy to attempt) |
| **Impact** | 3 — High (if Guardian Agent is hijacked, all AI defenses are compromised) |
| **Inherent Risk Score** | 9 — High |
| **Primary Controls** | Tier 8 Prompt Injection Firewall (3-stage: regex + semantic + ML); RAG Trust Scorer; Semantic Flow Monitor; Tool Call Auditor |
| **Residual Risk** | 4 — Medium (no injection defense is 100%) |
| **Residual Risk Justification** | Defense-in-depth across 4 Guardian nodes reduces probability significantly; HITL gate prevents catastrophic action |
| **Status** | Active — mitigated by Tier 8 |
| **Review Date** | Monthly; update classifier after each new ATLAS TTP publication |

---

### R-004 — Supply Chain Compromise

| Field | Value |
|-------|-------|
| **Risk ID** | R-004 |
| **Category** | Supply Chain — ATLAS AML.T0010 |
| **Description** | Compromised Python package, Docker base image, or ML model weights deliver malicious code into the ecosystem. |
| **Likelihood** | 2 — Medium |
| **Impact** | 3 — High (silent, persistent access; hard to detect) |
| **Inherent Risk Score** | 6 — Medium |
| **Primary Controls** | SUPPLY-CHAIN-POLICY.md; dependency pinning + hash verification; FIM on model files; Wazuh binary change detection |
| **Residual Risk** | 3 — Low |
| **Status** | Mitigated with accepted residual |
| **Review Date** | Quarterly; after each major dependency update |

---

### R-005 — Compromised IoT Device / Lateral Movement

| Field | Value |
|-------|-------|
| **Risk ID** | R-005 |
| **Category** | Internal Threat — Compromised Endpoint |
| **Description** | Smart home device or IoT hardware is compromised and used as a foothold to attempt lateral movement into TRUSTED or MANAGEMENT VLANs. |
| **Likelihood** | 2 — Medium (IoT firmware vulnerabilities common) |
| **Impact** | 1 — Low (VLAN isolation makes lateral movement to high-trust segments very difficult) |
| **Inherent Risk Score** | 2 — Low |
| **Primary Controls** | Tier 2 IOT VLAN (VLAN 30) isolated by default-deny; no inbound routing from IOT; Wazuh alerts on cross-VLAN attempts |
| **Residual Risk** | 1 — Low |
| **Status** | Mitigated by architecture |
| **Review Date** | Quarterly |

---

### R-006 — Security Stack Availability / Single Point of Failure

| Field | Value |
|-------|-------|
| **Risk ID** | R-006 |
| **Category** | Operational — Availability |
| **Description** | Failure of the MGMT VLAN server running Wazuh, MISP, Guardian, and other security tools creates blind spots or complete loss of detection capability. |
| **Likelihood** | 2 — Medium (hardware failure; update failure) |
| **Impact** | 2 — Medium (reduced visibility; core network still functions) |
| **Inherent Risk Score** | 4 — Medium |
| **Primary Controls** | Docker Compose auto-restart policies; Restic encrypted backups; config backup for OPNsense |
| **Residual Risk** | 2 — Low |
| **Status** | Partially mitigated — RECOVER runbook needed |
| **Gap** | No tested restoration procedure. See FRAMEWORK-MAPPING.md Gap Register R-006. |
| **Review Date** | After RECOVER runbook is created |

---

### R-007 — Unpatched Vulnerability in Core Security Component

| Field | Value |
|-------|-------|
| **Risk ID** | R-007 |
| **Category** | Vulnerability Management |
| **Description** | A vulnerability in OPNsense, Suricata, Wazuh, or another security component is exploited before patching. |
| **Likelihood** | 2 — Medium |
| **Impact** | 3 — High (security component compromise = blind spots + potential attacker foothold) |
| **Inherent Risk Score** | 6 — Medium |
| **Primary Controls** | Automated update schedules; OPNsense quarterly update cadence; defense-in-depth means no single component compromise = full compromise |
| **Residual Risk** | 3 — Low |
| **Status** | Mitigated with accepted residual |
| **Review Date** | Quarterly; immediately on critical CVE publication |

---

### R-008 — ML Model Drift / Performance Degradation

| Field | Value |
|-------|-------|
| **Risk ID** | R-008 |
| **Category** | AI/ML — Model Quality |
| **Description** | Tier 7 ML models drift from baseline as network traffic patterns change, causing false negative rate increase (missed attacks) or false positive rate increase (alert fatigue). |
| **Likelihood** | 3 — High (all ML models drift over time) |
| **Impact** | 2 — Medium (other tiers still detect; not a single-point-of-failure) |
| **Inherent Risk Score** | 6 — Medium |
| **Primary Controls** | MLflow model versioning; scheduled retraining; drift detection metrics; anomaly score thresholds monitored |
| **Residual Risk** | 3 — Low |
| **Status** | Mitigated by MLflow + retraining schedule |
| **Review Date** | Monthly model performance review |

---

## 3. Acceptable Exceptions (Documented)

The following gaps are accepted with documented justification.

| Exception | Framework | Justification | Review |
|---|---|---|---|
| No centralized IdP (Okta/Azure AD) | NSA ZT User Pillar | Single-operator environment; certificate-based identity provides sufficient assurance | Annual |
| No formal third-party security assessment | CIS #18 / CSF GV.OV | Solo operation; no budget for external pentest; self-assessment via phase test suites | Annual |
| No formal data classification scheme | NSA ZT Data Pillar | Personal network; data types known implicitly; formal scheme adds overhead without benefit | Annual |
| No dedicated RECOVER phase (yet) | CSF RC | Restic backup exists; formal runbook is backlogged in roadmap | Q2 2025 |

---

## 4. Risk Register Change Log

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2025-01-01 | Initial risk register — Phase 0 |

---

*AEGIS-ZERO Risk Register v1.0.0 — AetherHorizon 🦉*
