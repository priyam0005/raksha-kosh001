<div align="center">

<img src="https://img.shields.io/badge/MERN-Stack-blue?style=flat-square" />
<img src="https://img.shields.io/badge/Security-4--Layer%20Defense-red?style=flat-square" />



# 🛡️ Raksha Kosh
### Secure File Upload Portal — 4-Layer Defense-in-Depth Architecture

*Analyze suspicious files with advanced multi-layer security validation. Detect malware and breaches through comprehensive file scanning.*

</div>

---

> ⚠️ **Beta / Preview Notice**
>
> This is an early preview of Raksha Kosh. The frontend is hosted on **Vercel** and the backend on **Render (free tier)**.
>
> **Known limitations on the live version:**
> - Free tier backend on Render **spins down after inactivity** — expect a **30–60 second cold start** when the backend hasn't been used recently. Just wait and retry.
> - File scans above **~10 MB** may fail or time out due to free tier resource constraints.
>
> For the best and most reliable experience, we recommend **cloning the repo and running the app on your local machine** until we upgrade to a premium backend plan:
> ```bash
> git clone https://github.com/priyam0005/raksha_kosh001.git
> ```
> Full local setup instructions are in the [Getting Started](#getting-started) section.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Supported File Types](#supported-file-types)
- [4-Layer Security Architecture](#4-layer-security-architecture)
- [Scan History & Threat Explanation](#scan-history--threat-explanation)
- [Processing Flow](#processing-flow)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Environment Variables](#environment-variables)
- [API Endpoints](#api-endpoints)
- [Security Benchmarks](#security-benchmarks)
- [References](#references)

---

## Overview

Raksha Kosh is a full-stack file security platform that runs every uploaded file through a **4-Layer Defense-in-Depth pipeline**. Each layer performs a distinct type of analysis — from binary signature validation to deep content inspection, metadata forensics, and multi-engine AV scanning. Execution halts at the first detected threat, and the user receives a precise, layer-specific explanation of what was found and why the file was rejected.

**Key stats:**
- Consistent detection across known malware signatures via multi-engine consensus
- `AES-256` encryption for all files that pass all layers
- Full audit trail and quarantine log for every rejected file

---



## Screenshots

### Home
![Home](./ScreenShot/Home.png)

### Login
![Login](./ScreenShot/login.png)

### Dashboard
![Dashboard](./ScreenShot/Dashboard.png)

### Scan
![Scan](./ScreenShot/scan.png)



## Supported File Types

Raksha Kosh supports scanning across **6 file categories**:

| Category | Extensions | Notes |
|----------|-----------|-------|
| **Images** | `.jpg`, `.jpeg`, `.png` | EXIF stripping, steganography removal, structure validation |
| **Documents** | `.pdf` | Script injection scan, embedded file detection |
| **Office — Word** | `.docx` | XML extraction, VBA macro scan, XXE injection detection |
| **Office — Spreadsheet** | `.xlsx` | XML extraction, macro and formula injection scan |
| **Office — Presentation** | `.pptx` | Embedded script and macro detection |
| *(Blocked)* | All others | Rejected at client-side pre-validation |

> Max file size: **10 MB**  
> Magic bytes are verified against the actual binary header — not just the file extension.

---

## 4-Layer Security Architecture

Each file passes through all 4 layers **in sequence**. If any layer fails, the file is immediately rejected and quarantined. The user sees exactly which layer failed and why.

---

### Layer 1 — Magic Number Validation

> *"Never trust the extension. Read the bytes."*

Reads the binary file signature (first 4–16 bytes) and compares against a strict whitelist of known-safe magic numbers.

| Format | Magic Bytes |
|--------|-------------|
| JPEG | `FF D8 FF E0` |
| PNG | `89 50 4E 47` |
| PDF | `25 50 44 46` |
| DOCX/XLSX/PPTX | `50 4B 03 04` (ZIP header) |

**Catches:** Executables renamed as documents, polyglot files, extension spoofing (e.g., `.exe` disguised as `.pdf`)

---

### Layer 2 — Deep Content Analysis

> *"The extension is clean. The content might not be."*

Performs format-specific deep inspection of the actual file content.

- **Images** — Sharp library validates internal structure, strips all EXIF metadata, and re-encodes to eliminate steganographic payloads
- **PDFs** — Regex scan for dangerous PDF commands: `/JavaScript`, `/OpenAction`, `/EmbeddedFile`, `/Launch`
- **Office Documents** — Unzips DOCX/XLSX/PPTX archives, parses XML internals, scans for VBA macros and XXE injection patterns

**Catches:** Embedded scripts, macro viruses, hidden executables, file injection attacks, steganography

---

### Layer 3 — Metadata Forensics

> *"Even clean files can carry dangerous metadata."*

Extracts and forensically analyzes all EXIF/XMP metadata embedded in the file.

- Scans metadata fields for code injection patterns (`<script>`, `javascript:`, `eval()`)
- Validates timestamps — creation date in the future flags the file as suspicious
- Detects and strips GPS coordinates and privacy-leaking fields
- Identifies anomalies in authorship, software, and creation tool fields

**Catches:** Metadata-based injection attacks, timestamp manipulation, tracking payloads, suspicious authorship markers

---

### Layer 4 — AV Engine

> *"Cross-reference the signature against everything known."*

Multi-engine antivirus scanning for maximum coverage against known threats.

- **VirusTotal API** — Consensus from 70+ independent AV engines
- **SHA-256 hash check** — Against known malware hash databases

**Catches:** Known viruses, trojans, ransomware, spyware

---

### Security Flow

```
File Upload
    │
    ▼
[L1] Magic Number Validation      ──✗──▶  REJECTED
    │ ✓
    ▼
[L2] Deep Content Analysis        ──✗──▶  REJECTED
    │ ✓
    ▼
[L3] Metadata Forensics           ──✗──▶  REJECTED
    │ ✓
    ▼
[L4] AV Engine                    ──✗──▶  REJECTED
    │ ✓
    ▼
AES-256 Encrypted Storage ✓
Full Audit Trail Logged
```

> A file is **only stored** if it passes **all 4 layers**.

---

## Scan History & Threat Explanation

### Sign In to Access Your Scan History

Authenticated users can view their **complete personal scan history** from the Profile dashboard. Every scan is stored and accessible after login.

Each scan record shows:
- File name and size
- Scan timestamp
- Overall result: **CLEAN** or **THREAT**

### Threat Explanation — Know Exactly What Failed

When a file is detected as a threat, Raksha Kosh doesn't just say "upload failed." It tells you precisely:

**1. Which layer stopped the file** — e.g., `L3 · Metadata Forensics`

**2. The technical failure detail** — e.g.:
> `Parse error: 15 byte(s) of data found after IEND chunk — possible injected payload`

**3. A plain-language analysis** — e.g.:
> `Blocked at Layer 3 — Metadata contains injected payloads, malicious scripts, or suspicious timestamps.`

**4. How many layers the file passed** before being rejected — displayed as a visual summary (e.g., `2 layers passed`) so you understand exactly how far through the pipeline the file got.

Clean files show a full layer-by-layer breakdown with pass status for every layer in sequence (L1 → L2 → L3 → L4).

---

## Processing Flow

```
Client Side (0.5s)
├── Size check (max 50MB)
├── Extension whitelist validation
├── Magic bytes preview via FileReader API
└── Filename sanitization

Server Side
├── Step 1 · Magic Number Validation (0.2s)   — Binary header verification
├── Step 2 · Deep Content Analysis (1–2s)     — Format-specific content scan
├── Step 3 · Metadata Forensics (0.3s)        — EXIF/XMP extraction & analysis
└── Step 4 · AV Engine (2–3s)                 — VirusTotal + hash check

Result
├── PASS → AES-256 encrypt → Store in /secure → Log audit trail
└── FAIL → Quarantine → Log reason → Return layer-specific explanation
```

**Total processing time: 4–7 seconds per file**

---

## Tech Stack

### Frontend (React + Vite)

| Library | Version | Purpose |
|---------|---------|---------|
| `react` | ^19.2.0 | UI framework |
| `react-dom` | ^19.2.0 | DOM rendering |
| `react-router-dom` | ^7.13.0 | Client-side routing |
| `@reduxjs/toolkit` | ^2.11.2 | State management |
| `react-redux` | ^9.2.0 | Redux bindings |
| `redux` | ^5.0.1 | State container |
| `axios` | ^1.13.5 | HTTP client |
| `framer-motion` | ^12.34.3 | Animations |
| `lucide-react` | ^0.575.0 | Icons |
| `react-hook-form` | ^7.71.2 | Form handling |
| `react-icons` | ^5.5.0 | Icon library |
| `tailwindcss` | ^4.2.0 | Styling |

### Backend (Node.js + Express)

| Library | Version | Purpose |
|---------|---------|---------|
| `express` | ^5.2.1 | Web framework |
| `mongoose` | ^9.2.1 | MongoDB ODM |
| `multer` | ^2.0.2 | Multipart file handling & temp storage |
| `sharp` | ^0.34.5 | Image validation, EXIF stripping, re-encoding |
| `adm-zip` | ^0.5.16 | Office document XML extraction |
| `axios` | ^1.13.5 | VirusTotal API v3 calls |
| `bcrypt` | ^6.0.0 | Password hashing |
| `bcryptjs` | ^3.0.3 | Password hashing (JS fallback) |
| `jsonwebtoken` | ^9.0.3 | JWT authentication |
| `crypto` | ^1.0.1 | SHA-256 hashing, AES-256-CBC encryption |
| `dotenv` | ^17.3.1 | Environment variable management |
| `cors` | ^2.8.6 | Cross-origin resource sharing |
| `form-data` | ^4.0.5 | Multipart form data handling |
| `fs` | ^0.0.1-security | File system operations |
| `path` | ^0.12.7 | File path utilities |

### Database (MongoDB)

| Collection | Purpose |
|------------|---------|
| `users` | User accounts and auth |
| `scans` | Scan records with per-layer results |
| `quarantine` | Flagged/rejected files |
| `auditLogs` | Full access and event trail |

**Indexed fields:** `fileHash` (deduplication), `uploadedBy`, `scanStatus`

**Scan schema:** `originalName`, `storedName`, `mimeType`, `size`, `hash`, `timestamps`, `layers[]`, `scanStatus`, `failedLayerName`, `reason`

---

## Project Structure

```
raksha-kosh/
│
├── frontend/
│   ├── public/
│   └── src/
│       ├── assets/
│       ├── component/
│       │   ├── about.jsx
│       │   ├── footer.jsx
│       │   ├── header.jsx
│       │   ├── home.jsx                ← Upload terminal UI
│       │   ├── login.jsx
│       │   ├── profile.jsx             ← Scan history dashboard
│       │   ├── Register.jsx
│       │   └── terms.jsx
│       ├── loading/
│       │   └── loading.jsx
│       ├── others/
│       │   ├── rakshalogo.jsx
│       │   └── scrollToTop.jsx
│       ├── store/
│       │   ├── index.js
│       │   ├── login.js
│       │   ├── rootReducer.js
│       │   ├── scan.js
│       │   └── scanUser.js             ← Fetch user scan history
│       ├── App.jsx
│       └── main.jsx
│
└── backend/
    ├── controller/
    │   ├── auth.js                     ← Register / Login
    │   ├── layers.js                   ← Layer orchestration
    │   ├── scan.js                     ← Scan entry point
    │   ├── uploadController.js         ← File handling
    │   └── userScan.js                 ← Scan history retrieval
    ├── middleware/
    │   ├── authMiddleware.js
    │   └── protect.js
    ├── RAKSHA/                         ← Core scanning engine
    │   ├── MagicNumber.js              ← Layer 1: Magic Number Validation
    │   ├── deepContent.js              ← Layer 2: Deep Content Analysis
    │   ├── layer4MetadataForensics.js  ← Layer 3: Metadata Forensics
    │   └── layer3antivirus.js          ← Layer 4: AV Engine
    ├── router/
    │   ├── raksha.js
    │   └── scanRouter.js
    ├── schema/
    │   ├── scanSchema.js
    │   └── userSchema.js
    ├── uploads/
    ├── app.js
    └── package.json
```

---

## Getting Started

### Prerequisites

- Node.js >= 18
- MongoDB (local or Atlas)
- VirusTotal API key (free tier: 500 calls/day)

### Setup Instructions

**1. Clone the repository**
```bash
git clone https://github.com/your-username/raksha-kosh.git
cd raksha-kosh
```

**2. Backend setup**
```bash
cd backend
npm install
```

Create a `.env` file in the `/backend` directory:
```env

MONGO_URI=mongodb_connection_string
JWT_SECRET=jwt_secret
CLIENT_URL=Backend_server_url
VIRUSTOTAL_API_KEY=virustotal_key

```

Start the backend server:
```bash
npm run dev
```

**3. Frontend setup**
```bash
cd ../frontend
npm install
```

Start the frontend dev server:
```bash
npm run dev
```

**4. Open in browser**
```
http://localhost:5173
```

> Make sure MongoDB is running before starting the backend. The backend runs on `http://localhost:5000` by default.

---

## API Endpoints

### Auth

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/auth/register` | Register new user |
| `POST` | `/api/auth/login` | Login, returns JWT |

### Scan

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `POST` | `/api/scan/upload` | ✓ | Upload and scan a file through all 4 layers |
| `GET` | `/api/scan/user` | ✓ | Fetch authenticated user's full scan history |

> All protected routes require: `Authorization: Bearer <token>`

---

## Security Benchmarks

| Metric | Raksha Kosh | Industry Average |
|--------|-------------|-----------------|
| Threat detection rate | **Multi-engine** | 85% (single-layer) |
| File encryption | **AES-256** | optional |
| Storage deduplication | **SHA-256 hash** | rarely implemented |
| Audit trail | **Full** | rarely implemented |

---

## References

**Academic**
- IEEE 2024 — *Multi-Layer Security Framework for File Upload Systems*
- OWASP Top 10 — Unrestricted File Upload (#8)
- NIST Cybersecurity Framework v2.0

**Industry Standards**
- CWE-434: Unrestricted Upload of File with Dangerous Type
- MITRE ATT&CK: Malicious File (T1204)

**Open Source Libraries**
- [Multer](https://npmjs.com/package/multer) — MIT
- [file-type](https://github.com/sindresorhus/file-type) — MIT
- [sharp](https://sharp.pixelplumbing.com) — Apache 2.0
- [pdf-parse](https://npmjs.com/package/pdf-parse) — MIT

**APIs & Threat Intelligence**
- [VirusTotal API v3](https://developers.virustotal.com)
- MISP Threat Sharing Platform
- CVE Database

---

## 🔑 Environment Variables (Hackathon Reference)

> ⚠️ **These credentials are shared temporarily for hackathon evaluation purposes only. They will be rotated after the event.**

```env
VIRUSTOTAL_API_KEY=a2353b368ddc18812342a003aaa7e13b158e3933f9e3045059c52181055d5dd9
VT_TIMEOUT_MS=10000
MONGO_URI=mongodb+srv://pradumnpathak87_db_user:HgD6QzjDEAxE6mB2@raksha.saxrrag.mongodb.net/
JWT_SECRET_TOKEN=her
CLIENT_URL=https://raksha-kosh001.vercel.app
```

---

## Team

| # | Name | Role |
|---|------|------|
| 1 | Lakshya Nath | Team Lead |
| 2 | Pradumn Kumar Pathak | Developer |
| 3 | Kuldeep Bishnoi | Testing & QA |

---

<div align="center">

*Never trust. Always verify.*

</div>
