# AI-Powered-OT-Threat-Intel-Agent
# ⚡ OT & Ransomware Intelligence Platform
### Energy Sector Focus — Powered by n8n + Claude AI

> An automated weekly threat intelligence workflow that aggregates ransomware victim data, ICS/OT vulnerability advisories, and CISA known exploited vulnerabilities — then uses Claude AI to produce a structured, energy-sector-focused intelligence brief.

---

## 📸 Overview

This n8n workflow runs every Sunday at 08:00, pulls from four free public threat intelligence sources, enriches IOCs against VirusTotal and AbuseIPDB, and feeds all signals into a Claude AI agent that produces a downloadable HTML intelligence brief tailored specifically to **Operational Technology (OT) and energy sector** defenders.

The report covers:
- OT/ICS-specific TTPs mapped to the **ICS Cyber Kill Chain** and **MITRE ATT&CK for ICS**
- **Purdue Model** level targeting analysis
- Threat actor profiles with OT capability ratings
- ICS vendor vulnerability spotlight from CISA KEV
- Physical impact and IT/OT convergence risk assessment
- Prioritised, **OT-safe** recommendations

---

## 🏗 Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│  PHASE 1 — Data Collection (parallel, all free)                 │
│                                                                 │
│  ransomware.live/posts.json  ──► Filter OT Victims             │
│  ransomware.live/groups.json ──► Parse Threat Groups           │
│  CISA KEV JSON feed          ──► Parse CISA KEV (ICS filter)   │
│  CISA ICS Advisories RSS     ──► Parse ICS Advisories          │
└────────────────────────┬────────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────────────┐
│  PHASE 2 — Normalise & Enrich                                   │
│                                                                 │
│  Extract IOCs ──► VirusTotal Lookup  ──► Enrich IOC Results    │
│               └──► AbuseIPDB Lookup  ──┘                       │
│                                                                 │
│  Merge All Intel Sources (unified payload)                      │
└────────────────────────┬────────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────────────┐
│  PHASE 3 — AI Analysis                                          │
│                                                                 │
│  OT Threat Intelligence Agent (Claude Sonnet)                   │
│  + OT Intel Output Parser (structured JSON schema)             │
└────────────────────────┬────────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────────────┐
│  PHASE 4 — Output                                               │
│                                                                 │
│  Output Enhanced HTML (v300) ──► HTML File Download            │
│                              ──► Slack Alert (disabled)        │
│                              ──► Email Report (disabled)       │
│                              ──► Google Doc (disabled)         │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📡 Data Sources

| Source | Endpoint | Auth Required | What it provides |
|--------|----------|---------------|-----------------|
| ransomware.live | `raw.githubusercontent.com/jmousqueton/ransomware.live/main/posts.json` | None | Recent ransomware victims |
| ransomware.live | `raw.githubusercontent.com/jmousqueton/ransomware.live/main/groups.json` | None | Threat group TTPs, tools, IOCs |
| CISA KEV | `cisa.gov/sites/default/files/feeds/known_exploited_vulnerabilities.json` | None | Known exploited CVEs (ICS/OT filtered) |
| CISA ICS Advisories | `us-cert.cisa.gov/ics/advisories/advisories.xml` | None | ICS/OT vulnerability advisories RSS |
| VirusTotal | `virustotal.com/api/v3/domains/{domain}` | Free API key | IOC reputation scoring |
| AbuseIPDB | `api.abuseipdb.com/api/v2/check` | Free API key | IP abuse confidence scoring |

---

## 🤖 AI Analysis Output

Claude produces a structured JSON report containing:

| Field | Description |
|-------|-------------|
| `executive_summary` | 3–4 sentence CISO-level summary |
| `threat_level` | CRITICAL / HIGH / MEDIUM / LOW |
| `confidence_score` | 0–100 attribution confidence |
| `key_findings` | Top 5 most important findings |
| `ttps` | MITRE ATT&CK for ICS TTPs with ICS Kill Chain stage |
| `ot_specific_risks` | Protocols at risk, Purdue levels, affected vendors, physical impact |
| `threat_actors` | Actor profiles with OT capability and energy sector focus ratings |
| `vulnerability_spotlight` | Top ICS CVEs with patch priority and OT impact |
| `ioc_summary` | Malicious domains, IPs, hashes, BTC addresses |
| `targeting_patterns` | Sectors, regions, victim profile, attack vectors |
| `operational_intelligence` | Narrative situational awareness |
| `recommendations` | Prioritised, OT-safe actions with MITRE references |
| `negotiation_intel` | Demand ranges, payment methods, decryptor reliability |

---

## ⚙️ Prerequisites

