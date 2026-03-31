# AEGIS-ZERO Framework Mapping

> **Version:** 1.0.0  
> **Methodology:** STRIDE + NIST CSF 2.0 + NIST AI RMF 1.0 + NIST AI 600-1 + NSA ZT ZIGs +
> MITRE ATT&CK + MITRE ATLAS + CIS Controls v8 + CISA CPGs + NIST SP 800-53 Rev 5 + NIST SP 800-207

This document maps every AEGIS-ZERO tier and component to specific framework controls and
subcategories. Coverage ratings: ✅ Full  ⭐ Exceeds  🟡 Partial  🔴 Gap

---

## Coverage Summary

| Framework | Rating | Notes |
|-----------|--------|-------|
| NIST CSF 2.0 | 🟡 5/6 Functions | RECOVER phase not yet built |
| NIST AI RMF 1.0 | ✅ All 4 Functions | First-class implementation |
| NIST AI 600-1 (GenAI Profile) | ⭐ Exceeds | Agentic AI controls operationalized |
| NIST SP 800-53 Rev 5 | ✅ Full | 8 control families covered |
| NIST SP 800-207 (ZTA) | ✅ Full | All three ZTA core tenets |
| NSA ZT Guidelines (ZIGs) | ✅ Advanced Step 4 | AI/ML-driven automation |
| MITRE ATT&CK Enterprise | ✅ Full TTP | All alerts mapped, STIX 2.1 |
| MITRE ATLAS | ⭐ Exceeds | Custom converter — unique tool |
| CIS Controls v8 | ✅ IG1+IG2+IG3 | All implementation groups |
| CISA Cybersecurity Performance Goals | ✅ Full | All cross-sector goals |

---

## NIST CSF 2.0

> Released February 2024. Six functions, 22 categories, 106 subcategories.  
> Reference: https://nvlpubs.nist.gov/nistpubs/CSWP/NIST.CSWP.29.pdf

### GV — GOVERN  |  Rating: 🟡 Partial

The GOVERN function establishes cybersecurity strategy, risk management policy, oversight, and
supply chain risk management. Three of six categories are addressed through planning documents.
Three require additional formalization (see gap-closing docs in `docs/governance/`).

| CSF Category | Subcategory | AEGIS-ZERO Implementation | Gap? |
|---|---|---|---|
| GV.OC — Organizational Context | GV.OC-01 Mission/objectives documented | ARCHITECTURE.md § Guiding Principles | — |
| GV.OC — Organizational Context | GV.OC-04 Legal/regulatory requirements | FRAMEWORK-MAPPING.md (this doc) | — |
| GV.RM — Risk Management Strategy | GV.RM-01 Risk tolerance established | RISK-REGISTER.md — risk appetite defined | — |
| GV.RM — Risk Management Strategy | GV.RM-02 Risk strategy established | THREAT-MODEL.md + RISK-REGISTER.md | — |
| GV.RR — Roles/Responsibilities | GV.RR-01 Leadership roles defined | Single-operator; documented in RISK-REGISTER.md | 🟡 Solo |
| GV.PO — Policies | GV.PO-01 Policy established | AI-USE-POLICY.md, SUPPLY-CHAIN-POLICY.md | — |
| GV.OV — Oversight | GV.OV-01 Cybersecurity strategy reviewed | Phase approval gate process in PHASES.md | — |
| GV.SC — Supply Chain Risk | GV.SC-01 Supply chain risk managed | SUPPLY-CHAIN-POLICY.md | — |
| GV.SC — Supply Chain Risk | GV.SC-06 Supplier dependencies assessed | TECH-STACK.md § Rejected + SUPPLY-CHAIN-POLICY.md | — |

**Remaining Gap:** No formal audit cadence document. No third-party assessment (single operator).
These are documented acceptable exceptions for the deployment context.

---

### ID — IDENTIFY  |  Rating: ✅ Full

