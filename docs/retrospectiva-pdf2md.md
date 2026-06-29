---
tags: [fullstack-agent, retrospectiva, hill-climbing, licoes, pdf2md]
type: retrospective
system: Fullstack Agent System
version: "2.1.0"
project: pdf2md
cycles: "1–4"
date: 2026-05-29
feeds: agent-memory
---

# Retrospectiva — pdf2md (Ciclos 1–4)

> Documento de aprendizado para o **hill-climbing** do sistema (Constitution #6).
> Não re-lista bugs — isso está em [[code-review-ciclo2]]. Aqui ficam as **lições
> transferíveis** e as **falhas de processo** que o sistema deve internalizar.
> Entradas formatadas como [[_template|agent-memory]] (`DECISION/PATTERN/CONSTRAINT/FAILURE`)
> para propagação direta aos arquivos de `memory/`.

## Contexto

pdf2md exercitou o ciclo completo 4 vezes: spec → código (Python core + SwiftUI)
→ empacotamento (PyInstaller + .app + .dmg) → release público. Stack com
dependências de **binário de sistema** (Tesseract, antiword) e **bridge
Process↔JSON** Swift↔Python. Repo público → segurança máxima.

---

## ⚠️ As duas meta-falhas (o aprendizado central)

### META-FALHA 1 — O sistema não generalizou um fix que já tinha em código
**O mesmo bug-raiz foi shippado DUAS vezes.**

1. v0.1.0: Tesseract "não encontrado" no app empacotado. Causa: binário
   PyInstaller tem **PATH mínimo** (sem `/opt/homebrew/bin/`). Fix: `_configurar_tesseract_cmd()` resolve o path explícito.
2. Word support (`.doc`) foi adicionado depois com `subprocess.run(["antiword", ...])` — **sem** aplicar o fix idêntico que já existia a alguns commits de distância.
3. O usuário bateu no **mesmo bug** com antiword (reportado na v0.2.0).
4. v0.3.0 aplicou o fix idêntico (`_resolver_antiword()`).

**Por que falhou:** Stratum adicionou um novo `subprocess` de binário de sistema
sem buscar fixes análogos no próprio código. Maestro não roteou "nova dependência
externa via subprocess" para uma checagem contra o catálogo de fixes conhecidos.
A solução estava **escrita no repo** e não foi propagada.

**Correção de processo:** toda adição de `subprocess.run([<binário de sistema>])`
dispara `grep subprocess.run core/` + comparação com fixes existentes. Binário de
sistema nunca é chamado por nome cru em código que será congelado.

### META-FALHA 2 — Gate pulado, com auto-engano por "score estimado"
`progress.md` do Ciclo 2 admite textualmente:
> "Gate 2 formal (Forge 5E + Probe scan) não executado — complexidade emergencial
> resolvida inline. Score estimado: ≥85."

**Realidade:** o code-review max-effort posterior achou **15 bugs**, sendo 3
**críticos já em produção** (v0.1.0/v0.2.0): perda de dados (colisão de nome no
batch), deadlock de pipe (app congela), encoding PT-BR (`.doc` quebrado).

"Score estimado ≥85" sem evidência **viola a Constituição** (#1 Evidence before
code, #3 Tests not optional, #5 Fail visibly). "Resolvido inline" ≠ revisado.

**Correção de processo:** gate não é estimável. Forge/Sentinel rodam **antes** do
release, com evidência real (findings + score), nunca "estimado".

### META-LIÇÃO 3 — O code-review pós-hoc teve ROI altíssimo
O review (5 finders + verify + sweep) pegou exatamente o que os gates pularam:
15/15 corrigidos. **Institucionalizar como gate obrigatório pré-release**, não
como faxina posterior.

---

## Lições por agente (memory-ready)

### Bastion — Infra & Empacotamento
- `[2026-05-29] CONSTRAINT` Binário PyInstaller one-file = PATH mínimo (sem `/opt/homebrew/bin/`). **Todo** subprocess de binário de sistema precisa resolução de path explícita. Catálogo: `_configurar_tesseract_cmd`, `_resolver_antiword` (PATH → fallback Homebrew → nome literal).
- `[2026-05-29] PATTERN` DMG: **não** usar `hdiutil create -srcfolder` com symlink `/Applications` — auto-size segue o symlink e estoura ("no space") com binário grande. Usar imagem RW de **tamanho explícito** (`du -sm + margem`) → montar → copiar app + symlink → detach por **device node** com `-force` (detach por mountpoint dá "resource busy") → `convert UDZO`.
- `[2026-05-29] PATTERN` Em binário PyInstaller, usar `ThreadPoolExecutor`, não `ProcessPoolExecutor`: o `spawn` re-executa o binário congelado com flags que o Typer rejeita. fitz/tesseract liberam o GIL em C → threads têm paralelismo real.
- `[2026-05-29] CONSTRAINT` Build reproduzível em script versionado, não comandos one-off. O build manual perdeu reprodutibilidade entre v0.1.0 e v0.2.0; `scripts/build_app.sh` (paths dinâmicos, versão do pyproject) resolveu.

### Facet — Frontend & Swift bridge
- `[2026-05-29] FAILURE` `Process` + `Pipe`: ler `readDataToEndOfFile()` **depois** de `waitUntilExit()`, com stderr nunca drenado, causa **deadlock** — o pipe (~64KB) enche, o filho bloqueia em `write()` e nunca sai. Drenar stdout **e** stderr concorrentemente, em paralelo ao wait.
- `[2026-05-29] FAILURE` Estado de cancelamento com dono dividido → corrida ao reconverter (zera estado da nova execução). **Dono único**: `cancelar()` só termina o processo; a cauda do loop liquida a UI. `guard !estaProcessando` na reentrada.
- `[2026-05-29] PATTERN` Confinamento de path por componente: `path == home || path.hasPrefix(home + "/")`. `hasPrefix(home)` cru deixa `/Users/bob` prefixar `/Users/bobby`.

### Stratum — Backend & core
- `[2026-05-29] FAILURE` Nome de saída por `stem` só → `a.pdf` + `a.docx` colidem em `a.md` → perda de dados silenciosa sob ThreadPool. Desambiguar por extensão, determinístico.
- `[2026-05-29] FAILURE` `subprocess.run(text=True)` decodifica com o locale (UTF-8 no macOS); antiword emite **Latin-1** → `UnicodeDecodeError` em PT-BR (o público-alvo). Capturar **bytes** + decode defensivo (utf-8 → cp1252 → latin-1, que nunca falha).
- `[2026-05-29] FAILURE` Bibliotecas C (MuPDF via pymupdf4llm) escrevem no **fd 1 nativo**; `contextlib.redirect_stdout` (que troca só `sys.stdout`) **não** captura. Para proteger protocolo JSON no stdout, redirecionar no nível de fd (`os.dup2(2, 1)`) ao redor da chamada.
- `[2026-05-29] PATTERN` Eficiência: `pymupdf4llm.to_markdown(str(path), page_chunks=True)` em **uma passada**, não por-página (reabrir+reparsear o PDF por página é O(n)).

### Sentinel — Segurança
- `[2026-05-29] PATTERN` Detecção de traversal por **componente** (`".." in path.parts`), não substring (`".." in str(path)` dá falso positivo em `relatorio..final.pdf`).
- `[2026-05-29] CONSTRAINT` Mensagens de erro não vazam path absoluto do usuário — usar `.name`.
- `[2026-05-29] DECISION` Containment home-only **não** vai no core (quebraria CLI legítimo em `/tmp`, `/Volumes`). Confinamento home é responsabilidade da **camada GUI** (bridge), não do core. Threat model por camada, não global.

### Neuron — Data & OCR
- `[2026-05-29] PATTERN` Memoizar `verificar_tesseract`/`_configurar_tesseract_cmd` (`lru_cache`): um subprocess `tesseract --version` por página/imagem é desperdício; a disponibilidade não muda durante a execução.
- `[2026-05-29] FAILURE` `pytesseract` decodifica o stderr do Tesseract como UTF-8 → `UnicodeDecodeError` em stderr não-UTF8 (imagens em branco). **Mesma classe** do bug do antiword — dependência externa emitindo não-UTF8. Capturar bytes. (Pendente: hardening em `image_converter.py`.)

### Maestro — Orquestração
- `[2026-05-29] DECISION` Rotear "adição de subprocess externo" → checagem obrigatória contra o catálogo de fixes (anti META-FALHA 1).
- `[2026-05-29] CONSTRAINT` Gate exige **evidência**, nunca "score estimado" (anti META-FALHA 2).
- `[2026-05-29] DECISION` Code-review max-effort (5 finders + verify + sweep) como **gate pré-release obrigatório**.
- `[2026-05-29] PATTERN` Distinguir falha de teste **ambiental** de regressão: validar contra o base limpo (`git stash`) antes de culpar a mudança. 5 falhas de OCR aqui eram bug do pytesseract no ambiente local, não regressão — confirmado no base.

---

## Ações de propagação (próximo hill-climb)

- [x] Append das entradas acima em `memory/bastion.md`, `memory/stratum.md`, `memory/facet.md`, `memory/sentinel.md`, `memory/neuron.md`, `memory/maestro.md` (criados 2026-05-29 a partir de [[_template]]).
- [ ] Adicionar a `docs/standards-anti-patterns.md`: o catálogo **subprocess-de-binário-de-sistema → resolução de path** e o **idioma hdiutil** (RW explícito + device-node detach).
- [ ] Adicionar ao checklist do Maestro: "nova dep externa via subprocess → grep + catálogo de fixes" e "gate com evidência, não estimativa".
- [ ] Promover o code-review max-effort a etapa fixa do pipeline de release.

## Meta

A maior lição não é técnica — é que o sistema **tinha a solução em código e não a
reusou**, e **pulou um gate se auto-avaliando**. Hill-climbing real exige: (1)
memória que propaga fixes entre dependências da mesma classe; (2) gates com
evidência, imunes a "estimativa". As duas falhas custaram 2 releases quebrados
(v0.1.0 Tesseract, v0.2.0 antiword) que um único princípio bem aplicado teria
evitado.

Ver também: [[code-review-ciclo2]] · [[progress]] · [[README|Fullstack Agent System]]
