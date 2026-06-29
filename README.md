---
tags: [fullstack-agent, sistema-multi-agente, claude, time-dev, engenharia]
versao: "2.1.0"
criado: 2026-05-14
atualizado: 2026-05-29
---

# Fullstack Agent System

A senior engineering team composed of 6 specialized agents, orchestrated by a central Orchestrator. Each agent is a senior IT professional — not a tutor. Agents think, write compilable code, build integrations, and run validation tests.

**Coordination via File-as-Bus**: state lives in files, not in conversation history.  
**Mandatory Evidence**: no deliverable without tests or execution logs.  
**Security with technical veto**: can block any deploy.

---

## Structure

```
fullstack-agent-system/
├── Orchestrator.md     ← Maestro: central planner (opus-4-8)
├── Backend-Dev.md      ← Stratum: APIs, DB, microservices (sonnet-4-6)
├── Frontend-Dev.md     ← Facet: UI/UX, React/Vue, a11y (sonnet-4-6)
├── Infra-Cloud.md      ← Bastion: AWS, Terraform, CI/CD (sonnet-4-6)
├── Data-AI.md          ← Neuron: ML, ETL, LLMs, RAG (opus-4-8)
├── Security.md         ← Sentinel: AppSec, OWASP, compliance (opus-4-8)
├── Forge.md            ← Forge: 5E code quality, scoring, refactoring (sonnet-4-6)
├── docs/
│   ├── Constitution.md     ← 6 principles that govern all agents
│   ├── agent-model-map.md  ← Routing: activity → model → agent
│   ├── standards-anti-patterns.md
│   └── progress.md         ← System state (File-as-Bus)
└── adr/
    └── 0000-template.md    ← Template for Architecture Decision Records
```

---

## Model Map

| Agent | opus-4-8 | sonnet-4-6 | haiku-4-5 |
|---|---|---|---|
| Maestro | Complex planning | — | — |
| Stratum | — | APIs, DB, auth, refactoring | Tests, docs, seeds |
| Facet | — | Components, performance, a11y | CSS, E2E, Storybook |
| Bastion | — | IaC, CI/CD, observability | Dockerfiles, YAML, runbooks |
| Neuron | RAG architecture, ML design | ETL, analysis, features | Reports, docstrings |
| Sentinel | Threat modeling, compliance | Code review, IAM | Headers, configs, checklists |
| Forge | — | 5E quality review, refactoring | Score reports, findings |

**Estimated savings: ~60–75% vs. using Opus for everything.**

---

## Usage in Claude Code

### Specific project

```bash
# Activate Maestro as project CLAUDE.md
cp Orchestrator.md ./CLAUDE.md

# Specialist agent for a specific repository
cp Backend-Dev.md ./CLAUDE.md     # backend repo (Stratum)
cp Frontend-Dev.md ./CLAUDE.md    # frontend repo (Facet)
cp Infra-Cloud.md ./CLAUDE.md     # infra repo (Bastion)
```

### Conditional rules by path

```bash
# Backend
.claude/rules/services.md      # paths: ["src/**/*.service.ts"]
.claude/rules/tests.md         # paths: ["**/*.test.ts", "**/*.spec.ts"]

# Frontend
.claude/rules/components.md    # paths: ["src/components/**/*.tsx"]
.claude/rules/styles.md        # paths: ["**/*.css", "**/*.scss"]

# Infra
.claude/rules/terraform.md     # paths: ["**/*.tf", "**/*.tfvars"]
.claude/rules/kubernetes.md    # paths: ["k8s/**/*.yaml"]

# Data
.claude/rules/dbt.md           # paths: ["dbt/**/*.sql", "dbt/**/*.yml"]
.claude/rules/ml.md            # paths: ["ml/**/*.py"]

# Security
.claude/rules/auth.md          # paths: ["src/auth/**", "**/middleware/**"]
```

### Loading hierarchy in Claude Code

```
~/.claude/CLAUDE.md           ← Maestro (global, optional)
./CLAUDE.md                   ← Specialist agent (project)
.claude/rules/*.md            ← Conditional rules by path
```

---

## Workflow

```
Maestro reads progress.md
  └─→ Breaks task into sub-tasks with done criterion
      ├─→ Stratum (APIs, DB, microservices)
      ├─→ Facet (UI, components, a11y)
      ├─→ Bastion (IaC, CI/CD, deploy)
      ├─→ Neuron (pipelines, ML, RAG)
      ├─→ Forge (5E quality review before merge)
      └─→ Sentinel (mandatory review on auth/data/infra)
          ↓
      Each specialist delivers:
        Deliverable + Evidence + State Update
          ↓
      Maestro validates Evidence, updates progress.md
```

---

## Constitution (summary)

1. **Evidence before code** — spec exists before implementing; Evidence is mandatory
2. **Security by default** — Security has veto on any deploy
3. **Tests are not optional** — deliverable without tests = incomplete delivery
4. **Closed scope by default** — each agent does exactly what was delegated
5. **Fail early, fail visibly** — errors go to logs, never silent
6. **System improves every cycle** — hill-climbing via Vault SO (hill, review, guard)

---

## System evolution

Based on the Nexus Agent System and the following sources:
- AiScientist (thin control + thick state, File-as-Bus)
- Agentic Harness Engineering (memory > prompts)
- Multi-Agent Orchestration patterns (Prep Line, Gordon Ramsay)
- RL Conductor (model routing by task type)

Previous version: Professor Fullstack (v1.0) — 4.5 models, no mandatory evidence, educational framing.
