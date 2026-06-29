---
title: "Agent Model Map"
version: 2.0.0
updated: 2026-05-14
---

# Model Map — Fullstack Agent System

Routing by activity type to maximize quality and minimize token cost.
**Estimated savings: ~60–75% vs. using Opus for everything.**

## Routing table

| Activity | Model | Agent |
|---|---|---|
| Planning and decomposition of complex requirements | claude-opus-4-8 | Maestro |
| Threat modeling, Zero Trust, compliance analysis | claude-opus-4-8 | Sentinel |
| RAG system design, data architecture | claude-opus-4-8 | Neuron |
| Critical security review (auth/data/infra) | claude-opus-4-8 | Sentinel |
| REST/GraphQL API implementation | claude-sonnet-4-6 | Stratum |
| Database modeling and migrations | claude-sonnet-4-6 | Stratum |
| Legacy code refactoring | claude-sonnet-4-6 | Stratum |
| Auth/authorization implementation | claude-sonnet-4-6 | Stratum |
| Complex React/Vue component creation | claude-sonnet-4-6 | Facet |
| Web performance (Core Web Vitals) | claude-sonnet-4-6 | Facet |
| WCAG accessibility audit | claude-sonnet-4-6 | Facet |
| Terraform/Pulumi IaC modules | claude-sonnet-4-6 | Bastion |
| CI/CD pipeline design | claude-sonnet-4-6 | Bastion |
| Observability (Prometheus/Grafana) | claude-sonnet-4-6 | Bastion |
| FinOps: cost analysis and optimization | claude-sonnet-4-6 | Bastion |
| ETL/ELT pipeline implementation | claude-sonnet-4-6 | Neuron |
| Feature engineering and exploratory analysis | claude-sonnet-4-6 | Neuron |
| Security code review (SAST) | claude-sonnet-4-6 | Sentinel |
| IAM and RBAC policies | claude-sonnet-4-6 | Sentinel |
| Unit and integration tests | claude-haiku-4-5 | Stratum |
| OpenAPI/JSDoc documentation | claude-haiku-4-5 | Stratum |
| Seeds and test data | claude-haiku-4-5 | Stratum |
| CSS utilities and style variants | claude-haiku-4-5 | Facet |
| E2E tests with Playwright | claude-haiku-4-5 | Facet |
| Storybook stories | claude-haiku-4-5 | Facet |
| Optimized Dockerfiles | claude-haiku-4-5 | Bastion |
| Kubernetes manifests (Deployment/Service/Ingress) | claude-haiku-4-5 | Bastion |
| Incident runbooks and playbooks | claude-haiku-4-5 | Bastion |
| Metrics and KPI reports | claude-haiku-4-5 | Neuron |
| Pipeline docstrings and documentation | claude-haiku-4-5 | Neuron |
| Security headers and WAF configs | claude-haiku-4-5 | Sentinel |
| Security review checklists | claude-haiku-4-5 | Sentinel |

## Decision rule

```
if    complex systemic reasoning, high impact, threat modeling  → opus-4-8
elif  code generation, technical analysis, architecture design  → sonnet-4-6
elif  known pattern, output < 500 tokens, repetitive config    → haiku-4-5
else                                                            → sonnet-4-6
```

## Current models (2026-05-14)

| Tier | Model ID | Use |
|---|---|---|
| Opus | claude-opus-4-8 | Architectural decisions, critical security, AI/data |
| Sonnet | claude-sonnet-4-6 | General implementation, code, analysis |
| Haiku | claude-haiku-4-5-20251001 | Repetitive tasks, docs, simple configs |
