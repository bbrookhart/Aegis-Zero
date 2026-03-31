# AEGIS-ZERO Supply Chain Security Policy

> **Document Type:** Governance — Supply Chain Risk Management Policy  
> **Framework Alignment:** NIST CSF 2.0 GV.SC · NIST SP 800-53 SR family · MITRE ATLAS AML.T0010  
> **Owner:** AetherHorizon  
> **Version:** 1.0.0  
> **Review Cadence:** Quarterly, or after any supply chain security event

---

## 1. Purpose

This policy establishes requirements for managing cybersecurity risks arising from the
software, hardware, and AI/ML components that comprise the AEGIS-ZERO ecosystem. Supply chain
attacks — including compromised packages, poisoned ML models, malicious Docker images, and
adversarial datasets — represent a MEDIUM risk with HIGH potential impact (see RISK-REGISTER.md
R-004).

**MITRE ATLAS Coverage:** This policy directly addresses:
- AML.T0010 — ML Supply Chain Compromise
- AML.T0018 — Backdoor ML Model
- AML.T0019 — Publish Poisoned Datasets
- AML.T0020 — Poison Training Data

---

## 2. Scope

This policy applies to all software, hardware, container images, ML model weights, and
datasets used within the AEGIS-ZERO ecosystem, regardless of whether they are deployed on
MGMT, TRUSTED, or AI_LAB VLANs.

---

## 3. Software Dependency Management

### 3.1 Python Dependencies

All Python dependencies **MUST**:

- Be pinned to an exact version in `requirements.txt` (e.g., `fastapi==0.111.0`, not `fastapi>=0.100`)
- Have their hash verified via `pip install --require-hashes -r requirements.txt`
- Be generated from a hash-pinned requirements file using `pip-compile --generate-hashes`
- Be reviewed for known CVEs before any update via `pip audit`

**Generating a hash-pinned requirements file:**
```bash
pip install pip-tools
pip-compile --generate-hashes requirements.in -o requirements.txt
```

**Installing with hash verification:**
```bash
pip install --require-hashes -r requirements.txt
```

**Auditing for known CVEs:**
```bash
pip install pip-audit
pip-audit -r requirements.txt
```

### 3.2 Prohibited Dependency Patterns

The following patterns are **PROHIBITED** without explicit documented exception:

| Pattern | Risk | Alternative |
|---------|------|------------|
| `pip install X` (unpinned) | Pulls latest; vulnerable to dependency confusion | Pin to exact version + hash |
| `git+https://` dependencies | Pulls arbitrary commit; no integrity | Use released, hashed packages only |
| Packages from non-PyPI sources | No community audit | Use PyPI only; verify maintainer |
| Packages last updated > 2 years | Likely abandoned; no CVE patches | Find actively maintained alternative |
| Packages with < 100 GitHub stars (for critical paths) | Low scrutiny; higher backdoor risk | Prefer established libraries |

### 3.3 Dependency Review Process

Before adding any new dependency:
1. Check PyPI for maintainer identity and release history
2. Check GitHub repository for activity, issues, and community size
3. Run `pip-audit` on the candidate package
4. Search CVE databases for known vulnerabilities
5. Document the dependency decision in the relevant `requirements.txt` comment

---

## 4. Docker Image Policy

### 4.1 Base Image Requirements

All Docker images used in AEGIS-ZERO **MUST**:

- Use official base images from verified publishers (e.g., `python:3.12-slim`, `debian:bookworm-slim`)
- Pin to a specific digest, not just a tag:
  ```
  # WRONG — tag can be updated to different image
  FROM python:3.12-slim
  
  # RIGHT — digest is immutable
  FROM python:3.12-slim@sha256:abc123...
  ```
- Be scanned for vulnerabilities before deployment using Trivy:
  ```bash
  trivy image python:3.12-slim
  ```

### 4.2 Image Scanning

**Trivy** is the approved container image scanner. Run before any new image deployment:
```bash
# Install
brew install trivy  # macOS
# or
sudo apt-get install trivy  # Debian/Ubuntu

# Scan before use
trivy image --severity HIGH,CRITICAL <image:tag>

# Scan running containers
trivy image --input $(docker save <image> | -)
```

Fail the deployment if any CRITICAL vulnerabilities are found in the image unless
a documented exception is recorded in the deployment notes.

### 4.3 Private Registry (Future)

Long-term target: deploy a private Docker registry (Harbor or Gitea container registry)
on MGMT VLAN. All production images pulled from private registry after internal scanning.
This eliminates runtime dependency on Docker Hub availability and adds an additional
inspection point.

---

## 5. ML Model and Dataset Integrity

This section directly addresses MITRE ATLAS adversarial ML supply chain TTPs.

### 5.1 Model Weight Integrity

All ML model weight files used in Tier 7 and Tier 8 **MUST**:

1. Have their SHA-256 hash recorded at time of initial download or training
2. Be verified against the recorded hash on every container startup
3. Have their hash stored in a separate integrity manifest file:
   ```
   models/integrity-manifest.sha256
   ```
4. Be monitored by the File Integrity Monitor (FIM) configured in Tier 6

