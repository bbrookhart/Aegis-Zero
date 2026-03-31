# AEGIS-ZERO Build Phases

> Each phase is fully functional before the next begins.  
> No phase proceeds without architect review and approval.  
> Every phase produces working, deployed, documented deliverables.

---

## Phase Dependency Map

```
Phase 0 (Planning)
    └── Phase 1 (Perimeter)
            └── Phase 2 (Zero Trust + DNS)
                    └── Phase 3 (Threat Intel)
                            └── Phase 4 (SIEM/SOAR)
                                    └── Phase 5 (EDR)
                                            └── Phase 6 (AI Detection)
                                                    └── Phase 7 (Guardian)
```

Each phase assumes all previous phases are operational. Do not skip phases.

---

## PHASE 0 — Planning & Architecture

**Status:** ✅ Complete  
**Duration:** 1 session  
**Approval Gate:** Owner review of all planning documents

### Deliverables

- [x] `README.md` — Project overview and mission
- [x] `ARCHITECTURE.md` — Full 8-tier architecture specification
- [x] `PHASES.md` — This document
- [x] `TECH-STACK.md` — Technology decisions with rationale
- [x] `THREAT-MODEL.md` — Adversary profiles and attack surface analysis

### Success Criteria

- Architecture clearly defines all 8 tiers
- Technology stack decisions are justified against threat model
- VLAN segmentation plan is complete and coherent
- All phases have clear deliverables and success criteria

---

## PHASE 1 — Perimeter Defense

**Status:** 🔄 Ready to Build  
**Estimated Duration:** 1-2 sessions  
**Dependencies:** Hardware appliance available; ISP modem in bridge mode

### Objectives

Deploy a hardened perimeter firewall with inline IDS/IPS and distributed threat intelligence.
Replace consumer router with OPNsense. Begin logging all network events.

### Deliverables

```
phase-01-perimeter/
├── README.md                           # Phase overview and deployment steps
├── opnsense/
│   ├── config-baseline.xml             # OPNsense baseline configuration export
│   ├── firewall-rules.md               # Human-readable firewall rule documentation
│   └── hardening-checklist.md          # OPNsense hardening steps
├── suricata/
│   ├── suricata.yaml                   # Suricata main configuration
│   ├── rules/
│   │   ├── local.rules                 # Custom local rules
│   │   └── atlas-ai.rules              # ATLAS-derived rules (placeholder, filled Phase 3)
│   └── rule-management.md              # Rule update automation docs
├── crowdsec/
│   ├── config.yaml                     # CrowdSec agent configuration
│   ├── acquis.yaml                     # Log acquisition configuration
│   └── bouncers/
│       └── opnsense-bouncer.md         # OPNsense bouncer setup docs
├── docker/
│   └── docker-compose.yml             # Auxiliary services (GeoIP update, CrowdSec dashboard)
└── tests/
    ├── connectivity-tests.sh           # Verify allowed traffic passes
    └── block-tests.sh                  # Verify blocked traffic is denied + alerted
```

### Key Configuration Tasks

1. Install OPNsense on dedicated appliance
2. Configure WAN interface (replace ISP router)
3. Create trunk port to managed switch (all VLANs)
4. Deploy initial firewall ruleset (default-deny)
5. Install and configure Suricata in IPS mode (inline, not just IDS)
6. Enable ET Open rule set + schedule daily updates
7. Install CrowdSec agent + OPNsense bouncer
8. Configure syslog forwarding to MGMT VLAN (Tier 5 prep)
9. Enable GeoIP blocking for high-risk countries
10. Configure management interface restrictions

### Success Criteria (Approval Gate)

- [ ] OPNsense is the default gateway for all internal traffic
- [ ] Suricata is operating in IPS mode (drops, not just alerts)
- [ ] Test attack traffic (safe, benign scans) generates Suricata alerts
- [ ] CrowdSec bouncer is blocking known-bad IPs
- [ ] All firewall events are being logged to syslog
- [ ] Management interface is accessible only from MGMT VLAN IP range
- [ ] Default-deny inbound policy is confirmed via external port scan
- [ ] `connectivity-tests.sh` passes for all expected traffic
- [ ] `block-tests.sh` confirms expected denials trigger alerts

