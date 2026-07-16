---
title: Breadcrumbs e eventos úteis para depurar apps mobile
draft: false
tags:
  - dev
  - mobile
  - observability
  - sentry
socialDescription: Como escolher breadcrumbs que contam a história de uma falha mobile sem poluir o monitoramento.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# TL;DR;

Breadcrumb bom explica o caminho até a falha. Breadcrumb ruim copia log de debug para produção.

## Helper pequeno

```ts
import * as Sentry from "@sentry/react-native";

type BreadcrumbData = Record<string, string | number | boolean | undefined>;

export function addAppBreadcrumb(
  message: string,
  data: BreadcrumbData = {},
  category = "app"
) {
  Sentry.addBreadcrumb({
    category,
    message,
    level: "info",
    data: removeEmpty(data),
  });
}

function removeEmpty(data: BreadcrumbData) {
  return Object.fromEntries(
    Object.entries(data).filter(([, value]) => value !== undefined)
  );
}
```

O helper limita o formato e reduz a chance de alguém registrar objetos grandes ou dados sensíveis por acidente.

## Exemplo em navegação

```ts
addAppBreadcrumb("screen_view", {
  screen: "OrderList",
  source: "bottom_tab",
});
```

## Exemplo em sincronização

```ts
addAppBreadcrumb(
  "sync_finished",
  {
    pendingBefore: 12,
    synced: 10,
    failed: 2,
    durationMs: 2450,
  },
  "sync"
);
```

Esse breadcrumb ajuda a responder se o erro aconteceu antes, durante ou depois da sincronização.

## Exemplo em feature flag

```ts
Sentry.setContext("feature_flags", {
  newCheckout: true,
  offlineMode: true,
});
```

Use contexto para estado relativamente estável da sessão. Use breadcrumb para eventos ordenados no tempo.

## Critério editorial

Antes de adicionar um breadcrumb, pergunte: "isso ajudaria alguém que não estava comigo no momento da falha?". Se a resposta for não, mantenha fora da telemetria de produção.

