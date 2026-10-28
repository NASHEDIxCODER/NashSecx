# 🛡️ NashSecx — AI-Powered Autonomous Pentesting Agent

<p align="center">
  <img src="https://img.shields.io/badge/python-3.11%20%7C%203.14-blue.svg" alt="Python 3.11 | 3.14">
  <img src="https://img.shields.io/badge/platform-Linux-orange.svg" alt="Linux">
  <img src="https://img.shields.io/badge/security-scope--enforced-success.svg" alt="Scope Enforced">
  <img src="https://img.shields.io/badge/license-MIT-green.svg" alt="MIT License">
</p>

<p align="center"><em>An autonomous, authorized web application security assessment and pentesting agent.<br>
Deterministic Scope Boundaries · Multi-Provider AI (NVIDIA NIM & Google Gemini) · Independent Vulnerability Verification · Cryptographic Evidence Vault.</em></p>

---

## 1. Vision & Workflow

**NashSecx** is an AI-powered security assessment agent engineered for authorized web security audits, internal pentests, and bug bounty workflows. 

Unlike conversational assistants or unrestricted scripts, **NashSecx strictly enforces security policies outside the AI model**. The AI is utilized for reasoning, attack surface modeling, and hypothesis formulation, but is structurally incapable of bypassing scope controls, exceeding request budgets, or executing arbitrary host shell commands.

### The Assessment Lifecycle

```text
[Project Creation] ➔ [Scope Policy Validation] ➔ [Reconnaissance Probe] ➔ [Endpoint Inventory]
                                                                                │
[Verified Deliverable] ◀─ [Cryptographic Evidence Vault] ◀─ [Independent Verifier] ◀─ [Hypothesis Testing]
```

1. **Project & Scope Definition**: Establish explicit allowlists, wildcard domains (`*.target.com`), CIDR ranges, port limits, and TTL expiration.
2. **Deterministic Pre-Flight Checks**: Destination IPs are validated at connection time; loopback and RFC 1918 private addresses are unconditionally blocked unless `allow_local_lab=True`.
3. **Reconnaissance & Asset Inventory**: Baseline probes passively discover endpoints, forms, and routes, filtering out out-of-scope targets.
4. **Security Hypothesis Generation**: AI models formulate testable, bounded assessment steps via a unified provider gateway (NVIDIA NIM or Google Gemini).
5. **Controlled Execution**: Typed tools execute requests with rate budgets (requests per minute), byte size limits, and audit logs.
6. **Independent Empirical Verification**: Candidate anomalies are tested against a baseline; assertions without reproducible empirical proof are rejected as false positives.
7. **Evidence & Cryptographic Vault**: Every request/response pair is hashed with SHA-256 for provenance and auditability.
8. **Report Generation**: Deliverables are exported to standardized Markdown and machine-readable JSON.

---

## 2. Key Architecture & Security Guardrails

### 🛡️ Out-of-Model Scope Containment
- **Zero Scope Escape**: AI hallucinations or prompt injections embedded in target web pages cannot extend scope or target unauthorized assets.
- **Connection-Time DNS Check**: Protects against DNS rebinding, malformed IP representations, and out-of-scope HTTP redirects.
- **Budget & Concurrency Controls**: Sliding-window rate limiting, run total request caps, and an operator emergency kill-switch.
- **Sensitive Data Redaction**: Bearer tokens, passwords, Basic auth headers, JWTs, and cookies are scrubbed prior to model transmission.

### 🧠 Unified AI Provider Gateway
- **NVIDIA NIM Integration**: OpenAI-compatible adapter targeting NVIDIA NIM microservices (e.g. `meta/llama-3.1-70b-instruct`).
- **Google Gemini Integration**: REST API adapter with structured schema generation (e.g. `gemini-2.0-flash`).
- **Deterministic Mock Provider**: Zero-cost, reproducible offline testing in CI without API keys.
- **Resilience**: Configurable primary/fallback providers, exponential backoff on HTTP 429/503, and health probes.

### 🔬 Empirical Verification vs. Model Hallucination
- Scanner alerts and LLM assertions are never accepted as proof of a vulnerability.
- An independent verification engine replicates the observation with altered baselines, categorizing results into:
  - `Verified Finding`: Empirically confirmed and reproducible.
  - `Rejected False Positive`: Failed reproduction.
  - `Inconclusive`: Ambiguous or rate-limited.

---

## 3. Installation

### Requirements
- **Linux** (tested on Arch Linux, Debian, Ubuntu)
- **Python 3.11** or **3.14**

```bash
# 1. Clone the repository
git clone https://github.com/NASHEDIxCODER/NashSecx.git
cd NashSecx

# 2. Set up virtual environment
python3 -m venv .venv
source .venv/bin/activate

# 3. Install dependencies
pip install -r requirements.txt
```

---

## 4. Quickstart: CLI Usage

NashSecx features an interactive and scriptable command-line interface:

```bash
# Initialize local database and evidence vault
python3 -m nashsecx.interfaces.cli init

# 1. Create an assessment project
python3 -m nashsecx.interfaces.cli project create "Acme Web Audit" --description "Authorized external assessment"

# 2. Define assessment scope boundary (e.g. local lab or authorized domain)
python3 -m nashsecx.interfaces.cli scope add \
  --project-id <PROJECT_ID> \
  --name "Local Lab Scope" \
  --pattern "127.0.0.1" \
  --allow-local-lab

# 3. Validate a target URL before execution
python3 -m nashsecx.interfaces.cli scope validate \
  --scope-id <SCOPE_ID> \
  --target-url "http://127.0.0.1:8000/api"

# 4. Launch an autonomous assessment run
python3 -m nashsecx.interfaces.cli run start \
  --project-id <PROJECT_ID> \
  --scope-id <SCOPE_ID> \
  --target-url "http://127.0.0.1:8000"

# 5. List verified findings
python3 -m nashsecx.interfaces.cli findings list --project-id <PROJECT_ID>
```