---

## PHASE 2 — Zero Trust Network Access & DNS Security

**Status:** ⏳ Pending Phase 1 Approval  
**Estimated Duration:** 1-2 sessions  
**Dependencies:** Phase 1 complete; managed switch with 802.1Q support

### Objectives

Implement full VLAN segmentation per the architecture spec. Deploy Pi-hole DNS sinkhole.
Establish internal PKI with Step-CA. Configure WireGuard for privileged access. Enable DoH/DoT.

### Deliverables

```
phase-02-zero-trust/
├── README.md
├── vlans/
│   ├── vlan-design.md                  # VLAN map and policy documentation
│   ├── switch-config.md                # Managed switch configuration guide
│   └── inter-vlan-rules.md             # OPNsense inter-VLAN firewall rules
├── wireguard/
│   ├── server-setup.md                 # WireGuard server configuration
│   ├── wg0.conf.template               # Server config template
│   └── peer-provisioning.sh            # Peer certificate + config generator
├── pki/
│   ├── step-ca/
│   │   ├── ca-setup.md                 # Step-CA installation and configuration
│   │   ├── ca.json                     # Step-CA configuration file
│   │   └── provisioners.json           # ACME + JWK provisioner config
│   ├── root-ca-ceremony.md             # Air-gapped root CA key generation procedure
│   └── certificate-policies.md        # Cert profiles, validity, renewal policy
├── dns/
│   ├── pihole/
│   │   ├── docker-compose.yml          # Pi-hole deployment
│   │   ├── custom-blocklists.txt       # AetherHorizon curated block lists
│   │   └── pihole.env                  # Environment configuration
│   ├── unbound/
│   │   ├── unbound.conf                # Recursive resolver configuration
│   │   └── forward-records.conf        # DoT forwarding configuration
│   └── doh-setup.md                    # DNS-over-HTTPS configuration
├── mtls/
│   ├── nginx-mtls.conf                 # Nginx mTLS termination template
│   └── mtls-testing.md                # mTLS verification steps
└── tests/
    ├── vlan-isolation-tests.sh         # Confirm VLAN separation
    ├── dns-tests.sh                    # Confirm DNS filtering works
    └── pki-tests.sh                   # Verify cert chain and mTLS
```

### Success Criteria (Approval Gate)

- [ ] All 7 VLANs are active and correctly tagged on the switch
- [ ] VLAN isolation confirmed: IOT cannot reach TRUSTED (and vice versa)
- [ ] Pi-hole is the DNS server for all VLANs
- [ ] DNS queries to known malicious domains are blocked and logged
- [ ] DoH/DoT upstream is configured and confirmed via tcpdump (no plaintext DNS leaving network)
- [ ] Step-CA is operational and issuing certificates
- [ ] WireGuard tunnel established from at least one test peer
- [ ] mTLS handshake verified between two internal services
- [ ] All DNS queries are being forwarded to Wazuh (Tier 5 prep)

---

## PHASE 3 — Cyber Threat Intelligence Platform

**Status:** ⏳ Pending Phase 2 Approval  
**Estimated Duration:** 1-2 sessions  
**Dependencies:** Phase 2 complete; MGMT VLAN server with Docker installed

### Objectives

Deploy MISP and OpenCTI. Build the ATLAS-to-Wazuh TTP converter. Establish automated
threat feed ingestion. Create the intelligence pipeline that feeds all downstream tiers.

### Deliverables

