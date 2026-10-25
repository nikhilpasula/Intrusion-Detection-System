# Adaptive AI-Based Intrusion Detection System

A web-based Intrusion Detection System (IDS) that detects simulated cyberattacks in real time, classifies them using machine learning, and **adapts to new attack patterns** through a human-in-the-loop validation workflow.

Built for an **internship review and demonstration** — practical, explainable, and end-to-end.

---

## What This Project Does

The system follows a complete adaptive loop:
Observe → Collect Event → Extract Features → Detect → Classify
↓
Improve Future Detection ← Learn ← Validate ← Store Pattern
↑
Alert

Every event is classified. Every attack is stored as a pattern. Every pattern can be validated by an analyst. Every approval retrains the model. The system gets measurably smarter as it's used.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | HTML, CSS, JavaScript |
| Backend | Python, Flask |
| Machine Learning | scikit-learn (RandomForest + IsolationForest) |
| Database | SQLite |
| Packet Capture | scapy |
| Numerics | numpy, pandas |

**No frontend framework. No build step. No external services.** Everything runs from one folder.

---

## Features

- **Real-time classification** — RandomForest classifier with 150 decision trees
- **Anomaly detection** — IsolationForest flags statistically unusual traffic
- **14 engineered features** per event — ports, protocol, flags, bytes, duration, rate, failed logins
- **7 attack classes** — normal, port scan, DoS, brute force, SQL injection, XSS, malformed packet
- **Explainable predictions** — full probability distribution, feature vector, and decision trace per event
- **Live packet capture** — scapy-based flow aggregation for real network traffic
- **Synthetic traffic generator** — 7 attack types for reproducible demos
- **Human-in-the-loop validation** — approve or reject new signatures from the web UI
- **Automatic retraining** — approved patterns join the training set, model rebuilds automatically
- **Model metrics dashboard** — accuracy, precision, recall, and F1 on holdout data
- **Fully browser-based demo** — same ML pipeline in pure JavaScript for zero-install demos

---

## The Adaptive Loop — What Makes This Different

Traditional IDSs are static. They're trained once and deployed. When a new attack appears, they miss it.

## This system closes the loop:
Event arrives → classified as "port_scan" (high, 0.94)

Pattern stored → "port_scan:TCP:800" with status=pending

Analyst visits → /patterns page shows the pending signature

Analyst clicks → "Approve"

Model retrains → new RandomForest trained with the approved pattern

Metrics update → accuracy / precision / recall / F1 refreshed

Next event uses → the improved model

The system gets smarter every time an analyst validates a pattern.

---

## Architecture
┌─────────────────────────────────────────────────────────────────┐
│ DATA SOURCES │
│ │
│ capture.py simulate.py │
│ (real packets via (synthetic events │
│ scapy + Npcap) for demos) │
└──────────┬────────────────────────┬─────────────────────────────┘
│ │
▼ ▼
POST /api/events (JSON: src_ip, dst_ip, ports, flags,
bytes, duration, rate, failed_logins)
│
▼
┌─────────────────────────────────────────────────────────────────┐
│ FLASK BACKEND (app.py) │
│ │
│ 1. Receive event │
│ 2. ml_engine.classify(event) │
│ ├─ Extract 14 numeric features │
│ ├─ RandomForest → attack class + confidence │
│ ├─ IsolationForest → anomaly score │
│ ├─ Apply severity rules │
│ └─ Return prediction, severity, action │
│ 3. database.insert_event(...) │
│ 4. If attack → database.upsert_pattern(...) │
└──────────┬──────────────────────────────────────────────────────┘
│
▼
┌─────────────────────────────────────────────────────────────────┐
│ SQLITE (data/ids.db) │
│ │
│ events — every classified packet │
│ patterns — attack signatures (pending/approved/rejected) │
│ metrics — accuracy / precision / recall / F1 over time │
│ users — analyst credentials │
└──────────┬──────────────────────────────────────────────────────┘
│
▼
┌─────────────────────────────────────────────────────────────────┐
│ DASHBOARD (Flask + Jinja + CSS) │
│ │
│ /dashboard → stat cards, recent alerts, distribution │
│ /alerts → filterable alert feed │
│ /events → dense event table │
│ /patterns → pattern validation │
│ /model → model health + retraining history │
│ /settings → thresholds + API endpoints │
└─────────────────────────────────────────────────────────────────┘

---

## Two Ways to Run

### Option A — Browser Demo (Fastest)

Open **`adaptive-ids.html`** in Chrome or Edge. That's it.

- **Model:** RandomForest implemented from scratch in JavaScript
- **Data:** synthetic traffic generator with realistic distributions
- **Storage:** in-memory (resets on reload)
- **Best for:** live demos, zero-setup environments

**Login:** `admin` / `admin`

### Option B — Full Python Stack

