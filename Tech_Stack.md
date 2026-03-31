# AEGIS-ZERO Technology Stack

> Technology decisions documented with rationale.  
> Every choice is justified against the threat model and operational constraints.

---

## Stack Selection Philosophy

1. **Open source preferred** — auditability over convenience; no closed-source security theater
2. **Production-grade only** — tools must be used in real enterprise and government deployments
3. **Active maintenance required** — abandoned projects are attack surface, not protection
4. **Minimal attack surface** — fewer components = fewer vulnerabilities; integrate where possible
5. **Observable by default** — every component must expose logs/metrics in a standard format

---

## Tier 1 — Perimeter Defense

### OPNsense (Primary Firewall)
- **Chosen over:** pfSense, commercial NGFW (Fortinet, Palo Alto), iptables/nftables
- **Rationale:** OPNsense is the most actively maintained open-source NGFW platform. FreeBSD base
  provides a minimal attack surface. Strongly typed configuration via XML API enables automation.
  HardenedBSD option available for maximum kernel hardening. Business Edition available if
  commercial support becomes required.
- **Key capabilities:** Stateful firewall, NAT, VPN (WireGuard/OpenVPN), traffic shaping,
  VLAN management, HA failover, Suricata integration, CrowdSec integration.
- **Version:** 24.x (quarterly updates)

### Suricata (IDS/IPS)
- **Chosen over:** Snort, Zeek (for inline IPS), Bro
- **Rationale:** Suricata is the de facto standard for high-performance IDS/IPS. Multi-threaded
  architecture handles gigabit traffic without packet loss. Natively integrated with OPNsense.
  Supports PCRE, Lua scripting, and file extraction. OISF (Open Information Security Foundation)
  maintains it with US government backing.
- **Mode:** Inline IPS (drops malicious traffic; does not just alert)
- **Rule sources:** ET Open (base), custom ATLAS-derived rules (Phase 3)

### CrowdSec (Distributed Threat Intelligence)
- **Chosen over:** Fail2ban (alone), commercial threat intel feeds
- **Rationale:** CrowdSec combines local behavioral analysis with a crowdsourced IP reputation
  network of millions of contributors. When your IDS triggers on an IP, CrowdSec validates
  against its community consensus and blocks accordingly. OPNsense bouncer integration is native.
- **Key feature:** Community threat intelligence that improves as the global user base grows.

---

## Tier 2 — Network Segmentation

### 802.1Q VLAN (Network Segmentation)
- **Standard:** IEEE 802.1Q (industry universal)
- **Implementation:** Managed switch (Ubiquiti UniFi, Netgear, Cisco — any 802.1Q capable)
- **Rationale:** Hardware-enforced segmentation. VLANs are the industry standard for network
  microsegmentation. Combined with OPNsense inter-VLAN firewall rules, each segment is
  an independent security domain.

