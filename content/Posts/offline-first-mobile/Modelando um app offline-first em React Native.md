---
title: Modelando um app offline-first em React Native
draft: false
tags:
  - dev
  - mobile
  - react-native
  - offline-first
socialDescription: Como separar cache, estado local, fila de operações e estado de sincronização em um app mobile.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# TL;DR;

Um app offline-first precisa persistir dados, intenção e metadados de sincronização. O erro comum é tratar o banco local apenas como cache.

## Modelo mental

Separe quatro coisas:

- `entities`: dados que o usuário enxerga, como pedidos, produtos ou clientes.
- `sync_queue`: operações pendentes que ainda precisam chegar ao servidor.
- `sync_state`: cursor, última sincronização, versão do schema e flags de erro.
- `outbox_status`: estado visual para a UI mostrar "pendente", "sincronizando" ou "falhou".

## Schema mínimo

```ts
import * as SQLite from "expo-sqlite";

const db = await SQLite.openDatabaseAsync("app.db");

export async function migrate() {
  await db.execAsync(`
    CREATE TABLE IF NOT EXISTS orders (
      id TEXT PRIMARY KEY NOT NULL,
      customer_name TEXT NOT NULL,
      total_cents INTEGER NOT NULL,
      updated_at TEXT NOT NULL,
      sync_status TEXT NOT NULL DEFAULT 'synced'
    );

    CREATE TABLE IF NOT EXISTS sync_queue (
      id TEXT PRIMARY KEY NOT NULL,
      operation TEXT NOT NULL,
      entity TEXT NOT NULL,
      entity_id TEXT NOT NULL,
      payload_json TEXT NOT NULL,
      attempts INTEGER NOT NULL DEFAULT 0,
      created_at TEXT NOT NULL,
      last_error TEXT
    );
  `);
}
```

`orders` guarda o estado de leitura. `sync_queue` guarda a intenção. Se o usuário cria um pedido offline, o app não depende da chamada HTTP para considerar a ação concluída localmente.

## Criando uma operação local

```ts
import { randomUUID } from "expo-crypto";

type CreateOrderInput = {
  customerName: string;
  totalCents: number;
};

export async function createOrderOffline(input: CreateOrderInput) {
  const id = randomUUID();
  const now = new Date().toISOString();

  await db.withTransactionAsync(async () => {
    await db.runAsync(
      `INSERT INTO orders (id, customer_name, total_cents, updated_at, sync_status)
       VALUES (?, ?, ?, ?, ?)`,
      id,
      input.customerName,
      input.totalCents,
      now,
      "pending"
    );

    await db.runAsync(
      `INSERT INTO sync_queue
       (id, operation, entity, entity_id, payload_json, created_at)
       VALUES (?, ?, ?, ?, ?, ?)`,
      randomUUID(),
      "create",
      "order",
      id,
      JSON.stringify({ id, ...input }),
      now
    );
  });

  return id;
}
```

A transação é o ponto crítico: ou o pedido e a fila entram juntos, ou nada entra. Isso evita uma UI mostrando um pedido que nunca será sincronizado.

## Regra prática

Se a operação muda algo que o usuário considera concluído, grave primeiro no banco local e depois sincronize. Se a operação depende de uma validação remota forte, deixe isso explícito na UI com um estado como "aguardando confirmação".

