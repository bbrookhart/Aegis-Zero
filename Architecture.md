# AEGIS-ZERO Architecture Specification

> **Version:** 1.0.0  
> **Classification:** Public Portfolio  
> **Status:** Planning Phase Complete

---

## 1. Architecture Overview

AEGIS-ZERO implements a layered, defense-in-depth architecture across eight distinct tiers.
Each tier operates as an independent security domain with its own detection, logging, and
response capabilities. Compromise of any single tier triggers alerting in all upstream tiers.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    INTERNET / THREAT SURFACE                        │
└───────────────────────────┬─────────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────────┐
│  TIER 1 │ PERIMETER DEFENSE                                         │
│  OPNsense · Suricata IDS/IPS · CrowdSec · GeoIP Block · DDoS Mitg  │
└───────────────────────────┬─────────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────────┐
│  TIER 2 │ NETWORK SEGMENTATION & ZERO TRUST                         │
│  VLAN 802.1Q · Microsegmentation · WireGuard · ZTNA Policy Engine   │
└───────────────────────────┬─────────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────────┐
│  TIER 3 │ DNS SECURITY & ENCRYPTED COMMUNICATIONS                   │
│  Pi-hole · DoH/DoT · Step-CA PKI · mTLS · Certificate Rotation      │
└───────────────────────────┬─────────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────────┐
│  TIER 4 │ CYBER THREAT INTELLIGENCE                                  │
│  MISP · OpenCTI · ATLAS Framework · Custom TTP Converters           │
└───────────────────────────┬─────────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────────┐
│  TIER 5 │ SIEM / SOAR                                               │
│  Wazuh · Elasticsearch · Kibana · Automated Response Playbooks      │
└───────────────────────────┬─────────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────────┐
│  TIER 6 │ ENDPOINT DETECTION & RESPONSE                             │
│  Wazuh Agents · osquery · Auditd · Sysmon · File Integrity Monitor  │
└───────────────────────────┬─────────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────────┐
│  TIER 7 │ AI/ML BEHAVIORAL THREAT DETECTION                         │
│  Zeek · Isolation Forest · LSTM Anomaly · UEBA · Flow Analysis      │
└───────────────────────────┬─────────────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────────────┐
│  TIER 8 │ AGENTIC AI SECURITY GUARDIAN                              │
│  LangGraph Agent · Prompt Injection Firewall · Semantic Monitor     │
│  RAG Trust Scorer · Tool Call Auditor · AI Honeypot API             │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Tier Specifications

---

### TIER 1 — Perimeter Defense

**Mission:** Intercept, inspect, and block threats at the network boundary before they reach
internal resources. First line of defense against internet-based adversaries including
automated scanning, exploit frameworks, and AI-driven attack tooling.

**Hardware Target:**  
- Dedicated x86 appliance (Protectli, PC Engines APU, or equivalent) or pfSense/OPNsense VM
- Minimum: 4-core CPU, 8GB RAM, 64GB SSD, 4× GbE NIC
- Recommended: 6-core, 16GB RAM, 256GB NVMe, 2.5GbE capable

**Core Components:**

| Component | Role | Version Target |
|-----------|------|---------------|
| OPNsense | Primary stateful firewall, NAT, routing | 24.x |
| Suricata | Inline IDS/IPS, signature + anomaly detection | 7.x |
| CrowdSec | Distributed threat intelligence, IP reputation | 1.6.x |
| Nginx (reverse proxy) | SSL termination, WAF for exposed services | Latest |
| GeoIP Blocking | Country-level block lists via MaxMind | MaxMind GeoLite2 |
| Fail2ban | Brute-force protection for management interfaces | 1.x |

**Suricata Rule Sets (Priority Order):**
1. Emerging Threats Open (ET Open) — base signatures
2. ET Pro (if licensed) — advanced/zero-day signatures
3. ATLAS AI-specific rules (custom, via Phase 3 converter)
4. Local custom rules — environment-specific detections
5. PT Research rules — APT-focused signatures

