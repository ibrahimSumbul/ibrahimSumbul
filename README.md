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

#### 🧩 Domain-Based AI Orchestration Platform *(in active development — private)*
![Status](https://img.shields.io/badge/Status-Active%20Development-blue)
![Stack](https://img.shields.io/badge/Next.js%20·%20TypeScript%20·%20Anthropic%20·%20Vercel%20AI-000000)
![Repo](https://img.shields.io/badge/Repository-Private-lightgrey)

Multi-agent orchestration layer that gives users a single natural-language surface (chat, dashboards, proactive notifications) over many internal systems (ERP, fleet platform, QMS, asset registry, document stores). Designed for **mid-size enterprises** where data exists but isn't usable.

**Architecture highlights:**
- Four-layer separation: Channels · Orchestration · Domain agents · Federated data sources (no central warehouse, query-time API access)
- **Capability-based model routing** — domain reasoning on Claude Sonnet, horizontal/data tasks on Haiku, with fallback chains
- Multi-channel context continuity (a chat thread started in Teams continues in the web dashboard)
- KVKK / GDPR-grade audit trail, prompt-injection defense, tool whitelisting per domain

**Engineering rigor — every agent ships with a 12-axis checklist:**
prompt engineering & versioning · Anthropic prompt caching · structured outputs (Zod) · typed error taxonomy · streaming with progressive tool-call UI · OpenTelemetry observability · per-user / per-agent / per-tool rate limits (Upstash Redis) · PII detect+mask pipeline (TC kimlik, phone — deterministic + Haiku) · golden-query evals (Vitest + Promptfoo) · semver agent versioning · per-call cost log + Vercel AI Gateway budget caps · LLM hallucination guard via citation cross-check.

**Stack:** Next.js · TypeScript · Anthropic SDK · Vercel AI SDK + AI Gateway · Supabase (Auth + Postgres) · Upstash Redis · Sentry · Promptfoo · Vitest

> Framed not as "another chatbot" but as **a data strategy project** — orchestration becomes the forcing function that disciplines underlying data sources and turns latent data into used data.

> *Repository private during development. Architecture deck and engineering deep-dive available on request.*

---

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

#### 🏋️ AI Fitness Coaching App *(side project)*
![Status](https://img.shields.io/badge/Status-Shipped-success)
![Stack](https://img.shields.io/badge/React%20Native%20·%20Instagram%20OAuth%20·%20LLM-FF6B6B)

Mobile fitness app that pairs a **trainer (hoca)** with their athletes and lets an LLM act as the explainer-in-the-middle. Users sign in with **Instagram OAuth**, and an AI layer analyzes their **diet plan** and **training program** — clarifying *why* the coach prescribed what they did, surfacing inconsistencies, and translating jargon for non-expert users.

- **Two-sided assistance** — helps the coach build/explain programs, helps the athlete actually understand them
- AI-generated explanations and progress summaries from training logs
- Personalized weekly check-ins triggered by usage patterns
- LLM-translated nutrition and movement vocabulary

> Built end-to-end as a side project — interesting design problem: an AI that **augments a human expert** instead of replacing them.

---

#### 🚗 [AI-Powered Vehicle System Automation](https://github.com/ibrahimSumbul/ai-vehicle-automation)
AI-driven automotive assistant automating diagnostic and reporting workflows — GPT-based reasoning over IoT data, vehicle fault detection, automatic logging.

---

### 🌱 Currently Working On

- Lead engineer on the **AI Orchestration Platform** above — moving from architecture deck to multi-agent runtime: prompt versioning, eval pipelines (Promptfoo + Vitest golden queries), Anthropic prompt caching strategy, KVKK PII pipeline, per-agent cost attribution.
- Building **AI NVR** scaffold (Milestone 1: Docker stack, bridge service, Postgres schema) — code phase begins after documentation freeze.
- Exploring **capability-based model routing** (Sonnet for reasoning, Haiku for retrieval/data tasks) to keep multi-agent systems under tight monthly LLM budgets.

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
