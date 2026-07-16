---
title: Validação e Result Pattern em APIs .NET
draft: false
tags:
  - dev
  - dotnet
  - api
socialDescription: Como retornar erros de domínio de forma explícita sem transformar fluxo esperado em exceção.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# TL;DR;

Use exceção para falha inesperada. Use `Result` para validação e regra de negócio prevista.

## Tipo Result simples

```csharp
public sealed record Result<T>(
    bool IsSuccess,
    T? Value,
    string? Error,
    string? Code)
{
    public static Result<T> Success(T value) => new(true, value, null, null);
    public static Result<T> Failure(string error, string code) => new(false, default, error, code);
}
```

Esse tipo força o chamador a olhar para sucesso e falha.

## Serviço

```csharp
public async Task<Result<OrderDto>> CreateAsync(
    CreateOrderRequest request,
    CancellationToken ct)
{
    if (request.Items.Count == 0)
        return Result<OrderDto>.Failure("Order must have items.", "EMPTY_ORDER");

    var customer = await _customers.GetByIdAsync(request.CustomerId, ct);
    if (customer is null)
        return Result<OrderDto>.Failure("Customer not found.", "CUSTOMER_NOT_FOUND");

    var order = Order.Create(customer.Id, request.Items);
    await _orders.AddAsync(order, ct);

    return Result<OrderDto>.Success(OrderDto.From(order));
}
```

O serviço descreve decisões de negócio sem depender de `BadRequest`, `NotFound` ou detalhes HTTP.

## Endpoint

```csharp
app.MapPost("/orders", async (
    CreateOrderRequest request,
    OrderService service,
    CancellationToken ct) =>
{
    var result = await service.CreateAsync(request, ct);

    return result.IsSuccess
        ? Results.Created($"/orders/{result.Value!.Id}", result.Value)
        : Results.BadRequest(new { error = result.Error, code = result.Code });
});
```

Em projetos maiores, crie um mapeador central de `Result` para HTTP para evitar repetição.

