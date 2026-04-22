#Team Name : AlgoX
#Team Member
Manasi Bhole
Mahi Kala
#Domain : Cybersecurity

#Description of problem

Video Link : https://drive.google.com/file/d/1KvEpx6jZH8djLlb5u_jpGuaZaTpfUbMu/view?usp=sharing
# Smart-Cyber-Defense-System
# 🛡️ SCDS — Smart Cybersecurity Detection System

> A real-time, graph-based cybersecurity threat detection platform that combines intelligent password security analysis, network intrusion detection, and a fusion threat engine to automatically identify, score, and isolate suspicious users.

---

## 📌 Problem Statement

In modern digital systems, cyberattacks such as brute-force login attempts, credential stuffing, and unauthorized geographic access are increasingly common and difficult to detect in real time. Traditional security systems rely on static rules and manual monitoring, which fail to adapt to evolving attack patterns.

**SCDS** addresses this by building an intelligent, self-learning detection system that:
- Evaluates password strength and vulnerability in real time
- Monitors network activity for anomalous behavior using graph-based node modeling
- Combines multiple risk signals into a unified threat score using a Fusion Threat Engine
- Automatically isolates high-risk nodes and blocks further access
- Learns and stores recurring attack patterns for future threat intelligence

The system provides a live dashboard for security operators to monitor users, threat scores, suspicious nodes, and system logs — enabling proactive threat response rather than reactive incident handling.

---

## 🧠 Data Structures Used

### 1. Graph (Nodes and Edges)
- **Used in:** Network Intrusion Detection module
- **Tables:** `nodes`, `edges`, `suspicious_nodes`
- Each user session is represented as a **node** in the network graph. Connections between users (shared IPs, locations) are represented as **edges**.
- Anomalous nodes (high failed logins, location jumps, excessive requests) are flagged and stored as suspicious nodes.
- This graph structure allows the system to detect **lateral movement** and **clustered attack patterns** across multiple users.

### 2. Hash Map (Dictionary / Key-Value Store)
- **Used in:** Password Analyzer service (`passwordAnalyzer.js`)
- A **dictionary array** (hash-mapped lookup) stores common weak passwords (`password`, `123456`, `admin`, etc.).
- Password input is checked against this dictionary in O(1) average time using `.includes()` on the pre-loaded array.
- The `attack_patterns` table functions as a persistent key-value store where the **pattern string is the key** and `occurrence_count` is the value, incremented on each match.

### 3. Weighted Scoring Vector (Feature Vector)
- **Used in:** Fusion Threat Engine (`fusionEngine.js`)
- The system builds a **two-dimensional feature vector** `[passwordRisk, networkRisk]` for each user session.
- A **weighted linear combination** is applied:
  ```
  ThreatScore = (passwordRisk × 0.4) + (networkRisk × 0.6)
  ```
- This vector-based scoring allows the system to combine heterogeneous risk signals into a single comparable scalar value for threshold-based decision making.

### 4. Queue (FIFO Log Stream)
- **Used in:** `logs` table and dashboard live feed
- System events (registrations, scans, isolations) are appended in **insertion order** and retrieved in reverse-chronological order, simulating a FIFO log queue.
- The dashboard displays the 15 most recent log entries, functioning as a **bounded queue** with automatic overflow trimming via SQL `LIMIT`.

### 5. Threshold-Based Decision Tree
- **Used in:** Fusion Engine + Isolation System
- A simple **decision tree** structure governs isolation logic:
  ```
  ThreatScore < 40   → SAFE
  40 ≤ Score < 70    → SUSPICIOUS
  Score ≥ 70         → ISOLATED → Block login + insert into isolated_nodes
  ```
- This tree is evaluated on every fusion call, with the outcome written to the `decisions` table.

### 6. Relational Database Schema (Linked Tables)
- **Used in:** MySQL (`scds_db`)
- The 12-table schema forms a **relational graph** of foreign-key linked entities:
  - `users` → `password_security`, `network_activity`, `nodes`, `fusion_scores`, `decisions`, `logs`
  - `nodes` → `edges`, `suspicious_nodes`, `isolated_nodes`
