---
title: Organizando rotas com Expo Router
draft: false
tags:
  - dev
  - mobile
  - expo-router
socialDescription: Um desenho simples para tabs, stack autenticada e telas de detalhe no Expo Router.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# TL;DR;

Agrupe por fluxo: público, autenticado, abas e modais. Evite deixar regra de autenticação espalhada em telas.

## Estrutura

```txt
app/
  _layout.tsx
  (public)/
    login.tsx
  (app)/
    _layout.tsx
    (tabs)/
      orders.tsx
      customers.tsx
      settings.tsx
    orders/
      [id].tsx
  modal/
    sync-status.tsx
```

Grupos entre parênteses organizam sem entrar na URL. Isso permite separar layout e proteção por contexto.

## Layout autenticado

```tsx
import { Redirect, Stack } from "expo-router";
import { useSession } from "@/features/auth/useSession";

export default function AppLayout() {
  const { session, isLoading } = useSession();

  if (isLoading) return null;
  if (!session) return <Redirect href="/login" />;

  return <Stack screenOptions={{ headerShown: false }} />;
}
```

A checagem fica no layout. As telas internas não precisam repetir `if (!session)`.

## Navegação explícita

```tsx
import { router } from "expo-router";

function openOrder(id: string) {
  router.push({ pathname: "/orders/[id]", params: { id } });
}
```

Prefira passar o mínimo de dados por parâmetro. A tela de detalhe deve carregar o recurso pelo `id` ou pelo cache local.

