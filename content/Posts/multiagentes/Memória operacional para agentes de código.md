---
title: Memória operacional para agentes de código
draft: false
tags:
  - dev
  - agents
  - knowledge-management
socialDescription: O que guardar como contexto para agentes sem transformar memória em lixeira de logs.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# TL;DR;

Memória boa para agente é curta, verificável e operacional. Ela diz onde mexer, como validar e quais armadilhas evitar.

## Formato útil

```md
# Projeto exemplo

## Comandos
- Testes: `pnpm test`
- Build: `pnpm build`

## Convenções
- Imports públicos saem de `src/index.ts`.
- Não usar aliases do app dentro da lib.

## Validação
- Rodar `git diff --check`.
- Conferir busca por termos sensíveis antes de publicar posts.
```

## O que guardar

- Comandos reais de validação.
- Estrutura do projeto.
- Decisões arquiteturais.
- Padrões de teste.
- Restrições de segurança.
- Fluxos de deploy.

## O que não guardar

- Segredos.
- Logs brutos.
- Dados de clientes.
- Dumps extensos.
- Opiniões sem evidência.

## Critério de frescor

Toda memória operacional deveria responder: quando foi observada, em qual projeto, como validar se ainda é verdade e qual comando confirma.

