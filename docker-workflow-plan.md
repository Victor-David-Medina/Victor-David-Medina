# Docker Integration Plan
## Cloud Ops Workflow & Project Enhancement

**Author:** Victor David Medina
**Date:** March 2026
**Purpose:** Personal planning reference — Docker as the next automation layer in cloud ops workflow

---

## Why Docker Now

The Docker AI article reinforced what was already apparent from a workflow standpoint: containers are the missing layer between "local tooling" and "cloud-ready delivery." The specific patterns that apply directly to this work:

| Article Concept | Cloud Ops Application |
|---|---|
| Docker Compose as a service definition layer | Define Terraform + tfsec + checkov runner as a Compose stack |
| Docker Offload (local → cloud compute) | ECS Fargate Spot for batch/ephemeral infra jobs |
| Model Context Protocol (MCP) servers | Claude Code integration with AWS environment via Docker-run MCP servers |
| Portable, reproducible environments | Eliminate "works on my machine" from client engagements |

---

## Goals

1. **Workflow automation** — containerize the tools I already use so every engagement starts from a consistent, version-locked environment
2. **Portfolio expansion** — build a second portfolio project (Docker + ECS Fargate + Terraform) that extends the existing VPC + Auto Scaling work
3. **Certification path** — add Docker Certified Associate (DCA) as the third certification after AWS CCP and Terraform Associate
4. **Claude co-work enhancement** — use Docker-run MCP servers to give Claude Code persistent, structured access to project context

---

## Phase 1 — Containerized Terraform Runner

**Goal:** A Docker image with all IaC tooling baked in. Run locally or in GitHub Actions. Zero setup per engagement.

### What goes in the image:
- Terraform (pinned version)
- tfsec
- checkov
- AWS CLI v2
- jq + yq for config parsing

### How it fits into GitHub Actions:
```
jobs:
  terraform:
    runs-on: ubuntu-latest
    container:
      image: ghcr.io/victor-david-medina/tf-runner:latest
    steps:
      - terraform fmt
      - terraform validate
      - tfsec .
      - terraform plan
```

### Outcome:
- CI always uses the exact same tool versions
- New client repos get a working pipeline in minutes (copy Dockerfile + workflow)
- Dockerfile itself becomes a deliverable — shows clients exactly what's running

---

## Phase 2 — Docker + ECS Fargate Portfolio Project

**Goal:** Second portfolio project that adds the container layer on top of the existing Terraform VPC stack.

### Component Breakdown:

| Component | Details |
|---|---|
| Containerization | Dockerfile with multi-stage build for a lightweight web service |
| Image Registry | Amazon ECR — push via GitHub Actions on merge to main |
| Orchestration | AWS ECS Fargate cluster provisioned entirely with Terraform |
| CI/CD | Build → Tag → Push → ECS rolling deploy (GitHub Actions) |
| Networking | Existing VPC + new Application Load Balancer + ECS service |
| IAM | Task execution role, ECR pull permissions, least-privilege task role |
| Observability | CloudWatch Container Insights + FireLens log routing |
| Cost Controls | Fargate Spot capacity provider for non-prod; Budget alert |

### Portfolio narrative:
> Project 1 (existing): Provision the cloud infrastructure
> Project 2 (Docker + ECS): Deploy something INTO that infrastructure — containers, end to end

This makes the portfolio tell a complete story: **IaC → Containers → Orchestration → CI/CD → Observability**

---

## Phase 3 — Claude Co-Work Enhancement via Docker MCP

**Goal:** Use Docker to run Model Context Protocol (MCP) servers locally, giving Claude Code structured access to project files, AWS context, and documentation.

### What this enables:
- Claude can query live project state (Terraform state, cost data, runbooks) without copy-pasting
- MCP servers run as Docker containers — start/stop without polluting the local environment
- Compose file defines the full "Claude workspace": Claude Code + MCP servers + project context

### Relevant MCP servers (from Docker's ecosystem):
- **Filesystem MCP** — structured file access to project directories
- **AWS MCP** (community) — read-only access to AWS resource state
- **PostgreSQL MCP** — if project tracking moves to a local DB

### Docker Compose pattern:
```yaml
services:
  filesystem-mcp:
    image: mcp/filesystem
    volumes:
      - ./:/project:ro

  aws-mcp:
    image: mcp/aws-readonly
    environment:
      - AWS_PROFILE=default
```

---

## Learning Path & Timeline

| Milestone | Target | Notes |
|---|---|---|
| AWS Cloud Practitioner (CLF-C02) | Current priority | Foundation cert — do this first |
| Docker fundamentals | Alongside CCP prep | Docker's own "Get Started" + Dockerfile practice |
| Phase 1: Terraform runner image | After CCP | 1–2 day build; immediate workflow value |
| Phase 2: ECS Fargate project | After Terraform Associate | Larger project — 2–4 weeks |
| HashiCorp Terraform Associate (004) | Mid-2026 | Second cert |
| Docker Certified Associate (DCA) | Late 2026 | Third cert — after building real Docker experience |

---

## Skills Added by Docker (for interview positioning)

Once Phase 1 and 2 are complete, the honest skill set becomes:

**Current:**
AWS · Terraform · GitHub Actions · Linux · Bash

**Added:**
Docker · Amazon ECR · AWS ECS Fargate · Multi-stage builds · Container-native CI/CD

**Full stack story:**
> "I provision the cloud foundation with Terraform, package applications as containers, push to ECR via GitHub Actions, and deploy to ECS Fargate — then monitor the whole stack with CloudWatch."

---

## Notes / Reference

- Docker Model Runner (DMR) — local LLM hosting via Docker; relevant for future AI tooling experiments
- Docker Offload — transparent cloud GPU offload; interesting for compute-heavy automation jobs
- `docker compose` as IaC analog — define services declaratively, version-controlled, reproducible
- DCA exam guide: [docs.docker.com/certification](https://docs.docker.com)

---

*This document is a working reference — update it as phases complete and plans evolve.*
