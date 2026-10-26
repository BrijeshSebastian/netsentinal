# NetSentinel MCA Mini Project Report: Action Items & Verification Log (TODO.md)

This document catalogs all administrative confirmations, manual signature requirements, technical adjustments, and architectural discrepancies between the initial project presentation (`review1.pptx`) and the final codebase implementation (`D:\netsentinel`).

---

## 1. Items to Fill or Confirm by Hand

Before final printing and hard-binding of the project report, please verify and complete the following details:

- [ ] **Declaration Date:**
  - Located on Page **iii** (Declaration page).
  - Currently formatted as: `Date: ........................` with `Place: Chengannur`.
  - Fill in the actual date of project submission (e.g., `26-10-2026`).
- [ ] **Project Guide Designation & Department:**
  - Guide Name: **Smt. Beena K**, Assistant Professor.
  - Department: **Department of Computer Engineering**, College of Engineering Chengannur.
  - Confirm whether any additional academic degrees (e.g., M.Tech, Ph.D.) should be appended to the guide's name.
- [ ] **Project Coordinators Confirmation:**
  - Template and department records list:
    - **Smt. Josmi Jose**, Assistant Professor, Dept. of Computer Engineering
    - **Smt. Santhy Viswam**, Assistant Professor, Dept. of Computer Engineering
    - **Smt. Celin T**, Assistant Professor, Dept. of Computer Engineering
  - *Action:* Confirm that this coordinator committee corresponds to the current academic year/semester (S3 MCA 2026).
- [ ] **Head of Department (HOD) Confirmation:**
  - Listed as: **Dr. Renu George**, Professor & Head of the Department, Dept. of Computer Engineering.
  - *Action:* Confirm if there has been any change in departmental headship prior to signing the Certificate (Page **ii**).
- [ ] **Principal Name Confirmation:**
  - Listed as: **Dr. Hari V.S**, Principal, College of Engineering Chengannur.
  - *Action:* Verify against current college administrative records.
- [ ] **Signatures Required:**
  - Certificate page (Page **ii**): Project Guide, Project Coordinator, Head of Department.
  - Declaration page (Page **iii**): Student signature above `Sameer Sha (CHN25MCA-2045)`.

---

## 2. Typography & Font Route Verification

- **Mandate:** Strict adherence to Times New Roman across the entire document (Cover, Front Matter, Headings, Body, Tables, Captions, and Code Listings). Zero sans-serif and zero monospace text.
- **Font Inspection Status:** Real TrueType Times New Roman was detected on the host system (`C:\Windows\Fonts\times.ttf`, `timesbd.ttf`, `timesi.ttf`, `timesbi.ttf`).
- **Compilation Engine:** **Dual-Engine Auto-Adaptive** (`xelatex` preferred, with automatic fallback for `pdflatex`).
  - Added `% !TEX program = xelatex` and `% !TEX TS-program = xelatex` at line 1 of `report/main.tex` for IDE auto-selection (VS Code, TeXstudio, Overleaf).
  - Uses `iftex` conditional branching:
    - Under **XeLaTeX / LuaLaTeX**: loads `fontspec` and binds system TrueType `Times New Roman`.
    - Under **pdfLaTeX**: loads `[utf8]{inputenc}`, `[T1]{fontenc}`, and `mathptmx` (Times), mapping all font families (`\familydefault`, `\sfdefault`, `\ttdefault`) to `\rmdefault`.
- **Font Configuration Applied in `report/main.tex`:**
  ```latex
  \usepackage{iftex}
  \ifxetex
    \usepackage{fontspec}
    \setmainfont{Times New Roman}
    \setsansfont{Times New Roman}
    \setmonofont{Times New Roman}
  \else\ifluatex
    \usepackage{fontspec}
    \setmainfont{Times New Roman}
    \setsansfont{Times New Roman}
    \setmonofont{Times New Roman}
  \else
    \usepackage[utf8]{inputenc}
    \usepackage[T1]{fontenc}
    \usepackage{mathptmx}
    \renewcommand{\familydefault}{\rmdefault}
    \renewcommand{\sfdefault}{\rmdefault}
    \renewcommand{\ttdefault}{\rmdefault}
  \fi\fi
  ```
