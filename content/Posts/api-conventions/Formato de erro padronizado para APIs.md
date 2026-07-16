---
title: Formato de erro padronizado para APIs
draft: false
tags:
  - dev
  - api
  - backend
socialDescription: Como desenhar um corpo de erro previsível para clientes web, mobile e integrações.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# TL;DR;

Erro de API deve ser estável para máquina e legível para humano. Não devolva mensagens diferentes para o mesmo problema.

## Contrato

```json
{
  "code": "VALIDATION_ERROR",
  "message": "The request has invalid fields.",
  "fields": {
    "customerId": ["Customer is required."]
  },
  "traceId": "00-abcd"
}
```

`code` é para o cliente tratar. `message` é genérica. `fields` detalha validação. `traceId` conecta suporte, log e monitoramento.

## Middleware simples

```csharp
app.UseExceptionHandler(errorApp =>
{
    errorApp.Run(async context =>
    {
        var traceId = context.TraceIdentifier;

        context.Response.StatusCode = StatusCodes.Status500InternalServerError;
        context.Response.ContentType = "application/json";

        await context.Response.WriteAsJsonAsync(new
        {
            code = "UNEXPECTED_ERROR",
            message = "Unexpected error.",
            traceId
        });
    });
});
```

Não exponha stack trace para cliente. Stack trace pertence ao log, associado ao `traceId`.

## Códigos bons

- `VALIDATION_ERROR`
- `NOT_FOUND`
- `CONFLICT`
- `UNAUTHORIZED`
- `FORBIDDEN`
- `RATE_LIMITED`
- `UNEXPECTED_ERROR`

Evite códigos específicos demais que vazam implementação interna.

