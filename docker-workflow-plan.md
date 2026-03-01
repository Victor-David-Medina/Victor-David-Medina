# Docker Integration Plan — Detailed Build & Analysis Reference

**Author:** Victor David Medina
**Date:** March 2026
**Version:** 2.0
**Purpose:** Comprehensive planning, build, and analysis reference for Docker integration into cloud ops workflow and portfolio. Intended for use across Claude co-work sessions, build sprints, and printed reference.

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Current State Assessment](#2-current-state-assessment)
3. [Strategic Rationale — Why Docker](#3-strategic-rationale--why-docker)
4. [Architecture Overview](#4-architecture-overview)
5. [Phase 1 — Containerized Terraform Runner](#5-phase-1--containerized-terraform-runner)
6. [Phase 2 — Docker + ECS Fargate Portfolio Project](#6-phase-2--docker--ecs-fargate-portfolio-project)
7. [Phase 3 — Claude Co-Work Enhancement via Docker MCP](#7-phase-3--claude-co-work-enhancement-via-docker-mcp)
8. [Learning Path & Certification Roadmap](#8-learning-path--certification-roadmap)
9. [Cost Analysis](#9-cost-analysis)
10. [Decision Log](#10-decision-log)
11. [Interview Preparation](#11-interview-preparation)
12. [Troubleshooting Reference](#12-troubleshooting-reference)
13. [Glossary](#13-glossary)

---

## 1. Executive Summary

This document defines a three-phase plan to integrate Docker into existing cloud operations workflow and portfolio. The driver is a Docker AI article highlighting Docker Compose, Docker Offload, Model Context Protocol (MCP) servers, and container-native runtimes as foundational patterns for modern cloud delivery.

**The three phases:**

| Phase | Name | Value | Timeline |
|---|---|---|---|
| 1 | Containerized Terraform Runner | Eliminate tool-version drift across all projects and CI environments | 1–2 days after CLF-C02 |
| 2 | Docker + ECS Fargate Portfolio Project | Second portfolio project; completes the IaC → container → orchestration story | 2–4 weeks after Terraform Associate |
| 3 | Claude Co-Work via Docker MCP | Persistent, structured AI-assisted workflow; Claude reads live project context | Ongoing; start simple |

**Certifications unlocked by this plan:**
- AWS Cloud Practitioner (CLF-C02) → *in progress*
- HashiCorp Terraform Associate (004) → *planned*
- Docker Certified Associate (DCA) → *planned after Phase 2*

---

## 2. Current State Assessment

### What exists today

| Asset | Location | Status |
|---|---|---|
| AWS Terraform Portfolio | `github.com/Victor-David-Medina/aws-terraform-portfolio` | Active |
| Multi-AZ VPC (public/private subnets) | Portfolio repo | Complete |
| Auto Scaling Group + CloudWatch | Portfolio repo | Complete |
| GitHub Actions CI pipeline (fmt/validate/tfsec/plan) | Portfolio repo | Complete |
| Runbook + ADRs + cost table | Portfolio repo | Complete |
| Profile README | `github.com/Victor-David-Medina` | Current |

### What is missing

- No containerization layer — nothing runs in Docker locally or in CI
- No ECS/ECR resources — container orchestration is absent from portfolio
- CI pipeline installs tools fresh each run (slower, version-inconsistent)
- No persistent tool context between Claude co-work sessions
- No second major portfolio project to show breadth

### Gap analysis

```
Current:  [Terraform] → [GitHub Actions] → [AWS VPC + ASG]
Target:   [Terraform] → [Docker + GitHub Actions] → [AWS VPC + ECS Fargate + ECR + ALB]
                                ↑
                     Containerized runner (Phase 1)
                     brings Docker into the CI layer immediately
```

---

## 3. Strategic Rationale — Why Docker

### Mapping the Docker AI article to this workflow

| Article Pattern | Concept | This Workflow's Application |
|---|---|---|
| Docker Model Runner (DMR) | Run LLMs locally via Docker API | Future: local AI model for offline development sessions |
| Docker Compose as service layer | Define multi-service apps declaratively | Define Terraform runner + MCP servers as a Compose stack |
| Docker Offload | Transparently offload compute to cloud | Fargate Spot for ephemeral CI jobs; no local GPU needed |
| MCP Servers in Docker | Pre-built tool integrations for AI agents | Run MCP servers as containers; Claude reads project files + AWS state |
| GPU-optimized base images | Training/inference on PyTorch/TF images | Not relevant for cloud ops; skip this angle |

### Why ECS Fargate (not EKS/Kubernetes)

| Factor | ECS Fargate | EKS (Kubernetes) |
|---|---|---|
| Operational overhead | Very low — AWS manages the control plane | High — you manage node groups, upgrades, networking |
| Cost at small scale | Lower — pay per task, no idle nodes | Higher — control plane + node group minimum costs |
| Portfolio signal | "I understand container orchestration at the managed-service level" | "I run Kubernetes" — overkill for a solo portfolio project |
| Learning curve | 1–2 weeks to competency | 2–3 months to production-ready |
| AWS certification alignment | Directly tested in AWS SysOps + DevOps Pro | Tested in separate Kubernetes certifications |
| Time to build | 2–4 weeks for a complete portfolio project | 2–3 months |

**Verdict:** ECS Fargate is the right choice for this stage. It extends the existing Terraform + AWS knowledge directly, appears on AWS certification exams, and produces a complete portfolio project faster.

---

## 4. Architecture Overview

### End-state architecture (after all three phases)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│  Developer Workstation                                                      │
│                                                                             │
│  ┌──────────────┐    ┌──────────────────────────────────┐                  │
│  │ Claude Code  │◄──►│  Docker Compose (MCP Workspace)  │                  │
│  │ (AI session) │    │  ┌────────────┐ ┌─────────────┐  │                  │
│  └──────────────┘    │  │ filesystem │ │  aws-mcp    │  │                  │
│                      │  │    MCP     │ │  (read-only)│  │                  │
│                      │  └────────────┘ └─────────────┘  │                  │
│                      └──────────────────────────────────┘                  │
│                                           │                                 │
│  ┌─────────────────────────────────────── │ ──────────────┐                │
│  │  Terraform Runner Container            │               │                │
│  │  ghcr.io/victor-david-medina/tf-runner │               │                │
│  │  [terraform | tfsec | checkov | awscli]│               │                │
│  └─────────────────────────────────────── │ ──────────────┘                │
└─────────────────────────────────────────── │ ──────────────────────────────┘
                                             │ git push
                                             ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│  GitHub                                                                     │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │  GitHub Actions CI/CD Pipeline                                       │  │
│  │                                                                      │  │
│  │  [checkout] → [build image] → [push ECR] → [ECS rolling deploy]     │  │
│  │      ↓              ↓               ↓               ↓               │  │
│  │  [tf fmt]    [tfsec scan]    [checkov scan]    [tf plan/apply]       │  │
│  │                                                                      │  │
│  │  Container: ghcr.io/victor-david-medina/tf-runner:latest             │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
                                             │
                                             ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│  AWS (us-east-1)                                                            │
│                                                                             │
│  ┌────────────────────────────────────────────────────────────────────┐    │
│  │  VPC (10.0.0.0/16)                                                 │    │
│  │                                                                    │    │
│  │  ┌─────────────────────────┐  ┌─────────────────────────────────┐ │    │
│  │  │   Public Subnet AZ-1    │  │       Public Subnet AZ-2        │ │    │
│  │  │  10.0.1.0/24            │  │       10.0.2.0/24               │ │    │
│  │  │  ┌───────────────────┐  │  │  ┌───────────────────────────┐ │ │    │
│  │  │  │ ALB (port 80/443) │  │  │  │  ALB (cross-AZ)           │ │ │    │
│  │  │  └─────────┬─────────┘  │  │  └─────────────┬─────────────┘ │ │    │
│  │  └────────────│────────────┘  └────────────────│───────────────┘ │    │
│  │               │                                 │                  │    │
│  │  ┌────────────│─────────────────────────────────│───────────────┐ │    │
│  │  │  Private Subnet AZ-1     │  Private Subnet AZ-2              │ │    │
│  │  │  10.0.3.0/24             │  10.0.4.0/24                      │ │    │
│  │  │                          │                                    │ │    │
│  │  │  ┌─────────────────┐     │    ┌─────────────────┐            │ │    │
│  │  │  │  ECS Task       │     │    │  ECS Task       │            │ │    │
│  │  │  │  (Fargate)      │     │    │  (Fargate)      │            │ │    │
│  │  │  │  [app container]│     │    │  [app container]│            │ │    │
│  │  │  └─────────────────┘     │    └─────────────────┘            │ │    │
│  │  └──────────────────────────────────────────────────────────────┘ │    │
│  │                                                                    │    │
│  │  ┌──────────────────────────────────────────────────────────────┐ │    │
│  │  │  Supporting Services                                          │ │    │
│  │  │  Amazon ECR (image registry)                                  │ │    │
│  │  │  CloudWatch (Container Insights + logs + alarms)              │ │    │
│  │  │  IAM (task execution role + task role)                        │ │    │
│  │  │  AWS Budget (cost alert)                                      │ │    │
│  │  └──────────────────────────────────────────────────────────────┘ │    │
│  └────────────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 5. Phase 1 — Containerized Terraform Runner

### Goal
A single Docker image containing all IaC tooling pinned to specific versions. The image runs identically on a developer laptop, in GitHub Actions, or in any CI environment. Zero manual tool installation per project.

### Tool inventory

| Tool | Purpose | Version strategy |
|---|---|---|
| Terraform | IaC provisioning | Pin to latest stable (currently 1.7.x) |
| tfsec | Static security scanning of Terraform | Pin to latest stable |
| checkov | Policy-as-code compliance checks | Pin via pip install |
| AWS CLI v2 | AWS API calls, credential validation | Install from official zip |
| jq | JSON parsing in bash scripts | Alpine package |
| yq | YAML parsing (for CI config inspection) | Alpine package |
| git | Source control ops inside container | Alpine package |
| bash | Scripting, entrypoint | Alpine package |

### Dockerfile (multi-stage build)

```dockerfile
# ============================================================
# tf-runner — Containerized Terraform + Security Tooling
# Image: ghcr.io/victor-david-medina/tf-runner
# ============================================================

# ----- Stage 1: Alpine base with system packages -----
FROM alpine:3.19 AS base

RUN apk add --no-cache \
    bash \
    curl \
    git \
    jq \
    python3 \
    py3-pip \
    unzip \
    wget \
    yq

# ----- Stage 2: Install Terraform binary -----
FROM base AS terraform-installer

ARG TERRAFORM_VERSION=1.7.5

RUN curl -fsSL \
    "https://releases.hashicorp.com/terraform/${TERRAFORM_VERSION}/terraform_${TERRAFORM_VERSION}_linux_amd64.zip" \
    -o /tmp/terraform.zip \
    && unzip /tmp/terraform.zip -d /usr/local/bin \
    && rm /tmp/terraform.zip \
    && terraform version

# ----- Stage 3: Install tfsec -----
FROM terraform-installer AS tfsec-installer

ARG TFSEC_VERSION=1.28.5

RUN curl -fsSL \
    "https://github.com/aquasecurity/tfsec/releases/download/v${TFSEC_VERSION}/tfsec-linux-amd64" \
    -o /usr/local/bin/tfsec \
    && chmod +x /usr/local/bin/tfsec \
    && tfsec --version

# ----- Stage 4: Install checkov via pip -----
FROM tfsec-installer AS checkov-installer

ARG CHECKOV_VERSION=3.2.0

RUN pip3 install --no-cache-dir checkov==${CHECKOV_VERSION} \
    && checkov --version

# ----- Stage 5: Install AWS CLI v2 -----
FROM checkov-installer AS awscli-installer

RUN curl -fsSL "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" \
    -o /tmp/awscliv2.zip \
    && unzip /tmp/awscliv2.zip -d /tmp \
    && /tmp/aws/install \
    && rm -rf /tmp/awscliv2.zip /tmp/aws \
    && aws --version

# ----- Final stage: Lean production image -----
FROM awscli-installer AS final

WORKDIR /workspace

# Non-root user — security best practice
RUN addgroup -S tfrunner \
    && adduser -S tfrunner -G tfrunner \
    && chown tfrunner:tfrunner /workspace

USER tfrunner

# Verify all tools available
RUN terraform version \
    && tfsec --version \
    && checkov --version \
    && aws --version \
    && jq --version

LABEL org.opencontainers.image.source="https://github.com/Victor-David-Medina/tf-runner"
LABEL org.opencontainers.image.description="Terraform + tfsec + checkov + AWS CLI runner"
LABEL maintainer="Victor David Medina"

ENTRYPOINT ["/bin/bash"]
```

### Image publish workflow (GitHub Actions)

```yaml
# .github/workflows/publish-tf-runner.yml
# Builds and publishes the tf-runner image to GitHub Container Registry (GHCR)
# Triggers: push to main when Dockerfile changes; manual dispatch

name: Publish tf-runner Image

on:
  push:
    branches: [main]
    paths:
      - 'Dockerfile'
      - '.github/workflows/publish-tf-runner.yml'
  workflow_dispatch:

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: victor-david-medina/tf-runner

jobs:
  build-and-push:
    name: Build and Push to GHCR
    runs-on: ubuntu-latest

    permissions:
      contents: read
      packages: write

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Log in to GHCR
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata (tags, labels)
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=sha,prefix=sha-
            type=raw,value=latest,enable={{is_default_branch}}
            type=semver,pattern={{version}}

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          build-args: |
            TERRAFORM_VERSION=1.7.5
            TFSEC_VERSION=1.28.5
            CHECKOV_VERSION=3.2.0
```

### Terraform CI pipeline using the runner image

```yaml
# .github/workflows/terraform-ci.yml
# Uses tf-runner container image — no tool installation step needed

name: Terraform CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  terraform:
    name: Format · Validate · Security · Plan
    runs-on: ubuntu-latest

    container:
      image: ghcr.io/victor-david-medina/tf-runner:latest
      credentials:
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}

    permissions:
      id-token: write       # required for OIDC AWS auth
      contents: read
      pull-requests: write  # to post plan output as PR comment

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Configure AWS credentials (OIDC)
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: us-east-1

      - name: Terraform Format Check
        run: terraform fmt -check -recursive
        working-directory: ./terraform

      - name: Terraform Init
        run: terraform init -backend=false
        working-directory: ./terraform

      - name: Terraform Validate
        run: terraform validate
        working-directory: ./terraform

      - name: tfsec Security Scan
        run: tfsec ./terraform --format=default --minimum-severity=HIGH

      - name: checkov Policy Scan
        run: |
          checkov -d ./terraform \
            --framework terraform \
            --output cli \
            --soft-fail-on MEDIUM

      - name: Terraform Plan
        if: github.event_name == 'pull_request'
        run: terraform plan -no-color -out=tfplan 2>&1 | tee plan-output.txt
        working-directory: ./terraform

      - name: Post Plan as PR Comment
        if: github.event_name == 'pull_request'
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const plan = fs.readFileSync('plan-output.txt', 'utf8');
            const truncated = plan.length > 65000 ? plan.substring(0, 65000) + '\n... (truncated)' : plan;
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `## Terraform Plan\n\`\`\`hcl\n${truncated}\n\`\`\``
            });
```

### Local usage

```bash
# Run the Terraform runner locally against a project directory
docker run --rm \
  -v $(pwd):/workspace \
  -e AWS_PROFILE=default \
  -v ~/.aws:/home/tfrunner/.aws:ro \
  ghcr.io/victor-david-medina/tf-runner:latest \
  -c "terraform fmt -check && terraform validate && tfsec ."

# Interactive shell inside the runner
docker run --rm -it \
  -v $(pwd):/workspace \
  -v ~/.aws:/home/tfrunner/.aws:ro \
  ghcr.io/victor-david-medina/tf-runner:latest
```

### File structure for Phase 1

```
tf-runner/                        ← new standalone repo
├── Dockerfile
├── .github/
│   └── workflows/
│       └── publish-tf-runner.yml
└── README.md

aws-terraform-portfolio/          ← existing repo — only workflow file changes
├── .github/
│   └── workflows/
│       └── terraform-ci.yml      ← updated to use container: image
├── terraform/
│   └── (existing .tf files)
└── docs/
    └── (existing runbooks/ADRs)
```

---

## 6. Phase 2 — Docker + ECS Fargate Portfolio Project

### Goal
A second portfolio project that deploys a containerized application to AWS ECS Fargate using Terraform and GitHub Actions. This project extends the existing VPC foundation and tells a complete infrastructure story.

### Repository name suggestion
`docker-ecs-fargate-portfolio` or `aws-container-ops-portfolio`

### Application choice
A lightweight Python Flask "health check API" — intentionally simple. The point is the infrastructure and deployment pipeline, not the application. The app exposes:
- `GET /health` → `{"status": "ok", "version": "1.0.0"}`
- `GET /` → plain HTML status page

This is realistic: in production, similar health endpoints are often the first containerized service teams deploy.

### Full AWS resource inventory

| Terraform Resource | AWS Resource | Purpose |
|---|---|---|
| `aws_ecr_repository` | Amazon ECR Repository | Store container images |
| `aws_ecr_lifecycle_policy` | ECR Lifecycle Policy | Auto-delete old images (keep last 10) |
| `aws_ecs_cluster` | ECS Cluster | Logical grouping of ECS services |
| `aws_ecs_cluster_capacity_providers` | Cluster Capacity Providers | Enable FARGATE + FARGATE_SPOT |
| `aws_ecs_task_definition` | Task Definition | Container spec (image, CPU, memory, env) |
| `aws_ecs_service` | ECS Service | Maintains desired task count, rolling deploy |
| `aws_lb` | Application Load Balancer | Ingress, health checks, SSL termination |
| `aws_lb_listener` | ALB Listener (port 80) | Routes HTTP → target group |
| `aws_lb_target_group` | ALB Target Group | Routes to ECS tasks on port 5000 |
| `aws_security_group` (alb) | ALB Security Group | Allow 80/443 inbound from internet |
| `aws_security_group` (ecs) | ECS Task Security Group | Allow traffic only from ALB SG |
| `aws_iam_role` (execution) | ECS Task Execution Role | Pull from ECR, write to CloudWatch |
| `aws_iam_role_policy_attachment` | Managed policy attachment | AmazonECSTaskExecutionRolePolicy |
| `aws_iam_role` (task) | ECS Task Role | Application-level AWS permissions |
| `aws_cloudwatch_log_group` | CloudWatch Log Group | Container stdout/stderr logs |
| `aws_appautoscaling_target` | Application Auto Scaling Target | Register ECS service for scaling |
| `aws_appautoscaling_policy` | Scaling Policy (CPU) | Scale out when CPU > 70% |
| `aws_budgets_budget` | AWS Budget | Alert when monthly cost exceeds threshold |
| `aws_s3_bucket` (tfstate) | S3 Bucket | Remote Terraform state |
| `aws_dynamodb_table` (lock) | DynamoDB Table | Terraform state locking |

**Total: ~20 Terraform resources — a substantial but buildable portfolio scope.**

### Terraform module structure

```
docker-ecs-fargate-portfolio/
├── terraform/
│   ├── main.tf               ← provider config, backend (S3 + DynamoDB)
│   ├── variables.tf
│   ├── outputs.tf
│   ├── versions.tf           ← terraform required_version + provider versions
│   │
│   ├── modules/
│   │   ├── ecr/
│   │   │   ├── main.tf       ← aws_ecr_repository + lifecycle_policy
│   │   │   ├── variables.tf
│   │   │   └── outputs.tf    ← repository_url
│   │   │
│   │   ├── ecs/
│   │   │   ├── main.tf       ← cluster, task_definition, service
│   │   │   ├── variables.tf  ← image_uri, cpu, memory, desired_count
│   │   │   └── outputs.tf    ← cluster_name, service_name
│   │   │
│   │   ├── alb/
│   │   │   ├── main.tf       ← aws_lb, aws_lb_listener, aws_lb_target_group
│   │   │   ├── variables.tf  ← vpc_id, subnet_ids, security_group_id
│   │   │   └── outputs.tf    ← alb_dns_name, target_group_arn
│   │   │
│   │   ├── iam/
│   │   │   ├── main.tf       ← execution_role, task_role, policy attachments
│   │   │   ├── variables.tf
│   │   │   └── outputs.tf    ← execution_role_arn, task_role_arn
│   │   │
│   │   └── scaling/
│   │       ├── main.tf       ← appautoscaling_target + policy
│   │       ├── variables.tf  ← min_capacity, max_capacity, target_cpu_pct
│   │       └── outputs.tf
│   │
│   └── environments/
│       ├── dev.tfvars        ← desired_count=1, use FARGATE_SPOT
│       └── prod.tfvars       ← desired_count=2, use FARGATE
│
├── app/
│   ├── app.py                ← Flask health check app
│   ├── requirements.txt      ← flask==3.0.0
│   └── Dockerfile            ← multi-stage Python build
│
├── .github/
│   └── workflows/
│       ├── ci.yml            ← lint + build + scan on PR
│       └── deploy.yml        ← push image + ECS deploy on merge to main
│
└── docs/
    ├── architecture.md       ← diagram + component descriptions
    ├── runbook.md            ← deployment, rollback, scaling procedures
    ├── adr-001-fargate-vs-ec2.md
    ├── adr-002-alb-vs-nlb.md
    └── cost-analysis.md
```

### Application Dockerfile

```dockerfile
# ============================================================
# app/Dockerfile — Flask health check API
# Multi-stage build: build deps separate from runtime
# ============================================================

# ----- Stage 1: Build dependencies -----
FROM python:3.12-slim AS builder

WORKDIR /build

COPY requirements.txt .
RUN pip install --no-cache-dir --target=/build/deps -r requirements.txt

# ----- Stage 2: Runtime image -----
FROM python:3.12-slim AS runtime

# Security: non-root user
RUN useradd -r -u 1001 -g root appuser

WORKDIR /app

# Copy installed deps from builder
COPY --from=builder /build/deps /usr/local/lib/python3.12/site-packages/

# Copy application code
COPY app.py .

# Metadata
LABEL org.opencontainers.image.source="https://github.com/Victor-David-Medina/docker-ecs-fargate-portfolio"
LABEL org.opencontainers.image.description="Health check API — ECS Fargate portfolio demo"

USER appuser

EXPOSE 5000

# Use gunicorn for production (not flask dev server)
CMD ["python", "-m", "gunicorn", "--bind", "0.0.0.0:5000", "--workers", "2", "app:app"]
```

### CI/CD Pipeline (complete)

```yaml
# .github/workflows/deploy.yml
# Triggered on merge to main:
# 1. Build and push image to ECR
# 2. Update ECS service with new image
# 3. Wait for rollout to stabilize

name: Build, Push, Deploy

on:
  push:
    branches: [main]

env:
  AWS_REGION: us-east-1
  ECR_REPO: docker-ecs-fargate-demo
  ECS_CLUSTER: ecs-fargate-cluster
  ECS_SERVICE: fargate-service

jobs:
  build-and-push:
    name: Build & Push to ECR
    runs-on: ubuntu-latest

    outputs:
      image-uri: ${{ steps.build.outputs.image-uri }}

    permissions:
      id-token: write
      contents: read

    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials (OIDC)
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build, tag, and push image
        id: build
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPO:$IMAGE_TAG ./app
          docker push $ECR_REGISTRY/$ECR_REPO:$IMAGE_TAG
          # Also tag as latest
          docker tag $ECR_REGISTRY/$ECR_REPO:$IMAGE_TAG $ECR_REGISTRY/$ECR_REPO:latest
          docker push $ECR_REGISTRY/$ECR_REPO:latest
          echo "image-uri=$ECR_REGISTRY/$ECR_REPO:$IMAGE_TAG" >> $GITHUB_OUTPUT

  deploy:
    name: Deploy to ECS Fargate
    runs-on: ubuntu-latest
    needs: build-and-push

    permissions:
      id-token: write
      contents: read

    steps:
      - name: Configure AWS credentials (OIDC)
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Download current task definition
        run: |
          aws ecs describe-task-definition \
            --task-definition ${{ env.ECS_SERVICE }} \
            --query taskDefinition \
            > task-def.json

      - name: Update task definition with new image
        id: task-def
        uses: aws-actions/amazon-ecs-render-task-definition@v1
        with:
          task-definition: task-def.json
          container-name: app
          image: ${{ needs.build-and-push.outputs.image-uri }}

      - name: Deploy to ECS service (rolling update)
        uses: aws-actions/amazon-ecs-deploy-task-definition@v1
        with:
          task-definition: ${{ steps.task-def.outputs.task-definition }}
          service: ${{ env.ECS_SERVICE }}
          cluster: ${{ env.ECS_CLUSTER }}
          wait-for-service-stability: true
```

### IAM roles — detailed policy breakdown

**Task Execution Role** (used by ECS agent to start your task):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ecr:GetAuthorizationToken",
        "ecr:BatchCheckLayerAvailability",
        "ecr:GetDownloadUrlForLayer",
        "ecr:BatchGetImage"
      ],
      "Resource": "*",
      "Sid": "PullFromECR"
    },
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:us-east-1:ACCOUNT_ID:log-group:/ecs/fargate-demo:*",
      "Sid": "WriteToCloudWatch"
    }
  ]
}
```

**Task Role** (permissions your application container has at runtime):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ssm:GetParameter",
        "ssm:GetParameters"
      ],
      "Resource": "arn:aws:ssm:us-east-1:ACCOUNT_ID:parameter/app/*",
      "Sid": "ReadSSMParameters"
    }
  ]
}
```

*Note: Task role starts minimal. Add only permissions your application actually needs.*

### CloudWatch Container Insights configuration

Container Insights provides CPU/memory per-task metrics, not just per-service. Enable it on the cluster:

```hcl
# In modules/ecs/main.tf
resource "aws_ecs_cluster" "main" {
  name = var.cluster_name

  setting {
    name  = "containerInsights"
    value = "enabled"
  }
}
```

**Key CloudWatch alarms to create:**

| Alarm | Metric | Threshold | Action |
|---|---|---|---|
| High CPU | `ECS/ContainerInsights CPUUtilization` | > 80% for 5 min | SNS alert |
| High Memory | `ECS/ContainerInsights MemoryUtilization` | > 85% for 5 min | SNS alert |
| Task count low | `ECS/ContainerInsights RunningTaskCount` | < 1 | SNS alert (service down) |
| ALB 5xx errors | `ApplicationELB HTTPCode_Target_5XX_Count` | > 10 per minute | SNS alert |
| ALB response time | `ApplicationELB TargetResponseTime` | > 1 second avg | SNS alert |

### Networking — how ECS connects to the existing VPC

```
Existing VPC (from Portfolio Project 1):
  - VPC: 10.0.0.0/16
  - Public subnets: 10.0.1.0/24 (AZ-1), 10.0.2.0/24 (AZ-2)
  - Private subnets: 10.0.3.0/24 (AZ-1), 10.0.4.0/24 (AZ-2)
  - NAT Gateway in public subnet (enables outbound from private)
  - Internet Gateway

Added by Phase 2:
  - ALB in public subnets (internet-facing)
  - ECS tasks in private subnets (no direct internet access)
  - ALB security group: allow 80/443 from 0.0.0.0/0
  - ECS task security group: allow port 5000 only from ALB security group
  - Traffic flow: Internet → IGW → ALB → ECS Task (private subnet, via ALB SG rule)
```

### Portfolio narrative this creates

| Project | What it demonstrates |
|---|---|
| Project 1: aws-terraform-portfolio | I can provision cloud infrastructure with IaC |
| Project 2: docker-ecs-fargate-portfolio | I can containerize applications and deploy them into that infrastructure |
| Combined | Full cloud ops stack: network → compute → containers → CI/CD → observability |

Interview answer:
> "My first project builds the VPC foundation with Terraform — subnets, NAT gateway, security groups, CloudWatch alarms. My second project deploys into that foundation: I containerize a Flask API, push it to ECR via GitHub Actions, and deploy it to ECS Fargate with a Terraform-provisioned cluster and Application Load Balancer. The whole thing runs through a GitHub Actions pipeline that goes from `git push` to live deployment in under 5 minutes."

---

## 7. Phase 3 — Claude Co-Work Enhancement via Docker MCP

### What MCP is

Model Context Protocol (MCP) is an open standard that lets AI assistants like Claude connect to external data sources and tools through a standardized interface. Instead of copy-pasting file contents or AWS output into a chat, MCP servers expose that data directly to Claude in a structured way.

Docker's ecosystem includes pre-built MCP server images, making it easy to run them without polluting the local environment.

### How Claude Code uses MCP

Claude Code reads MCP server configurations from `.mcp.json` in the project root (or global Claude settings). Each server exposes "tools" that Claude can call — for example, a filesystem server exposes a `read_file` tool.

**Configuration file: `.mcp.json`**

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "docker",
      "args": [
        "run", "--rm", "-i",
        "--mount", "type=bind,src=/home/user/Victor-David-Medina,dst=/projects",
        "mcp/filesystem",
        "/projects"
      ]
    },
    "aws-docs": {
      "command": "docker",
      "args": [
        "run", "--rm", "-i",
        "mcp/aws-documentation"
      ]
    }
  }
}
```

### MCP servers relevant to this workflow

| Server | Source | What it provides to Claude |
|---|---|---|
| `mcp/filesystem` | Docker Hub (official) | Read/write access to local files; Claude can read your Terraform files directly |
| `mcp/aws-documentation` | Docker Hub (official) | Search and retrieve official AWS docs; Claude cites accurate, current documentation |
| `mcp/github` | Docker Hub (official) | Read GitHub repos, issues, PRs; Claude can track project state |
| `mcp/postgres` | Docker Hub (official) | Query a local PostgreSQL database; useful if project tracking moves to DB |
| `mcp/puppeteer` | Docker Hub (official) | Control a browser; useful for scraping or testing web UIs |

### Docker Compose workspace definition

```yaml
# docker-compose.mcp.yml
# Start all MCP servers for a Claude co-work session
# Usage: docker compose -f docker-compose.mcp.yml up -d

services:
  filesystem-mcp:
    image: mcp/filesystem
    restart: unless-stopped
    volumes:
      - type: bind
        source: /home/user/Victor-David-Medina
        target: /projects
        read_only: true
    command: ["/projects"]
    labels:
      - "mcp.server=filesystem"
      - "mcp.description=Read-only access to all project files"

  aws-docs-mcp:
    image: mcp/aws-documentation
    restart: unless-stopped
    labels:
      - "mcp.server=aws-docs"
      - "mcp.description=Search and retrieve AWS documentation"
```

### Session workflow

```
1. Start MCP servers:
   docker compose -f docker-compose.mcp.yml up -d

2. Open Claude Code in project directory:
   claude .

3. Claude now has:
   - Direct read access to all project files (no manual copy-paste)
   - Ability to search AWS docs inline
   - Full conversation context from .mcp.json config

4. End session:
   docker compose -f docker-compose.mcp.yml down
```

### What this changes about the co-work experience

| Without MCP | With Docker MCP |
|---|---|
| Manually paste file contents into chat | Claude reads files directly |
| Manually describe project structure | Claude explores it via filesystem MCP |
| Claude's AWS knowledge may be dated | Claude queries live AWS docs via MCP |
| Context resets each session | `.mcp.json` in repo preserves tool config across sessions |
| Single context window constraint | MCP tools let Claude fetch only what it needs |

---

## 8. Learning Path & Certification Roadmap

### Sequence (order matters)

```
NOW          → AWS Cloud Practitioner (CLF-C02)
               Foundation: AWS services, billing, IAM basics, shared responsibility
               Study time: 4–6 weeks
               Exam: 65 questions, 90 minutes, $150

ALONGSIDE    → Docker fundamentals (no cert yet — just build)
               Resource: Docker's official "Get Started" tutorial
               Practice: Build Dockerfiles, run containers, use Compose
               Goal: Understand image layers, volumes, networking, multi-stage builds

AFTER CCP    → Phase 1: Containerized Terraform Runner
               1–2 day build project
               Deliverable: Published image at ghcr.io/victor-david-medina/tf-runner

NEXT         → HashiCorp Terraform Associate (004)
               Validates: HCL, state, modules, workspaces, providers
               Study time: 4–6 weeks
               Exam: 57 questions, 60 minutes, $70.50
               Resource: HashiCorp official study guide + Terraform Up & Running (book)

AFTER TF     → Phase 2: Docker + ECS Fargate Portfolio Project
               2–4 week build project
               Deliverable: Public GitHub repo with working ECS deployment

THEN         → Docker Certified Associate (DCA)
               Validates: Container lifecycle, networking, storage, security, orchestration, registries
               Study time: 4–6 weeks
               Exam: 55 questions, 90 minutes, $195
               Resource: Docker's official study guide + DCA practice exams

ONGOING      → Phase 3: Claude MCP co-work enhancement
               Iterative — add MCP servers as new needs arise
```

### Docker fundamentals topics to cover

| Topic | Key concepts | Practice task |
|---|---|---|
| Image basics | Layers, cache, base images | Pull and run `nginx`, inspect layers with `docker history` |
| Dockerfile | FROM, RUN, COPY, EXPOSE, CMD, ENTRYPOINT | Write a Dockerfile for a simple Python script |
| Multi-stage builds | Builder pattern, final stage size | Convert single-stage to multi-stage; compare image sizes |
| Volumes | Bind mounts vs named volumes | Mount a local directory into a running container |
| Networking | bridge, host, overlay networks | Run two containers that communicate by name |
| Docker Compose | services, networks, volumes, depends_on | Write a Compose file for app + database |
| ECR integration | `aws ecr get-login-password`, `docker push` | Push a local image to a personal ECR repository |
| Security | Non-root user, minimal base image, secret handling | Add `USER` directive; use build args for secrets |

### DCA exam domain breakdown

| Domain | Weight | Key topics |
|---|---|---|
| Orchestration | 25% | ECS concepts, task definitions, services, scaling |
| Image creation & management | 20% | Dockerfile best practices, multi-stage builds, registries |
| Installation & configuration | 15% | Docker Engine, daemon config, storage drivers |
| Networking | 15% | bridge/overlay/macvlan, DNS, publishing ports |
| Security | 15% | Content trust, secrets, least-privilege, scanning |
| Storage & volumes | 10% | Volume drivers, bind mounts, tmpfs |

---

## 9. Cost Analysis

### Phase 1 — Containerized Terraform Runner

| Resource | Cost |
|---|---|
| GitHub Container Registry (GHCR) | Free for public repos |
| GitHub Actions minutes | Free tier: 2,000 min/month on public repos |
| Developer time to build | 1–2 days |
| **Total ongoing cost** | **$0/month** |

### Phase 2 — Docker + ECS Fargate (dev environment)

*Estimates based on us-east-1 pricing, March 2026*

| Resource | Config | Estimated Monthly Cost |
|---|---|---|
| ECS Fargate task (1x) | 0.25 vCPU, 0.5 GB RAM, 730 hrs | ~$11/month |
| Application Load Balancer | 1 ALB, ~1 LCU | ~$18/month |
| Amazon ECR | ~1 GB storage | ~$0.10/month |
| CloudWatch Container Insights | Metrics + logs, low volume | ~$2–5/month |
| NAT Gateway | 1 gateway, low traffic | ~$32/month |
| Data transfer | Minimal for portfolio demo | ~$1–2/month |
| **Total (dev, running 24/7)** | | **~$64–68/month** |

**Cost optimization strategies:**
- Use FARGATE_SPOT for dev environment (up to 70% cheaper) → reduces Fargate cost to ~$3/month
- Tear down when not actively demoing (ECS service desired_count = 0)
- NAT Gateway is the largest line item — reuse from existing VPC if possible
- With spot + teardown: **~$20–25/month active, $0 when torn down**

### Phase 3 — Claude MCP

| Resource | Cost |
|---|---|
| Docker containers (local) | $0 — runs on local machine |
| Claude Code subscription | Existing subscription |
| **Total** | **$0 additional** |

---

## 10. Decision Log

| Decision | Options considered | Choice | Rationale |
|---|---|---|---|
| Container registry for tf-runner | ECR, GHCR, Docker Hub | GHCR | Free for public repos, integrates with GITHUB_TOKEN, no separate auth config |
| App runtime for Phase 2 | Node.js, Python Flask, Go | Python Flask | Simplest for a demo; `requirements.txt` is minimal; aligns with checkov (Python-based) |
| Fargate vs EC2 launch type | EC2, Fargate, Fargate Spot | Fargate (+ Spot for dev) | No node management; pay per task; cleaner portfolio story |
| ALB vs NLB | ALB, NLB | ALB | HTTP health checks, path-based routing, easier Terraform config; NLB overkill for this use case |
| Image registry for app | ECR, GHCR | ECR | Native AWS integration; no external auth; already in the AWS ecosystem |
| State backend | Local, S3, Terraform Cloud | S3 + DynamoDB | Standard enterprise pattern; matches what hiring managers expect to see |
| Docker base image | ubuntu:22.04, debian:slim, alpine:3.19 | alpine:3.19 | Smallest attack surface; fastest pull; sufficient for CLI tools |

---

## 11. Interview Preparation

### Questions to be ready for

**Docker fundamentals:**

Q: "What is the difference between CMD and ENTRYPOINT in a Dockerfile?"
A: ENTRYPOINT defines the main command that always runs. CMD provides default arguments to ENTRYPOINT (or is the full command if no ENTRYPOINT is set). Together they are often combined: ENTRYPOINT is the binary, CMD is the default flags. ENTRYPOINT cannot be overridden at runtime without `--entrypoint`; CMD can be overridden by appending arguments to `docker run`.

Q: "What is a multi-stage build and why use it?"
A: A multi-stage build uses multiple FROM instructions in a single Dockerfile. Early stages handle compilation, dependency installation, or tooling that is large or not needed at runtime. The final stage copies only the artifacts from earlier stages, resulting in a much smaller production image with a smaller attack surface.

Q: "How do you handle secrets in Docker?"
A: Never embed secrets in a Dockerfile (they become part of the image layer history). Use build args only for non-sensitive values. For runtime secrets: use AWS Secrets Manager or SSM Parameter Store referenced from the ECS task definition via `secrets` field — ECS injects them as environment variables at task startup without them appearing in the task definition in plaintext.

**ECS Fargate:**

Q: "Walk me through deploying a containerized application to ECS Fargate."
A: (Use the Phase 2 architecture) Create an ECR repository for the image. Write a Dockerfile, build and push via GitHub Actions. In Terraform: create an ECS cluster with Fargate capacity provider, write a task definition specifying the ECR image URI, CPU, memory, port mappings, log configuration, and IAM roles. Create an ECS service that references the task definition and targets an ALB target group. Create the ALB, listener, and target group separately. On deploy, GitHub Actions builds a new image, pushes to ECR, and updates the ECS service — ECS performs a rolling update.

Q: "What is the difference between a task definition and an ECS service?"
A: A task definition is a blueprint — it describes the container image, CPU, memory, networking mode, environment variables, secrets, and log configuration. An ECS service is the runtime management layer — it ensures a desired number of task definition instances are always running, handles rolling updates when the task definition changes, and integrates with load balancers.

Q: "What is the ECS task execution role versus the task role?"
A: The task execution role is used by the ECS agent (infrastructure layer) to start your task — it needs permissions to pull the image from ECR and write logs to CloudWatch. The task role is used by the application running inside your container — it has the permissions your application code needs (e.g., reading from S3, querying SSM). Keep them separate and grant minimum permissions to each.

**CI/CD integration:**

Q: "How does GitHub Actions authenticate to AWS in your pipeline?"
A: Using OIDC (OpenID Connect) rather than long-lived access keys. AWS creates an IAM role with a trust policy that allows GitHub's OIDC provider to assume it. The GitHub Actions workflow requests a short-lived token from GitHub's OIDC endpoint and uses it to assume the IAM role. No AWS credentials are stored as GitHub secrets — just the role ARN.

### Skills summary for resume / LinkedIn

```
Docker (containers, multi-stage builds, ECR)
AWS ECS Fargate (task definitions, services, Fargate Spot)
Amazon ECR (image lifecycle management, pull-through cache)
Container-native CI/CD (build → push → rolling deploy via GitHub Actions)
CloudWatch Container Insights (task-level metrics, log routing)
Application Load Balancer (target group health checks, listener rules)
IAM (task execution role, task role, OIDC for GitHub Actions)
```

---

## 12. Troubleshooting Reference

### Phase 1 — Terraform Runner

| Symptom | Likely cause | Fix |
|---|---|---|
| `terraform: not found` in CI | Container image not pulled correctly | Verify image tag in `container: image:` field; check GHCR auth with GITHUB_TOKEN |
| tfsec exits with code 1 on valid code | tfsec finding a real issue | Read the output; either fix the finding or add `#tfsec:ignore:rule-id` comment with justification |
| AWS CLI fails with "Unable to locate credentials" | AWS credentials not passed to container | Use OIDC action before AWS CLI steps; verify role ARN in secrets |
| Multi-stage build is slow in CI | No layer cache | Add `cache-from: type=gha` and `cache-to: type=gha,mode=max` to build-push-action |

### Phase 2 — ECS Fargate

| Symptom | Likely cause | Fix |
|---|---|---|
| ECS task fails with `CannotPullContainerError` | Task execution role lacks ECR permissions | Verify `AmazonECSTaskExecutionRolePolicy` is attached; check ECR repo is in same account |
| Tasks stop immediately after start | Application crashes on startup | Check CloudWatch logs in `/ecs/fargate-demo` log group for the error; run container locally first with `docker run` |
| ALB returns 502 Bad Gateway | ECS tasks not passing health checks | Verify the health check path and port in the ALB target group match what your app actually serves |
| ECS service stuck in `DRAINING` | Old tasks not terminating | Check if there are long-running connections; adjust deregistration delay on target group |
| Terraform plan shows image URI as `null` | ECR push hasn't happened yet | Phase 1 in the pipeline (build + push) must complete before phase 2 (deploy); check `needs:` dependency |

### Phase 3 — MCP

| Symptom | Likely cause | Fix |
|---|---|---|
| MCP server not found by Claude Code | `.mcp.json` not in project root | Check file location; Claude Code looks for it in `cwd` |
| `docker: command not found` in MCP config | MCP server configured with `command: docker` but Docker not in PATH | Use full path: `command: /usr/local/bin/docker` or verify Docker Desktop is running |
| Filesystem MCP can't read files | Volume mount path wrong | Check `src` path in `--mount` arg; must be absolute path on host |

---

## 13. Glossary

| Term | Definition |
|---|---|
| **ECS** | Elastic Container Service — AWS's container orchestration service |
| **Fargate** | Serverless compute engine for ECS — no EC2 instances to manage |
| **Fargate Spot** | Spare Fargate capacity at up to 70% discount; tasks can be interrupted |
| **ECR** | Elastic Container Registry — AWS's private Docker image registry |
| **ALB** | Application Load Balancer — HTTP/HTTPS load balancer with path routing and health checks |
| **Task Definition** | Blueprint for an ECS task: image, CPU, memory, environment, ports, roles |
| **ECS Service** | Runtime manager that maintains desired task count and handles rolling updates |
| **Task Execution Role** | IAM role used by ECS agent to start a task (ECR pull, CloudWatch write) |
| **Task Role** | IAM role used by your application container at runtime |
| **OIDC** | OpenID Connect — federated auth that allows GitHub Actions to assume AWS roles without long-lived keys |
| **MCP** | Model Context Protocol — open standard for connecting AI models to external tools and data sources |
| **Multi-stage build** | Dockerfile pattern using multiple FROM stages to keep the final image small |
| **Container Insights** | CloudWatch feature providing CPU/memory metrics at the ECS task level |
| **GHCR** | GitHub Container Registry — free Docker registry integrated with GitHub |
| **tfsec** | Static security scanner for Terraform code |
| **checkov** | Policy-as-code tool that checks Terraform (and other IaC) against security benchmarks |
| **DCA** | Docker Certified Associate — vendor certification validating Docker fundamentals |
| **Rolling update** | ECS deployment strategy that replaces tasks incrementally with zero downtime |
| **Target group** | ALB component that routes requests to registered targets (ECS tasks) and performs health checks |

---

*Last updated: March 2026 — update this document as phases complete and decisions evolve.*