**Key Firewall Policies:**
- Default-deny inbound; explicit allowlist only
- Egress filtering enabled — outbound traffic requires justification
- Management interface accessible only from MGMT VLAN
- All WAN-facing services rate-limited and logged
- Automatic IP blocking on Suricata drop-severity alerts

**Detection Capabilities:**
- Protocol anomaly detection (HTTP, DNS, TLS, SMB)
- Known malware C2 traffic (ET signatures)
- Port scanning and reconnaissance
- DDoS pattern detection
- TLS certificate anomaly detection (JA3/JA3S fingerprinting)
- DNS exfiltration detection

---

### TIER 2 — Network Segmentation & Zero Trust

**Mission:** Eliminate the concept of a trusted internal network. Every VLAN is its own
security domain. Inter-segment traffic is treated as potentially hostile. Implement
Zero Trust Network Access (ZTNA) for all privileged operations.

**VLAN Architecture:**

| VLAN ID | Name | Subnet | Description |
|---------|------|--------|-------------|
| VLAN 10 | MANAGEMENT | 10.0.10.0/24 | Security tools, firewall mgmt, SIEM |
| VLAN 20 | TRUSTED | 10.0.20.0/24 | Primary workstations, approved devices |
| VLAN 30 | IOT | 10.0.30.0/24 | Smart home, cameras — no outbound except cloud APIs |
| VLAN 40 | DMZ | 10.0.40.0/24 | Publicly exposed services, honeypots |
| VLAN 50 | AI_LAB | 10.0.50.0/24 | AI/ML workloads, Guardian Agent, sandboxed LLM calls |
| VLAN 60 | GUEST | 10.0.60.0/24 | Isolated internet-only, no internal routing |
| VLAN 70 | QUARANTINE | 10.0.70.0/24 | Compromised/suspected devices, internet blocked |
| VLAN 99 | NATIVE | 10.0.99.0/24 | Untagged/default — blocked everywhere |

**Inter-VLAN Firewall Rules (Default-Deny Matrix):**

```
FROM\TO    MGMT  TRUSTED  IOT   DMZ   AI_LAB  GUEST  QUARANTINE
MGMT       SELF   ✅      ✅    ✅     ✅      ✅       ✅
TRUSTED     ❌   SELF     ❌    ✅     ✅      ❌       ❌
IOT         ❌    ❌     SELF   ❌     ❌      ❌       ❌
DMZ         ❌    ❌      ❌   SELF    ❌      ❌       ❌
AI_LAB      ✅    ❌      ❌    ❌    SELF     ❌       ❌
GUEST       ❌    ❌      ❌    ❌     ❌     SELF       ❌
QUARANTINE  ❌    ❌      ❌    ❌     ❌      ❌      SELF
```

**Zero Trust Implementation:**
- WireGuard for all privileged administrative access (even on local network)
- mTLS enforced for all service-to-service API calls
- Device certificates required for VLAN 10 and 20 access
- Continuous authentication (re-challenge on session timeout or anomaly)
- Network access based on identity + device posture, not IP address

---

### TIER 3 — DNS Security & Encrypted Communications

**Mission:** Prevent DNS-based attacks, command-and-control beaconing via DNS, data exfiltration
through DNS tunneling, and man-in-the-middle attacks on internal communications. Establish
a private Certificate Authority for the entire ecosystem.

**Core Components:**

| Component | Role |
|-----------|------|
| Pi-hole | DNS sinkhole, ad/malware domain blocking |
| Unbound | Recursive DNS resolver (DNSSEC validation) |
| Cloudflare DoH / Quad9 DoT | Encrypted upstream resolution |
| Step-CA (Smallstep) | Internal PKI / private Certificate Authority |
| Certbot / ACME | Public cert management for DMZ services |
| Nginx + mTLS | Mutual TLS termination for internal APIs |

