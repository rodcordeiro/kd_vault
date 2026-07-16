---
title: Idempotência em integrações backend
draft: false
tags:
  - dev
  - backend
  - integrations
socialDescription: Como evitar duplicidade em integrações usando chave de idempotência e registro de processamento.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# TL;DR;

Toda integração de escrita deve responder bem a reenvio. A forma mais simples é persistir uma chave de idempotência por operação.

## Tabela de controle

```sql
CREATE TABLE integration_messages (
  id TEXT PRIMARY KEY,
  source TEXT NOT NULL,
  message_key TEXT NOT NULL,
  status TEXT NOT NULL,
  received_at TEXT NOT NULL,
  processed_at TEXT,
  error TEXT,
  UNIQUE(source, message_key)
);
```

`message_key` pode vir do parceiro ou ser calculada com hash de campos estáveis do evento. Evite usar timestamp como chave.

## Fluxo

```csharp
public async Task<Result> ProcessAsync(IntegrationEvent input, CancellationToken ct)
{
    var alreadyProcessed = await _messages.ExistsAsync(
        input.Source,
        input.MessageKey,
        ct);

    if (alreadyProcessed)
        return Result.Success("duplicate_ignored");

    await _messages.RegisterReceivedAsync(input, ct);

    try
    {
        await _handler.HandleAsync(input, ct);
        await _messages.MarkProcessedAsync(input.Source, input.MessageKey, ct);
        return Result.Success("processed");
    }
    catch (Exception ex)
    {
        await _messages.MarkFailedAsync(input.Source, input.MessageKey, ex.Message, ct);
        throw;
    }
}
```

O importante é gravar recebimento antes de executar efeito colateral. Em sistemas concorrentes, a restrição `UNIQUE` precisa ser a defesa final contra duplicidade.

## Regra prática

Se a operação cria pedido, pagamento, nota, movimentação ou qualquer evento financeiro, idempotência não é opcional.

