---
title: Processando webhooks com fila e retry
draft: false
tags:
  - dev
  - backend
  - webhooks
  - queue
socialDescription: Um desenho simples para receber webhook rápido, enfileirar e processar com retry controlado.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# TL;DR;

Endpoint de webhook deve validar, persistir e responder rápido. O processamento pesado fica fora da requisição.

## Endpoint mínimo

```csharp
app.MapPost("/webhooks/orders", async (
    WebhookEnvelope envelope,
    IWebhookInbox inbox,
    CancellationToken ct) =>
{
    if (string.IsNullOrWhiteSpace(envelope.MessageId))
        return Results.BadRequest(new { error = "missing_message_id" });

    await inbox.SaveAsync(envelope, ct);

    return Results.Accepted();
});
```

`202 Accepted` comunica que o evento foi aceito para processamento, não que a regra de negócio terminou.

## Worker

```csharp
public sealed class WebhookWorker : BackgroundService
{
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            var item = await _inbox.GetNextAsync(stoppingToken);

            if (item is null)
            {
                await Task.Delay(TimeSpan.FromSeconds(5), stoppingToken);
                continue;
            }

            await _processor.ProcessAsync(item, stoppingToken);
        }
    }
}
```

Em produção, prefira uma fila externa quando houver múltiplas instâncias ou necessidade de throughput alto. O padrão conceitual continua igual: receber rápido, processar de forma controlada e registrar resultado.

## Retry com classificação

- `400`: payload inválido, não adianta retry automático.
- `401/403`: erro de credencial, retry só depois de correção operacional.
- `409`: conflito esperado, tratar com idempotência.
- `429/5xx`: candidato a retry com backoff.