**DNS Security Layers:**
1. **Pi-hole** blocks known malicious, tracker, and C2 domains at query time
2. **Unbound** performs recursive resolution with DNSSEC validation
3. **DoH/DoT upstream** encrypts all queries leaving the network
4. **Custom DNS block lists** fed from MISP threat intel (Tier 4)
5. **DNS query logging** — all queries normalized and sent to Wazuh (Tier 5)
6. **DNS anomaly detection** — high-volume, entropy-based, rare-domain alerting

**PKI Architecture (Step-CA):**
```
Root CA (offline — air-gapped USB)
└── Intermediate CA (Step-CA, VLAN 10)
    ├── Management certs (admin workstations)
    ├── Service certs (internal APIs, mTLS)
    ├── Device certs (endpoint identity)
    └── WireGuard PKI (peer authentication)
```

**Certificate Policies:**
- Max validity: 90 days (auto-renewed via ACME protocol)
- RSA 4096 or ECDSA P-384 minimum
- Certificate Transparency logging for all issued certs
- Revocation via OCSP stapling
- Root CA private key stored offline; never touches a networked device

---

### TIER 4 — Cyber Threat Intelligence

**Mission:** Ingest, correlate, and operationalize threat intelligence from global sources,
specialized AI/ML attack databases, and ATLAS adversarial AI threat framework. Convert
intelligence into actionable detection rules across the ecosystem.

**Core Components:**

| Component | Role |
|-----------|------|
| MISP | Threat intelligence sharing platform, IOC management |
| OpenCTI | Structured CTI platform (STIX 2.1, TAXII) |
| ATLAS Framework | MITRE ATLAS adversarial AI TTP database |
| atlas-to-wazuh | Custom Python converter (ATLAS TTPs → Wazuh rules) |
| threat-feed-ingest | Automated OSINT and feed aggregation pipeline |

**Threat Feed Sources:**
- MITRE ATT&CK (enterprise + mobile + ICS matrices)
- MITRE ATLAS (adversarial ML attacks)
- Abuse.ch (malware, botnet C2 IOCs)
- AlienVault OTX (open threat exchange)
- Emerging Threats (ET Pro if licensed, ET Open base)
- PhishTank (phishing domains)
- Proofpoint Emerging Threats (AI-specific TTPs)
- Custom AetherHorizon research feeds

**ATLAS → Wazuh Converter (Custom Build):**
The `atlas-to-wazuh` script is the crown jewel of Tier 4. It:
1. Fetches the ATLAS TTP database via MITRE's STIX API
2. Maps each adversarial ML technique to a detection hypothesis
3. Generates Wazuh custom rules targeting observable indicators
4. Exports rules in Wazuh XML format ready for deployment
5. Maintains a version-controlled rule history

**MISP ↔ OpenCTI ↔ Wazuh Pipeline:**
```
External Feeds → MISP (IOC management)
     ↓
OpenCTI (structural CTI, STIX 2.1 enrichment)
     ↓
Wazuh (rule generation, active response)
     ↓
Guardian Agent (Tier 8: AI-aware correlation)
```

---

### TIER 5 — SIEM / SOAR

**Mission:** Aggregate, normalize, correlate, and respond to security events across the entire
ecosystem. Provide real-time threat detection, investigation workflows, compliance monitoring,
and automated incident response.

**Core Components:**

| Component | Role |
|-----------|------|
| Wazuh Manager | Central SIEM, rule correlation, active response |
| Wazuh Indexer | OpenSearch-based event storage |
| Wazuh Dashboard | Kibana-based visualization + alerting UI |
| Custom Playbooks | Python-based SOAR automation |
| MITRE ATT&CK mappings | TTP tagging on every alert |

**Log Sources (Normalized to Wazuh):**
- OPNsense / Suricata (Tier 1) — network events, IDS alerts
- CrowdSec (Tier 1) — community threat intelligence decisions
- Pi-hole / Unbound (Tier 3) — all DNS queries
- All VLAN gateway logs (Tier 2) — inter-segment traffic
- Wazuh agents (Tier 6) — endpoint events
- Zeek (Tier 7) — network flow metadata
- Guardian Agent (Tier 8) — AI-specific threat events

