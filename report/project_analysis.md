# Project Analysis: NetSentinel (Network Traffic Analyzer and Intrusion Detection System)

## 1. Project Identity & Administrative Details (from First Review / PPTX)

- **Project Title:** Network Traffic Analyzer and Intrusion Detection System
- **Project Subtitle:** A Web-Based Platform for Real-Time Network Monitoring and Threat Detection
- **Student Full Name:** Sameer Sha
- **Roll Number:** 45
- **Register Number:** CHN25MCA-2045
- **Course / Degree:** Master of Computer Applications (MCA)
- **Semester:** Third Semester (S3 MCA) *(Source: Standard APJ AKTU Curriculum; noted in TODO.md)*
- **Subject Code:** 20MCA245 MINI PROJECT *(Source: Template / Course curriculum; noted in TODO.md)*
- **Project Guide:** Smt. Beena K, Assistant Professor, Department of Computer Engineering
- **Project Coordinators:** Smt. Josmi Jose, Smt. Santhy Viswam, Smt. Celin T (Assistant Professors, Dept. of Computer Engineering) *(Source: Template institutional records; review file: NOT FOUND)*
- **Head of the Department (HOD):** Dr. Renu George, Professor & Head of the Department, Dept. of Computer Engineering *(Source: Template institutional records; review file: NOT FOUND)*
- **Principal:** Dr. Hari V.S, Principal, College of Engineering Chengannur *(Source: Template institutional records; review file: NOT FOUND)*
- **Department:** Department of Computer Engineering
- **Institution:** College of Engineering Chengannur (CEC), Alappuzha, Kerala - 689121
- **Affiliating University:** APJ Abdul Kalam Technological University (KTU), Kerala
- **Month and Year:** October 2026

---

## 2. Technical Profile (Extracted from Real Source Code)

### 2.1 Technology Stack & Real Versions
- **Frontend:**
  - React 18.3.1 with Vite 5.2.0 build tool
  - React Router DOM v6.23.1 (client-side routing, protected routes, role-based route guards)
  - Chart.js v4.4.2 & React-ChartJS-2 v5.2.0 (real-time bandwidth, throughput, and protocol donut charts)
  - Lucide React v0.378.0 (vector UI icons)
  - Socket.io-client v4.7.5 (WebSocket live telemetry ingestion)
  - Axios v1.6.8 (HTTP REST client with JWT interceptors)
  - Pure CSS design system with Soft Claymorphism aesthetic (`theme.css`)
- **Backend API Server:**
  - Node.js v18+ / v20+ / v24+ runtime with CommonJS modules
  - Express.js v4.19.2 (REST API controllers, middleware, routing)
  - Socket.io v4.7.5 (`/stream` namespace for bidirectional real-time event streaming)
  - MySQL2 v3.9.7 with `mysql2/promise` connection pooling
  - JSON Web Tokens (`jsonwebtoken` v9.0.2) and SHA-256 session token management
  - Bcrypt.js v2.4.3 (salted password hashing with 10 rounds)
  - PDFKit v0.15.0 (vector PDF executive forensic report compilation)
  - CSV-Stringify v6.5.0 (raw CSV telemetry data export engine)
  - CORS v2.8.5, Dotenv v16.4.5
- **Packet Capture & Intrusion Detection Service:**
  - Python 3.10+ / 3.14 (standalone service in `capture/`)
  - Scapy v2.8.0 (promiscuous socket packet sniffing, protocol disassembly, TCP/UDP/ICMP/DNS feature extraction)
  - Requests v2.34.2 (HTTP client pushing telemetry to `/api/ingest/packets` and `/api/ingest/alerts` via `x-api-key`)
  - Python-dotenv v1.2.4 (environment variable management)
  - Synthetic Traffic Simulator (`simulator.py`) for reproducible offline attack generation
  - Multi-Rule Sliding Window Threat Detector (`detector.py`)
  - Rotating PCAP Writer (`pcap_writer.py`) for raw forensic capture archiving
- **Database:**
  - MySQL 8.0 / 8.4 InnoDB engine, character set `utf8mb4`, collation `utf8mb4_unicode_ci`

