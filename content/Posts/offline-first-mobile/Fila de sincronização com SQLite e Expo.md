---
title: Fila de sincronização com SQLite e Expo
draft: false
tags:
  - dev
  - mobile
  - expo
  - sqlite
socialDescription: Um worker simples para enviar operações pendentes com backoff, transação e atualização de status.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# TL;DR;

Uma fila de sincronização precisa ser idempotente, reprocessável e observável. O objetivo não é "tentar de novo para sempre"; é saber o que falhou e por quê.

## Buscando o próximo item

```ts
type QueueItem = {
  id: string;
  operation: "create" | "update" | "delete";
  entity: string;
  entity_id: string;
  payload_json: string;
  attempts: number;
};

async function getNextQueueItem() {
  return db.getFirstAsync<QueueItem>(
    `SELECT *
     FROM sync_queue
     WHERE attempts < 5
     ORDER BY created_at
     LIMIT 1`
  );
}
```

Ordene por criação para preservar intenção do usuário. Limite tentativas para impedir loop silencioso.

## Enviando com idempotência

```ts
async function sendQueueItem(item: QueueItem) {
  const response = await fetch(`https://api.example.com/sync/${item.entity}`, {
    method: "POST",
    headers: {
      "content-type": "application/json",
      "idempotency-key": item.id,
    },
    body: item.payload_json,
  });

  if (!response.ok) {
    const body = await response.text();
    throw new Error(`Sync failed: ${response.status} ${body}`);
  }
}
```

A chave de idempotência permite repetir a mesma operação sem duplicar registros no backend. O servidor deve persistir o resultado associado a essa chave por uma janela de tempo.

## Worker simples

```ts
export async function syncOnce() {
  const item = await getNextQueueItem();
  if (!item) return { synced: 0 };

  try {
    await sendQueueItem(item);

    await db.withTransactionAsync(async () => {
      await db.runAsync("DELETE FROM sync_queue WHERE id = ?", item.id);
      await db.runAsync(
        "UPDATE orders SET sync_status = ? WHERE id = ?",
        "synced",
        item.entity_id
      );
    });

    return { synced: 1 };
  } catch (error) {
    await db.runAsync(
      `UPDATE sync_queue
       SET attempts = attempts + 1, last_error = ?
       WHERE id = ?`,
      error instanceof Error ? error.message : "unknown error",
      item.id
    );

    return { synced: 0 };
  }
}
```

Esse worker é pequeno de propósito. Em produção, execute-o quando o app abre, quando a conectividade volta e em intervalos controlados enquanto a fila existir.

## Checklist de produção

- Use `idempotency-key` em operações de escrita.
- Grave `last_error` para suporte e diagnóstico.
- Tenha uma tela ou badge para pendências.
- Pare depois de algumas tentativas e exija ação manual quando fizer sentido.
- Meça tempo de fila, quantidade de pendências e taxa de falha.