```
phase-03-threat-intel/
├── README.md
├── misp/
│   ├── docker-compose.yml              # MISP deployment stack
│   ├── misp.env                        # Environment variables
│   ├── feed-config.json                # Automated feed subscriptions
│   └── misp-hardening.md               # MISP security configuration
├── opencti/
│   ├── docker-compose.yml              # OpenCTI deployment stack
│   ├── opencti.env                     # Environment variables
│   └── opencti-setup.md                # Initial configuration guide
├── atlas-to-wazuh/
│   ├── README.md                       # Tool documentation
│   ├── atlas_to_wazuh.py               # Main converter script
│   ├── models/
│   │   ├── atlas_technique.py          # ATLAS TTP data model (Pydantic)
│   │   └── wazuh_rule.py               # Wazuh rule data model (Pydantic)
│   ├── templates/
│   │   └── wazuh_rule.xml.j2           # Jinja2 rule template
│   ├── output/
│   │   └── atlas_rules.xml             # Generated Wazuh rules (auto-updated)
│   ├── tests/
│   │   └── test_converter.py           # Unit tests
│   └── requirements.txt
├── threat-feed-ingest/
│   ├── ingest_pipeline.py              # Orchestrated feed aggregation
│   ├── sources/
│   │   ├── abusech.py                  # Abuse.ch integration
│   │   ├── alienvault.py               # OTX integration
│   │   ├── emerging_threats.py         # ET feed integration
│   │   └── phishtank.py               # PhishTank integration
│   ├── scheduler.py                    # Cron-like scheduling (APScheduler)
│   └── requirements.txt
├── docker/
│   └── docker-compose.yml             # Combined CTI stack (MISP + OpenCTI)
└── tests/
    ├── test_misp_connection.py
    ├── test_opencti_connection.py
    └── test_atlas_converter.py
```

### Success Criteria (Approval Gate)

- [ ] MISP is operational and ingesting at least 3 automated threat feeds
- [ ] OpenCTI is operational and connected to MISP via STIX/TAXII
- [ ] `atlas_to_wazuh.py` successfully converts the full ATLAS TTP database to Wazuh rules
- [ ] Generated Wazuh rules are valid XML and load without errors
- [ ] At least one live threat feed IOC is visible in MISP dashboard
- [ ] Threat feed ingest pipeline runs on schedule without errors
- [ ] All tests in `tests/` pass

---

## PHASE 4 — SIEM / SOAR

**Status:** ⏳ Pending Phase 3 Approval  
**Estimated Duration:** 1-2 sessions  
**Dependencies:** Phase 3 complete; all log sources from Tiers 1-3 available

### Objectives

Deploy Wazuh as the central SIEM. Load all custom rules including ATLAS-derived rules.
Build and test automated SOAR playbooks. Establish the alert severity matrix and escalation paths.

### Deliverables

```
phase-04-siem-soar/
├── README.md
├── wazuh/
│   ├── docker-compose.yml              # Wazuh manager + indexer + dashboard
│   ├── wazuh.env                       # Environment configuration
│   ├── ossec.conf                      # Wazuh manager main config
│   └── rules/
│       ├── local_rules.xml             # Custom correlation rules
│       ├── atlas_rules.xml             # ATLAS-generated rules (from Phase 3)
│       ├── ai_threat_rules.xml         # AI-specific threat rules
│       └── network_rules.xml           # Custom network anomaly rules
├── playbooks/
│   ├── README.md
│   ├── base/
│   │   └── playbook_base.py            # Abstract base class for all playbooks
│   ├── playbook_ip_block.py            # IP blocking via CrowdSec + OPNsense
│   ├── playbook_device_quarantine.py   # Move device to VLAN 70
│   ├── playbook_dns_sinkhole.py        # Push IOC to Pi-hole blocklist
│   ├── playbook_cert_revoke.py         # Revoke device cert via Step-CA
│   ├── playbook_ai_threat_response.py  # Escalate to Guardian Agent
│   └── tests/
│       └── test_playbooks.py
├── dashboards/
│   ├── overview-dashboard.json         # Kibana/OpenSearch dashboard export
│   ├── threat-hunt-dashboard.json      # Threat hunting dashboard
│   └── ai-threats-dashboard.json       # AI-specific threat dashboard
└── tests/
    ├── test_rule_load.sh               # Verify rules load without errors
    └── test_alert_generation.sh        # Generate test events, verify alerts
```

### Success Criteria (Approval Gate)

- [ ] Wazuh manager is receiving logs from all deployed tiers (1-3)
- [ ] ATLAS-derived rules are loaded and functional
- [ ] Test events trigger appropriate alert levels
- [ ] At least 3 SOAR playbooks are functional and tested
- [ ] Kibana dashboard shows real-time event flow
- [ ] Alert → Playbook pipeline tested end-to-end
- [ ] Human escalation gate tested (CRITICAL severity → manual confirmation required)