- **n8n** — self-hosted (Docker recommended) or n8n Cloud
- **Anthropic API key** — for Claude Sonnet (claude-sonnet-4-5)
- **VirusTotal API key** — free tier (500 requests/day). [Get one here](https://www.virustotal.com/gui/join-us)
- **AbuseIPDB API key** — free tier available. [Get one here](https://www.abuseipdb.com/register)

---

## 🚀 Installation & Setup

### 1. Clone this repository

```bash
git clone https://github.com/YOUR_USERNAME/ot-ransomware-intelligence-platform.git
cd ot-ransomware-intelligence-platform
```

### 2. Run n8n with Docker

```bash
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n
```

Or with Docker Compose — see [`docker-compose.yml`](./docker-compose.yml).

### 3. Import the workflow

1. Open n8n at `http://localhost:5678`
2. Click the **⋮ menu** (top right) → **Import from file**
3. Select `OT_Ransomware_Intelligence_Platform_v300_FIXED.json`

### 4. Configure credentials

**Anthropic (required):**
1. In n8n go to **Settings → Credentials → Add Credential**
2. Search for **Anthropic** and add your API key
3. Open the workflow and connect this credential to the **Anthropic Chat Model** node

**VirusTotal & AbuseIPDB (optional but recommended):**
1. Go to **Settings → Variables**
2. Add the following variables:

| Variable | Value |
|----------|-------|
| `VT_API_KEY` | Your VirusTotal API key |
| `ABUSEIPDB_API_KEY` | Your AbuseIPDB API key |

### 5. Enable optional output channels

By default, Slack, Email, and Google Docs output nodes are **disabled**. To enable:
1. Add the relevant credential in **Settings → Credentials**
2. Right-click the node in the workflow → **Enable**
3. Set the following variables in **Settings → Variables**:

| Variable | Description |
|----------|-------------|
| `SLACK_CHANNEL` | e.g. `#threat-intel` |
| `FROM_EMAIL` | Sender email address |
| `TO_EMAIL` | Recipient email address |

### 6. Run it

Click **Execute Workflow** to run immediately, or activate the workflow to run automatically every Sunday at 08:00.

---

## 📄 Output Report

The HTML intelligence brief includes:

- **Header** — Threat level pill, confidence ring, TLP classification
- **Stat cards** — Victim count, trend, ICS CVE count, advisory count, TTPs, actors
- **Executive Summary** — With numbered key findings
- **OT/ICS Risk Profile** — Protocols, vendors, physical impact, IT/OT convergence
- **Purdue Model Diagram** — Visual of which network levels are targeted
- **Threat Actor Table** — Motivation, OT capability, energy sector focus
- **MITRE ATT&CK for ICS TTP Table** — With ICS Kill Chain stage and OT relevance
- **ICS Vulnerability Spotlight** — CISA KEV CVEs with patch priority
- **IOC Enrichment** — Domains, IPs, hashes, BTC addresses
- **Targeting Patterns** — Sectors, regions, victim profile, attack vectors
- **Negotiation Intelligence** — Demand ranges, payment methods, decryptor reliability
- **Operational Intelligence Narrative** — AI-written situational awareness
- **Prioritised Recommendations** — With OT-safe indicators and MITRE references

The report is styled for dark-mode viewing and is print-friendly.

---

## 🔧 Troubleshooting

### ransomware.live returns 502
The ransomware.live API server is intermittently unreliable. The workflow uses GitHub raw URLs as a stable mirror. If you still see errors, the parse nodes return graceful empty fallbacks so the rest of the workflow continues unaffected.

### "Model output doesn't fit required format"
This means Claude's response didn't match the output schema. The simplified schema in v300 minimises this. If it recurs, check the **OT Intel Output Parser** node and ensure the schema has no deeply nested required fields.

### CISA KEV returns no data
Ensure the **Fetch CISA KEV** node has `Accept: application/json` and `User-Agent: Mozilla/5.0` headers set. CISA blocks requests without a proper User-Agent.

### Workflow won't execute (shows issues)
Right-click any nodes with orange warning triangles (Slack, Email, Google Docs) → **Disable**. These require credentials to pass validation even when not in the execution path.

### Credential test shows error on save
This is a known n8n false positive for Anthropic credentials. If your workflow executes successfully, the credential is working correctly — ignore the save-time warning.

---

## 📁 Repository Structure

```
.
├── OT_Ransomware_Intelligence_Platform_v300_FIXED.json  # Main workflow
├── docker-compose.yml                                    # Docker Compose setup
├── .env.example                                          # Environment variable template
├── CHANGELOG.md                                          # Version history
├── CONTRIBUTING.md                                       # Contribution guidelines
├── LICENSE                                               # MIT License
└── README.md                                             # This file
```

---

## 🗺 Roadmap

- [ ] Shodan integration for exposed ICS device discovery
- [ ] MITRE ATT&CK for ICS navigator layer export
- [ ] PDF report output option
- [ ] Telegram / Teams alert channel support
- [ ] Historical trending using n8n persistent storage
- [ ] Sector-specific sub-reports (oil & gas, electric, water)

---

## ⚠️ Disclaimer

This tool aggregates **publicly available** threat intelligence data. It does not access, store, or process any classified or proprietary information. All data sources used are open and free. The intelligence produced is for **defensive purposes only** — to help OT/ICS security teams understand the threat landscape and prioritise mitigations.

This project is not affiliated with ransomware.live, CISA, Anthropic, or any other data source referenced.

---

## 📜 License

MIT License — see [`LICENSE`](./LICENSE) for details.

---

## 🙏 Acknowledgements

- [ransomware.live](https://www.ransomware.live) — Julien Mousqueton, for maintaining the public ransomware tracker
- [CISA](https://www.cisa.gov) — for the KEV catalogue and ICS advisory feeds
- [Anthropic](https://www.anthropic.com) — Claude AI
- [n8n](https://n8n.io) — workflow automation platform
- [Dragos](https://www.dragos.com) — OT threat intelligence research referenced in AI prompts
