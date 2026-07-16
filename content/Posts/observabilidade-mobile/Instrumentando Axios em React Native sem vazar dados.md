---
title: Instrumentando Axios em React Native sem vazar dados
draft: false
tags:
  - dev
  - mobile
  - axios
  - sentry
socialDescription: Como registrar falhas HTTP com contexto suficiente e sem capturar payloads sensíveis.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# TL;DR;

Logue método, rota normalizada, status, duração e tipo de erro. Evite payload, token, documento, e-mail, telefone e qualquer identificador de cliente.

## Cliente HTTP com metadata

```ts
import axios from "axios";
import * as Sentry from "@sentry/react-native";

export const api = axios.create({
  baseURL: "https://api.example.com",
  timeout: 15_000,
});

api.interceptors.request.use((config) => {
  config.metadata = { startedAt: Date.now() };
  return config;
});
```

O `metadata` não existe no tipo padrão do Axios. Em TypeScript, adicione uma declaração global:

```ts
declare module "axios" {
  export interface InternalAxiosRequestConfig {
    metadata?: { startedAt: number };
  }
}
```

## Normalizando rotas

```ts
function normalizeUrl(url?: string) {
  if (!url) return "unknown";

  return url
    .replace(/[0-9a-f]{8}-[0-9a-f-]{27,}/gi, ":uuid")
    .replace(/\b\d+\b/g, ":id")
    .split("?")[0];
}
```

`/orders/123/items?token=abc` vira `/orders/:id/items`. Isso preserva a forma da falha sem capturar dados pessoais ou tokens.

## Interceptor de erro

```ts
api.interceptors.response.use(
  (response) => response,
  (error) => {
    const config = error.config;
    const durationMs = config?.metadata?.startedAt
      ? Date.now() - config.metadata.startedAt
      : undefined;

    Sentry.captureException(error, {
      tags: {
        feature: "http",
        method: config?.method?.toUpperCase() ?? "UNKNOWN",
        route: normalizeUrl(config?.url),
        status: String(error.response?.status ?? "network"),
      },
      extra: {
        durationMs,
        timeout: error.code === "ECONNABORTED",
        hasResponse: Boolean(error.response),
      },
    });

    return Promise.reject(error);
  }
);
```

`hasResponse=false` costuma indicar falha de rede, timeout, DNS ou bloqueio antes de receber resposta HTTP. `status=500` já aponta para resposta do servidor.

## O que não capturar

- `Authorization`, cookies e refresh tokens.
- Corpo de request/response.
- Dados pessoais ou financeiros.
- URLs com query string.
- Identificadores reais de cliente, pedido ou usuário.