**Alert Severity Matrix:**

| Level | Wazuh Score | Response | Human Required |
|-------|------------|----------|---------------|
| INFO | 1-4 | Log only | No |
| LOW | 5-6 | Log + tag | No |
| MEDIUM | 7-9 | Alert + Slack notification | No |
| HIGH | 10-12 | Auto-block + alert | Recommended |
| CRITICAL | 13-15 | Full lockdown + alert | **Yes — mandatory** |

**SOAR Playbooks (Automated):**
1. `playbook-ip-block.py` — CrowdSec ban + firewall rule on HIGH+ IOC match
2. `playbook-device-quarantine.py` — Move device to VLAN 70 on anomaly detection
3. `playbook-dns-sinkhole.py` — Push malicious domain to Pi-hole blocklist
4. `playbook-cert-revoke.py` — Revoke compromised device certificate via Step-CA
5. `playbook-ai-threat-response.py` — Guardian Agent escalation handler

---

### TIER 6 — Endpoint Detection & Response (EDR)

**Mission:** Monitor every endpoint for indicators of compromise, unauthorized changes, lateral
movement, and privilege escalation. Provide deep visibility into host behavior.

**Core Components:**

| Component | Role |
|-----------|------|
| Wazuh Agent | EDR agent — log forwarding, active response, rootkit detection |
| osquery | SQL-based endpoint telemetry (processes, network, files) |
| Auditd (Linux) | Kernel-level syscall auditing |
| Sysmon (Windows) | Windows event enrichment |
| File Integrity Monitor | Real-time critical file/directory monitoring |

**osquery Packs Deployed:**
- `incident-response` — process trees, open sockets, loaded modules
- `vuln-management` — installed packages, CVE surface
- `hardware-info` — USB devices, removable media
- `network-connections` — all active connections + DNS cache
- `startup-items` — persistence mechanisms (cron, systemd, registry)

**FIM Critical Paths:**
```
Linux:   /etc/, /bin/, /sbin/, /usr/bin/, /boot/, /root/,
         ~/.ssh/, /etc/sudoers, /etc/passwd, /etc/shadow
Windows: C:\Windows\System32\, HKLM\Software\, Startup folders,
         C:\Users\*\AppData\Roaming\Microsoft\Windows\Start Menu\
```

**Rootkit Detection:**
- Wazuh built-in rootkit scanner (daily scheduled + on-demand)
- rkhunter integration
- chkrootkit integration
- Anomaly-based detection: hidden processes, port discrepancies, kernel module changes

---

### TIER 7 — AI/ML Behavioral Threat Detection

**Mission:** Detect novel attacks that bypass signature-based defenses using machine learning
models trained on network and endpoint behavior. Establish baselines; alert on statistical
deviations that indicate compromise, data exfiltration, or adversarial AI activity.

**Core Components:**

| Component | Role |
|-----------|------|
| Zeek | Network flow metadata extraction, protocol analysis |
| NetFlow/IPFIX | Traffic volume and flow telemetry |
| Isolation Forest | Unsupervised anomaly detection (network flows) |
| LSTM Autoencoder | Temporal sequence anomaly detection |
| UEBA Engine | User and entity behavioral baselines + deviation scoring |
| FastAPI + Redis | Real-time scoring microservice |
| MLflow | Model versioning, experiment tracking |

**Detection Models:**

