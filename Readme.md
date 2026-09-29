# 🌉 Vidya-Setu — Digital Inclusion Platform for Rural & Tribal Higher Education

> *"Vidya-Setu" = "Bridge of Knowledge."* An offline-first, multilingual, voice-enabled, AI-assisted learning platform for **Windows, Linux, macOS and Android**, built for students with limited connectivity, language barriers and low career awareness.

![Status](https://img.shields.io/badge/status-in%20development-orange)
![Platforms](https://img.shields.io/badge/platforms-Windows%20%7C%20Linux%20%7C%20macOS%20%7C%20Android-blue)
![License](https://img.shields.io/badge/license-Apache%202.0-green)

---

## 📑 Table of Contents

1. [Problem Statement](#1-problem-statement)
2. [Solution Overview](#2-solution-overview)
3. [Key Features](#3-key-features)
4. [System Architecture](#4-system-architecture)
5. [Tech Stack](#5-tech-stack)
6. [Repository Structure](#6-repository-structure)
7. [Prerequisites](#7-prerequisites)
8. [Quick Start](#8-quick-start)
9. [Building for Each Platform](#9-building-for-each-platform)
10. [Step-by-Step Implementation Guide](#10-step-by-step-implementation-guide)
11. [Offline-First & Sync Design](#11-offline-first--sync-design)
12. [AI Pipeline (Doubt Resolution, Voice, Translation)](#12-ai-pipeline)
13. [Low-Bandwidth Optimization](#13-low-bandwidth-optimization)
14. [Scholarship Discovery Engine](#14-scholarship-discovery-engine)
15. [Mentoring & Career Counseling](#15-mentoring--career-counseling)
16. [Database Schema](#16-database-schema-simplified)
17. [API Reference (Summary)](#17-api-reference-summary)
18. [Security & Privacy](#18-security--privacy)
19. [Accessibility & Inclusion](#19-accessibility--inclusion)
20. [Testing Strategy](#20-testing-strategy)
21. [CI/CD & Release](#21-cicd--release)
22. [Roadmap](#22-roadmap)
23. [Success Metrics](#23-success-metrics)
24. [Contributing](#24-contributing)
25. [License](#25-license)

---

## 1. Problem Statement

Students from rural and tribal regions face four compounding barriers:

| Barrier | Impact |
|---|---|
| Limited/intermittent internet | Cannot stream lectures, submit work, or use cloud tools |
| Language barriers | Most quality content is in English/Hindi; local languages are under-served |
| Lack of career awareness | Students choose courses without knowing job outcomes or pathways |
| Limited access to quality resources | Few libraries, teachers, mentors and lab facilities |

**Goal:** Build one platform that works with *zero or poor internet*, speaks the student's *own language*, and connects them to *knowledge, money (scholarships), people (mentors) and futures (careers)*.

---

## 2. Solution Overview

Vidya-Setu is a **local-first application**. All core functions (lessons, quizzes, AI tutor, voice, scholarship matching, career guidance) run **on the device**. When any connectivity appears (2G, Wi-Fi at a cafe, college hotspot), the app **syncs small deltas** in the background. Where even that is impossible, content and data move **device-to-device** (Wi-Fi Direct / local hotspot / USB / SD card) via a **Community Hub** (a Raspberry Pi/PC placed at a college or village center).

**Design principles**

1. **Offline is the default, online is a bonus.**
2. **Voice and local language first, text second.**
3. **Every byte counts** — compress, delta-sync, degrade gracefully.
4. **Low-end hardware friendly** — must run on 3 GB RAM Android phones and 4 GB RAM old laptops.
5. **Privacy by design** — data stays on device unless the student opts in.

---

## 3. Key Features

| # | Feature | Description |
|---|---|---|
| 1 | **Offline-first learning** | Downloadable course packs (text, audio, low-res video, quizzes) stored locally; progress tracked offline |
| 2 | **Regional language support** | UI + content in Hindi, Marathi, Gujarati, Bengali, Odia, Tamil, Telugu, Kannada, Malayalam, Punjabi, Assamese, and tribal languages (e.g., Gondi, Santali, Bhili — as community datasets grow) |
| 3 | **AI doubt resolution** | On-device small LLM + retrieval over course material (RAG); optional cloud LLM when online |
| 4 | **Voice-based interaction** | Speak a question, hear the answer; voice navigation for low-literacy users |
| 5 | **Low-bandwidth optimization** | Audio-only mode, 144p/240p video, delta sync, resumable downloads, text-only fallback |
| 6 | **Government scholarship discovery** | Rule-based eligibility engine over a locally cached scholarship database (NSP, state, private/CSR) with deadline reminders |
| 7 | **Digital mentoring** | Asynchronous voice/text mentoring (store-and-forward), group sessions when online |
| 8 | **Career counseling** | Interest/aptitude assessment, career roadmaps, skill-gap analysis, local job/internship/exam info |
| 9 | **Community Hub mode** | Village/college server that distributes content over local Wi-Fi |
| 10 | **SMS/IVR/WhatsApp fallback** | Deadline alerts and basic Q&A for feature-phone users (optional module) |

---

## 4. System Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│                       CLIENT (Windows/Linux/macOS/Android)           │
│                                                                      │
│  ┌────────────────────── UI (React + TypeScript) ─────────────────┐  │
│  │ Lessons │ AI Tutor │ Voice │ Scholarships │ Mentor │ Career    │  │
│  └───────────────────────────┬────────────────────────────────────┘  │
│                              │ IPC (Tauri commands)                  │
│  ┌───────────────────────────▼────────────────────────────────────┐  │
│  │                    Rust Core (Local-First Engine)              │  │
│  │  SQLite + FTS5  │ Sync Engine (CRDT/outbox) │ Content Manager  │  │
│  │  Eligibility Engine │ Crypto │ P2P Transfer │ Scheduler        │  │
│  └───────┬───────────────────────┬────────────────────────┬───────┘  │
│          │                       │                        │          │
│  ┌───────▼────────┐   ┌──────────▼──────────┐   ┌─────────▼───────┐  │
│  │ On-device AI   │   │ Vector Store        │   │ Voice Engine    │  │
│  │ (llama.cpp,    │   │ (sqlite-vec /       │   │ (Whisper.cpp /  │  │
│  │  quantized LLM,│   │  embeddings)        │   │  Vosk STT,      │  │
│  │  IndicTrans2)  │   │                     │   │  Piper TTS)     │  │
│  └────────────────┘   └─────────────────────┘   └─────────────────┘  │
└───────────────┬──────────────────────────────────────┬───────────────┘
                │ (opportunistic, compressed deltas)   │ (Wi-Fi Direct / LAN)
        ┌───────▼──────────────┐              ┌────────▼──────────────┐
        │  CLOUD BACKEND       │              │  COMMUNITY HUB        │
        │  FastAPI + Postgres  │◄────sync─────┤  (Raspberry Pi / PC)  │
        │  pgvector + MinIO    │              │  Content cache + sync │
        │  Redis + Celery      │              └───────────────────────┘
        │  Cloud LLM gateway   │
        │  Bhashini/IndicTrans │
        │  Scholarship crawler │
        └──────────────────────┘
```

---

## 5. Tech Stack

| Layer | Technology | Why |
|---|---|---|
| **Cross-platform shell** | **Tauri 2** (Rust + WebView) | One codebase for Windows, Linux, macOS **and Android**; tiny binaries (~10 MB vs 150 MB Electron) |
| **Frontend** | React + TypeScript + Vite + Tailwind + i18next | Fast, component-driven, strong i18n |
| **Local DB** | SQLite (+FTS5, sqlite-vec) via `rusqlite`/`sqlx` | Zero-config, offline, full-text and vector search |
| **On-device LLM** | `llama.cpp` (GGUF, 4-bit quantized: Gemma-2B / Qwen2.5-1.5B / Llama-3.2-1B/3B) | Runs on low-end CPUs |
| **Translation** | AI4Bharat **IndicTrans2** (distilled/CTranslate2 int8) offline; **Bhashini** API online | Best-in-class Indian languages |
| **Speech-to-Text** | `whisper.cpp` (tiny/base) + Vosk Indian models; AI4Bharat IndicWhisper/Bhashini ASR online | Offline voice input |
| **Text-to-Speech** | Piper TTS / Android TTS / AI4Bharat Indic-TTS | Offline voice output |
| **Embeddings** | `multilingual-e5-small` (ONNX, int8) | Multilingual RAG on device |
| **Backend API** | Python **FastAPI**, Pydantic, SQLAlchemy | Rapid dev, async, ML ecosystem |
| **Cloud DB** | PostgreSQL + `pgvector` | Relational + vector search |
| **Object storage** | MinIO / S3-compatible | Content packs |
| **Queue / cache** | Redis + Celery | Crawlers, packaging jobs |
| **Real-time (optional)** | WebRTC + fallback to async audio | Mentor calls when online |
| **Auth** | OAuth2/OIDC (Keycloak), phone-OTP, offline PIN | Works for students without email |
| **Infra** | Docker, Docker Compose, Kubernetes (prod), Terraform | Reproducible |
| **CI/CD** | GitHub Actions | Multi-OS builds |

> **Alternative stack:** Flutter (Dart) for UI + FFI to Rust/C++ for AI. Choose this if your team is stronger in Flutter. The architecture remains the same.

---

## 6. Repository Structure

```
vidya-setu/
├── README.md
├── LICENSE
├── docker-compose.yml
├── .github/workflows/            # CI: build, test, release
│
├── app/                          # Cross-platform client (Tauri 2)
│   ├── src/                      # React + TS frontend
│   │   ├── features/
│   │   │   ├── learn/            # courses, lessons, quizzes
│   │   │   ├── tutor/            # AI doubt resolution UI
│   │   │   ├── voice/            # mic, waveform, TTS controls
│   │   │   ├── scholarships/
│   │   │   ├── mentoring/
│   │   │   └── career/
│   │   ├── i18n/locales/         # hi.json, mr.json, ta.json, ...
│   │   ├── components/
│   │   └── main.tsx
│   ├── src-tauri/                # Rust core
│   │   ├── src/
│   │   │   ├── db/               # SQLite, migrations
│   │   │   ├── sync/             # outbox, CRDT, delta sync
│   │   │   ├── ai/               # llama.cpp, whisper, piper bindings
│   │   │   ├── content/          # pack download/verify/unpack
│   │   │   ├── p2p/              # Wi-Fi Direct / LAN transfer
│   │   │   ├── eligibility/      # scholarship rules engine
│   │   │   └── main.rs
│   │   ├── gen/android/          # Android project (generated)
│   │   └── tauri.conf.json
│   └── package.json
│
├── backend/                      # Cloud API
│   ├── app/
│   │   ├── api/                  # routers: auth, content, sync, mentor...
│   │   ├── services/             # rag, translate, scholarship, career
│   │   ├── models/
│   │   ├── workers/              # celery tasks (crawler, packager)
│   │   └── main.py
│   ├── migrations/               # Alembic
│   └── requirements.txt
│
├── hub/                          # Community Hub (Raspberry Pi image / Docker)
│   ├── docker-compose.hub.yml
│   └── README.md
│
├── ml/                           # Model prep & evaluation
│   ├── quantize/                 # GGUF / CT2 / ONNX conversion scripts
│   ├── evals/                    # Indic QA, WER, BLEU benchmarks
│   └── datasets/
│
├── content-tools/                # Course-pack builder CLI
│   └── packbuilder/              # markdown/video → compressed .vspack
│
├── data/
│   ├── scholarships/             # seed JSON + schema
│   └── careers/                  # career graph, exam calendars
│
└── docs/                         # architecture, ADRs, user guides
```

---

## 7. Prerequisites

### Common (all platforms)
- **Git** ≥ 2.40
- **Node.js** ≥ 20 and **pnpm** ≥ 9
- **Rust** (stable) via [rustup](https://rustup.rs) ≥ 1.80
- **Python** ≥ 3.11 (backend & ML tooling)
- **Docker** & Docker Compose (backend)
- **CMake** ≥ 3.24 and a C/C++ compiler (for llama.cpp / whisper.cpp)

### Per-OS
| OS | Extra requirements |
|---|---|
| **Windows 10/11** | Visual Studio Build Tools (C++ workload), WebView2 runtime |
| **Linux (Ubuntu/Debian)** | `sudo apt install libwebkit2gtk-4.1-dev build-essential libssl-dev libayatana-appindicator3-dev librsvg2-dev libasound2-dev` |
| **macOS** | Xcode Command Line Tools (`xcode-select --install`) |
| **Android** | Android Studio, SDK 34+, NDK r26+, JDK 17; set `ANDROID_HOME`, `NDK_HOME`; `rustup target add aarch64-linux-android armv7-linux-androideabi x86_64-linux-android` |

---

## 8. Quick Start

```bash
# 1. Clone
git clone https://github.com/<your-org>/vidya-setu.git
cd vidya-setu

# 2. Start the backend (Postgres, Redis, MinIO, API)
cp backend/.env.example backend/.env
docker compose up -d

# 3. Download AI models (quantized, ~1.5–2 GB total)
cd ml/quantize
python -m venv .venv && source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
python fetch_models.py --profile lite               # lite | standard | full
cd ../..

# 4. Run the desktop app in dev mode
cd app
pnpm install
pnpm tauri dev

# 5. Run on an Android device/emulator
pnpm tauri android init      # first time only
pnpm tauri android dev
```

**Model profiles**

| Profile | RAM needed | Contents |
|---|---|---|
| `lite` | 3 GB | 1B LLM (Q4), Whisper-tiny, Piper voices, e5-small |
| `standard` | 6 GB | 3B LLM (Q4), Whisper-base, IndicTrans2-distilled |
| `full` | 12 GB+ | 7B LLM (Q4), Whisper-small, full translation |

---

## 9. Building for Each Platform

```bash
cd app
pnpm install

# ---------- Windows (run on Windows) ----------
pnpm tauri build --bundles msi,nsis
# → src-tauri/target/release/bundle/msi/*.msi

# ---------- macOS (run on macOS) ----------
pnpm tauri build --bundles dmg,app
# → universal binary: pnpm tauri build --target universal-apple-darwin

# ---------- Linux (run on Linux) ----------
pnpm tauri build --bundles deb,appimage,rpm
# → src-tauri/target/release/bundle/{deb,appimage,rpm}/

# ---------- Android ----------
pnpm tauri android build --apk          # debug/side-loadable APK
pnpm tauri android build --aab          # Play Store bundle
# Signing: configure keystore in src-tauri/gen/android/keystore.properties
```

> **Tip for low-end Android:** ship ABI-specific APKs (`--split-per-abi`) and keep the base APK < 50 MB. Download AI models and course packs **after install** (resumable, Wi-Fi or hub).

---

## 10. Step-by-Step Implementation Guide

The project is designed to be delivered in **8 phases**. Each phase produces a working, testable increment.

### Phase 0 — Discovery & Planning (Week 1–2)

1. **Interview 30–50 students, teachers and NGO workers** in target regions. Capture: devices used, data plans, languages, literacy levels, peak connectivity times.
2. Define **personas** (e.g., first-generation learner on a ₹6,000 Android phone; college student sharing a village PC).
3. Pick **pilot languages** (start with 3: e.g., Hindi + Marathi + Gondi/Santali, or per your region) and **pilot subjects** (e.g., first-year Maths, English, Computer Basics, Commerce).
4. Partner with a local college/NGO/CSC (Common Service Centre) to host a **Community Hub**.
5. Write the **Definition of Done** and success metrics (see §23).

**Deliverable:** PRD, personas, language & content plan, risk register.

---

### Phase 1 — Project Setup & Foundations (Week 2–3)

1. Create the monorepo (structure in §6); set up `pnpm` workspaces and a `Cargo` workspace.
2. Bootstrap the client:
   ```bash
   pnpm create tauri-app app --template react-ts
   cd app && pnpm tauri android init
   ```
3. Bootstrap the backend:
   ```bash
   mkdir backend && cd backend
   python -m venv .venv && source .venv/bin/activate
   pip install fastapi uvicorn[standard] sqlalchemy alembic asyncpg pydantic-settings celery redis
   ```
4. Write `docker-compose.yml` with services: `postgres` (with pgvector), `redis`, `minio`, `api`, `worker`.
5. Configure **GitHub Actions** matrix: `windows-latest`, `macos-latest`, `ubuntu-latest`, plus an Android job.
6. Set up **linting/formatting** (ESLint, Prettier, `cargo clippy`, Ruff, Black) and **pre-commit** hooks.
7. Set up **i18n** skeleton with `i18next` and a translation workflow (Weblate/Crowdin/Pontoon or plain JSON PRs).

**Deliverable:** "Hello World" app running on all 4 platforms; backend health-check endpoint; green CI.

---

### Phase 2 — Local-First Data Layer & Content Packs (Week 3–6)

1. **Local DB (Rust):** Create SQLite with migrations (`sqlx migrate`). Tables: `users`, `courses`, `lessons`, `quizzes`, `progress`, `outbox`, `scholarships`, `mentor_messages`, `career_profiles` (see §16). Enable WAL mode and FTS5.
2. **Content pack format (`.vspack`)** — a zip/zstd archive:
   ```
   maths101.vspack
   ├── manifest.json     # id, version, language, size, checksum, dependencies
   ├── lessons/*.md      # Markdown/MDX, lightweight
   ├── audio/*.opus      # 12–24 kbps Opus narration
   ├── video/*.webm      # 144p/240p AV1/VP9 optional
   ├── quizzes/*.json
   ├── embeddings.bin    # pre-computed vectors for RAG
   └── signature.sig     # Ed25519 signature
   ```
3. Build the **pack builder CLI** (`content-tools/packbuilder`): Markdown → compressed pack; auto-generates narration audio (Piper TTS), transcripts, embeddings and signature.
4. **Content Manager (Rust):** resumable HTTP range downloads, checksum verify (SHA-256), signature verify, atomic install, delta updates (bsdiff/zstd `--patch-from`).
5. Build UI: **Library**, **Lesson viewer** (text + audio + optional video), **Quiz runner**, **Progress**.
6. Implement **storage manager** (show MB used, delete packs, move to SD card on Android).

**Deliverable:** A student can install a pack once and study fully in airplane mode.

---

### Phase 3 — Sync Engine (Week 5–8)

1. Use an **outbox pattern**: every local write (progress, quiz attempt, doubt, mentor message) is appended to `outbox` with a UUIDv7 and logical clock.
2. Implement the **sync protocol** (`POST /sync/push`, `GET /sync/pull?since=<cursor>`):
   - Batch and compress payloads (zstd/CBOR — ~5–10× smaller than JSON).
   - Idempotent, resumable, retry with exponential backoff.
   - Conflict resolution: **last-writer-wins** for simple fields, **CRDT (Automerge/Yjs-style)** for notes.
3. **Connectivity detection:** don't rely on "online" flag; probe with a tiny 1 KB endpoint and measure RTT/throughput; classify as `none | poor | ok | good`.
4. **Background sync:**
   - Android: `WorkManager` (via Tauri plugin/Kotlin bridge) — constraints: any network, battery not low.
   - Desktop: Rust `tokio` scheduler with tray-icon status.
5. **Hub sync:** implement mDNS discovery (`_vidyasetu._tcp`) so devices auto-find the Community Hub on LAN and sync/download without internet.
6. **P2P sharing:** Android Wi-Fi Direct / Nearby Connections, desktop LAN HTTP share, QR-code pairing; allow **USB/SD card import** of packs.

**Deliverable:** Progress syncs when connected; packs can be received from a hub or a friend's phone.

---

### Phase 4 — Language & Voice (Week 7–11)

1. **UI localization:** externalize all strings; support RTL/Indic fonts (Noto Sans family, bundled subset fonts for size); test with long strings.
2. **Language selection at first launch** using **icons + audio prompts** (not just text).
3. **Speech-to-Text (STT):**
   - Integrate `whisper.cpp` via Rust FFI (`whisper-rs`) — use tiny/base multilingual models; optionally fine-tune on Indic data.
   - Add VAD (Silero) to trim silence and save compute.
   - Online upgrade path: Bhashini ASR when connectivity is good.
4. **Text-to-Speech (TTS):** Piper voices per language (~20–60 MB each) as optional downloads; fall back to Android/OS TTS.
5. **Voice navigation:** wake-by-button (push-to-talk) → intents like *"open maths lesson 3"*, *"read this aloud"*, *"find scholarships for me"* using a small intent classifier (rule-based + embedding similarity).
6. **Translation layer:** IndicTrans2 (CTranslate2 int8) for on-device translation of lessons/answers; cache results in SQLite.
7. **Code-switching:** handle Hinglish/mixed-language input with language ID (fastText) before routing.

**Deliverable:** A student can ask a question aloud in their language and hear an answer.

---

### Phase 5 — AI Doubt Resolution (Week 9–14)

1. **Local RAG pipeline** (Rust or Python sidecar):
   ```
   Question (voice/text, any language)
        → detect language → translate to pivot (English) if needed
        → embed → top-k retrieval from installed packs (sqlite-vec)
        → build prompt with retrieved passages + safety/system prompt
        → local LLM (llama.cpp) generates grounded answer with citations
        → translate back → show text + speak via TTS
   ```
2. **Prompt design:** a *tutor persona* — step-by-step, simple vocabulary, local examples (farming, markets, local geography), asks a follow-up check question, never invents facts (must cite lesson & section; say "I'm not sure" if no context).
3. **Hybrid mode:** if online and question is out-of-scope of installed packs, route to cloud LLM via backend gateway (with caching, rate-limits, PII stripping).
4. **Deferred doubts:** if the device is too slow or answer confidence is low, save the doubt in the outbox → teacher/mentor/cloud answers later → notification on next sync.
5. **Math/Science support:** integrate a symbolic engine (SymPy via sidecar or `mathjs` in the UI) for step-by-step verified math.
6. **Guardrails:** profanity/self-harm/misinformation filters; academic-integrity mode; bias evaluation across languages.
7. **Evaluation:** build a benchmark of 500+ questions per language; track exact-match/F1, faithfulness (answer supported by citation), human rating by teachers, latency on target devices.

**Deliverable:** Grounded, cited AI answers offline in <15 s on a 4 GB device (lite profile).

---

### Phase 6 — Scholarships, Mentoring, Career Counseling (Week 12–18)

**Scholarship discovery** *(see §14)*
1. Build `scholarship-crawler` (Celery): scrape/ingest from the National Scholarship Portal (scholarships.gov.in), state portals, AICTE/UGC, and CSR/private schemes; normalize to a common JSON schema; human-verify before publishing. Always link back to the **official source**.
2. Encode eligibility as **rules** (category, income, gender, state, course, marks, disability, tribal status, etc.).
3. Ship the DB as a signed, versioned pack that syncs weekly (few KB–MB).
4. Client-side matching (Rust) → ranked list + **"why you match"** explanation + **document checklist** (income certificate, caste certificate, Aadhaar, bank details) + **deadline reminders** (local notifications).

**Digital mentoring** *(see §15)*
1. Mentor onboarding (verification, languages, domains, availability).
2. **Async-first** channel: voice notes (Opus 8–16 kbps) + text, store-and-forward via outbox; sessions when online (WebRTC audio-first, video optional).
3. Matching algorithm: language + subject + region + career interest + mentor load.
4. Safety: report/block, moderated groups, no sharing of phone numbers by default.

**Career counseling**
1. **Assessment:** short, voice-enabled interest (RIASEC) + aptitude questions; no literacy barrier.
2. **Career graph:** JSON knowledge graph (Course → Skills → Jobs → Exams → Salary bands → Institutions) curated with local relevance (government jobs, agri-tech, ITI/polytechnic, forest services, teaching, healthcare, entrepreneurship).
3. **Roadmap generator:** given profile, produce a step timeline (courses, entrance exams, scholarships, skills, internships) — LLM only *explains*; recommendations come from the curated graph to avoid hallucination.
4. **Skill-gap analysis** and micro-course recommendations from installed packs.
5. **Success stories** in local languages (audio/video of alumni from similar backgrounds).

**Deliverable:** Full feature set functional offline, with async sync.

---

### Phase 7 — Low-Bandwidth Hardening, Accessibility & Security (Week 16–20)

1. Apply all techniques in §13; test with **network throttling** (2G/EDGE, 5–10% packet loss) using `tc netem` or Chrome DevTools.
2. Accessibility audit (§19): screen readers (TalkBack, NVDA, VoiceOver), large fonts, high contrast, color-blind safe palettes.
3. Security hardening (§18): encrypted DB (SQLCipher), secure key storage (Android Keystore, macOS Keychain, Windows DPAPI, Linux Secret Service), threat model, dependency scanning (`cargo audit`, `pip-audit`, `npm audit`).
4. Performance budgets: cold start < 3 s on a 3 GB phone; idle RAM < 250 MB (excluding models); APK < 50 MB.

**Deliverable:** Release Candidate.

---

### Phase 8 — Pilot, Feedback, Scale (Week 20+)

1. Deploy **2–3 Community Hubs** (Raspberry Pi 5 + 512 GB SSD + Wi-Fi AP).
2. Train **digital champions** (student volunteers/teachers).
3. Run a **4–8 week pilot** with 100–300 students; measure metrics in §23 (privacy-preserving analytics, opt-in).
4. Iterate on content, AI accuracy, UX pain points; add more languages/subjects.
5. Open-source content packs; onboard universities, NGOs, and state education departments.
6. Plan sustainability: government partnerships, CSR funding, institutional licenses (students always free).

---

## 11. Offline-First & Sync Design

| Concern | Approach |
|---|---|
| Source of truth | **Local SQLite** on device; server is a replica/aggregator |
| Writes | Append to `outbox` (idempotent ops with UUIDv7) |
| Sync transport | HTTPS/2 + zstd + CBOR; resumable; chunked ≤ 64 KB when link is poor |
| Conflicts | LWW for scalars; CRDT for notes/collaborative text |
| Large files | HTTP Range + SHA-256 chunk verification; delta patches |
| Sync triggers | App open, connectivity change, every N hours, manual "Sync now" |
| Offline auth | Long-lived refresh token + device-bound key + offline PIN/biometric |
| Data expiry | Scholarship/deadline data has TTL & "last updated" label |

---

## 12. AI Pipeline

```
 Voice/Text ─► Lang-ID ─► (STT) ─► Normalize ─► Intent Router
                                                │
                    ┌───────────────────────────┼──────────────────────┐
                    ▼                           ▼                      ▼
              Navigation cmd            Doubt / Q&A (RAG)       Scholarship / Career
                                        │                        (rules + graph,
                              Retrieve (sqlite-vec)               LLM only explains)
                                        │
                                Local LLM (GGUF Q4)
                                        │
                              Cite + Safety filter
                                        │
                              Translate ─► TTS ─► Speaker
```

**Model management**
- Models are **optional downloads**, versioned and checksummed; device capability probe (RAM, CPU features, NPU) selects a profile automatically.
- Use **NNAPI/GPU delegates** on Android and **Metal** on macOS where available; AVX2/NEON CPU kernels otherwise.
- Keep an **evaluation harness** in `ml/evals/` and run it in CI whenever prompts/models change.

**Responsible AI**
- Ground answers in vetted content; show sources; display "AI can make mistakes — verify with your teacher".
- Evaluate fairness across languages/dialects; collect feedback (👍/👎) to improve.

---

## 13. Low-Bandwidth Optimization

| Technique | Detail |
|---|---|
| **Adaptive media** | Audio-only default; video 144p/240p (AV1/VP9), auto-degrade; thumbnails as tiny WebP/AVIF |
| **Compression** | Opus for audio, zstd/brotli for text, CBOR instead of JSON |
| **Delta updates** | Only changed lessons/scholarships downloaded |
| **Resumable + scheduled downloads** | Download at night/off-peak or when on Wi-Fi/hub |
| **Progressive loading** | Text first, media on demand |
| **Data-saver mode** | One-tap; blocks images/video/cloud AI calls; shows data usage |
| **Prefetching** | Predict next lessons/quizzes; prefetch on good links |
| **Sneakernet** | USB/SD/QR/Wi-Fi Direct transfers for whole course packs |
| **Fallback channels** | SMS alerts (scholarship deadlines), IVR for voice Q&A, WhatsApp bot (optional) |
| **No heavy web assets** | Bundle fonts/icons locally; no CDN dependencies; no trackers |

---

## 14. Scholarship Discovery Engine

**Schema (excerpt)**
```json
{
  "id": "nsp-postmatric-st-2026",
  "name": "Post Matric Scholarship for ST Students",
  "provider": "Ministry of Tribal Affairs",
  "level": ["UG", "PG"],
  "amount": {"type": "annual", "min": 5000, "max": 20000},
  "eligibility": {
    "category": ["ST"],
    "income_max": 250000,
    "states": ["*"],
    "min_marks_percent": 50,
    "gender": ["*"]
  },
  "documents": ["income_certificate", "caste_certificate", "aadhaar", "bank_passbook", "marksheet"],
  "deadline": "2026-10-31",
  "apply_url": "https://scholarships.gov.in",
  "languages": ["en", "hi", "mr"],
  "last_verified": "2026-09-01"
}
```

**Matching flow:** profile → rule evaluation (`eligibility/` in Rust) → score (fit, amount, deadline urgency) → explanation → checklist → reminders.

> ⚠️ **Always verify details on the official portal.** Data may change; show `last_verified` date. The platform is a discovery aid, not an official application channel.

---

## 15. Mentoring & Career Counseling

**Mentoring modes**

| Mode | Connectivity | Description |
|---|---|---|
| Async voice/text | Low | Student records question → syncs later → mentor replies |
| Scheduled call | Medium | Audio-first WebRTC |
| Group AMA | Medium/Good | Weekly live sessions + recordings distributed via hubs |
| Peer circles | Offline/LAN | Study groups on hub Wi-Fi |

**Career counseling outputs:** interests profile, top career clusters, personalized roadmap, exam calendar, scholarships, "people like me" stories, recommended micro-courses.

---

## 16. Database Schema (Simplified)

```sql
CREATE TABLE users (
  id TEXT PRIMARY KEY, name TEXT, phone_hash TEXT, language TEXT,
  state TEXT, category TEXT, income_band TEXT, course TEXT, created_at INTEGER
);
CREATE TABLE courses (id TEXT PRIMARY KEY, title TEXT, language TEXT, version TEXT, size_bytes INTEGER);
CREATE TABLE lessons (id TEXT PRIMARY KEY, course_id TEXT, title TEXT, body_md TEXT, audio_path TEXT, position INTEGER);
CREATE VIRTUAL TABLE lessons_fts USING fts5(title, body_md, content='lessons');
CREATE TABLE progress (user_id TEXT, lesson_id TEXT, percent REAL, updated_at INTEGER, PRIMARY KEY(user_id, lesson_id));
CREATE TABLE quiz_attempts (id TEXT PRIMARY KEY, user_id TEXT, quiz_id TEXT, score REAL, taken_at INTEGER);
CREATE TABLE doubts (id TEXT PRIMARY KEY, user_id TEXT, question TEXT, answer TEXT, sources TEXT, status TEXT, created_at INTEGER);
CREATE TABLE scholarships (id TEXT PRIMARY KEY, json TEXT, deadline TEXT, last_verified TEXT);
CREATE TABLE mentor_messages (id TEXT PRIMARY KEY, thread_id TEXT, sender TEXT, kind TEXT, body TEXT, audio_path TEXT, created_at INTEGER);
CREATE TABLE career_profiles (user_id TEXT PRIMARY KEY, riasec TEXT, interests TEXT, roadmap_json TEXT);
CREATE TABLE outbox (id TEXT PRIMARY KEY, entity TEXT, op TEXT, payload BLOB, created_at INTEGER, synced INTEGER DEFAULT 0);
```

---

## 17. API Reference (Summary)

| Method | Endpoint | Purpose |
|---|---|---|
| POST | `/auth/otp/request` · `/auth/otp/verify` | Phone-OTP login |
| GET | `/content/catalog?lang=hi&since=…` | List available packs (delta) |
| GET | `/content/packs/{id}` | Download pack (Range supported) |
| POST | `/sync/push` | Push outbox batch |
| GET | `/sync/pull?cursor=…` | Pull server changes |
| POST | `/ai/ask` | Cloud doubt resolution (hybrid mode) |
| POST | `/translate` | Cloud translation fallback |
| GET | `/scholarships?since=…` | Signed scholarship data delta |
| POST | `/mentors/match` | Suggest mentors |
| POST | `/mentors/threads/{id}/messages` | Async messages |
| GET | `/careers/graph?version=…` | Career graph delta |
| GET | `/health` | Liveness probe |

OpenAPI docs: `http://localhost:8000/docs` (when running locally).

---

## 18. Security & Privacy

- **Data minimization:** collect only what's needed; no Aadhaar storage on servers; sensitive fields stay on device unless the student opts in.
- **Encryption:** TLS 1.3 in transit; SQLCipher at rest; OS keystore for keys.
- **Compliance:** align with India's **Digital Personal Data Protection Act, 2023** (consent, purpose limitation, right to erasure); parental/guardian consent flows for minors if applicable.
- **Integrity:** Ed25519-signed content packs & scholarship data; verify before install.
- **AI safety:** prompt-injection defenses for retrieved content; PII scrubbing before cloud calls.
- **Mentor safety:** verification, moderation, reporting, audit logs.
- **Supply chain:** pin dependencies, SBOM, reproducible builds, signed releases.
- **Analytics:** opt-in, aggregated, no third-party trackers.

---

## 19. Accessibility & Inclusion

- WCAG 2.2 AA; TalkBack/VoiceOver/NVDA support; minimum 48 dp touch targets.
- **Low-literacy UX:** icons, colors, audio prompts, voice navigation, pictorial menus.
- Adjustable font size, high-contrast and dark modes, dyslexia-friendly font option.
- Captions/transcripts for all audio & video; sign-language video option for key lessons.
- Gender-inclusive and culturally respectful content; involve tribal community members as co-designers and content reviewers.
- Local examples and idioms; avoid stereotypes.

---

## 20. Testing Strategy

| Type | Tools |
|---|---|
| Unit | `cargo test`, Vitest, Pytest |
| Integration | Testcontainers (Postgres/Redis/MinIO), Tauri WebDriver |
| E2E | Playwright (desktop), Appium/Maestro (Android) |
| Offline scenarios | Airplane mode scripts; kill network mid-download; corrupt pack tests |
| Network | `tc netem` / Toxiproxy: 2G, 300 ms RTT, 10% loss |
| AI quality | `ml/evals/`: WER (STT), BLEU/chrF (translation), Faithfulness & Exact-match (QA) |
| Performance | Low-end device lab (3 GB RAM Android Go, old i3 laptops) |
| Usability | Field tests with real students; task-completion & SUS scores |
| Security | SAST (Semgrep), dependency audit, pen-test before pilot |

---

## 21. CI/CD & Release

```yaml
# .github/workflows/release.yml (outline)
on:
  push:
    tags: ['v*']
jobs:
  build:
    strategy:
      matrix:
        os: [windows-latest, macos-latest, ubuntu-22.04]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: pnpm }
      - uses: dtolnay/rust-toolchain@stable
      - run: pnpm install --frozen-lockfile
        working-directory: app
      - uses: tauri-apps/tauri-action@v0     # builds & uploads bundles to a GitHub Release
        with: { projectPath: app }
  android:
    runs-on: ubuntu-22.04
    steps:
      # setup JDK 17, Android SDK/NDK, Rust android targets
      - run: pnpm tauri android build --aab
        working-directory: app
```

- **Distribution:** GitHub Releases (desktop), Google Play + **direct APK** + F-Droid (Android), plus **offline installer bundles** (USB) for hubs.
- **Auto-update:** Tauri updater with signed manifests; small delta updates.
- **Versioning:** SemVer for the app; independent versions for content packs & data packs.

---

## 22. Roadmap

- [x] Concept, architecture, and documentation
- [ ] **v0.1** — Offline lessons, quizzes, 3 languages, packs, desktop + Android
- [ ] **v0.2** — Sync engine, Community Hub, P2P sharing
- [ ] **v0.3** — Voice STT/TTS, translation
- [ ] **v0.4** — AI doubt resolution (RAG), hybrid cloud mode
- [ ] **v0.5** — Scholarships + career counseling
- [ ] **v0.6** — Mentoring (async + live)
- [ ] **v1.0** — Pilot-ready, accessibility & security audited
- [ ] **v1.x** — More tribal languages, SMS/IVR/WhatsApp, teacher dashboard, NPU acceleration, iOS

---

## 23. Success Metrics

| Category | Metric | Target (pilot) |
|---|---|---|
| Access | % of sessions fully offline | > 60% |
| Engagement | Weekly active learners / lessons completed | ≥ 40% WAL |
| Learning | Pre/post-test score improvement | ≥ 20% |
| Language | Sessions in regional languages | > 70% |
| AI | Doubts answered satisfactorily (👍) | ≥ 80% |
| Scholarships | Students who discovered ≥1 eligible scheme / applied | ≥ 70% / ≥ 30% |
| Mentoring | Mentor response time (async) | < 48 h |
| Career | Students with a saved career roadmap | ≥ 60% |
| Performance | Cold start on 3 GB phone | < 3 s |
| Data | Avg. MB per active week | < 20 MB |

---

## 24. Contributing

1. Fork the repository and create a branch: `git checkout -b feature/my-feature`
2. Follow the code style (pre-commit hooks run automatically).
3. Add tests and update docs.
4. Submit a Pull Request with a clear description and screenshots/logs.

**We especially welcome:** translators, tribal-language speakers, teachers (content review), Indic NLP researchers, accessibility experts, and designers with field experience.

See `CONTRIBUTING.md` and `CODE_OF_CONDUCT.md`.

---

## 25. License

- **Code:** Apache License 2.0
- **Educational content:** CC BY-SA 4.0 (unless otherwise noted)
- **Third-party models/datasets:** respect their individual licenses (Whisper — MIT, Piper — MIT, llama.cpp — MIT, IndicTrans2 — MIT; check each model card before redistribution).

---

<p align="center"><b>Built so that geography is never a barrier to education. 🌾📚📱</b></p>
