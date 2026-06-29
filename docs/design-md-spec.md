---
title: "DESIGN.md — spec de identidade visual p/ agentes"
type: reference
for: facet (frontend-dev), pixel
source: https://github.com/google-labs-code/design.md
created: 2026-06-27
---

# DESIGN.md — identidade visual persistente

Formato p/ dar a Facet/pixel entendimento estruturado e persistente de um
design system (em vez de redescrever a cada tarefa). Análogo ao Constitution.md,
mas p/ o visual.

## Quando usar
Projeto frontend novo → Facet gera/lê um `DESIGN.md` na raiz com:
tokens (cor/tipo/espaço), componentes-base, tom/voz, regras de a11y.

## Por que
- Consistência visual entre sessões (sem drift de estilo).
- Mesma filosofia File-as-Bus: estado de design vive em arquivo, não no chat.

Spec/exemplos: https://github.com/google-labs-code/design.md