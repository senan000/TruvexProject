# 🛡️ Truvex Core – CTI Platform

**Multi-Source Threat Intelligence & IOC Analysis**

Truvex is a lightweight, single-file Flask application that analyzes Indicators of Compromise (IOCs) – URLs, domains, IP addresses, file hashes and uploaded files – against several public threat-intelligence sources. It combines the results into a single **Truvex Threat Index (TTI)**, maps findings to **MITRE ATT&CK**, generates incident tickets, and produces PDF / JSON / CSV reports.

> ⚠️ **Educational project.** Truvex is intended for learning, demos and prototyping. It has no authentication and should not be exposed to the internet as-is.

---

## ✨ Features

- **Multi-source CTI consensus** – six engines queried in parallel (see below)
- **Truvex Threat Index (TTI)** – a 0–100 risk score with a verdict (Critical / High / Medium / Low / Clean)
- **Automatic indicator detection** – URL, domain, IPv4/IPv6, MD5, SHA1, SHA256
- **File analysis by hash** – the file is hashed locally (MD5/SHA1/SHA256) and only the SHA256 is looked up on VirusTotal; the file itself is never uploaded
- **Network context** – IP resolution, geolocation, reverse DNS, domain age (RDAP) and SSL certificate check
- **MITRE ATT&CK mapping** – only for strong evidence (TTI ≥ 50)
- **Trust-Decay trajectory** – a simple risk trend/forecast indicator
- **Automatic incident tickets** – `INC-xxxxx` IDs generated for scans with TTI ≥ 50
- **AI-assisted triage prototype** – template-based natural-language summary of each scan
- **SOC-style dashboard** – statistics, severity distribution, indicator types, top tags, search and filters, live telemetry feed
- **Analyst notes** – per-incident notes stored with the scan
- **Reporting & export** – PDF incident report, JSON export, CSV export of all telemetry
- **Bilingual UI** – English and Azerbaijani (default: Azerbaijani); switching language also re-renders historical scans
- **Demo mode** – generates a synthetic malicious scan for presentations
- **Engine health page** and JSON health endpoint

---

## 🔌 Threat Intelligence Sources