---

## PHASE 5 — Endpoint Detection & Response

**Status:** ⏳ Pending Phase 4 Approval  
**Estimated Duration:** 1 session  
**Dependencies:** Phase 4 complete; Wazuh manager reachable from all VLANs

### Deliverables

```
phase-05-edr/
├── README.md
├── wazuh-agents/
│   ├── linux-agent-install.sh          # Linux agent deployment script
│   ├── windows-agent-install.ps1       # Windows agent deployment script
│   ├── agent-ossec.conf                # Agent-side configuration template
│   └── agent-hardening.md              # Agent security configuration
├── osquery/
│   ├── osquery.conf                    # Main osquery configuration
│   ├── packs/
│   │   ├── incident-response.conf      # IR-focused queries
│   │   ├── vuln-management.conf        # Vulnerability surface queries
│   │   ├── hardware-info.conf          # Hardware/USB telemetry
│   │   ├── network-connections.conf    # Active connection monitoring
│   │   └── startup-items.conf          # Persistence mechanism detection
│   └── osquery-wazuh-integration.py   # Forward osquery events to Wazuh
├── auditd/
│   ├── audit.rules                     # Linux audit rules (NIST-aligned)
│   └── auditd-setup.md                # Installation and configuration guide
├── sysmon/ (Windows)
│   ├── sysmon-config.xml               # SwiftOnSecurity + custom rules
│   └── sysmon-setup.md                 # Installation guide
└── tests/
    ├── test_agent_connection.sh
    └── test_fim_alert.sh               # Trigger FIM alert, verify detection
```

### Success Criteria (Approval Gate)

- [ ] Wazuh agents deployed on all primary endpoints
- [ ] osquery running on all Linux endpoints, queries executing on schedule
- [ ] FIM alert generated when test file is modified in monitored path
- [ ] Rootkit scanner completes without errors
- [ ] Auditd events visible in Wazuh dashboard
- [ ] All endpoint events appear in Wazuh SIEM with correct normalization

---

## PHASE 6 — AI/ML Behavioral Threat Detection

**Status:** ⏳ Pending Phase 5 Approval  
**Estimated Duration:** 2 sessions  
**Dependencies:** Phase 5 complete; sufficient baseline data (7+ days recommended)

### Deliverables

```
phase-06-ai-detection/
├── README.md
├── zeek/
│   ├── docker-compose.yml              # Zeek deployment (mirror port or tap)
│   ├── zeek-config/
│   │   └── local.zeek                  # Custom Zeek scripts
│   └── zeek-wazuh-integration.py       # Normalize Zeek logs → Wazuh
├── models/
│   ├── README.md
│   ├── network_anomaly/
│   │   ├── train_isolation_forest.py   # Train network flow anomaly model
│   │   ├── score_flow.py               # Real-time scoring inference
│   │   └── features.py                 # Feature engineering from Zeek logs
│   ├── temporal_anomaly/
│   │   ├── train_lstm_autoencoder.py   # Train temporal sequence model
│   │   └── score_sequence.py           # Real-time sequence scoring
│   ├── dns_anomaly/
│   │   └── dns_entropy_detector.py     # DNS tunneling / DGA detection
│   └── ueba/
│       ├── build_baselines.py          # Entity behavioral baseline construction
│       └── score_deviation.py          # Real-time deviation scoring
├── scoring-service/
│   ├── main.py                         # FastAPI scoring microservice
│   ├── routers/
│   │   ├── network.py                  # Network anomaly scoring endpoint
│   │   ├── temporal.py                 # Temporal anomaly scoring endpoint
│   │   └── ueba.py                     # UEBA scoring endpoint
│   ├── Dockerfile
│   └── requirements.txt
├── mlflow/
│   └── docker-compose.yml              # MLflow tracking server
└── tests/
    ├── test_models.py
    └── test_scoring_service.py
```

### Success Criteria (Approval Gate)