```
Model 1: Network Flow Anomaly (Isolation Forest)
  Input:  Zeek conn.log features (bytes, pkts, duration, ports, proto)
  Output: Anomaly score per flow (0.0 - 1.0)
  Alert:  score > 0.85 → MEDIUM; score > 0.95 → HIGH

Model 2: Temporal Sequence Anomaly (LSTM Autoencoder)
  Input:  Time-windowed sequences of network events (15-min windows)
  Output: Reconstruction error (normal = low; attack = high)
  Alert:  error > 2σ → MEDIUM; error > 3σ → HIGH

Model 3: DNS Entropy Anomaly (Statistical)
  Input:  DNS query entropy, query rate, unique domain ratio
  Output: Exfiltration/tunneling probability score
  Alert:  score > 0.7 → HIGH (DNS tunneling or DGA C2)

Model 4: UEBA Baseline Deviation
  Input:  Per-entity behavioral vectors (login times, destinations, volumes)
  Output: Deviation score per entity per time window
  Alert:  score > 3σ → MEDIUM (unusual entity behavior)
```

**AI-Specific Detection (Pre-Guardian):**
- LLM API call volume spikes (potential prompt injection campaigns)
- Unusual token consumption patterns on local LLM endpoints
- Model file modification detection (model poisoning)
- Embedding space shift detection (RAG poisoning)

---

### TIER 8 — Agentic AI Security Guardian

**Mission:** The crown jewel. An autonomous AI security agent purpose-built to defend against
the next generation of threats — other AI agents. The Guardian monitors all AI activity
within the ecosystem, enforces policy at the semantic level, and responds to adversarial
AI threats in real time.

**Architecture Overview:**
```
                    ┌─────────────────────────────┐
                    │    GUARDIAN AGENT (LangGraph)│
                    │                             │
  Incoming AI ─────►│  ┌──────────────────────┐  │
  Traffic /         │  │ Prompt Injection      │  │
  API Calls         │  │ Firewall              │  ├──► ALLOW / DENY / SANDBOX
                    │  └──────────┬───────────┘  │
                    │             │               │
                    │  ┌──────────▼───────────┐  │
                    │  │ Semantic Flow         │  │
                    │  │ Monitor               │  ├──► Alert to Wazuh (Tier 5)
                    │  └──────────┬───────────┘  │
                    │             │               │
                    │  ┌──────────▼───────────┐  │
                    │  │ RAG Trust Scorer      │  │
                    │  └──────────┬───────────┘  │
                    │             │               │
                    │  ┌──────────▼───────────┐  │
                    │  │ Tool Call Auditor     │  │
                    │  └──────────┬───────────┘  │
                    │             │               │
                    │  ┌──────────▼───────────┐  │
                    │  │ AI Honeypot API       │  ├──► Adversary Profiling
                    │  └─────────────────────┘  │
                    └─────────────────────────────┘
```

**Guardian Components:**

**1. Prompt Injection Firewall**
- Multi-stage detection pipeline (regex + semantic + ML classifier)
- Detects: direct injection, indirect injection, jailbreak patterns, role-play attacks
- Action: BLOCK (high confidence), SANDBOX (uncertain), ALLOW + LOG (clean)
- Response time target: < 50ms p99

**2. Semantic Flow Monitor (MAScope-inspired)**
- Tracks semantic intent across multi-turn agent conversations
- Detects: goal hijacking, semantic drift, hidden instruction accumulation
- Maintains a conversation-level intent graph
- Alerts when current semantic trajectory diverges from authorized intent

**3. RAG Trust Scorer**
- Scores every document chunk retrieved during RAG operations
- Trust factors: source provenance, content entropy, embedding drift, freshness
- Documents below trust threshold are quarantined before LLM ingestion
- Detects: RAG poisoning, adversarial document injection, data exfiltration via retrieval

**4. Tool Call Auditor**
- Logs and validates every tool/function call made by AI agents
- Policy enforcement: tool × context × agent identity = allow/deny
- Detects: unauthorized tool access, privilege escalation via tool chaining
- Maintains an immutable tool call audit log

**5. AI Honeypot API**
- Fake LLM endpoints designed to attract and fingerprint adversarial probing
- Serves plausible-but-monitored responses to attackers
- Profiles attack patterns, prompt structures, exfiltration attempts
- Feeds adversary intelligence back to MISP (Tier 4)

