# AEGIS-ZERO AI Use and Governance Policy

> **Document Type:** Governance — AI Use Policy  
> **Framework Alignment:** NIST AI RMF 1.0 GV-1.1 through GV-6.2 · NIST AI 600-1 · NIST CSF 2.0 GV.PO  
> **Owner:** AetherHorizon  
> **Version:** 1.0.0  
> **Review Cadence:** Quarterly, or after any AI-related security incident or major model update

---

## 1. Purpose

This policy governs the design, deployment, operation, and monitoring of all AI and ML systems
within the AEGIS-ZERO ecosystem. It establishes principles for trustworthy AI, defines the
risk management approach for AI-specific threats, and operationalizes relevant controls from
NIST AI RMF 1.0 and NIST AI 600-1 (Generative AI / Agentic AI Profile).

**Primary AI Systems Covered:**
- **Guardian Agent (Tier 8)** — LangGraph-based agentic AI security system
- **Tier 7 ML Detection Models** — Isolation Forest, LSTM Autoencoder, DNS entropy model, UEBA
- **AI Honeypot API** — FastAPI service serving synthetic LLM responses for adversary profiling
- **Any LLM or foundation model** running on the AI_LAB VLAN (VLAN 50)

---

## 2. Foundational AI Governance Principles

AEGIS-ZERO adopts the seven characteristics of trustworthy AI as defined in NIST AI RMF 1.0:

| Characteristic | Implementation in AEGIS-ZERO |
|---|---|
| **Valid and Reliable** | Models evaluated against test cases at every phase; MLflow tracks performance over time |
| **Safe** | Guardian Agent cannot take CRITICAL containment actions without explicit human approval (HITL gate) |
| **Secure and Resilient** | Guardian protected by its own multi-stage prompt firewall; isolated to AI_LAB VLAN |
| **Accountable and Transparent** | Every Guardian decision written to immutable SQLite audit log; full decision trace available |
| **Explainable and Interpretable** | Isolation Forest provides feature importance; Guardian audit log explains each classification |
| **Privacy-Enhanced** | AI systems operate on network metadata, not personal content; minimal data retention |
| **Fair with Bias Managed** | Security classification context; fairness in the traditional ML sense is not applicable; bias = false positive/negative rate, monitored via MLflow |

---

## 3. AI Risk Classification

AI systems within AEGIS-ZERO are classified by risk level, determining the governance
requirements that apply.

### 3.1 Risk Tiers

| Risk Tier | Criteria | AEGIS-ZERO Systems |
|---|---|---|
| **Tier A — Critical** | Can take autonomous actions with real-world consequences (network changes, certificate revocation, device quarantine) | Guardian Agent (containment-capable nodes) |
| **Tier B — High** | Provides recommendations that directly drive human decisions or automated playbooks | Guardian threat classifier; Tier 7 scoring service |
| **Tier C — Medium** | Provides intelligence or enrichment; no direct action capability | Tier 7 ML models; UEBA baselines |
| **Tier D — Low** | Passive monitoring; read-only; no downstream decision dependency | AI Honeypot API response synthesis |

### 3.2 Governance Requirements by Tier

| Requirement | Tier A | Tier B | Tier C | Tier D |
|---|---|---|---|---|
| HITL gate for CRITICAL actions | **REQUIRED** | N/A | N/A | N/A |
| Audit log for all decisions | **REQUIRED** | **REQUIRED** | Recommended | Optional |
| Performance monitoring | **REQUIRED** | **REQUIRED** | **REQUIRED** | Recommended |
| Test suite in CI | **REQUIRED** | **REQUIRED** | **REQUIRED** | Recommended |
| Rollback procedure documented | **REQUIRED** | **REQUIRED** | Recommended | Optional |
| Model update approval | Owner approval | Owner review | Self-service | Self-service |

---

## 4. The Guardian Agent — Specific Governance

The Guardian Agent (Tier 8) is the highest-risk AI system in the ecosystem. Its governance
requirements are the most stringent.

### 4.1 Authorized Actions

The Guardian Agent is authorized to take the following actions **autonomously** (without
human approval) at the specified severity thresholds:

| Action | Maximum Autonomous Severity | Higher Severity Requires |
|--------|----------------------------|------------------------|
| LOG threat event to Wazuh | Any | — |
| LOG to immutable audit log | Any | — |
| BLOCK prompt (Prompt Firewall) | HIGH | Human confirmation for CRITICAL |
| QUARANTINE RAG document | HIGH | Human confirmation for CRITICAL |
| DENY tool call (Tool Auditor) | HIGH | Human confirmation for CRITICAL |
| PUSH IOC to MISP | MEDIUM | Review recommended for HIGH+ |
| TRIGGER SOAR playbook | MEDIUM | Human approval for HIGH+ |
| NETWORK ISOLATION (VLAN 70) | — | **ALWAYS requires human approval** |
| CERTIFICATE REVOCATION | — | **ALWAYS requires human approval** |

### 4.2 Human-in-the-Loop Gate

The HITL gate (`nodes/human_escalation.py`) is **non-bypassable**. The Guardian Agent
must halt and await explicit human confirmation before executing any action at or above
the CRITICAL threshold. This is not a configurable preference — it is hardcoded into the
LangGraph graph topology as a conditional edge with no bypass path.

**Escalation notification:** When the HITL gate is triggered, the operator receives:
- Wazuh HIGH/CRITICAL alert with full threat context
- Guardian Streamlit dashboard alert with one-click approve/deny
- (Future) SMS/push notification via configured alerting channel

**Timeout behavior:** If no human response is received within 15 minutes:
- The action is **DENIED** by default (fail-safe posture)
- A CRITICAL Wazuh alert is generated: "Guardian HITL timeout — action blocked"
- The threat is logged with full context for post-incident review

### 4.3 Audit Log Requirements

Every Guardian Agent decision — allow, deny, escalate, or sandbox — **MUST** be written to
the immutable SQLite audit log with the following fields:

```python
class GuardianAuditRecord(BaseModel):
    event_id: str              # UUID
    timestamp: datetime        # UTC ISO 8601
    input_hash: str            # SHA-256 of the input (not plaintext — privacy)
    decision: Literal["ALLOW", "DENY", "SANDBOX", "ESCALATE"]
    threat_type: str           # e.g., "PROMPT_INJECTION", "SEMANTIC_DRIFT"
    confidence: float          # 0.0 - 1.0
    severity: Literal["INFO", "LOW", "MEDIUM", "HIGH", "CRITICAL"]
    firewall_stage: str        # Which detection node triggered
    action_taken: str          # Specific action executed
    human_approved: bool       # Was HITL gate triggered?
    human_decision: Optional[str]  # If HITL: "APPROVED" or "DENIED"
    wazuh_alert_id: Optional[str]  # Correlated Wazuh event
```

Audit log records are **append-only**. The table is created with a trigger that prevents
UPDATE or DELETE operations. Regular integrity checks verify the log has not been tampered with.

### 4.4 Prompt Injection Defense — This System Itself

The Guardian Agent defends against prompt injection attacks directed at other AI systems.
It is itself a target for prompt injection. The following additional controls apply specifically
to the Guardian Agent's own inputs:

- All inputs processed by the Guardian Agent pass through the Prompt Injection Firewall first
- The Guardian Agent has no ability to modify its own system prompt, configuration, or policy engine
- The Guardian Agent's tool access is bounded and audited — it cannot grant itself additional permissions
- Attempts to instruct the Guardian Agent via injected content to "ignore previous instructions" or "act as a different system" are classified as injection attempts with HIGH severity

---

## 5. AI/ML Model Lifecycle Governance

### 5.1 Model Introduction

A new ML model may be introduced to production only when:
- [ ] Test suite passes with documented performance metrics
- [ ] Baseline performance is recorded in MLflow
- [ ] Adversarial test cases have been run (can the model be evaded?)
- [ ] Rollback procedure is documented
- [ ] Model weight hash is added to `models/integrity-manifest.sha256`

### 5.2 Model Monitoring

All production models must be monitored for:
- **Performance drift:** Alert rate changes > 2σ from 30-day baseline
- **Data drift:** Input feature distributions shift significantly
- **Integrity:** SHA-256 hash verified on container startup

Monitoring frequency:
- Real-time: Integrity checks (on startup)
- Daily: Performance metric review via MLflow
- Monthly: Formal model review against adversarial test cases

### 5.3 Model Retirement