### WireGuard (VPN/ZTNA Transport)
- **Chosen over:** OpenVPN, IPsec, Cisco AnyConnect
- **Rationale:** WireGuard has the smallest codebase of any production VPN (~4,000 lines vs
  OpenVPN's ~100,000+). Smaller codebase = smaller attack surface = easier to audit. Modern
  cryptography (Noise protocol, ChaCha20, Curve25519). Mainlined into Linux kernel 5.6+.
  Performance far exceeds OpenVPN. Used by Cloudflare, Mullvad, and in enterprise deployments.
- **Use case:** All administrative access to MGMT VLAN goes through WireGuard, even from local
  network. This enforces Zero Trust — network location grants nothing.

---

## Tier 3 — DNS Security & PKI

### Pi-hole (DNS Sinkhole)
- **Chosen over:** AdGuard Home, commercial DNS filtering (Cisco Umbrella, Zscaler)
- **Rationale:** Pi-hole is the most widely deployed open-source DNS sinkhole. Massive community
  block list ecosystem. Integrates with Unbound for full recursive resolution. Lightweight
  enough to run on a Raspberry Pi or in Docker. MISP integration enables automated threat-intel-
  driven block list updates.
- **Enhancement:** Feeds from MISP (Tier 4) automatically update Pi-hole block lists with fresh IOCs.

### Unbound (Recursive DNS Resolver)
- **Chosen over:** dnsmasq (for recursive), public resolvers (8.8.8.8, 1.1.1.1)
- **Rationale:** Unbound performs full recursive DNS resolution with DNSSEC validation. No data
  leaves the network in plaintext. DNSSEC prevents DNS cache poisoning. Paired with DoT/DoH
  upstream for the final hop.

### Step-CA by Smallstep (Internal PKI)
- **Chosen over:** HashiCorp Vault PKI, cfssl, OpenSSL manual CA
- **Rationale:** Step-CA is the most operationally friendly internal PKI for the automation
  level required here. Native ACME protocol support means certificates auto-renew without
  manual intervention. OIDC integration for human authentication. Excellent CLI tooling.
  Used in production at Cloudflare, Datadog, and others.
- **Architecture:** Air-gapped root CA (USB, offline) → Step-CA intermediate CA (online, MGMT VLAN)

---

## Tier 4 — Threat Intelligence

### MISP (Malware Information Sharing Platform)
- **Rationale:** MISP is the global standard for threat intelligence sharing. Used by national
  CERTs, ISACs, government agencies, and enterprises worldwide. STIX/TAXII support enables
  interoperability with all major CTI platforms. Feeds can be ingested automatically from
  dozens of OSINT sources. The custom MISP → Pi-hole pipeline enables real-time IOC-driven
  DNS blocking.
- **Deployment:** Docker stack on MGMT VLAN server

### OpenCTI (Open Cyber Threat Intelligence)
- **Rationale:** OpenCTI structures threat intelligence in STIX 2.1 format. While MISP excels
  at IOC management, OpenCTI excels at structured adversary intelligence — actor profiles,
  campaign tracking, TTP attribution. Filigran (the maintainer) has strong EU government and
  enterprise backing. Directly integrates with MISP via TAXII.
- **Deployment:** Docker stack on MGMT VLAN server (same server as MISP)

### ATLAS-to-Wazuh Converter (Custom)
- **Why custom?** MITRE ATLAS (Adversarial Threat Landscape for AI Systems) does not have a
  native Wazuh rule exporter. This is an original AetherHorizon tool that bridges the gap
  between ATLAS adversarial ML TTPs and operational Wazuh detection rules.
- **Technology:** Python 3.12, Pydantic v2, Jinja2 templating, MITRE STIX API client
- **Output:** Valid Wazuh XML rule files ready for direct deployment

---

## Tier 5 — SIEM / SOAR

### Wazuh (SIEM + EDR + Compliance)
- **Chosen over:** Splunk (cost), IBM QRadar (cost), Elastic SIEM (complexity), Graylog
- **Rationale:** Wazuh is the most capable open-source SIEM available. It combines SIEM
  (log ingestion, correlation, alerting), EDR (endpoint agents, FIM, rootkit detection),
  and compliance monitoring (HIPAA, PCI-DSS, GDPR, NIST) in a single platform. Used by
  organizations worldwide including government agencies. OpenSearch-based indexer means
  full Kibana-compatible visualization. Active development with regular updates.
- **Stack:** Wazuh Manager + Wazuh Indexer (OpenSearch) + Wazuh Dashboard (Kibana)

### Custom Python SOAR Playbooks
- **Chosen over:** Cortex XSOAR, Splunk SOAR, IBM SOAR
- **Rationale:** Custom playbooks give complete control, zero licensing cost, and allow tight
  integration with every component in the ecosystem. Built on Python + the APIs of each
  platform. SOAR playbooks are simple Python scripts triggered by Wazuh active response hooks.

---

## Tier 6 — Endpoint Detection & Response

### osquery (Endpoint Telemetry)
- **Rationale:** osquery exposes the operating system as a relational database. This enables
  SQL-based querying of processes, network connections, loaded modules, file changes, users,
  and hundreds of other endpoint attributes. Originally developed by Facebook; now maintained
  by the Linux Foundation. Industry standard for endpoint visibility.
- **Integration:** osquery → Wazuh via osquery logger daemon

### Auditd (Linux) / Sysmon (Windows)
- **Rationale:** Kernel-level event capture that cannot be bypassed by userspace malware.
  Auditd provides Linux syscall auditing. Sysmon provides Windows process/network/file events.
  Both feed into Wazuh for correlation.

---

## Tier 7 — AI/ML Detection

### Zeek (Network Analysis Framework)
- **Chosen over:** Wireshark (not real-time), ntopng (less programmable), Arkime
- **Rationale:** Zeek (formerly Bro) is the gold standard for network metadata extraction.
  Used by national security agencies, research institutions, and major enterprises. Zeek
  generates rich structured logs (conn.log, dns.log, http.log, ssl.log, files.log) that
  become the training and inference data for ML models.

### scikit-learn Isolation Forest (Network Anomaly)
- **Rationale:** Isolation Forest is the most computationally efficient algorithm for
  unsupervised anomaly detection on tabular data (network flows). Fast inference, no labeled
  data required for training, interpretable anomaly scores. Production-ready via scikit-learn.

### PyTorch LSTM Autoencoder (Temporal Anomaly)
- **Rationale:** Network attacks have temporal signatures — the sequence of events matters.
  An LSTM autoencoder learns to compress and reconstruct normal event sequences. Anomalous
  sequences produce high reconstruction error, triggering alerts. PyTorch provides production-
  grade deep learning with excellent deployment options (TorchScript, ONNX export).

### FastAPI (Scoring Microservice)
- **Rationale:** FastAPI provides the fastest Python HTTP server available. Async by default.
  Automatic OpenAPI documentation. Used in production at Netflix, Microsoft, and Uber.
  The scoring microservice needs sub-50ms response time; FastAPI achieves this easily.

---

## Tier 8 — Agentic AI Guardian

### LangGraph (Agent Orchestration)
- **Chosen over:** CrewAI, AutoGen, raw LangChain chains
- **Rationale:** LangGraph provides stateful, graph-based agent orchestration with explicit
  control flow. This is critical for security — the Guardian Agent's decision flow must be
  auditable, deterministic, and interruptible. LangGraph's conditional edges and human-in-the-
  loop support are first-class features, not afterthoughts. State is Pydantic-typed.

### Claude / OpenAI API (LLM Backend)
- **Primary:** Claude claude-sonnet-4-20250514 (via Anthropic API) — chosen for superior instruction
  following and safety alignment, critical for a security agent
- **Secondary:** Local Ollama (llama3.2, mistral) — for air-gapped operations and high-throughput
  screening tasks that don't require frontier model capability

### Pydantic v2 (Data Validation)
- **Rationale:** All Guardian state, threat classifications, and audit records are Pydantic
  models. Type safety is non-negotiable in a security system. Pydantic v2's Rust core provides
  the performance needed for real-time threat processing.

### FastAPI (Honeypot API)
- **Rationale:** Same as Tier 7. The AI Honeypot needs to convincingly serve LLM API responses
  at realistic latencies to adversarial probers. FastAPI + async response synthesis achieves this.

### SQLite + SQLAlchemy (Audit Log)
- **Rationale:** The audit log must be immutable, local, and fast. SQLite is embedded (no
  network attack surface), supports WAL mode for concurrent writes, and is suitable for the
  log volumes expected at home network scale. Append-only table design enforces immutability.

### Streamlit (Guardian Dashboard)
- **Rationale:** Rapid dashboard development with zero frontend complexity. The Guardian dashboard
  provides real-time visibility into AI threat activity, Guardian decisions, and audit logs.

---

## Infrastructure & Supporting Tools

| Component | Choice | Rationale |
|-----------|--------|-----------|
| Containerization | Docker + Docker Compose | Universal, auditable, reproducible |
| Infrastructure as Code | Ansible | Agentless, simple, Python ecosystem |
| Secret Management | Doppler / Vault (future) | Environment-variable injection |
| Log Transport | Wazuh agents + syslog | Native, reliable, encrypted |
| Time Sync | NTP (chrony) | Critical for log correlation |
| Backup | Restic + rclone | Encrypted, deduplicated, portable |
| Monitoring | Prometheus + Grafana | Container and system metrics |
| Python Version | 3.12 | Latest stable, full type hint support |
| Package Management | pip + requirements.txt | Simple, auditable dependencies |
| Testing | pytest | Industry standard |

---

## What Was Explicitly Rejected

| Rejected | Why |
|----------|-----|
| Commercial NGFW | Cost, closed-source, vendor lock-in |
| Windows Server | Attack surface, cost, unnecessary complexity |
| Cloud-hosted SIEM (Datadog, Splunk Cloud) | Data sovereignty; all security data stays on-premise |
| PAN-OS / Fortinet | Cost; OPNsense meets the threat model |
| OpenVPN | WireGuard is objectively superior on every metric |
| Manual PKI (raw OpenSSL) | Operational burden; Step-CA automates safely |
| Kubernetes | Unnecessary complexity at this scale; Docker Compose is sufficient |
| GPT-4 only for Guardian | Vendor concentration risk; multi-model approach |
| Proprietary threat feeds only | OSINT + MISP community feeds provide equivalent coverage |

---

*AEGIS-ZERO Technology Stack v1.0.0 — AetherHorizon 🦉*