- [ ] Zeek is capturing network metadata from the monitored segment
- [ ] Isolation Forest model trained and deployed (baseline period complete)
- [ ] LSTM model trained and deployed
- [ ] DNS entropy detector is running and generating test alerts
- [ ] Scoring service is reachable from Wazuh for enrichment
- [ ] At least one synthetic anomaly detected by each model
- [ ] MLflow tracking all model versions
- [ ] AI-specific detection (LLM API anomalies) producing test alerts

---

## PHASE 7 — Agentic AI Security Guardian

**Status:** ⏳ Pending Phase 6 Approval  
**Estimated Duration:** 2-3 sessions  
**Dependencies:** Phase 6 complete; all prior tiers operational

### Deliverables

```
phase-07-guardian-agent/
├── README.md
├── guardian/
│   ├── main.py                         # Entry point
│   ├── graph.py                        # LangGraph agent graph definition
│   ├── state.py                        # Guardian state schema (Pydantic)
│   ├── nodes/
│   │   ├── ingest.py                   # Request ingestion node
│   │   ├── prompt_firewall.py          # Prompt injection detection node
│   │   ├── semantic_monitor.py         # Semantic flow monitoring node
│   │   ├── rag_trust_scorer.py         # RAG document trust scoring node
│   │   ├── tool_call_auditor.py        # Tool call validation node
│   │   ├── threat_classifier.py        # Final threat classification node
│   │   ├── human_escalation.py         # HITL gate for CRITICAL threats
│   │   └── audit_logger.py             # Immutable audit log writer
│   ├── detectors/
│   │   ├── injection/
│   │   │   ├── regex_detector.py       # Pattern-based injection detection
│   │   │   ├── semantic_detector.py    # Embedding-based injection detection
│   │   │   └── ml_classifier.py        # Fine-tuned injection classifier
│   │   └── rag_poison/
│   │       ├── provenance_checker.py   # Document source verification
│   │       ├── entropy_scorer.py       # Content entropy analysis
│   │       └── embedding_drift.py      # Embedding space drift detection
│   ├── honeypot/
│   │   ├── honeypot_api.py             # FastAPI AI honeypot server
│   │   ├── response_synthesizer.py     # Plausible decoy responses
│   │   └── adversary_profiler.py       # Attack pattern fingerprinting
│   ├── integrations/
│   │   ├── wazuh_client.py             # Push Guardian alerts to Wazuh
│   │   └── misp_client.py              # Push adversary intel to MISP
│   ├── dashboard/
│   │   └── app.py                      # Streamlit Guardian monitoring dashboard
│   ├── Dockerfile
│   └── requirements.txt
└── tests/
    ├── test_prompt_injection.py        # Injection attack test cases
    ├── test_semantic_monitor.py        # Semantic drift test cases
    ├── test_rag_trust_scorer.py        # RAG poisoning test cases
    ├── test_tool_auditor.py            # Tool misuse test cases
    └── test_honeypot.py               # Honeypot engagement test cases
```

### Success Criteria (Approval Gate)

- [ ] Guardian Agent running and processing synthetic AI traffic
- [ ] Prompt injection firewall blocks known injection patterns
- [ ] Semantic monitor detects goal-hijacking in multi-turn test conversations
- [ ] RAG trust scorer flags adversarially poisoned test documents
- [ ] Tool call auditor logs all tool invocations and blocks policy violations
- [ ] Honeypot API receives and logs synthetic probe traffic
- [ ] Adversary intel from honeypot visible in MISP
- [ ] All Guardian alerts appear in Wazuh SIEM
- [ ] Streamlit dashboard shows live Guardian activity
- [ ] HITL gate tested: CRITICAL threat generates human approval request
- [ ] All test suites pass

---

## Approval Process

After each phase:
1. Review the Success Criteria checklist
2. Run all tests in the `tests/` directory
3. Confirm all criteria are met in a review session
4. **Explicitly approve** the phase before the next build begins

> Say "**Phase N approved — proceed to Phase N+1**" to unlock the next phase.

---

*AEGIS-ZERO Phase Plan v1.0.0 — AetherHorizon 🦉*