### 2.2 Implemented Database Schema (8 Tables)
1. **`users`**: User identity, credential hashes, and RBAC roles (`id`, `name`, `email`, `password_hash`, `role` ['user', 'admin'], `created_at`, `updated_at`).
2. **`user_sessions`**: Active authentication sessions (`id`, `user_id` [FK -> `users.id`], `token`, `created_at`, `expires_at`, `is_active`).
3. **`devices`**: Automatically discovered network hosts (`id`, `ip_address` [UNIQUE], `mac_address`, `hostname`, `device_type` ['router', 'server', 'workstation', 'iot', 'unknown'], `status`, `first_seen`, `last_seen`).
4. **`network_traffic`**: Ingested packet telemetry (`id`, `device_id` [FK -> `devices.id`], `timestamp`, `src_ip`, `dst_ip`, `src_port`, `dst_port`, `protocol`, `packet_size`, `packet_count`, `bytes_transferred`, `duration`, `tcp_flags`, `status`).
5. **`detection_rules`**: Intrusion detection rule definitions (`id`, `rule_name`, `attack_type` [UNIQUE], `description`, `threshold_value`, `time_window`, `severity` ['low', 'medium', 'high', 'critical'], `is_active`, `created_at`, `updated_at`).
6. **`alerts`**: Detected security incident records (`id`, `device_id` [FK -> `devices.id`], `rule_id` [FK -> `detection_rules.id`], `traffic_id` [FK -> `network_traffic.id`], `timestamp`, `alert_type`, `severity`, `source_ip`, `destination_ip`, `description`, `status` ['new', 'acknowledged', 'resolved']).
7. **`reports`**: Generated forensic reports catalog (`id`, `user_id` [FK -> `users.id`], `report_type` ['traffic', 'alerts', 'devices'], `format` ['pdf', 'csv'], `start_time`, `end_time`, `generated_at`, `file_path`).
8. **`capture_settings`**: Global packet capture engine state (`id` [PK=1], `is_running`, `mode` ['simulator', 'live'], `interface_name`, `simulate_attacks`, `updated_at`).

### 2.3 User Roles & RBAC Matrix
- **Standard User (`user` - Security Analyst):**
  - Access to Live Dashboard, Throughput charts, and Protocol statistics.
  - Access to Live Traffic Explorer with multi-parameter filtering.
  - Access to Security Alerts triage drawer (read-only inspection).
  - Access to Tracked Devices inventory and threat scores.
  - Ability to generate and download personal PDF and CSV forensic reports.
- **Administrator (`admin` - SecOps Administrator):**
  - All standard user capabilities.
  - Start, stop, and configure the Python Packet Capture Engine (Simulator vs. Live Interface).
  - Acknowledge, triage, and resolve security alerts.
  - Edit detection rule sensitivity thresholds, time windows, and severity ratings.
  - Toggle detection rules between active and disabled states.
  - User management: view all registered accounts, elevate users to `admin`, and delete accounts (with self-protection guard).

### 2.4 Implemented Intrusion Detection Rules
1. **SYN Flood (`syn_flood`):** Sliding window tracking of TCP SYN flags without matching ACK completions (Default threshold: 150 packets within 15 seconds, Critical severity).
2. **Port Scan (`port_scan`):** Tracks distinct destination ports contacted by a single source host (Default threshold: 15 distinct ports within 10 seconds, Medium severity).
3. **ICMP Flood (`icmp_flood`):** High-frequency ICMP Echo Request bursts (Default threshold: 100 packets within 10 seconds, High severity).
4. **Suspicious DNS (`suspicious_dns`):** High volume of DNS lookups or anomalous query lengths exceeding 60 characters indicating DNS tunneling (Default threshold: 50 queries within 60 seconds, Medium severity).
5. **Suspicious Payload (`suspicious_payload`):** Deep packet inspection scanning application layer data against known attack signatures (SQL injection, XSS, path traversal, shell execution) (Default threshold: 1 match within 1 second, High severity).
6. **Malformed Packet (`malformed_packet`):** Identifies invalid header fields and illegal TCP flag combinations such as NULL scans, XMAS scans, and SYN+FIN (Default threshold: 1 match within 1 second, Medium severity).
- *Deduplication Mechanism:* A 30-second sliding cache prevents alert flooding for recurring malicious traffic bursts.

---

## 3. Review vs Project Differences