| CSF Category | Subcategory | AEGIS-ZERO Implementation | Tier |
|---|---|---|---|
| ID.AM — Asset Management | ID.AM-01 Physical devices inventoried | osquery hardware-info pack | T6 |
| ID.AM — Asset Management | ID.AM-02 Software inventoried | osquery vuln-management pack | T6 |
| ID.AM — Asset Management | ID.AM-03 Network flows mapped | Zeek + VLAN architecture in ARCHITECTURE.md | T2/T7 |
| ID.AM — Asset Management | ID.AM-07 IT/OT assets catalogued | VLAN segmentation map (IOT VLAN 30) | T2 |
| ID.RA — Risk Assessment | ID.RA-01 Vulnerabilities identified | osquery + Wazuh vulnerability detection | T5/T6 |
| ID.RA — Risk Assessment | ID.RA-02 CTI received/analyzed | MISP + OpenCTI + automated feeds | T4 |
| ID.RA — Risk Assessment | ID.RA-04 Risks identified/prioritized | THREAT-MODEL.md risk matrix | T4 |
| ID.RA — Risk Assessment | ID.RA-05 Threats identified | 6 adversary profiles in THREAT-MODEL.md | T4 |
| ID.IM — Improvement | ID.IM-01 Improvement plans | PHASES.md phase gates + success criteria | All |

---

### PR — PROTECT  |  Rating: ✅ Full

| CSF Category | Subcategory | AEGIS-ZERO Implementation | Tier |
|---|---|---|---|
| PR.AA — Identity Management & Auth | PR.AA-01 User identities managed | Device certificates via Step-CA | T3 |
| PR.AA — Identity Management & Auth | PR.AA-02 Identities proofed | WireGuard peer identity + PKI certs | T2/T3 |
| PR.AA — Identity Management & Auth | PR.AA-03 Users/devices authenticated | mTLS service auth + device certs | T3 |
| PR.AA — Identity Management & Auth | PR.AA-05 Access permissions managed | VLAN inter-segment default-deny matrix | T2 |
| PR.DS — Data Security | PR.DS-01 Data-at-rest protected | Disk encryption (OS-level; documented) | T6 |
| PR.DS — Data Security | PR.DS-02 Data-in-transit protected | mTLS + DoH/DoT + WireGuard encryption | T2/T3 |
| PR.DS — Data Security | PR.DS-10 Data integrity enforced | File Integrity Monitor (FIM) | T6 |
| PR.PS — Platform Security | PR.PS-01 Config baseline established | OPNsense hardening checklist | T1 |
| PR.PS — Platform Security | PR.PS-02 Software maintained | Update schedules in PHASES.md | All |
| PR.PS — Platform Security | PR.PS-04 Logs enabled/managed | Wazuh + full log pipeline | T5 |
| PR.IR — Technology Infrastructure | PR.IR-01 Networks protected | 7-VLAN segmentation, default-deny | T2 |
| PR.IR — Technology Infrastructure | PR.IR-02 Sensitive data protected | MGMT VLAN isolation + mTLS | T2/T3 |

---

### DE — DETECT  |  Rating: ⭐ Exceeds

| CSF Category | Subcategory | AEGIS-ZERO Implementation | Tier |
|---|---|---|---|
| DE.CM — Continuous Monitoring | DE.CM-01 Networks monitored | Zeek + Suricata + CrowdSec | T1/T7 |
| DE.CM — Continuous Monitoring | DE.CM-03 Personnel activity monitored | Wazuh agents + auditd + UEBA | T5/T6/T7 |
| DE.CM — Continuous Monitoring | DE.CM-06 External service provider activity | Supply chain monitoring policy | T6 |
| DE.CM — Continuous Monitoring | DE.CM-09 Computing hardware monitored | osquery hardware telemetry | T6 |
| DE.AE — Adverse Event Analysis | DE.AE-02 Potentially adverse events analyzed | Wazuh correlation engine + ML scoring | T5/T7 |
| DE.AE — Adverse Event Analysis | DE.AE-03 Event data aggregated/correlated | Wazuh → OpenSearch → Kibana pipeline | T5 |
| DE.AE — Adverse Event Analysis | DE.AE-04 Estimated impact of events | SOAR playbook severity matrix | T5 |
| DE.AE — Adverse Event Analysis | DE.AE-06 Information shared with authorized parties | MISP → OpenCTI intelligence sharing | T4 |
| **AI-Specific (exceeds CSF)** | **No CSF subcategory** | **Prompt injection detection, semantic drift, RAG poisoning — Guardian Agent** | **T8** |
| **AI-Specific (exceeds CSF)** | **No CSF subcategory** | **AI honeypot — adversary intelligence capture** | **T8** |

---

