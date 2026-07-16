---
title: Modelo agente e orquestrador - uma separação prática
draft: false
tags:
  - dev
  - agents
  - ai
socialDescription: Como separar provedor de modelo, agente, orquestrador e ferramentas em uma arquitetura simples.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# TL;DR;

O modelo gera respostas. O agente usa o modelo com uma função específica. O orquestrador escolhe agentes, controla permissões e executa ferramentas.

## Separação

```txt
Orquestrador
  -> escolhe agente
  -> chama modelo
  -> valida tool call
  -> executa ferramenta
  -> salva resultado
```

## Definição de agente

```ts
type AgentDefinition = {
  name: string;
  model: string;
  systemPrompt: string;
  tools: string[];
  maxSteps: number;
};

const codeReviewAgent: AgentDefinition = {
  name: "code-review",
  model: "local-or-remote-coder",
  systemPrompt: "Revise mudanças de código e priorize bugs reais.",
  tools: ["read_file", "search", "run_tests"],
  maxSteps: 8,
};
```

O agente não precisa ter um modelo exclusivo. Vários agentes podem chamar o mesmo provedor com prompts e ferramentas diferentes.

## Regra prática

Não comece por "quantos agentes?". Comece por "quais tarefas precisam de papéis, permissões e critérios diferentes?".

