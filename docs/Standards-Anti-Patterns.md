---
title: "standards-anti-patterns"
version: 2.2.0
updated: 2026-05-30
---

# standards-anti-patterns — Fullstack Agent System

## Universal standards (all agents)

### Required output format

Every specialist deliverable must have three sections:

```markdown
## Deliverable
[Complete code, config, or artifact]

## What & Why
[Each logical block → one-line reason. Non-obvious decisions explained.]

## Evidence
[Passing tests, scan output, execution log, metrics]

## State Update
[What changed: files, dependencies, progress.md state]
```

Deliverable without Evidence = incomplete delivery. Maestro rejects and re-delegates.

### File-as-Bus

Project state lives in files, not in conversation history:
- `docs/progress.md` — current state, current cycle, blockers
- `workspace/` — generated artifacts (code, IaC, notebooks)
- `docs/logs/` — execution evidence by domain

Agents read what they need, write where their frontmatter specifies.

### Secrets

Never in code. Never in logs. Never in API responses.
Always: environment variables, AWS Secrets Manager, HashiCorp Vault, SOPS.

---

## Backend

### Required Standards
- Input validation at every external boundary (Zod, Pydantic, Joi)
- Semantic HTTP: 200, 201, 400, 401, 403, 404, 422, 500
- Structured JSON logs with appropriate level
- Parameterized queries — no SQL string concatenation
- Migrations with `up()` and `down()`
- OpenAPI/Swagger spec generated for every API
- Runnable Postman/Insomnia collection in repo
- Pre-request script injecting auth token on EVERY collection route
- README with build + run-mock script

### Anti-patterns
- `console.log` in production
- Collection route without auth pre-request script (consumer hits 401, blames API)
- Endpoint shipped without OpenAPI entry or example request
- `any` in TypeScript without justification
- Hardcoded IPs, secrets, connection strings
- Unaddressed N+1
- Query without index on filtered field

### subprocess — system binary catalog

Frozen binaries (PyInstaller, Nuitka, cx_Freeze) run with minimal PATH — `/opt/homebrew/bin` not included. Any `subprocess.run(["<bin>"])` by name fails in the packaged app even if installed.

**Pattern (mandatory for every system binary):**
```python
import shutil
from pathlib import Path

_HOMEBREW_PATHS_MACOS = [
    "/opt/homebrew/bin/<bin>",   # Apple Silicon
    "/usr/local/bin/<bin>",      # Intel
    "/usr/bin/<bin>",
]

def _resolver_<bin>() -> str:
    if cmd := shutil.which("<bin>"):
        return cmd
    for p in _HOMEBREW_PATHS_MACOS:
        if Path(p).exists():
            return p
    return "<bin>"  # let subprocess fail with a clear error

# usage:
subprocess.run([_resolver_<bin>(), ...], capture_output=True, timeout=30)
```

**Catalog — resolvers already implemented in pdf2md (copy pattern):**
- `_configurar_tesseract_cmd()` — `core/image_converter.py`
- `_resolver_antiword()` — `core/doc_converter.py`

**Anti-patterns:**
- `subprocess.run(["tesseract", ...])` — breaks in .app
- `subprocess.run(["antiword", ...])` — breaks in .app
- `text=True` with tools that emit Latin-1 (antiword) — `UnicodeDecodeError` on PT-BR. Use `capture_output=True` (bytes) + defensive decode (utf-8 → cp1252 → latin-1)
- Reading subprocess output stdout after `waitUntilExit()` (Swift) with stderr undrained — pipe deadlock at 64KB

---

## Frontend

### Required Standards
- Strict TypeScript — no implicit `any`
- Semantic HTML — `<header>`, `<nav>`, `<main>`, `<article>`
- Touch targets ≥ 44×44px
- `alt` on every `<img>`
- `prefers-reduced-motion` respected

### Anti-patterns
- `!important` without a comment
- `<div>` where semantic element exists
- Inline styles for logic that belongs in CSS
- Click events on `<span>` or `<div>` without `role`
- Body text < 16px

---

## Bastion

### Required Standards
- Mandatory tags: `Project`, `Environment`, `Owner`, `CostCenter`
- IAM least privilege — no `*` permission without justification
- Remote state in Terraform (S3 + DynamoDB locking)
- Multi-AZ for production
- Resource limits on every Kubernetes container

### Anti-patterns
- Hardcoded credentials in IaC
- Infrastructure without observability alerts
- `terraform apply` without a reviewed `plan`
- Production resources without configured backup
- Security group with `0.0.0.0/0` inbound without justification

### macOS packaging — hdiutil DMG

**Pattern (3 traps — all must be handled):**

```bash
# 1. Compute explicit size — auto-size follows /Applications symlink and overflows
TAM_MB=$(du -sm "$APP" | awk '{print $1 + 40}')

# 2. Create RW image WITHOUT -format (requires -srcfolder); size flag only
hdiutil create -volname "$VOLNAME" -fs HFS+ -size "${TAM_MB}m" -ov "$DMG_RW" -quiet

# 3. Mount and capture device node — detach by mountpoint gives "resource busy"
DEV=$(hdiutil attach "$DMG_RW" -nobrowse -noverify \
      | grep -E '^/dev/' | head -1 | awk '{print $1}')
cp -RP "$APP" "/Volumes/$VOLNAME/"
ln -s /Applications "/Volumes/$VOLNAME/Applications"
sync

# 4. Detach by device node + -force (not by mountpoint)
hdiutil detach "$DEV" -force -quiet

# 5. Convert to compressed UDZO
hdiutil convert "$DMG_RW" -format UDZO -o "$DMG_OUT" -quiet
```

**Anti-patterns:**
- `hdiutil create -srcfolder <dir_with_symlink>` — auto-size follows `/Applications`, overflow
- `-format UDRW` without `-srcfolder` — flag requires source
- `hdiutil detach "$MOUNTPOINT"` — "resource busy" after `cp`
- `cp -R` (without `-P`) into DMG staging — follows symlinks, copies entire `/Applications`

**Reference:** `scripts/build_app.sh` in mchlcs/pdf2md

---

## Neuron

### Required Standards
- Idempotent pipelines — re-executable without duplicating data
- Input and output schema documented
- Model and dataset versioning
- Quality checks (Great Expectations / Soda) on input and output
- Traceable data lineage

### Anti-patterns
- Dataset overwritten without backup
- Model in production without baseline metrics
- PII without anonymization (privacy compliance)
- Pipeline that fails silently
- RAG without retrieval quality validation

---

## Sentinel

### Required Standards
- PASS/FAIL with evidence — never PASS with informal caveat
- CVSS v3.1 on every reported vulnerability
- ADR created for every new security architecture decision
- Dependency scanning in CI/CD

### Anti-patterns
- Approving with "can fix later" on High or Critical severity
- Disabling security controls as a workaround
- Review without scan or checklist executed
- Secrets in any versioned file — no exceptions