### RS — RESPOND  |  Rating: ✅ Full

| CSF Category | Subcategory | AEGIS-ZERO Implementation | Tier |
|---|---|---|---|
| RS.MA — Incident Management | RS.MA-01 Incident response executed | SOAR playbooks activated on alert | T5 |
| RS.MA — Incident Management | RS.MA-02 Incidents categorized/triaged | Wazuh severity matrix (INFO→CRITICAL) | T5 |
| RS.MA — Incident Management | RS.MA-04 Incidents escalated | HITL gate for CRITICAL — mandatory human | T8 |
| RS.AN — Incident Analysis | RS.AN-03 Analysis performed | ML enrichment + MISP threat lookup | T4/T5/T7 |
| RS.CO — Incident Response Reporting | RS.CO-02 Internal stakeholders notified | Slack/alert notification via playbooks | T5 |
| RS.MI — Incident Mitigation | RS.MI-01 Incidents contained | playbook_device_quarantine → VLAN 70 | T5 |
| RS.MI — Incident Mitigation | RS.MI-02 Incidents eradicated | playbook_ip_block + cert revocation | T3/T5 |

---

### RC — RECOVER  |  Rating: 🔴 Gap

| CSF Category | Subcategory | AEGIS-ZERO Implementation | Gap |
|---|---|---|---|
| RC.RP — Incident Recovery Plan | RC.RP-01 Recovery plan executed | ❌ Not yet built | Add Phase 8 |
| RC.CO — Incident Recovery Communication | RC.CO-01 Public relations managed | ❌ Not applicable (home network) | N/A |

**Gap Remediation:** Add a `docs/runbooks/RECOVERY-RUNBOOK.md` documenting:
- Restic backup schedule + offsite rclone sync
- Tested restoration procedures (RTO/RPO defined)
- Post-incident lessons-learned process
- Configuration backup for OPNsense/Wazuh

---

## NIST AI RMF 1.0

> Released January 2023. Four functions: GOVERN, MAP, MEASURE, MANAGE.  
> Reference: https://nvlpubs.nist.gov/nistpubs/ai/nist.ai.100-1.pdf

### GOVERN  |  Rating: ✅ Full

| Subcategory | AEGIS-ZERO Implementation | Component |
|---|---|---|
| GV-1.1 Legal/regulatory requirements understood | AI-USE-POLICY.md documents legal boundaries | Governance |
| GV-1.2 Trustworthy AI characteristics integrated | Guardian Agent policy engine + HITL gate | Tier 8 |
| GV-1.3 Risk management level determined | THREAT-MODEL.md adversary risk matrix | Threat Model |
| GV-1.4 Organizational risk tolerance established | RISK-REGISTER.md — AI risk tolerance section | Governance |
| GV-1.5 Organizational risks prioritized | AI adversaries prioritized: HIGH likelihood in risk matrix | Threat Model |
| GV-1.6 Policies include AI risks | AI-USE-POLICY.md + Guardian policy engine | Tier 8 |
| GV-2.1 Roles documented | AI-USE-POLICY.md § Roles and Responsibilities | Governance |
| GV-4.1 AI risk management integrated | Guardian Agent integrates with SIEM, CTI, and SOAR | Tier 8 |
| GV-6.1 AI risk management policies reviewed | Phase gate approval process | PHASES.md |

---

### MAP  |  Rating: ✅ Full

| Subcategory | AEGIS-ZERO Implementation | Component |
|---|---|---|
| MP-1.1 AI risk categorized | 6 adversary profiles include AI-specific risks | THREAT-MODEL.md |
| MP-1.5 Risk tolerance established per system | AI_LAB VLAN isolated; tolerance documented | T2 + Governance |
| MP-2.1 Scientific/technical knowledge documented | ATLAS TTP database integrated | T4 |
| MP-2.3 AI capabilities and limitations understood | Guardian Agent scoping in ARCHITECTURE.md § Tier 8 | ARCHITECTURE.md |
| MP-3.1 AI risks framed in context | AI adversaries framed in operational context | THREAT-MODEL.md |
| MP-5.1 Likelihood/magnitude of impacts documented | Risk matrix with L/I ratings per adversary | THREAT-MODEL.md |
| MP-5.2 Scientific uncertainty acknowledged | Tier 7 model limitations documented in TECH-STACK.md | TECH-STACK.md |

