---
title: EF Core sem surpresas em consultas de leitura
draft: false
tags:
  - dev
  - dotnet
  - ef-core
socialDescription: Padrões simples para evitar overfetching, tracking desnecessário e consultas frágeis no EF Core.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# TL;DR;

Para leitura, projete direto para DTO, use `AsNoTracking()` e limite colunas. Entidade completa é para regra de negócio, não para toda consulta.

## Consulta ruim

```csharp
var orders = await _db.Orders
    .Include(x => x.Items)
    .ThenInclude(x => x.Product)
    .ToListAsync(ct);
```

Essa consulta pode trazer dados demais, ativar tracking sem necessidade e crescer sem perceber.

## Consulta de leitura

```csharp
public async Task<IReadOnlyList<OrderListItemDto>> ListAsync(
    int page,
    int pageSize,
    CancellationToken ct)
{
    return await _db.Orders
        .AsNoTracking()
        .OrderByDescending(x => x.CreatedAt)
        .Skip((page - 1) * pageSize)
        .Take(pageSize)
        .Select(x => new OrderListItemDto(
            x.Id,
            x.CustomerName,
            x.TotalCents,
            x.Status,
            x.CreatedAt))
        .ToListAsync(ct);
}
```

O SQL gerado fica menor e a intenção da tela aparece no código.

## Quando usar Include

Use `Include` quando a regra precisa de grafo de entidade. Para tela, relatório e endpoint de listagem, comece com projeção.

## Teste útil

```csharp
[Fact]
public async Task ListAsync_ReturnsOnlyRequestedPage()
{
    var result = await _repository.ListAsync(page: 2, pageSize: 10, CancellationToken.None);

    Assert.Equal(10, result.Count);
}
```

Teste de paginação parece simples, mas evita endpoints que trazem tabela inteira por acidente.

