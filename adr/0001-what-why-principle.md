---
title: "ADR 0001 — Princípio What & Why"
status: accepted
date: 2026-06-27
deciders: [human, Maestro]
---

# ADR 0001 — Princípio "What & Why" obrigatório

## Contexto
Insight externo (Ronconi): após gerar código, a IA deve explicar o que fez,
mapeando cada bloco lógico à sua razão. Reduz erro Karpathy (intenção implícita)
e facilita revisão por humano/QA.

## Decisão
Adicionar 7º Princípio Fundamental à Constitution: todo deliverable de código
inclui seção "What & Why". Estender Output Format dos agentes de execução.

## Consequências
- Todo output de Stratum/Facet/Neuron ganha seção What & Why.
- Forge passa a marcar como finding a ausência da seção.
- Custo marginal de tokens (poucas linhas por deliverable).

## Status
Aprovado por revisão humana em 2026-06-27.