<h1 align="center">👋 Hi, I'm Ibrahim Sumbul</h1>

<p align="center">
  <b>Full-Stack Engineer · AI Architecture · Systems Thinking</b><br>
  Building hybrid AI systems where <b>local inference meets cloud LLMs</b> — practical, cost-aware, and production-grade.
</p>

<p align="center">
  <a href="mailto:ibrahimsumbulll@gmail.com"><img src="https://img.shields.io/badge/Email-D14836?style=flat&logo=gmail&logoColor=white" /></a>
  <a href="https://www.linkedin.com/in/ibrahim-s%C3%BCmb%C3%BCl-838800300"><img src="https://img.shields.io/badge/LinkedIn-0A66C2?style=flat&logo=linkedin&logoColor=white" /></a>
  <a href="https://github.com/ibrahimSumbul"><img src="https://img.shields.io/badge/GitHub-181717?style=flat&logo=github&logoColor=white" /></a>
</p>

---

### 🧭 About

Computer Engineer focused on **AI-powered systems** — from edge inference on cameras to multi-agent orchestration over enterprise data. I design and build complete ecosystems: backend APIs, mobile apps, AI integrations, and the architecture that ties them together.

My background blends **embedded systems & robotics** with modern web/mobile and ML. That combination shapes how I think: hardware constraints first, software abstractions second, never the other way around.

Currently exploring how **hybrid AI** (local detection + cloud LLM semantics) outperforms pure-cloud approaches on cost, latency, and reliability for real-world deployments.

---

### 🧠 Tech Stack

#### Languages & Frameworks
<p align="left">
  <img src="https://skillicons.dev/icons?i=python,typescript,javascript,cpp,cs,nodejs,nestjs,fastapi,react,nextjs,tailwind" />
</p>

#### Data, Cloud & Infra
<p align="left">
  <img src="https://skillicons.dev/icons?i=postgresql,mysql,supabase,prisma,docker,kubernetes,aws,azure,vercel,linux,githubactions,git" />
</p>

#### AI / ML
<p align="left">
  <img src="https://skillicons.dev/icons?i=python,pytorch,tensorflow" />
  <br>
  <b>Focus areas:</b> Hybrid AI architecture · Multi-agent orchestration · Vision LLMs (Claude, Gemini, GPT-4o) · Prompt engineering & caching · Local inference (Frigate, YOLO, Coral TPU) · Event-driven systems · KVKK / GDPR-aware design
</p>

---

### 💡 Featured Projects

#### 🎥 [AI NVR — Hybrid Camera Analytics](https://github.com/ibrahimSumbul/ai_nvr)
[![License](https://img.shields.io/badge/license-MIT-blue)](https://github.com/ibrahimSumbul/ai_nvr/blob/main/LICENSE)
[![Stack](https://img.shields.io/badge/Python%20·%20Frigate%20·%20Postgres%20·%20Claude-534AB7)](https://github.com/ibrahimSumbul/ai_nvr)

A production-grade reference architecture for adding AI on top of an **existing 100-camera Dahua NVR** without disturbing the original recording system. Combines **local detection** (Frigate + Coral USB) with **cloud semantic analysis** (Claude Haiku) for color recognition and anomaly verification.

- First-entry alarm with zone state machine — no spam when zone is already occupied
- Door traversal logging with **second-precision** entry/exit timestamps + email link to event clip
- Truck cab + trailer color recording (no plate reading, KVKK-friendly)
- Original Dahua DSS/SmartPSS panel sees events as "External Alarm"
- **Fixed budgets**: $10/mo (PoC) → $25/mo (production, ~25 AI cameras)
- 11 architecture documents — from tech decision records to max-capacity bottleneck analysis

> **Why interesting:** real cost math (pure cloud LLM ≈ $2.5M/mo vs hybrid ≈ $25/mo), explicit trade-off tables, NVR-load-aware design.

---

#### 🧩 Domain-Based AI Orchestration Architecture *(architecture case study — private)*
![Status](https://img.shields.io/badge/Status-Architecture%20Design-orange)
![Visibility](https://img.shields.io/badge/Repository-Private-lightgrey)

An independent architectural case study exploring how a mid-size enterprise can **unify access to multiple internal systems** (ERP, fleet management platform, QMS, asset registry, document stores) behind a **single AI orchestration layer**.

Four-layer mimari:
- **Channels** — chat (WhatsApp / Teams / web), structured dashboards, proactive notifications
- **Orchestration** — authz/policy, audit logging, conversation memory, domain intent routing
- **Domains** — eight specialized agent clusters (HR/SOP, procurement, demand & stock, logistics, quality, sales, decision support, finance)
- **Federated data** — sources stay in place, queried via APIs on demand; no central warehouse

Design priorities: **KVKK / GDPR-grade audit trail**, prompt-injection defense, intent routing with tool whitelisting, multi-channel context continuity.

> Frames AI not as "chatbot" but as **data strategy** — the orchestration layer becomes the forcing function that disciplines underlying data sources.

> *Repository private under development. Architecture deck available on request.*

---

#### 🚗 [AI-Powered Vehicle System Automation](https://github.com/ibrahimSumbul/ai-vehicle-automation)
AI-driven automotive assistant automating diagnostic and reporting workflows — GPT-based reasoning over IoT data, vehicle fault detection, automatic logging.

#### 🟢 [FlortApp](https://github.com/ibrahimSumbul/flortapp)
Real-time social networking & messaging platform — Node.js + PostgreSQL + React Native. End-to-end (backend API + admin dashboard + mobile), real-time chat, JWT auth, RLS policies.

#### 🟢 [Billing API Demo](https://github.com/ibrahimSumbul/billing-api-demo)
SaaS-grade subscription & payments API — Node.js + PostgreSQL + Docker, clean REST architecture, query optimization.

---

### 🌱 Currently Working On

- Building **AI NVR** scaffold (Milestone 1: Docker stack, bridge service, Postgres schema) — code phase begins after documentation freeze.
- Architectural deep-dive on the **AI Orchestration** case study — moving from concept deck to detailed component design (security, audit schemas, intent routing).
- Exploring **prompt caching + small-model routing** to keep multi-agent systems under tight monthly LLM budgets.

---

### 📬 Get in Touch

📧 **Email** — [ibrahimsumbulll@gmail.com](mailto:ibrahimsumbulll@gmail.com)
💼 **LinkedIn** — [linkedin.com/in/ibrahim-sümbül](https://www.linkedin.com/in/ibrahim-s%C3%BCmb%C3%BCl-838800300)
💻 **GitHub** — [github.com/ibrahimSumbul](https://github.com/ibrahimSumbul)

---

<p align="center">
  <i>"Hardware constraints first, software abstractions second."</i><br>
  <sub>From embedded systems to multi-agent AI — building across stacks, languages, and boundaries.</sub>
</p>
