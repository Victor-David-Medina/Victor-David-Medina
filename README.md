<!--
  PERSONAL GITHUB PROFILE README: github.com/Victor-David-Medina/Victor-David-Medina
  Reliable by design: NO capsule-render (flaky). shields.io + skillicons + readme-typing-svg + readme-stats only.
  Palette: navy / tokyonight (#58A6FF accents) for cohesion.
  Honesty (traces to LOCKED): pre-revenue; SOLO (no "we"); RelayLaunch 2026; use_lockfile (NOT DynamoDB);
  15 cloud models / 7 providers (+ optional local Ollama); 40+ migrations; ~27-container relay-infra;
  auto-graduation SPECCED not built; ONE early unpaid pilot; Orchestra QA = contract; ezCater = Staff Accountant.
  Lane order: AI/Applied-AI + Forward-Deployed = PRIMARY; QA = bridge; Cloud = last/growing.
-->

<h1 align="center">Victor David Medina</h1>

<p align="center">
  <img src="https://raw.githubusercontent.com/Victor-David-Medina/Victor-David-Medina/main/ai-harness-banner.svg" alt="Victor David Medina, applied AI engineer building multi-agent AI operations systems" width="100%" />
</p>

<p align="center">
  <strong>AI / Applied-AI Engineer</strong> &nbsp;·&nbsp; <strong>Forward-Deployed / Solutions Engineer</strong>
</p>

<p align="center">
  <em>I build AI systems that propose actions and wait for owner approval before they execute,<br/>not chatbots that talk about it. And I orchestrate four AI coding agents to ship them.</em>
</p>

<p align="center">
  <img src="https://readme-typing-svg.demolab.com?font=Fira+Code&size=17&duration=3200&pause=900&color=58A6FF&center=true&vCenter=true&width=860&lines=4+AI+coding+agents+%C2%B7+1+orchestrated+workflow+%C2%B7+trust-but-verify;LLM+gateways+%C2%B7+RAG+%C2%B7+eval+harness+in+CI+%C2%B7+owner-approval+gates;Cloudflare+Workers+%C2%B7+Supabase+RLS+%C2%B7+Python+%C2%B7+TypeScript;USMC+veteran+%E2%86%92+operations+%E2%86%92+QA+%E2%86%92+building+production+AI" alt="What I do" />
</p>

<p align="center">
  <a href="https://www.linkedin.com/in/victor-david-medina"><img src="https://img.shields.io/badge/%F0%9F%9F%A2_Open_to_Work-Contract_or_Full--time-2EA043?style=for-the-badge" alt="Open to Work" /></a>
  <a href="https://www.linkedin.com/in/victor-david-medina"><img src="https://img.shields.io/badge/LinkedIn-victor--david--medina-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white" alt="LinkedIn" /></a>
  <a href="mailto:v.davidmedina@gmail.com"><img src="https://img.shields.io/badge/Email-v.davidmedina-EA4335?style=for-the-badge&logo=gmail&logoColor=white" alt="Email" /></a>
  <a href="https://github.com/Victor-David-Medina/aws-terraform-portfolio"><img src="https://img.shields.io/badge/Public_Proof-aws--terraform--portfolio-181717?style=for-the-badge&logo=github&logoColor=white" alt="Public proof repo" /></a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Watertown,_MA-30363D?style=flat-square&logo=googlemaps&logoColor=white" alt="Location" />
  <img src="https://img.shields.io/badge/Remote_(US)_%2B_Boston-30363D?style=flat-square" alt="Remote US + Boston" />
  <img src="https://img.shields.io/badge/USMC_Veteran-2013-2017-30363D?style=flat-square" alt="USMC veteran" />
  <img src="https://img.shields.io/badge/Available-Now-2EA043?style=flat-square" alt="Available now" />
</p>

---

> 🟢 **Open to** AI / Applied-AI Engineer · Forward-Deployed / Solutions Engineer, *primary.* QA Automation, *bridge.* Cloud Support, *growing, hands-on.*
> **Remote (US) or Boston · contract or full-time · available now.**

## 🤖 The differentiator: four AI agents, one orchestrated workflow

I don't just *use* AI tools. I run a **multi-AI development harness** where **Claude Code + GitHub Copilot + Gemini CLI + Codex** work the same sprint in parallel, hand off through shared docs, and pass a **trust-but-verify** gate before anything merges. My day-to-day dev process *is* a working demo of orchestrating AI in production.

```text
                ┌──────────────────────────────────────────────────────┐
   one sprint → │  Claude Code   GitHub Copilot   Gemini CLI    Codex    │  ← 4 agents, parallel
                └───────┬────────────┬───────────────┬───────────┬──────┘
                        └────────────┴──── shared handoff docs ───┴──────┐
                                                                         ▼
                                          trust-but-verify gate  →  merge / ship
```

| The system I build | The infra I run it on |
|---|---|
| **Owner-approval autonomy**: 5-level progressive-autonomy model; the AI proposes, the owner approves *(auto-graduation specced next, not yet built)* | **`relay-infra`**: a **~27-container Docker Compose spine**: Traefik · Prometheus · Grafana · Loki · LiteLLM · Langfuse · Qdrant · n8n · Ollama (GPU) |
| **Model gateway**: **LiteLLM** routing **15 cloud models across 7 providers** behind one API *(+ optional local Ollama)* | **`CouncilVerse`**: open-source multi-agent council engine, published to public npm ([@relaylaunch](https://www.npmjs.com/org/relaylaunch)) |

## 🧠 What I build

- **AI / LLM** *(primary)*: multi-agent orchestration · LiteLLM gateway (15 cloud models / 7 providers) · RAG on **Qdrant + pgvector** · **LLM evaluation harness** (golden datasets, regression detection, grounding / faithfulness) wired into **CI** · LLM observability (**Langfuse**) · progressive-autonomy **owner-approval gates**
- **Forward-Deployed / Solutions** *(primary)*: multi-tenant execution engine on **Cloudflare Workers** (per-tenant **D1 / KV / R2 / Queues**, 40+ migrations) · **Supabase Postgres + Row-Level Security** · Hono · stayed in the loop through **one early unpaid field pilot** with a real service business
- **Reliability**: AI-specific **incident-response playbook** + severity-based **escalation matrix** · **IBM 6-pillar + EU AI Act** governance mapping
- **Cloud / DevOps** *(growing, hands-on)*: AWS (VPC, IAM, GuardDuty, S3) · Terraform (`use_lockfile` remote state, **tfsec in CI**) · Docker · GitHub Actions

## 🪖 Background

**U.S. Marine Corps, Sergeant (E-5), 2013-2017**, then a decade across enterprise operations, accounting systems, and software QA, now building production AI full-time.

Reliability-first by training: I design failure modes, escalation paths, and incident response **in from the start**. Track record of owning systems end to end: a **1,000+ user Expensify rollout** (NetSuite / Namely integration, UAT, 100% adoption in 90 days) at ezCater, **50+ defects** caught and ~30% faster fix turnaround as a QA engineer at Orchestra.so, and **$11K+** in vendor overcharges recovered via root-cause analysis at Blue Matter Consulting.

## 📂 Public work

- **[`aws-terraform-portfolio`](https://github.com/Victor-David-Medina/aws-terraform-portfolio)**: multi-AZ VPC · GuardDuty · **tfsec-in-CI** · S3 remote state with **`use_lockfile`** (S3-native locking, TF 1.10+) · **5 ADRs** + an operational runbook
- **[CouncilVerse](https://www.npmjs.com/org/relaylaunch)**: open-source multi-agent council engine on public npm
- **RelayLaunch** *(solo, pre-revenue)*: the multi-tenant AI operations platform the harness above builds and runs

## 🛠️ Stack

<p align="center">
  <img src="https://skillicons.dev/icons?i=ts,python,nodejs,react,nextjs,astro,cloudflare,supabase,postgres,docker,aws,terraform,githubactions,linux,bash,git&theme=dark&perline=8" alt="Tech stack" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/LiteLLM-gateway-58A6FF?style=flat-square" />
  <img src="https://img.shields.io/badge/Qdrant_+_pgvector-RAG-58A6FF?style=flat-square" />
  <img src="https://img.shields.io/badge/Langfuse-LLM_observability-58A6FF?style=flat-square" />
  <img src="https://img.shields.io/badge/Hono-edge_API-58A6FF?style=flat-square" />
  <img src="https://img.shields.io/badge/Ollama-local_GPU-58A6FF?style=flat-square" />
  <img src="https://img.shields.io/badge/tfsec-IaC_security-58A6FF?style=flat-square" />
</p>

## 📜 Certifications

AWS Certified Cloud Practitioner (CLF-C02), *in progress* &nbsp;·&nbsp; HashiCorp Terraform Associate, *in progress* &nbsp;·&nbsp; QA Engineering Certificate, *2024*

---

<p align="center">
  <img src="https://github-readme-stats.vercel.app/api?username=Victor-David-Medina&show_icons=true&theme=tokyonight&hide_border=true&include_all_commits=true&count_private=true" alt="GitHub stats" height="165" />
  <img src="https://github-readme-stats.vercel.app/api/top-langs/?username=Victor-David-Medina&layout=compact&theme=tokyonight&hide_border=true&langs_count=8" alt="Top languages" height="165" />
</p>

<p align="center">
  <strong>Hiring for AI / Applied-AI, Forward-Deployed / Solutions, or QA Automation?</strong><br/>
  I build production AI from the model gateway to the owner-approval gate, and stay in the loop until it works for real users.<br/><br/>
  <a href="mailto:v.davidmedina@gmail.com"><strong>v.davidmedina@gmail.com</strong></a> &nbsp;·&nbsp; <a href="https://www.linkedin.com/in/victor-david-medina"><strong>LinkedIn</strong></a> &nbsp;·&nbsp; <a href="https://github.com/Victor-David-Medina/aws-terraform-portfolio"><strong>Proof repo</strong></a> &nbsp;·&nbsp; Remote (US) + Boston
</p>

---

<h3 align="center">Contribution activity</h3>
<p align="center"><sub>auto-generated twice daily by a GitHub Action (Platane/snk)</sub></p>
<p align="center">
  <img src="https://raw.githubusercontent.com/Victor-David-Medina/Victor-David-Medina/output/github-snake-dark.svg" alt="Animated contribution snake" />
</p>