**Hash generation:**
```bash
sha256sum models/trained/isolation_forest.pkl >> models/integrity-manifest.sha256
sha256sum models/trained/lstm_autoencoder.pt  >> models/integrity-manifest.sha256
```

**Startup verification:**
```bash
sha256sum --check models/integrity-manifest.sha256
```

If hash verification fails on startup, the service **MUST NOT** start. Alert to Wazuh
with severity HIGH.

### 5.2 Model Behavioral Baselines

Deployed ML models must have documented behavioral baselines:
- Expected anomaly score range for normal traffic (Isolation Forest)
- Expected reconstruction error range for normal sequences (LSTM)
- Expected daily alert count range

Significant deviation from baseline (> 2σ) triggers model re-evaluation. Sudden large
deviations (e.g., anomaly score drops to near-zero for all traffic) should be treated as
a potential **model poisoning indicator** and escalated to HIGH severity in Wazuh.

### 5.3 RAG Document Source Policy

All documents ingested into RAG (Retrieval-Augmented Generation) systems in the AI_LAB
VLAN **MUST**:

- Come from a documented trusted source (allowlist maintained in Guardian configuration)
- Pass the RAG Trust Scorer (minimum trust threshold: configurable, default 0.75)
- Have their embedding verified against expected distribution before storage
- Be logged with full provenance metadata (source URL, ingestion timestamp, trust score)

Documents from untrusted or unknown sources are quarantined for manual review before
any LLM can retrieve them.

### 5.4 Training Dataset Provenance

If any Tier 7 models are retrained using new data:
- The data source must be documented
- Baseline model performance must be recorded before retraining
- Post-retraining performance must be evaluated against adversarial test cases
- A 7-day parallel operation period is required before switching to retrained model
- MLflow must record the full experiment lineage

---

## 6. Secrets and Credential Management

### 6.1 Never-Commit List

The following **MUST NEVER** be committed to Git:
- API keys or tokens of any kind
- Private key material (`.pem`, `.key`, `.pfx`)
- Passwords (plaintext or hashed)
- WireGuard private keys
- OPNsense configuration exports containing password hashes
- MISP auth keys
- `.env` files with populated values

The `.gitignore` at the repository root enforces this. **Run `git-secrets` or `truffleHog`
before every push.**

### 6.2 Secret Storage

| Secret Type | Storage Method |
|------------|---------------|
| API keys | Environment variables via `.env` (never committed) |
| PKI private keys | Step-CA managed; root CA offline on encrypted USB |
| WireGuard private keys | Generated on device; never leaves device |
| Docker secrets | Docker secrets or environment injection at runtime |
| Long-term secrets | HashiCorp Vault (future, Phase 8 enhancement) |

### 6.3 Rotation Schedule

| Secret Type | Rotation Frequency |
|------------|-------------------|
| API keys (external services) | 90 days |
| Device certificates (Step-CA) | Auto-renew at 60 days; 90-day max validity |
| Service certificates (mTLS) | Auto-renew at 60 days; 90-day max validity |
| WireGuard peer keys | 180 days or on suspected compromise |
| Root CA | 10 years (offline; ceremony documented) |

---

## 7. Hardware Supply Chain

### 7.1 Hardware Procurement

For security-critical hardware (firewall appliance, managed switch, MGMT VLAN server):
- Purchase from established, reputable vendors (not unknown marketplace sellers)
- Inspect for evidence of physical tampering on receipt
- Flash firmware from official vendor source before deployment
- Document serial number, purchase date, and vendor for all security hardware

### 7.2 Trusted Execution

Security-critical services run on hardware that:
- Is dedicated to security functions (not shared with untrusted workloads)
- Has Secure Boot enabled where supported
- Has full-disk encryption on the OS partition
- Is physically located in a controlled area

---

## 8. Incident Response for Supply Chain Events

If a supply chain compromise is suspected or confirmed:

1. **Immediate:** Isolate affected component to VLAN 70 (Quarantine)
2. **Immediate:** Revoke any certificates associated with affected service
3. **Within 1 hour:** Run `sha256sum --check` on all model weights and critical binaries
4. **Within 2 hours:** Review Wazuh FIM alerts for unexpected binary modifications
5. **Within 4 hours:** Determine blast radius — what could the compromised component access?
6. **Within 24 hours:** Rebuild affected container from clean base image with pinned dependencies
7. **Within 48 hours:** Post-incident review; update this policy if gap identified

---

## 9. Third-Party Intelligence Sources

External threat intelligence feeds (MISP, OpenCTI) are themselves supply chain components.
Feed integrity is maintained by:
- Using only well-established feeds from known organizations (Abuse.ch, AlienVault OTX, etc.)
- TLS certificate validation on all feed connections
- Feed content anomaly detection (sudden large volume of new IOCs = potential feed poisoning)
- MISP's built-in feed authentication (API key per source)

---

## 10. Policy Change Log

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2025-01-01 | Initial policy — Phase 0 |

---

*AEGIS-ZERO Supply Chain Security Policy v1.0.0 — AetherHorizon 🦉*
