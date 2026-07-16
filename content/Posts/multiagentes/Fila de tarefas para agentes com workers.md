---
title: Fila de tarefas para agentes com workers
draft: false
tags:
  - dev
  - agents
  - queues
socialDescription: Um desenho simples para executar agentes de forma assíncrona usando fila, workers e idempotência.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# TL;DR;

Fila separa pedido de execução. Worker consome tarefa, escolhe agente e grava resultado.

## Modelo de tarefa

```ts
type AgentTask = {
  id: string;
  type: "review" | "summarize" | "monitor";
  input: string;
  createdAt: string;
};
```

## Worker

```ts
while (true) {
  const task = await queue.consume<AgentTask>("agent.tasks");
  const agent = chooseAgent(task.type);

  const result = await runAgent(agent, {
    taskId: task.id,
    input: task.input,
  });

  await results.save({
    taskId: task.id,
    status: "completed",
    output: result.output,
  });
}
```

## Pontos obrigatórios

- `task.id` como chave de idempotência.
- Timeout por tarefa.
- Limite de passos do agente.
- Lista explícita de ferramentas.
- Log de tool calls.
- Fila de erro ou status `failed`.

## Critério

Se a tarefa pode alterar arquivo, banco, deploy ou ticket, ela precisa de permissão explícita e trilha de auditoria.

