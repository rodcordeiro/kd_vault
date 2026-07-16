---
title: DTOs, entidades e fronteiras de arquitetura
draft: false
tags:
  - dev
  - api
  - architecture
socialDescription: Por que entidades de domínio não devem ser o contrato público da API.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# TL;DR;

DTO é contrato. Entidade é modelo de domínio ou persistência. Misturar os dois torna toda refatoração uma quebra de API.

## Entidade

```csharp
public sealed class Customer
{
    public Guid Id { get; private set; }
    public string Name { get; private set; } = "";
    public string DocumentNumber { get; private set; } = "";
    public DateTime CreatedAt { get; private set; }
}
```

Essa classe pode conter dados que não devem sair pela API.

## DTO

```csharp
public sealed record CustomerSummaryDto(
    Guid Id,
    string Name);

public static CustomerSummaryDto ToSummaryDto(Customer customer)
{
    return new CustomerSummaryDto(customer.Id, customer.Name);
}
```

O DTO mostra a intenção do endpoint. Se amanhã a entidade mudar, o contrato público continua estável.

## Fronteiras

- `Api`: HTTP, autenticação, serialização, DTOs.
- `Application`: casos de uso, validação, orquestração.
- `Domain`: regras e invariantes.
- `Infrastructure`: banco, fila, cache, serviços externos.

A regra prática é simples: camada interna não deve depender de detalhe externo.

