---
title: "Golden Example — Go Backend Clean Architecture"
type: reference
for: stratum (backend-dev)
source: https://github.com/amitshekhariitbhu/go-backend-clean-architecture
created: 2026-06-27
---

# Golden Example — Go Backend Clean Architecture

Exemplar concreto para Stratum quando o stack for Go.

## Camadas
Router → Controller → Usecase → Repository → Domain
(separação estrita; Domain no centro, sem dependência de fora)

## Stack de referência
- Gin (HTTP framework)
- MongoDB (mongo-go-driver)
- JWT (access + refresh token) como middleware
- Viper (config via .env)
- Testify + Mockery (test + mocks gerados)
- Docker

## Padrões a copiar
- JWT middleware como camada separada (alinha com T2/T3: pre-request token na collection)
- Mockery para gerar mocks de Repository → testes de Usecase isolados
- Domain layer sem imports de framework (testável puro)

## Quando usar
Stratum recebe tarefa backend em Go → seguir esta estrutura de camadas.
Outros stacks (Node/Python/Java) → manter Clean Arch equivalente, este é o mapa Go.