---

### MEASURE  |  Rating: ⭐ Exceeds

| Subcategory | AEGIS-ZERO Implementation | Component |
|---|---|---|
| MS-1.1 AI risk measurement approaches established | Multi-model scoring: IF anomaly score + LSTM recon error + semantic drift | T7/T8 |
| MS-2.1 AI risk assessment conducted | Continuous scoring via FastAPI scoring service | T7 |
| MS-2.3 AI system performance monitoring | MLflow model versioning + drift detection | T7 |
| MS-2.5 AI system robustness tested | Adversarial test suites in each phase's `tests/` dir | All phases |
| MS-2.6 Interpretability/explainability enabled | Isolation Forest feature importance; audit log explains every Guardian decision | T7/T8 |
| MS-2.8 AI system impact assessed | Guardian threat classification with confidence scores | T8 |
| MS-2.11 Fairness/bias evaluated | N/A — security classification system, not fairness-sensitive deployment | — |
| MS-3.1 Testing/evaluation performed | Test suites required by every phase approval gate | PHASES.md |
| MS-4.1 Measurement results documented | Wazuh SIEM + Guardian audit log + MLflow | T5/T7/T8 |
| **Exceeds: quantitative AI risk scoring** | Anomaly scores, semantic drift deltas, trust scores — measurable, logged, alertable | T7/T8 |

---

### MANAGE  |  Rating: ✅ Full

| Subcategory | AEGIS-ZERO Implementation | Component |
|---|---|---|
| MG-1.1 AI risks prioritized and treated | Guardian threat classification → SOAR playbook routing | T8 → T5 |
| MG-1.3 Responses to AI risks taken | playbook_ai_threat_response.py + Guardian containment | T5/T8 |
| MG-2.2 Mechanisms for reverting AI decisions | HITL gate — human can override any Guardian action | T8 |
| MG-2.4 AI system risks monitored | Continuous Guardian monitoring + Wazuh AI threat dashboard | T5/T8 |
| MG-3.1 AI risks communicated | Guardian alerts → Wazuh → MISP IOC sharing | T4/T5/T8 |
| MG-4.1 Post-deployment monitoring | Guardian always-on + Wazuh continuous correlation | T5/T8 |
| MG-4.2 Incidents documented | Immutable SQLite audit log + Wazuh event store | T5/T8 |

---

## NIST AI 600-1 (GenAI / Agentic AI Profile)

> Released July 2024. Extension of AI RMF 1.0 specifically for generative and agentic AI.  
> Reference: https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.600-1.pdf

| AI 600-1 Risk Area | AEGIS-ZERO Implementation | Component |
|---|---|---|
| Prompt injection (direct + indirect) | Prompt Injection Firewall: regex + semantic + ML classifier | T8 |
| Data privacy (PII exfiltration via prompts) | Tool Call Auditor blocks data access policy violations | T8 |
| Confabulation / false information generation | RAG Trust Scorer validates document provenance before LLM ingestion | T8 |
| Harmful content generation | Guardian policy engine defines authorized output categories | T8 |
| Intellectual property / copyright | Tool Call Auditor enforces tool × context × identity policy | T8 |
| Obscene/abusive content | Out of scope for network security context | N/A |
| Dangerous/violent content | Out of scope for network security context | N/A |
| CSAM / NCII | Out of scope; referenced in AI-USE-POLICY.md | Governance |
| Cybersecurity attacks via AI | Core mission of entire Tier 8 Guardian stack | T8 |
| Environmental impact | Noted; AI workloads isolated to AI_LAB VLAN | T2 |
| Agentic decision-making risks | Semantic Flow Monitor detects unauthorized goal drift | T8 |
| Multi-agent trust boundaries | Tool Call Auditor enforces agent × tool policy per identity | T8 |
| Transparency / explainability | Guardian audit log explains every decision; Streamlit dashboard | T8 |

**Key Differentiator:** NIST AI 600-1 explicitly calls out agentic systems as a specific risk
category. The Guardian Agent is the only known public home-lab implementation of AI 600-1
suggested actions at the control level.

---

## NIST SP 800-53 Rev 5

> Control catalog referenced by all US federal agencies. 20 control families.  
> Reference: https://csrc.nist.gov/publications/detail/sp/800-53/rev-5/final

