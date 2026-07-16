---
title: Instrumentando métricas no React Native e Expo
draft: false
description: Tutorial para medir duração de telas, ações, recursos e jornadas críticas em React Native e Expo.
tags:
  - dev
  - mobile
  - react-native
  - expo
  - performance
  - observabilidade
  - serie
socialDescription: Como instrumentar métricas de duração em React Native e Expo para gerar p95 e p99.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# Instrumentando métricas no React Native e Expo

Antes de ter p95 e p99, você precisa gerar medições consistentes. A unidade mais comum é duração em milissegundos.

Série: parte 2 de 6. Anterior: [[P95 e p99 - o que medir em apps mobile]]. Próximo: [[Datadog RUM - p95 e p99 em React Native]].

## O padrão mínimo

Toda métrica de duração deve ter:

- nome estável;
- início;
- fim;
- duração em `ms`;
- resultado: `success`, `error`, `cancelled` ou `timeout`;
- contexto com baixa cardinalidade.

Exemplo de evento:

```ts
{
  metric: "screen_load_ms",
  value: 1840,
  screen: "Home",
  result: "success",
  platform: "android",
  app_version: "1.8.0"
}
```

## Helper local

Comece com um helper pequeno. Depois conecte esse helper ao Datadog, Firebase, Sentry, OpenTelemetry ou outro destino.

```ts
type MetricContext = Record<string, string | number | boolean | undefined>

type MetricSink = {
  timing(name: string, valueMs: number, context?: MetricContext): void
}

export const metrics: MetricSink = {
  timing(name, valueMs, context) {
    console.log("[metric]", name, valueMs, context)
  },
}

export async function measureAsync<T>(
  name: string,
  context: MetricContext,
  operation: () => Promise<T>,
): Promise<T> {
  const start = performance.now()

  try {
    const result = await operation()
    metrics.timing(name, performance.now() - start, { ...context, result: "success" })
    return result
  } catch (error) {
    metrics.timing(name, performance.now() - start, { ...context, result: "error" })
    throw error
  }
}
```

Esse helper evita espalhar cálculo de tempo pelo app e padroniza o envio.

## Medir carregamento de tela

Meça quando a tela ficou útil, não apenas quando o componente montou.

```tsx
import { useEffect, useRef } from "react"
import { metrics } from "./metrics"

type HomeScreenProps = {
  isReady: boolean
}

export function HomeScreen({ isReady }: HomeScreenProps) {
  const startRef = useRef(performance.now())
  const sentRef = useRef(false)

  useEffect(() => {
    if (!isReady || sentRef.current) return

    sentRef.current = true
    metrics.timing("screen_load_ms", performance.now() - startRef.current, {
      screen: "Home",
      result: "success",
    })
  }, [isReady])

  return null
}
```

`isReady` deve representar conteúdo útil: dados carregados, skeleton removido, botão principal habilitado ou estado final da tela.

## Medir uma jornada

Para login, checkout ou sincronização:

```ts
await measureAsync(
  "flow_duration_ms",
  {
    flow: "login",
    screen: "Login",
  },
  async () => {
    await authService.login(credentials)
    await bootstrapUser()
    router.replace("/home")
  },
)
```

Esse evento permite responder se a jornada ficou lenta e se a lentidão está concentrada em uma versão.

## Medir recurso de rede crítico

Se você controla o client HTTP, instrumente em um ponto central:

```ts
export async function request<T>(input: RequestInfo, init?: RequestInit): Promise<T> {
  const start = performance.now()
  const endpoint = normalizeEndpoint(input)

  try {
    const response = await fetch(input, init)

    metrics.timing("resource_duration_ms", performance.now() - start, {
      endpoint,
      method: init?.method ?? "GET",
      status: response.status,
      result: response.ok ? "success" : "error",
    })

    return response.json() as Promise<T>
  } catch (error) {
    metrics.timing("resource_duration_ms", performance.now() - start, {
      endpoint,
      method: init?.method ?? "GET",
      result: "error",
    })

    throw error
  }
}
```

`normalizeEndpoint` deve remover IDs e query strings variáveis. Prefira `/orders/:id` a `/orders/123456?token=...`.

## Expo: cuidado com o ambiente

Em Expo, métricas de JavaScript podem ser coletadas em desenvolvimento, mas decisões de performance precisam de build próximo de produção.

Use:

```bash
npx expo start --no-dev --minify
```

Para medições mais confiáveis, use EAS Build ou development build com as mesmas configurações relevantes de produção. Expo Go não é bom alvo para validar SDKs nativos, startup real ou performance final.

## Checklist

- [ ] Nomear métricas com unidade: `_ms`, `_count`, `_rate`.
- [ ] Enviar resultado da operação.
- [ ] Normalizar endpoints.
- [ ] Evitar IDs e payloads sensíveis.
- [ ] Medir tela útil, não apenas mount.
- [ ] Separar cold start de warm/hot start.
- [ ] Validar em device real e build release-like.

## Resumo

P95 e p99 dependem de eventos consistentes. Centralize a instrumentação, envie duração em milissegundos e preserve contexto suficiente para quebrar por tela, versão, plataforma, aparelho e resultado.

---

Série: [[index|Métricas p95 e p99 em React Native e Expo]]  
Anterior: [[P95 e p99 - o que medir em apps mobile]]  
Próximo: [[Datadog RUM - p95 e p99 em React Native]]