- **Heading & Listing Overrides:**
  - Section headings configured with `\titleformat*{\section}{\large\rmfamily\bfseries}`.
  - Subsections and subsubsections enforce `\rmfamily\bfseries`.
  - Code listings environment configured with `basicstyle=\small\rmfamily` to prevent fallback to Computer Modern Typewriter (`cmtt`) or Courier.
- **PDF Font Audit (`pdffonts D:\netsentinel\report\main.pdf`):**
  - `TimesNewRomanPSMT` (CID TrueType) — Regular body text
  - `TimesNewRomanPS-BoldMT` (CID TrueType) — Headings, titles, table headers
  - `TimesNewRomanPS-ItalicMT` (CID TrueType) — Captions, emphasis, Latin names
  - `TimesNewRomanPS-BoldItalicMT` (CID TrueType) — Sub-emphasis
  - `CMSY10` (Type 1C Builtin) — Mathematical operational symbols ($\times$, $\ge$, $\rightarrow$)
  - **Result:** **100% Compliant**. No sans-serif (`cmss`), no monospace (`cmtt`), and no default Computer Modern Roman (`cmr12`) text font.

---

## 3. Discrepancies: First Review File (`review1.pptx`) vs. Final Implementation

The authoritative source of truth for all technical content in the report is the **actual functioning source code** in `D:\netsentinel`. The differences between the initial project presentation and the implemented system are summarized below:

| Feature / Dimension | First Review Presentation (`review1.pptx`) | Implemented Codebase (`D:\netsentinel`) | Report Action & Justification |
| :--- | :--- | :--- | :--- |
| **Primary Database** | Proposed **MongoDB** NoSQL document store (Slides 21, 23). | Implemented using **MySQL 8.4** relational database with InnoDB storage engine, relational foreign keys, and indexes (`backend/src/db/schema.sql`). | Documented MySQL relational architecture. Relational integrity is essential for linking ingested network traffic, devices, detection rules, alerts, and forensic reports. |
| **Backend Architecture** | Proposed **Python (Flask / FastAPI)** for the web backend (Slide 18). | Implemented using **Node.js + Express.js** with **Socket.io** for real-time WebSocket event streaming. Python is utilized exclusively for the standalone packet capture service (`capture/`). | Detailed the dual-service decoupled architecture: high-concurrency Node.js event-driven API server paired with Python Scapy promiscuous packet sniffing engine. |
| **Data Storage Model** | Proposed flat CSV / JSONL / PCAP files for telemetry storage. | Implemented structured tables in MySQL 8.4 for fast indexed querying, accompanied by rotated `.pcap` files on disk for deep forensic packet inspection. | Documented the hybrid storage pipeline: indexed relational storage for dashboard analytics and rotating PCAP writer for binary forensic capture. |
| **User Roles & RBAC** | Did not specify distinct user roles or permissions. | Implemented strict **Role-Based Access Control (RBAC)** distinguishing Security Analysts (`user`) and SecOps Administrators (`admin`). | Formally documented the RBAC security model, permission matrices, and protected frontend/backend route guards. |
| **Packet Capture Modes** | Only mentioned live packet capture via Scapy. | Implemented **Dual-Mode Engine**: Live Npcap network sniffing AND an asynchronous synthetic traffic generator (`simulator.py`) for safe reproducible testing. | Fully documented both Live and Simulator modes, explaining their role in reproducible lab evaluation and offline demonstration. |
| **Administrative Controls** | Omitted administrative controls and dynamic rule tuning. | Implemented interactive capture engine start/stop toggles, runtime detection rule threshold editing, and user account management. | Detailed in System Architecture, Use Case model, DFD Level 1 (Admin), and Testing chapters. |

---

## 4. Verification of Diagram Artifacts

All diagrams incorporated into Chapter 5 and Chapter 6 have been verified for accuracy against the running system:

1. **Figure 5.1 (`SystemArchitecture.png`):**
   - Verified that the Packet Capture Engine is correctly represented as an integrated component of NetSentinel rather than an external third-party service.
   - Accurately depicts Npcap/Scapy, Express API, Socket.io stream, MySQL database, and React frontend.
2. **Figure 5.2 (`usecase.png`):**
   - Verified that both actors (**Security Analyst** and **SecOps Administrator**) and their respective use cases match the implemented routes and UI controls.
3. **Figure 5.3 (`dfd-level-0.png`), Figure 5.4 (`dfd-level-1-user-large.png`), Figure 5.5 (`dfd-level-1-admin-large.png`):**
   - Verified process flow and data stores (`D1: Users`, `D2: Devices`, `D3: Network Traffic`, `D4: Detection Rules`, `D5: Alerts`, `D6: Reports`).
4. **Figure 5.6 (`dbschema.png`):**
   - Verified all 8 tables, column types, primary keys, and foreign key relationships against `backend/src/db/schema.sql`.

---

## 5. Screen Inventory & Skipped Images Rationale

- **Included Screenshots (7 Primary Interface Views):**
  - `Screenshots/Screenshot-user-dashboard.png` (Figure 7.1) — Analyst live telemetry dashboard.
  - `Screenshots/Screenshot-admin-dashboard.png` (Figure 7.2) — Administrator dashboard with capture control header.
  - `Screenshots/Screenshot-admin-live-traffic.png` (Figure 7.3) — Live Traffic Explorer with protocol filtering.
  - `Screenshots/Screenshot-admin-realtime-anyalysing.png` (Figure 7.4) — Real-time packet inspection drawer and hex view.
  - `Screenshots/Screenshot-admin-alerts.png` (Figure 7.5) — Security alerts triage and status management.
  - `Screenshots/Screenshot-admin-tracked-devices.png` (Figure 7.6) — Discovered network hosts and risk scoring.
  - `Screenshots/Screenshot-admin-detection-rules.png` (Figure 7.7) — Rule parameter configuration and threshold sliders.
- **Omitted Views & Rationale:**
  - *Login and Registration Screens (`/login`, `/register`):* Excluded from the diagrams folder to maintain focus on the core technical security operations.
  - *User Management Screen (`/users`):* Excluded to fit Chapter 7 screenshots cleanly into 3 sequential pages without creating orphaned, single-image pages.
  - *Note for Student:* If your project guide specifically requests authentication screenshots in Chapter 7, take browser screenshots of `http://localhost:5173/login` and `http://localhost:5173/register`, place them in `report/Screenshots/`, and insert them before Figure 7.1 in `report/main.tex`.

---

## 6. Software Testing & Execution Notes

- **Automated Verification Suite Results (`scripts/e2e-check.ps1`):**
  - Unit and integration tests TC-UT-01 through TC-UT-05, TC-IT-01 through TC-IT-06, and TC-ST-01 through TC-ST-07 executed with a **100% Pass** rate.
- **Unexecuted Test Case:**
  - **TC-ST-08: High-Throughput Hardware Promiscuous Sniffing Under Multi-Gigabit Load (10 Gbps+ line rate).**
  - *Status:* **Not Executed (Deferred)**.
  - *Reason:* Standard college lab workstations and student laptops operate on 1 Gbps integrated Ethernet NICs. Multi-gigabit continuous line-rate capture requires specialized enterprise packet-acceleration hardware (e.g., Intel DPDK, PF\_RING, or dedicated hardware capture cards) not available in the mini-project development environment.

---

## 7. External Code Repository Citation

- In `report/main.tex`, Reference **[6]** cites the project code repository:
  ```latex
  \bibitem{ref:netsentinel_repo}
  S.~Sha, ``NetSentinel: Web-Based Network Traffic Analyzer and Intrusion Detection System,''
  GitHub repository, 2026. [Online]. Available: \url{https://github.com/sameersha/netsentinel}
  ```
- *Action:* If your repository is hosted at a different personal URL (or on GitLab / college intranet), update the URL on line 1008 of `report/main.tex` and recompile with `xelatex main.tex`.