- Cascading deletes (`ON DELETE CASCADE`) maintain referential integrity across the graph when a user is removed.

---

## ⚙️ Tech Stack

| Layer | Technology |
|---|---|
| Backend | Node.js, Express.js |
| Frontend | HTML5, CSS3, Vanilla JavaScript |
| Database | MySQL |
| Charts | Chart.js |
| Security | bcryptjs |
| Environment | dotenv |
| Dev Server | nodemon |

---

## 🗂️ Project Structure

```
scds/
├── backend/
│   ├── config/
│   │   └── db.js                  # MySQL connection pool
│   ├── routes/
│   │   ├── auth.js                # Register & login endpoints
│   │   ├── password.js            # Password analysis endpoint
│   │   ├── network.js             # Network scan endpoint
│   │   ├── fusion.js              # Threat fusion + isolation endpoint
│   │   ├── dashboard.js           # Dashboard stats endpoint
│   │   └── patterns.js            # Attack patterns endpoint
│   ├── services/
│   │   ├── passwordAnalyzer.js    # Strength scoring, dictionary check
│   │   ├── networkAnalyzer.js     # Anomaly detection logic
│   │   └── fusionEngine.js        # Weighted threat score calculator
│   └── server.js                  # Express app entry point
├── frontend/
│   ├── index.html                 # Main terminal (register/scan/analyze)
│   ├── dashboard.html             # Live monitoring dashboard
│   ├── css/
│   │   └── style.css              # Dark cybersecurity theme
│   └── js/
│       ├── main.js                # Terminal page logic
│       └── dashboard.js           # Dashboard charts and data
├── .env                           # Environment variables (DB credentials)
└── package.json                   # Node dependencies
```

---

## 🚀 How to Run

### Prerequisites
- Node.js (v18+)
- MySQL (v8+)
- MySQL Workbench

### Steps

**1. Clone / set up the project folder**
```bash
cd scds
npm install
```

**2. Configure environment**
```
# .env
DB_HOST=localhost
DB_USER=root
DB_PASSWORD=your_mysql_password
DB_NAME=scds_db
PORT=3000
```

**3. Run the database schema in MySQL Workbench**
- Open MySQL Workbench
- Open a new query tab
- Paste the full schema SQL and press `Ctrl + Shift + Enter`

**4. Start the server**
```bash
npm run dev
```

**5. Open in browser**
- Terminal: `http://localhost:3000`
- Dashboard: `http://localhost:3000/dashboard`

---

## 🎯 Key Features

- ✅ Real-time password strength scoring and dictionary attack detection
- ✅ Brute-force cracking time estimation per password
- ✅ Graph-based network node modeling with anomaly detection
- ✅ Location jump detection (impossible travel alert)
- ✅ Fusion Threat Engine with weighted multi-signal scoring
- ✅ Automatic node isolation when threat score ≥ 70
- ✅ Login blocking for isolated nodes
- ✅ Self-learning attack pattern storage with occurrence tracking
- ✅ Live dashboard with Chart.js visualizations (trend + distribution)
- ✅ Severity-tagged system log feed
- ✅ Dark cybersecurity UI with scanline effects and animated threat ring

---

## 🗃️ Database Tables

| Table | Purpose |
|---|---|
| `users` | Stores registered user credentials |
| `password_security` | Password strength scores and risk metrics |
| `network_activity` | Raw network session data per user |
| `nodes` | Graph nodes representing user sessions |
| `edges` | Connections between nodes |
| `suspicious_nodes` | Nodes flagged with anomaly reasons |
| `attack_paths` | Full attack chain paths for forensics |
| `fusion_scores` | Combined threat scores per user session |
| `decisions` | Final SAFE / SUSPICIOUS / ISOLATED decisions |
| `isolated_nodes` | Blocked nodes with isolation reason |
| `logs` | Full system event log with severity |
| `attack_patterns` | Self-learned patterns with occurrence count |