| Control Family | Controls Addressed | AEGIS-ZERO Implementation |
|---|---|---|
| **AC** — Access Control | AC-2, AC-3, AC-4, AC-6, AC-17, AC-19 | VLAN segmentation, default-deny, WireGuard remote access, PKI-based access, least privilege |
| **AU** — Audit and Accountability | AU-2, AU-3, AU-6, AU-9, AU-12 | Wazuh event logging, immutable audit log, osquery telemetry, Zeek network audit |
| **CM** — Configuration Management | CM-2, CM-3, CM-6, CM-7 | OPNsense hardening checklist, FIM (CM-3/CM-6), osquery config monitoring |
| **IA** — Identification and Authentication | IA-2, IA-3, IA-5, IA-8, IA-9 | Step-CA PKI, device certificates, mTLS service auth, WireGuard peer identity |
| **IR** — Incident Response | IR-4, IR-5, IR-6, IR-8 | SOAR playbooks, Wazuh incident tracking, MISP IOC sharing, incident response playbooks |
| **RA** — Risk Assessment | RA-2, RA-3, RA-5, RA-7 | THREAT-MODEL.md, RISK-REGISTER.md, osquery vulnerability surface, continuous CTI |
| **SC** — System and Comm. Protection | SC-5, SC-7, SC-8, SC-17, SC-28 | DDoS mitigation, boundary protection (OPNsense), TLS/mTLS, Step-CA PKI, disk encryption |
| **SI** — System and Information Integrity | SI-2, SI-3, SI-4, SI-7, SI-10 | Suricata IPS, Wazuh SIEM, FIM, rootkit detection, input validation (Guardian prompt firewall) |

---

## NIST SP 800-207 (Zero Trust Architecture)

> The definitive NIST specification for Zero Trust. Three core tenets.  
> Reference: https://csrc.nist.gov/publications/detail/sp/800-207/final

| ZTA Core Tenet | Implementation | AEGIS-ZERO Component |
|---|---|---|
| **Verify explicitly** — Authenticate/authorize using all available data points | Device certificates + WireGuard identity + mTLS per-connection auth | T2/T3 |
| **Use least privilege access** — Limit access with Just-In-Time, Just-Enough-Access | VLAN default-deny matrix; MGMT VLAN accessible only via WireGuard | T2 |
| **Assume breach** — Minimize blast radius, segment access, verify E2E encryption | 7-VLAN segmentation; VLAN 70 quarantine; encrypted everything | T2/T3 |

**SP 800-207 Deployment Model:** Network-based ZTA (VLAN segmentation + identity-based access).
With Step-CA PKI providing device identity, AEGIS-ZERO satisfies the enhanced deployment model
described in SP 800-207 § 3.3 (ZTA with enhanced identity governance).

---

## NSA Zero Trust Guidelines

> NSA released Phase 1/2 Zero Trust Implementation Guidelines (ZIGs), January 2026.  
> Prior: NSA CSI "Advancing Zero Trust Maturity Throughout the Network and Environment Pillar," March 2024.

The NSA ZT model has seven pillars. AEGIS-ZERO maturity per pillar:

| NSA ZT Pillar | Maturity Level | AEGIS-ZERO Implementation |
|---|---|---|
| **User** | Step 2: Basic | WireGuard identity + device certs. No centralized IdP (documented exception). |
| **Device** | Step 3: Intermediate | Device certificates, osquery posture, FIM, Wazuh agents on all endpoints. |
| **Network & Environment** | Step 4: **Advanced** | 7-VLAN macro + inter-VLAN default-deny micro-segmentation. Data flow mapping per architecture. |
| **Application & Workload** | Step 2: Basic | mTLS for internal APIs, DMZ isolation. Per-app WAF policy not yet implemented. |
| **Data** | Step 2: Basic | Encryption in transit (mTLS, DoH/DoT). Formal data classification not documented. |
| **Automation & Orchestration** | Step 4: **Advanced** | SOAR playbooks + Guardian AI/ML-driven automated risk-based response satisfies NSA Step 4. |
| **Visibility & Analytics** | Step 4: **Advanced** | Wazuh + Zeek + ML scoring + Guardian dashboard = full-stack continuous observability. |