| Dimension | First Review File (`review1.pptx`) | Real Source Code Implementation | Report Action Taken |
| :--- | :--- | :--- | :--- |
| **Database Engine** | Mentions **MongoDB** (Slide 21, 233) | Implemented in **MySQL 8.0/8.4** with InnoDB, relational constraints, and SQL migrations (`backend/src/db/schema.sql`). | Documented MySQL 8.4 database architecture in accordance with real project code. |
| **Backend Framework** | Mentions **Python (Flask / FastAPI)** (Slide 18) | Backend API is built in **Node.js + Express.js** with **Socket.io**. Python is utilized exclusively for the capture/sniffing engine (`capture/`). | Documented the exact Node.js Express REST API + Python Scapy modular split. |
| **Data Storage** | Mentions CSV / JSONL / PCAP for primary storage | Uses MySQL relational tables for structured storage and maintains rotated `.pcap` files on disk for raw packet archiving. | Documented the dual MySQL + PCAP storage mechanism. |
| **User Roles & RBAC** | Does not specify user roles or permissions | Strict Role-Based Access Control (RBAC) implemented: Standard User (`user`) vs. Administrator (`admin`). | Detailed the two distinct roles and permissions throughout Chapters 1, 3, 4, 5, and 6. |
| **Capture Modes** | Only mentions live packet capture via Scapy | Dual-mode capture engine implemented: Live Npcap sniffer AND multi-host synthetic traffic simulator. | Fully described both Simulator and Live capture modes. |
| **Administrative Names** | Only Guide name is present; Coordinator, HOD, and Principal are missing | Institutional hierarchy extracted from template/department records for College of Engineering Chengannur. | Documented in front matter and recorded in `TODO.md` for student verification. |

---

## 4. Image Inventory & Mapping

| File Name | Dimensions | Content Description | Image Type | Target Report Section |
| :--- | :--- | :--- | :--- | :--- |
| `CEC.png` | 5000x5000 | Official crest of College of Engineering Chengannur | Institutional Logo | Front Matter (Cover Page & Certificate) |
| `SystemArchitecture.png` | 1500x800 | Multi-tier architecture diagram: Network Interface, Python Capture Engine, Express Backend, MySQL Database, and React Frontend | System Architecture Diagram | Chapter 5: System Architecture (`\ref{fig:architecture}`) |
| `usecase.png` | 1660x860 | UML Use Case diagram showing Standard User and SecOps Administrator interacting with 12 system use cases | Use Case Diagram | Chapter 5: Use Case Diagram (`\ref{fig:usecase}`) |
| `dfd-level-0.png` | 1395x400 | Context-level Data Flow Diagram (Process 0.0) with Standard User and Administrator data flows | DFD Level 0 | Chapter 5: Data Flow Diagrams (`\ref{fig:dfd0}`) |
| `dfd-level-1-user-large.png` | 2520x1960 | DFD Level 1 showing Analyst processes: Auth, Monitor, Query Traffic, Inspect Alerts, Browse Devices, and Export Reports | DFD Level 1 (User) | Chapter 5: Data Flow Diagrams (`\ref{fig:dfd1_user}`) |
| `dfd-level-1-admin-large.png` | 2520x2440 | DFD Level 1 showing Administrator processes: Capture Control, Rule Tuning, Alert Triage, and User Management | DFD Level 1 (Admin) | Chapter 5: Data Flow Diagrams (`\ref{fig:dfd1_admin}`) |
| `dbschema.png` | 1940x960 | Complete relational database schema with 8 entities, attributes, primary keys, foreign keys, and indexes | Relational Database Schema | Chapter 5: Database Schema (`\ref{fig:dbschema}`) |
| `Screenshots/Screenshot-user-dashboard.png` | 1917x925 | Standard user dashboard with live throughput charts, protocol distribution donut, and read-only recent alerts | Application Screenshot | Chapter 7: Screenshots (`\ref{fig:screen_user_dash}`) |
| `Screenshots/Screenshot-admin-dashboard.png` | 1917x930 | Administrator dashboard with topbar capture control bar (Simulator/Live mode), KPI cards, and live analytics | Application Screenshot | Chapter 7: Screenshots (`\ref{fig:screen_admin_dash}`) |
| `Screenshots/Screenshot-admin-live-traffic.png` | 1917x926 | Live Traffic Explorer with protocol dropdown, search query bar, and tabular packet telemetry stream | Application Screenshot | Chapter 7: Screenshots (`\ref{fig:screen_traffic}`) |
| `Screenshots/Screenshot-admin-realtime-anyalysing.png` | 1572x925 | Real-time threat analysis and incident investigation drawer displaying packet headers and payload hex dump | Application Screenshot | Chapter 7: Screenshots (`\ref{fig:screen_investigation}`) |
| `Screenshots/Screenshot-admin-alerts.png` | 1917x923 | Security alerts management page showing severity badges, source IPs, timestamps, and triage buttons | Application Screenshot | Chapter 7: Screenshots (`\ref{fig:screen_alerts}`) |
| `Screenshots/Screenshot-admin-tracked-devices.png` | 1917x917 | Discovered network host inventory displaying device cards, IP/MAC addresses, hostnames, and risk scores | Application Screenshot | Chapter 7: Screenshots (`\ref{fig:screen_devices}`) |
| `Screenshots/Screenshot-admin-detection-rules.png` | 1917x923 | Detection rules management page with threshold inputs, time windows, severity ratings, and active toggle switches | Application Screenshot | Chapter 7: Screenshots (`\ref{fig:screen_rules}`) |