You can also use the top-level executable:
```bash
python3 NashSecx.py --help
```

---

## 5. Desktop HUD Interface

Launch the desktop management interface for real-time telemetry, scope configuration, live run status, and emergency kill-switch controls:

```bash
python3 -m nashsecx.interfaces.desktop
# or
python3 NashSecx.py gui
```

Features:
- **Projects & Scope Editor**: Define and review target patterns, exclusions, and local lab toggles.
- **Live Run Telemetry**: Real-time progress, request counters, and emergency **⛔ Emergency Kill-Switch**.
- **Verified Findings**: Evidence inspection with SHA-256 hashes and reproduction curl commands.

---

## 6. Configuration & Centralized `.env`

Copy `.env.example` to `.env` to configure AI providers and operational parameters:

```bash
cp .env.example .env
```

All configuration variables are standardized on the `NASHSECX_` prefix:

| Variable | Default | Description |
| :--- | :--- | :--- |
| `NASHSECX_PROVIDER_PRIMARY` | `gemini` | Primary AI provider (`gemini`, `nvidia`, `mock`) |
| `NASHSECX_PROVIDER_FALLBACK`| `nvidia` | Fallback AI provider on transient or capability errors |
| `NASHSECX_GEMINI_API_KEY`   | `None` | Google Gemini API key |
| `NASHSECX_GEMINI_MODEL`     | `gemini-2.0-flash` | Gemini model (vision & text) |
| `NASHSECX_NVIDIA_API_KEY`   | `None` | NVIDIA NIM API key |
| `NASHSECX_NVIDIA_BASE_URL`  | `https://integrate.api.nvidia.com/v1` | NVIDIA NIM OpenAI-compatible endpoint |
| `NASHSECX_NVIDIA_MODEL`     | `meta/llama-3.1-70b-instruct` | NVIDIA model (`meta/llama-3.2-11b-vision-instruct` for vision) |
| `NASHSECX_REQUEST_TIMEOUT_SECONDS` | `60` | AI provider request timeout |
| `NASHSECX_MAX_PROVIDER_RETRIES` | `2` | Maximum retries on transient errors |
| `NASHSECX_SCREEN_VISION_ENABLED` | `true` | Enable screen vision analysis subsystem |
| `NASHSECX_SCREEN_UPLOAD_REQUIRES_CONFIRMATION` | `true` | Strict privacy gate: require operator confirmation before cloud upload |
| `NASHSECX_DATABASE_PATH`    | `~/.local/share/nashsecx/nashsecx.db` | Custom SQLite database location |
| `NASHSECX_LOG_LEVEL`        | `INFO` | Logging level (`DEBUG`, `INFO`, `WARNING`, `ERROR`) |

---

## 7. Screen Vision & Pentest Assistant

NashSecx includes a privacy-first screen vision assistant capable of analyzing security tooling displays:
- **Supported Contexts**: Burp Suite request/response pairs, browser DevTools network/console tabs, terminal outputs, Nmap/Nuclei scan outputs.
- **Privacy Enforcement**:
  - Zero background captures: captures only occur on explicit user request (`Capture Screen` / `Capture Region`).
  - Strict transmission confirmation: captures remain local in memory until the operator checks the confirmation box.
  - Secret and token redaction: sensitive credentials and tokens are scrubbed prior to model transmission.
- **Untrusted Boundary Framing**: All visual OCR text is framed as untrusted data. Prompt injection embedded in screenshots cannot modify scope, alter budgets, or execute host commands.
- **Linux Compatibility**: Automatically detects session type (`grim` on Wayland, `mss` / `scrot` / `import` on X11).

---

## 8. Running Tests

The test suite covers unit, integration, and end-to-end flows against local mock providers and disposable lab servers:

```bash
python3 -m pytest -v
```

All **86 automated tests** verify:
- Complete identity refactor and absence of obsolete branding in core code
- Centralized `.env` precedence and secret redaction
- Gemini and NVIDIA multimodal vision adapters & capability detection
- Screen capture service, region redaction, and privacy confirmation gates
- Scope boundary enforcement & out-of-scope redirect blocks
- Private IP blocking and local lab bypass controls
- Provider routing, bounded retries, and automatic fallback
- Token and secret data redaction
- Tool argument schema validation
- Independent empirical finding verification
- Evidence vault SHA-256 tamper detection
- Prompt injection scope defense (malicious model outputs blocked deterministically)
- End-to-end assessment against a local vulnerable server

---

## 9. Documentation

- [Rename Audit & Legacy Reference Tracking](docs/NASHSECX_RENAME_AUDIT.md)
- [Sprint Implementation Report](docs/NASHSECX_IMPLEMENTATION_REPORT.md)
- [System Architecture Specification](docs/ARCHITECTURE.md)
- [Engineering Implementation Plan](docs/IMPLEMENTATION_PLAN.md)
- [Security Hardening Report](docs/SECURITY_HARDENING_REPORT.md)
- [Security Audit Modules Report](docs/SECURITY_AUDIT_MODULES.md)
- [Repository Audit & Legacy Analysis](docs/REPOSITORY_AUDIT.md)
- [Component Reuse Map](docs/COMPONENT_REUSE_MAP.md)

---

## 10. License

MIT License — see [LICENSE](LICENSE).
# NashSecx
# NashSecx