**NSA Step 4 Advanced Maturity** (Automation & Orchestration): *"Establish automation and management
based on analytics and risk-based responses using policy and AI/ML technologies."* The
SOAR + Guardian combination is a textbook implementation.

---

## MITRE ATT&CK

> ATT&CK Enterprise Matrix — comprehensive adversary TTP taxonomy.  
> Reference: https://attack.mitre.org/

All Wazuh alerts are tagged with ATT&CK technique IDs. OpenCTI ingests ATT&CK via STIX 2.1.
The ATLAS-to-Wazuh converter also maps ATT&CK techniques where ATLAS and ATT&CK overlap.

| ATT&CK Tactic | Example Techniques | Detection Component |
|---|---|---|
| Reconnaissance | T1595, T1589, T1598 | Suricata scanning rules, CrowdSec reputation |
| Initial Access | T1190, T1566, T1133 | Suricata exploit signatures, DNS phishing block |
| Execution | T1059, T1053, T1203 | Wazuh process creation, osquery startup-items pack |
| Persistence | T1547, T1543, T1078 | FIM on startup dirs, osquery persistence packs |
| Privilege Escalation | T1055, T1068, T1134 | Auditd syscall monitoring, Wazuh priv-esc rules |
| Defense Evasion | T1027, T1070, T1562 | FIM + auditd detect log tampering and binary modification |
| Credential Access | T1003, T1110, T1555 | Wazuh brute-force rules, osquery credential store monitoring |
| Discovery | T1046, T1082, T1083 | Suricata port scan detection, UEBA (internal scan baseline) |
| Lateral Movement | T1021, T1091, T1550 | VLAN isolation blocks L.M. by default; Wazuh detects attempts |
| Collection | T1005, T1039, T1074 | DLP via egress filtering + DNS exfiltration detection |
| Exfiltration | T1048, T1041, T1071 | DNS entropy model, Zeek anomaly detection |
| Command & Control | T1071, T1095, T1573 | Suricata C2 signatures, Zeek LSTM temporal anomaly |
| Impact | T1486, T1491, T1561 | FIM detects ransomware file modifications; Wazuh active response |

---

## MITRE ATLAS

> Adversarial Threat Landscape for AI Systems — the ATT&CK equivalent for ML attacks.  
> Reference: https://atlas.mitre.org/

AEGIS-ZERO's ATLAS coverage is its most distinctive differentiator. The `atlas-to-wazuh` converter
(Phase 3) operationalizes ATLAS TTPs into live Wazuh detection rules — a capability not found in
any other public home-lab or small-enterprise deployment.

| ATLAS Tactic | TTP ID | Description | AEGIS-ZERO Defense |
|---|---|---|---|
| ML Attack Staging | AML.T0040 | Staging environment for ML attacks | Isolated AI_LAB VLAN + osquery monitoring |
| Craft Adversarial Data | AML.T0043 | Creating adversarial inputs | Guardian Prompt Firewall (semantic layer) |
| ML Supply Chain Compromise | AML.T0010 | Compromising ML libraries/models | Supply chain policy + FIM on model files |
| Backdoor ML Model | AML.T0018 | Inserting malicious behavior into models | FIM on model weights + embedding drift detection |
| Poison Training Data | AML.T0019/T0020 | Corrupting training datasets | RAG Trust Scorer + provenance checking |
| LLM Prompt Injection | AML.T0051 | Direct/indirect prompt injection | Prompt Injection Firewall (3-stage: regex+semantic+ML) |
| LLM Jailbreak | AML.T0054 | Bypassing LLM safety controls | Prompt Firewall jailbreak pattern library |
| Erode ML Model Integrity | AML.T0048 | Gradual model degradation | Embedding drift detection in RAG Trust Scorer |
| Model Extraction | AML.T0030 | Stealing model functionality | Honeypot API fingerprints extraction attempts |
| Infer Training Data | AML.T0024 | Membership inference attacks | Tool Call Auditor limits data access per identity |
| Evade ML Model | AML.T0015 | Crafting evasion inputs | LSTM autoencoder anomaly (behavioral, not signature) |
| ML-Enabled Product | AML.T0047 | Weaponizing AI for attacks | Guardian Honeypot profiles adversarial AI tooling |

**The ATLAS→Wazuh Converter** bridges the gap between ATLAS theory and operational defense.
No commercial or open-source tooling does this automatically. This is original AetherHorizon
portfolio work.