| Engine | Supports | API key required |
|---|---|---|
| [VirusTotal](https://www.virustotal.com/) | URL, domain, IP, hash, file | Yes (`VT_API_KEY`) |
| [URLhaus](https://urlhaus.abuse.ch/) | URL | Yes (`URLHAUS_AUTH_KEY`) |
| [AbuseIPDB](https://www.abuseipdb.com/) | IP (domains are resolved first) | Yes (`ABUSEIPDB_API_KEY`) |
| [AlienVault OTX](https://otx.alienvault.com/) | Domain, IP | No |
| [PhishTank](https://phishtank.org/) | URL | Optional (`PHISHTANK_APP_KEY`) |
| [OpenPhish](https://openphish.com/) | URL (public feed) | No |

Engines that are not applicable to the indicator type are marked `SKIPPED`; engines without a configured key are marked `NOT CONFIGURED`. Neither affects the score.

---

## 📊 Truvex Threat Index (TTI)

Scores are additive and capped at 100.

| Source | Condition | Points |
|---|---|---|
| VirusTotal | malicious ratio ≥ 20% | +35 |
| VirusTotal | malicious ratio ≥ 5% | +20 |
| VirusTotal | malicious ratio > 0% | +5 |
| VirusTotal | only suspicious detections | +3 |
| URLhaus | listed as malicious | +30 |
| PhishTank | verified phishing | +30 |
| OpenPhish | found in feed | +30 |
| AbuseIPDB | confidence ≥ 90 / ≥ 70 / ≥ 50 | +25 / +20 / +10 |
| AlienVault OTX | one or more threat pulses | +10 |

**Verdicts**

| TTI | Verdict |
|---|---|
| 75 – 100 | 🔴 Critical |
| 50 – 74 | 🟠 High |
| 25 – 49 | 🟡 Medium |
| 1 – 24 | 🔵 Low |
| 0 | 🟢 Clean |

**MITRE ATT&CK mappings used**

- `T1566` – Phishing (PhishTank / OpenPhish hit)
- `T1105` – Ingress Tool Transfer (URLhaus hit with payload indicators)
- `T1204` – User Execution (file analysis)

---

## 🚀 Getting Started

### Requirements

- Python 3.9+
- Internet access (for the CTI APIs, RDAP and IP geolocation)

### Installation

```bash
git clone <your-repo-url>
cd truvex

python -m venv venv
# Linux / macOS
source venv/bin/activate
# Windows
venv\Scripts\activate

pip install -r requirements.txt
```

### Configuration

Create a `.env` file in the project root:

```env
VT_API_KEY=your_virustotal_api_key
ABUSEIPDB_API_KEY=your_abuseipdb_api_key
URLHAUS_AUTH_KEY=your_urlhaus_auth_key
PHISHTANK_APP_KEY=optional_phishtank_app_key

# Optional: Unicode fonts for PDF reports (needed for Azerbaijani characters)
# TRUVEX_PDF_FONT=/path/to/Regular.ttf
# TRUVEX_PDF_FONT_BOLD=/path/to/Bold.ttf
```

All keys are optional – Truvex runs without them, but the corresponding engines will be reported as `NOT CONFIGURED`.

### Run

```bash
python app.py
```

Then open **http://127.0.0.1:5000**.

For a production-style run (Linux/macOS):

```bash
gunicorn -w 2 -b 127.0.0.1:5000 app:app
```

---

## 🧭 Usage

1. **Analyze an indicator** – on the *URL / Domain / IP / Hash* tab, enter a URL, domain, IP, or MD5/SHA1/SHA256 hash and click **Start CTI Analysis**.
2. **Analyze a file** – on the *File Analysis* tab, upload a file. Truvex computes its hashes and checks the SHA256 on VirusTotal.
3. **Review the report** – TTI score, network context, MITRE mapping, per-engine results, risk evidence and the AI triage summary.
4. **Manage incidents** – open the incident page to read details, add analyst notes, and export a PDF or JSON report.
5. **Filter history** – use the dashboard filters (search, verdict, type, date range) on the telemetry feed.
6. **Try the demo** – visit `/demo` to generate a synthetic high-risk scan.

> **Tip:** PhishTank, OpenPhish and URLhaus only work with full URLs (e.g. `https://example.com/login`). Entering just a domain will skip those engines.

---

## 🌐 Routes

| Route | Method | Description |
|---|---|---|
| `/` | GET | Dashboard, filters and telemetry feed |
| `/analyze-domain` | POST | Analyze a URL / domain / IP / hash |
| `/analyze-file` | POST | Hash-based file analysis |
| `/result/<id>` | GET | View a stored scan result |
| `/incident/<id>` | GET, POST | Incident details and analyst notes |
| `/generate-report/<id>` | GET | Download PDF incident report |
| `/export-json/<id>` | GET | Download scan as JSON |
| `/export-csv` | GET | Download all telemetry as CSV |
| `/delete-scan/<id>` | POST | Delete a scan |
| `/set-lang/<code>` | GET | Switch language (`en` / `az`) |
| `/engine-status` | GET | Engine configuration page |
| `/api/health` | GET | JSON health / engine configuration status |
| `/demo` | GET | Create a synthetic demo scan |

---

## 🗂️ Project Structure

```
truvex/
├── app.py             # Application (Flask routes, CTI engines, TTI, i18n, reporting, UI)
├── requirements.txt   # Python dependencies
├── .env               # API keys (you create this; do not commit)
└── truvex_soc.db      # SQLite database (created automatically on first run)
```

Scans are stored in a single SQLite `scans` table. Existing databases are migrated automatically (the `report_json` and `analyst_notes` columns are added if missing).

### Internationalization

Dynamic text (engine details, evidence, AI summary) is stored in the database as language-neutral message keys and translated at render time, so changing the language also updates previously saved scans. To add a string, add an entry to the `S` dictionary in `app.py`.

---

## 🔒 Security Notes

- No authentication or authorization – run locally or behind a trusted reverse proxy with access control.
- The server binds to `127.0.0.1` with `debug=False` by default.
- Keep your `.env` file out of version control (add it to `.gitignore`).
- Analyzed indicators are sent to third-party services (VirusTotal, URLhaus, AbuseIPDB, OTX, PhishTank, rdap.org, ip-api.com). Do not analyze indicators you are not permitted to share.
- Geolocation uses the free `ip-api.com` endpoint over plain HTTP.
- Free API tiers are rate limited; rate-limit responses are shown as `ERROR` for the affected engine.

---

## 🛠️ Tech Stack

- **Backend:** Python, Flask
- **Storage:** SQLite
- **Reporting:** ReportLab (PDF)
- **HTTP:** `requests`, `urllib`
- **Config:** `python-dotenv`
- **Frontend:** server-rendered Jinja2 template with embedded CSS (dark SOC theme)

---

## 📌 Limitations

- File analysis is hash-based only; no sandboxing or static/dynamic analysis.
- The "AI triage" is a template-based summary, not a machine-learning model.
- The Trust-Decay trend is a simplified illustration, not a historical time series.
- Scoring weights are heuristic and should be tuned for real-world use.

---

## 📄 License

Add your license here (e.g., MIT).