```bash
# 1. Create virtual environment
python -m venv venv
venv\Scripts\activate          # Windows
# source venv/bin/activate     # macOS/Linux

# 2. Install dependencies
pip install -r requirements.txt

# 3. Start the server
python app.py
```
Open http://localhost:5000 in your browser.
Login: admin / admin

## To generate attack traffic:
python simulate.py
To capture real network traffic (Windows — requires Npcap + Administrator):
python capture.py
## File Structure
adaptive-ids/
├── app.py                       Flask app — routes, auth, API
├── database.py                  SQLite schema + helpers
├── ml_engine.py                 RandomForest + IsolationForest
├── capture.py                   Live packet capture (scapy)
├── simulate.py                  Synthetic attack generator
├── requirements.txt             Python dependencies
├── Procfile                     Deployment config
├── runtime.txt                  Python version
├── .gitignore                   Excludes venv, DB, models
│
├── data/                        Auto-created on first run
│   ├── ids.db                   SQLite database
│   └── models/                  Trained model binaries
│
├── static/css/
│   └── main.css                 Design system
│
├── templates/
│   ├── base.html                Sidebar shell + drawer + toast
│   ├── landing.html             Marketing page
│   ├── login.html               Auth card
│   ├── dashboard.html           Stats + alerts + distribution
│   ├── alerts.html              Filterable feed
│   ├── events.html              Dense event table
│   ├── patterns.html            Validation workflow
│   ├── model.html               Model health + history
│   └── settings.html            Thresholds + API
│
└── adaptive-ids.html            Standalone browser demo
## ML Pipeline
Feature Extraction (14 Features per Event)
#	Feature	Meaning
1	src_port	Source port
2	dst_port	Destination port
3	bytes_sent	Total bytes
4	duration	Flow duration (seconds)
5	protocol_tcp	TCP indicator
6	protocol_udp	UDP indicator
7	protocol_icmp	ICMP indicator
8	flag_syn	SYN flag present
9	flag_ack	ACK flag present
10	flag_fin	FIN flag present
11	flag_rst	RST flag present
12	packet_size	Average packet size
13	failed_logins	Failed auth attempts
14	request_rate	Requests per second
## Models
Model	Purpose	Configuration
RandomForestClassifier	Multiclass attack classification	150 trees, max depth 12
IsolationForest	Unsupervised anomaly detection	contamination = 0.1
**Evaluation Metrics (20% Holdout Set)**
Metric	Value
Accuracy	~97%
Precision	~96%
Recall	~94%
F1 Score	~95%
Inference time	~5 ms per event
Training time	~2 seconds (Python) / ~1 second (Browser)
**Severity Mapping**
Attack Class	Severity	Action
DoS, brute force, confidence > 0.90	Critical	Blocked
Port scan, SQL injection, XSS	High	Blocked
Malformed packet	Medium	Flagged
Normal traffic	Low	Logged
Sanity guard: if the top prediction is on a common port (80, 443, 22, 53…) with no rate spike, no failed logins, and confidence below 0.90 — it defaults to normal. Reduces false positives on benign traffic.

## Known Limitations
This is a demo-grade system, not a production product. Documented honestly:

Limitation	Impact	Production Fix
Synthetic training data	Best on simulated attacks	Train on NSL-KDD or CICIDS2017
SHA256 password hashing	Not production-grade	Use bcrypt or argon2
SQLite	Single-node, no concurrency	Postgres + SQLAlchemy
No rate limiting	/api/events accepts unlimited POSTs	Flask-Limiter
No CSRF protection	Forms could be forged	Flask-WTF
Ephemeral cloud storage	DB resets on restart	Persistent volume
No auth on /api/events	Anyone can inject events	Bearer token
Catastrophic forgetting	Retraining can degrade old patterns	Replay buffers

## Project Objective
The goal was to build a practical, demonstrable cybersecurity project for internship review — one that shows:

Real machine learning (not rule-based if-statements)

A complete system (frontend + backend + ML + database)

Explainability (every decision is visible to the analyst)

Adaptability (the model improves through human validation)

Production thinking (documented limitations, deployment path, security considerations)

The project follows the workflow:
Observe → Collect → Extract Features → Detect → Classify →
Alert → Store Pattern → Validate → Learn → Improve

## Tech Stack Details
Layer	Technology	Version
Backend	Flask	3.0+
ML	scikit-learn	1.5+
Numerics	numpy, pandas	Latest stable
Persistence	SQLite	Built-in
Packet capture	scapy	2.5+
Packet driver	Npcap (Windows)	1.89+
Server	gunicorn	21.2+
Frontend	HTML + CSS + vanilla JS	No framework
## Author
Nikhil Pasula
Internship Project · October 2026
<https://github.com/nikhilpasula>

## Acknowledgements
Design system inspired by Linear, Sentry, Vercel, and NVIDIA. ML architecture follows standard IDS research baselines. Built with assistance from Claude for design and implementation guidance.