---

## CIS Controls v8

> Center for Internet Security Controls — 18 controls, three implementation groups.  
> Reference: https://www.cisecurity.org/controls/v8

| CIS Control | IG | Description | AEGIS-ZERO Implementation |
|---|---|---|---|
| 1 — Inventory of Enterprise Assets | 1 | Track all hardware assets | osquery hardware-info pack |
| 2 — Inventory of Software Assets | 1 | Track all installed software | osquery vuln-management pack |
| 3 — Data Protection | 1 | Data classification, encryption | mTLS + DoH/DoT + disk encryption |
| 4 — Secure Configuration | 1 | Secure baseline configs | OPNsense hardening checklist + Ansible |
| 5 — Account Management | 1 | Control user accounts | PKI device certs + WireGuard identity |
| 6 — Access Control Management | 1 | Enforce least privilege | VLAN default-deny + per-resource mTLS |
| 7 — Continuous Vulnerability Management | 2 | Patch/scan for vulns | osquery + Wazuh vuln detection |
| 8 — Audit Log Management | 1 | Collect/protect logs | Wazuh + OpenSearch + immutable audit log |
| 9 — Email and Web Browser Protections | 1 | Block malicious content | Pi-hole DNS filtering |
| 10 — Malware Defenses | 1 | Anti-malware on endpoints | Wazuh agent + rootkit detection |
| 11 — Data Recovery | 1 | Backup/restore capability | Restic (planned — RECOVER gap) |
| 12 — Network Infrastructure Management | 2 | Secure network devices | OPNsense hardening |
| 13 — Network Monitoring and Defense | 2 | Monitor network traffic | Zeek + Suricata + ML models |
| 14 — Security Awareness | 2 | Training | N/A — single operator |
| 16 — Application Software Security | 2 | Secure dev practices | Dependency pinning + hash verification |
| 17 — Incident Response Management | 2 | IR process | SOAR playbooks + Wazuh IR workflow |
| 18 — Penetration Testing | 3 | Test defenses | Phase test suites; formal pentest planned |

---

## CISA Cybersecurity Performance Goals (CPGs)

> Cross-Sector Cybersecurity Performance Goals — CISA's baseline expectations.  
> Reference: https://www.cisa.gov/cross-sector-cybersecurity-performance-goals

| CPG Goal Area | Implementation | AEGIS-ZERO Component |
|---|---|---|
| Account Security | PKI device certificates + WireGuard | T2/T3 |
| Device Security | osquery + Wazuh agent + FIM | T6 |
| Data Security | mTLS + DoH/DoT + disk encryption | T2/T3 |
| Governance and Training | Governance docs in `docs/governance/` | Governance |
| Vulnerability Management | osquery + Wazuh vuln rules | T5/T6 |
| Supply Chain / Third-Party Risk | SUPPLY-CHAIN-POLICY.md | Governance |
| Response and Recovery | SOAR playbooks (response); RECOVER gap documented | T5 |
| Network Security | 7-VLAN segmentation + OPNsense | T1/T2 |
| Operational Technology | IOT VLAN isolation (VLAN 30) | T2 |
| Email Security | DNS-based phishing protection via Pi-hole | T3 |
| Endpoint Detection | Wazuh agents + osquery + FIM | T6 |

---

## Gap Register

The following gaps are acknowledged, documented, and have defined remediation paths.

| Gap | Framework(s) | Severity | Remediation Path |
|-----|------------|---------|----------------|
| CSF RECOVER not built | CSF RC | Medium | Add recovery runbook + Restic schedule to Phase 4 or standalone Phase 8 |
| NSA ZT User pillar partial | NSA ZT | Low | Acceptable for single-operator; document exception in RISK-REGISTER.md |
| No centralized IdP | NSA ZT User | Low | Out of scope for home network; certificate-based identity is sufficient |
| NSA ZT Data pillar partial | NSA ZT | Low | Formal data classification doc needed; add to `docs/governance/` |
| DISA STIG not formally applied | DISA | Low | OPNsense has no official STIG; hardening checklist follows STIG principles |
| No formal pentest | CIS #18 | Medium | Add to roadmap post-Phase 7; document in RISK-REGISTER.md |

---

*AEGIS-ZERO Framework Mapping v1.0.0 — AetherHorizon 🦉*