A model is retired when:
- A newer version passes all test criteria and the 7-day parallel operation period
- The model produces unacceptable false positive or false negative rates
- The model's training data is found to be compromised (see SUPPLY-CHAIN-POLICY.md § 5.4)

Retired model weights are archived (not deleted) for forensic reference.

---

## 6. LLM / Foundation Model Usage

### 6.1 Approved Models

| Use Case | Approved Models | Not Approved |
|---|---|---|
| Guardian Agent reasoning (Tier A) | Claude claude-sonnet-4-20250514 (Anthropic API) | Unknown/unvetted models |
| Guardian Agent screening (Tier B/C) | Ollama: llama3.2, mistral (local) | Cloud models without data agreements |
| Offline / air-gapped operations | Ollama local models only | External API (no connectivity) |

**Model substitution requires owner approval.** Do not swap the Guardian Agent's LLM backend
without reviewing the impact on prompt injection detection performance.

### 6.2 Data Sent to External LLMs

The following data **MUST NOT** be sent to external LLM APIs:
- Network packet payloads or content
- Personal identifiable information (PII) from any source
- Internal IP addresses or network topology details
- Security tool credentials or configuration
- Threat intelligence that is confidential to a sharing agreement

Only anonymized/abstracted security event metadata may be sent to external APIs.

### 6.3 Prompt Engineering Standards

All system prompts used in the Guardian Agent:
- Are version-controlled in the repository
- Do not rely on secrecy for security (assume prompts may be extracted)
- Implement defense-in-depth: the prompt is one layer; the multi-stage firewall is another
- Are tested against known jailbreak patterns before deployment
- Define explicit refusal behaviors for unauthorized action requests

---

## 7. AI-Specific Incident Response

If an AI system behaves unexpectedly or is suspected to be compromised:

### Indicators of Compromise (AI-Specific)

| Indicator | Likely Cause | Response |
|---|---|---|
| Guardian Agent alert rate drops to near-zero | Model poisoning / evasion | CRITICAL — treat as active attack |
| Guardian Agent DENIES all inputs | Possible model corruption | HIGH — restart from clean image |
| Unexpected tool calls in audit log | Prompt injection success | CRITICAL — review last 24h audit log |
| Model weight hash mismatch on startup | Supply chain compromise | CRITICAL — do not start; restore from backup |
| HITL gate bypassed | Code integrity failure | CRITICAL — immediate shutdown |

### Response Procedure

1. **Isolate:** Move AI system container to quarantine network or stop container
2. **Preserve:** Export audit log immediately (immutable; preserve chain of custody)
3. **Investigate:** Review Guardian audit log for the incident window
4. **Root cause:** Determine whether input-based attack, supply chain, or code issue
5. **Remediate:** Rebuild from clean, verified base image with hash-verified dependencies
6. **Validate:** Run full test suite before reintroducing to production
7. **Document:** Record in RISK-REGISTER.md; update this policy if gap identified

---

## 8. Prohibited AI Uses

The following uses of AI are explicitly prohibited within the AEGIS-ZERO ecosystem:

1. **Offensive AI use** — Do not use any AI system to attack, probe, or test systems you do not own
2. **Deceptive use** — Do not use AI Honeypot to deceive authorized users
3. **Privacy violation** — Do not use AI to process personal communications content
4. **Unsupervised CRITICAL actions** — Never disable or bypass the HITL gate
5. **Unaudited AI** — No AI system may operate outside the audit log framework
6. **CSAM / NCII** — Absolute prohibition; not contextually relevant but stated for completeness

---

## 9. Roles and Responsibilities

| Role | Responsibility |
|------|---------------|
| **Owner/Operator (AetherHorizon)** | All AI governance responsibilities; approves model changes; reviews HITL escalations |
| **Guardian Agent (AI System)** | Autonomous threat detection and classification within defined policy bounds |
| **HITL Reviewer** | Owner/Operator when Guardian escalates a CRITICAL action for human decision |

For a single-operator deployment, these roles converge. The separation exists to maintain
conceptual clarity for future multi-operator environments and for portfolio documentation purposes.

---

## 10. Policy Change Log

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2025-01-01 | Initial policy — Phase 0 |

---

*AEGIS-ZERO AI Use and Governance Policy v1.0.0 — AetherHorizon 🦉*
