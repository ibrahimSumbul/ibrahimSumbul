<!-- HERO BANNER -->
<p align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=gradient&customColorList=12,20,24,30&height=200&section=header&text=Ibrahim%20S%C3%BCmb%C3%BCl&fontSize=60&fontColor=ffffff&fontAlignY=38&desc=AI%20Systems%20Engineer%20%C2%B7%20Hybrid%20AI%20Architectures&descSize=18&descAlignY=58" alt="Banner">
</p>

<p align="center">
  <b>🤖 AI Systems Engineer · Hybrid AI architectures</b><br>
  <b>🇹🇷 KVKK / GDPR-aware enterprise AI · Türkiye → EMEA</b><br>
  <b>🔧 Edge inference (Frigate · Coral · Ollama) ↔ Multi-agent orchestration (Claude)</b>
</p>

<p align="center">
  <b>Currently:</b> Multi-domain orchestrator <i>(stealth R&D)</i> + AI NVR camera analytics<br>
  <b>Looking for:</b> Forward Deployed / Applied AI Engineer · EMEA remote · Türkiye on-site
</p>

<p align="center">
  <a href="mailto:ibrahimsumbulll@gmail.com"><img src="https://img.shields.io/badge/Email-D14836?style=flat&logo=gmail&logoColor=white" /></a>
  <a href="https://www.linkedin.com/in/ibrahim-s%C3%BCmb%C3%BCl-838800300"><img src="https://img.shields.io/badge/LinkedIn-0A66C2?style=flat&logo=linkedin&logoColor=white" /></a>
  <a href="https://github.com/ibrahimSumbul"><img src="https://img.shields.io/badge/GitHub-181717?style=flat&logo=github&logoColor=white" /></a>
</p>

<p align="center">
  <a href="#-about">About</a> ·
  <a href="#-tech-stack">Tech Stack</a> ·
  <a href="#-featured-projects">Projects</a> ·
  <a href="#-past-projects">Past Work</a> ·
  <a href="#-certifications">Certifications</a> ·
  <a href="#%EF%B8%8F-writing">Writing</a> ·
  <a href="#-stats">Stats</a> ·
  <a href="#-get-in-touch">Connect</a>
</p>

---

### 🧭 About

Computer Engineer focused on **AI-powered systems** — from edge inference on cameras to multi-agent orchestration over enterprise data. I design and build complete ecosystems: backend APIs, mobile apps, AI integrations, and the architecture that ties them together.

My background blends **embedded systems & robotics** with modern web/mobile and ML. That combination shapes how I think: hardware constraints first, software abstractions second, never the other way around.

Currently exploring how **hybrid AI** — local detection + cloud LLM semantics, or fully on-premise vision models — outperforms pure-cloud approaches on cost, latency, reliability, and **data privacy** for real-world deployments.

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
  <b>Focus areas:</b> Hybrid AI architecture · Multi-agent orchestration · Vision LLMs (Claude, Gemini, GPT-4o, Qwen2.5-VL local) · Prompt engineering & caching · Local inference (Frigate, YOLO, Coral TPU, Ollama) · Event-driven systems · KVKK / GDPR-aware design
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