**LangGraph Agent Graph:**
```python
# Simplified node structure
nodes = [
    "ingest_request",
    "prompt_injection_screen",
    "semantic_intent_check",
    "rag_trust_evaluation",
    "tool_call_audit",
    "threat_classification",
    "human_escalation_gate",  # HITL for CRITICAL severity
    "response_synthesis",
    "audit_log_commit"
]
```

---

## 3. Data Flow Architecture

```
All Events → Wazuh (Tier 5) → Normalized Event Store
                                        │
                              ┌─────────▼──────────┐
                              │  Correlation Engine  │
                              │  (Wazuh Rules +     │
                              │   Custom Python)     │
                              └─────────┬──────────┘
                                        │
                              ┌─────────▼──────────┐
                              │  Threat Intel        │
                              │  Enrichment          │
                              │  (MISP + OpenCTI)    │
                              └─────────┬──────────┘
                                        │
                              ┌─────────▼──────────┐
                              │  ML Scoring          │
                              │  (Tier 7 Models)     │
                              └─────────┬──────────┘
                                        │
                              ┌─────────▼──────────┐
                              │  Guardian Agent      │
                              │  (AI-specific intel) │
                              └─────────┬──────────┘
                                        │
                              ┌─────────▼──────────┐
                              │  SOAR Playbooks      │
                              │  (Automated +        │
                              │   Human-gated)       │
                              └────────────────────┘
```

---

## 4. Hardware Architecture

```
[ISP Modem]
     │
[TIER 1 APPLIANCE]  ← OPNsense + Suricata + CrowdSec
     │
[MANAGED SWITCH]    ← 802.1Q VLAN trunk
     ├── VLAN 10 (MGMT)     → [Security Stack Server]
     │                           - Wazuh Manager (Tier 5)
     │                           - MISP (Tier 4)
     │                           - OpenCTI (Tier 4)
     │                           - Pi-hole (Tier 3)
     │                           - Step-CA (Tier 3)
     │                           - ML Engine (Tier 7)
     │                           - Guardian Agent (Tier 8)
     │
     ├── VLAN 20 (TRUSTED)   → Workstations, laptops
     ├── VLAN 30 (IOT)       → Smart home devices
     ├── VLAN 40 (DMZ)       → Public services, honeypots
     ├── VLAN 50 (AI_LAB)    → LLM workloads, AI experiments
     ├── VLAN 60 (GUEST)     → Guest WiFi
     └── VLAN 70 (QUARANTINE)→ Isolated compromised devices
```

---

## 5. Compliance & Framework Alignment

This architecture aligns with:

| Framework | Coverage |
|-----------|---------|
| NIST CSF 2.0 | Identify · Protect · Detect · Respond · Recover |
| NIST SP 800-53 Rev 5 | AC, AU, CM, IA, IR, RA, SC, SI controls |
| MITRE ATT&CK | Full TTP coverage mapped to detection rules |
| MITRE ATLAS | Adversarial ML TTP coverage (Tier 4 + Tier 8) |
| Zero Trust Architecture (NIST SP 800-207) | Tier 2 implementation |
| CIS Controls v8 | Implementation Groups 1, 2, 3 |

---

## 6. Threat Model Summary

See `THREAT-MODEL.md` for full adversary profiles.

**Primary Adversaries Modeled:**
1. **Automated AI Attack Agent** — LLM-orchestrated scanning, exploit, and exfiltration
2. **Prompt Injection Attacker** — Targeting AI pipelines in the AI_LAB VLAN
3. **Supply Chain Attacker** — Compromised dependencies, typosquatting, poisoned models
4. **APT-Level Human Attacker** — Nation-state TTPs, living-off-the-land, patience
5. **Insider/Compromised Device** — Lateral movement from compromised trusted endpoint
6. **RAG Poisoning Attacker** — Adversarial document injection targeting retrieval systems

---

*AEGIS-ZERO Architecture Specification v1.0.0 — AetherHorizon 🦉*