---

## 5. Verification of Automated Tests & Execution Status

- **Automated Verification Suite:** `scripts/e2e-check.ps1` (10-step full-stack integration test).
  - Step (a) - API & MySQL Health Check: **PASS**
  - Step (b) - Administrator Authentication: **PASS**
  - Step (c) - Simulator Capture Start & Attack Generator: **PASS**
  - Step (d) - Telemetry Ingestion & Batch Processing: **PASS**
  - Step (e) - Threat Detection Engine Rules Verification: **PASS**
  - Step (f) - Device Discovery & Risk Scoring: **PASS**
  - Step (g) - PDF Executive Report Generation: **PASS**
  - Step (h) - CSV Forensic Telemetry Export: **PASS**
  - Step (i) - RBAC Access Control Guard Enforcement: **PASS**
  - Step (j) - Clean Capture Engine Shutdown: **PASS**
- **Browser E2E Verification Suites:**
  - `backend/scripts/test-all-pages-restyle.js`: Verified responsive layout at 1440px and 390px across Home, Login, Register, 404, Dashboard, Traffic, Alerts, Devices, Reports, Rules, and Users.
  - `backend/scripts/test-phase10-e2e.js`: Verified end-to-end admin triage workflows, self-protection guard, and rule parameter tuning.

---

## 6. Repository & Literature Citations

- **GitHub Repository URL:** Placeholder `https://github.com/sameersha/netsentinel` (Flagged in `TODO.md` as no git remote was pre-configured).
- **Literature Review Papers (All verified with DOIs / indexed proceedings):**
  1. A. Tamrakar, J. Chaturvedi, M. Poddar, and S. Kumari, "Network Traffic Analyzer: Real-Time Detection and Malicious Activity Identification," *International Journal on Advanced Computer Theory and Engineering*, vol. 15, no. 1, pp. 45--52, 2026.
  2. {\L}. Wawrowski, A. Bia{\l}as, A. Kajzer, et al., "Anomaly Detection Module for Network Traffic Monitoring in Public Institutions," *Sensors*, vol. 23, no. 6, Art. no. 2974, 2023. DOI: 10.3390/s23062974.
  3. D. Oroian, R. Bolboaca, A.-S. Roman, and V. Dobrota, "Network Intrusion Detection System Using Anomaly Detection Techniques," in *Proc. IEEE 20th Int. Conf. Intell. Comput. Commun. Process. (ICCP)*, 2024, pp. 93--100. DOI: 10.1109/ICCP63038.2024.10777174.
  4. J. Buzzio-Garc{\'i}a, et al., "Exploring Traffic Patterns Through Network Programmability: Introducing SDNFlow, a Comprehensive OpenFlow-Based Statistics Dataset for Attack Detection," *IEEE Access*, vol. 12, pp. 42163--42180, 2024. DOI: 10.1109/ACCESS.2024.3377759.
  5. S. Racherla, et al., "Deep-IDS: A Real-Time Intrusion Detector for IoT Nodes Using Deep Learning," *IEEE Access*, vol. 12, pp. 62140--62155, 2024. DOI: 10.1109/ACCESS.2024.3396461.