> *Implementation repository private during development. Engineering deep-dive available on request.*
>
> 📖 **Architecture vision (public, CC BY-SA 4.0):** [`enterprise-ai-orchestra`](https://github.com/ibrahimSumbul/enterprise-ai-orchestra) · [live interactive demo](https://ibrahimsumbul.github.io/enterprise-ai-orchestra/)

---

#### 🎥 [AI NVR — Hybrid Camera Analytics](https://github.com/ibrahimSumbul/ai_nvr)
[![License](https://img.shields.io/badge/license-MIT-blue)](https://github.com/ibrahimSumbul/ai_nvr/blob/main/LICENSE)
[![Phase](https://img.shields.io/badge/Phase-M4%20done%20·%20M5%20in%20flight-green)](https://github.com/ibrahimSumbul/ai_nvr/blob/main/ROADMAP.md)
[![Stack](https://img.shields.io/badge/Python%20·%20Frigate%20·%20Postgres%20·%20Ollama%20·%20Grafana-534AB7)](https://github.com/ibrahimSumbul/ai_nvr)
[![Tests](https://img.shields.io/badge/60%20unit%20·%20ruff%20·%20mypy%20strict-success)](https://github.com/ibrahimSumbul/ai_nvr/tree/main/bridge/tests)

Production-grade reference architecture for adding AI on top of an **existing 100-camera Dahua NVR** without disturbing the original recording system. Combines **local detection** (Frigate + optional Coral USB) with **fully local vision LLM** (Ollama + Qwen2.5-VL) for color recognition — **images never leave the facility, $0 monthly LLM cost**.

**Current status (M4 done · M5 in flight):**
- ✅ Core pipeline running — 5 services healthy, end-to-end verified
- ✅ Zone state machine, first-entry alarm, snapshot capture
- ✅ Local Ollama vision (Qwen2.5-VL) for truck cab + trailer color analysis
- ✅ Dahua NVR external alarm bridge with retry queue (DSS/SmartPSS panel + mobile push)
- 🚧 M5: 5 cameras live + Grafana dashboard; production target 10 cameras
- ⬜ M6: Coral USB upgrade (hardware sourcing) · M6.5: door events + DMSS mobile push
- 60 unit tests · ruff · mypy strict · Alembic migrations · GitHub Actions CI
- 11 architecture documents — tech decisions to bottleneck analysis

> **Why interesting:** real cost math (pure cloud LLM ≈ $2.5M/mo vs hybrid Anthropic ≈ $25/mo vs **fully-local Ollama ≈ $0/mo**), explicit trade-off tables, NVR-load-aware design, privacy-first architecture — no images cross network boundary.

---

### 📂 Past Projects

#### 🏋️ AI Fitness Coaching App *(side project — shipped)*
Mobile fitness app pairing a **trainer** with their athletes; LLM acts as explainer-in-the-middle. Instagram OAuth, training plan analysis, weekly check-ins, jargon translation for non-expert users. **React Native · Instagram OAuth · LLM.** Design problem: AI that **augments** human expertise instead of replacing it.

#### 🚗 [AI-Powered Vehicle System Automation](https://github.com/ibrahimSumbul/ai-vehicle-automation)
AI-driven automotive assistant automating diagnostic and reporting workflows — GPT-based reasoning over IoT data, vehicle fault detection, automatic logging.

#### ⚖️ [seesaw_tork](https://github.com/ibrahimSumbul/seesaw_tork)
Tork-based seesaw physics simulation — JavaScript exploration of rigid-body dynamics. Earlier hobby project.

---

### 🎓 Certifications

> *Building toward CKA → AZ-104 → CKS → AZ-500 through 2026-2027. Credly badges will appear here as earned.*

- **CKA** — Certified Kubernetes Administrator · Linux Foundation · *target M6-7*
- **AZ-104** — Microsoft Azure Administrator · *target M9*
- **CKS** — Certified Kubernetes Security Specialist · *target M16*
- **AZ-500** — Microsoft Azure Security Engineer · *target M18*

---

### ✍️ Writing

> *Technical writing pipeline. Posts will auto-populate here as published.*

<!-- BLOG-POST-LIST:START -->
<!-- BLOG-POST-LIST:END -->

**Planned topics:**
- *Local-First Vision LLMs: Real Cost Math from a 100-Camera Production Deployment*
- *Hybrid AI Decision Matrix: When Edge, When Cloud, When Both*
- *KVKK-Compliant Multi-Agent Architecture: A Practical Reference*
- *Boundary Agent Doctrine: A Framework for AI Tool Calling at Production Scale*

---

### 📊 Stats

<p align="center">
  <img height="180" src="https://github-readme-stats.vercel.app/api?username=ibrahimSumbul&show_icons=true&theme=tokyonight&include_all_commits=true&count_private=true&hide_border=true" />
  <img height="180" src="https://github-readme-stats.vercel.app/api/top-langs/?username=ibrahimSumbul&layout=compact&theme=tokyonight&langs_count=8&hide_border=true" />
</p>

<p align="center">
  <img src="https://streak-stats.demolab.com?user=ibrahimSumbul&theme=tokyonight&hide_border=true" />
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/ibrahimSumbul/ibrahimSumbul/output/github-contribution-grid-snake-dark.svg" alt="snake animation" />
</p>

---

### 🌱 Currently Working On

- Lead engineer on the **AI Orchestration Platform** above — moving from architecture deck to multi-agent runtime: prompt versioning, eval pipelines (Promptfoo + Vitest golden queries), Anthropic prompt caching strategy, KVKK PII pipeline, per-agent cost attribution.
- **AI NVR M5 in flight** — production deployment to 10 cameras, Grafana dashboard polish, Coral USB hardware sourcing for M6.
- Exploring **capability-based model routing** (Sonnet for reasoning, Haiku for retrieval/data, local Qwen2.5-VL for vision) to keep multi-agent systems under tight monthly LLM budgets.

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
