---
title: Parâmetros e validação de tela no Expo Router
draft: false
tags:
  - dev
  - mobile
  - expo-router
  - typescript
socialDescription: Como validar parâmetros de rota e tratar estados inválidos de forma previsível.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# TL;DR;

Parâmetro de rota é entrada externa. Valide antes de usar em query, request ou acesso local.

## Hook de parâmetro

```tsx
import { useLocalSearchParams } from "expo-router";

export function useRequiredStringParam(name: string) {
  const params = useLocalSearchParams();
  const value = params[name];

  if (typeof value !== "string" || value.trim().length === 0) {
    return { ok: false as const, value: null };
  }

  return { ok: true as const, value };
}
```

Esse hook evita que cada tela invente uma validação diferente.

## Tela de detalhe

```tsx
export default function OrderDetailScreen() {
  const orderId = useRequiredStringParam("id");

  if (!orderId.ok) {
    return <InvalidRouteState message="Pedido inválido." />;
  }

  return <OrderDetail orderId={orderId.value} />;
}
```

O usuário vê um estado controlado. O app não dispara request com `undefined`, array ou string vazia.

## Quando usar params

Use params para identificadores e filtros pequenos. Não use params para payloads grandes, dados sensíveis ou objetos que podem ficar desatualizados.

