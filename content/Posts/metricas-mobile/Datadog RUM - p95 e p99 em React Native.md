---
title: Datadog RUM - p95 e p99 em React Native
draft: false
description: Tutorial para configurar Datadog RUM em React Native e gerar métricas p95 e p99 a partir de eventos.
tags:
  - dev
  - mobile
  - react-native
  - expo
  - datadog
  - observabilidade
  - performance
  - serie
socialDescription: Como enviar eventos RUM, gerar métricas e monitorar p95 e p99 no Datadog para React Native.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# Datadog RUM - p95 e p99 em React Native

No Datadog React Native, o caminho usual não é esperar uma métrica pronta chamada `p95`. O fluxo é instrumentar RUM, enviar eventos com atributos úteis, gerar métricas a partir desses eventos e habilitar percentis.

Série: parte 3 de 6. Anterior: [[Instrumentando métricas no React Native e Expo]]. Próximo: [[Monitorando TTI e FPS em React Native]].

## Instalar o SDK

```bash
npm install @datadog/mobile-react-native
```

Em iOS, instale os pods quando o projeto tiver pasta nativa:

```bash
cd ios
pod install
```

Em Expo, use development build ou EAS Build. Expo Go não é o ambiente certo para validar SDK nativo de observabilidade.

## Inicializar

Exemplo de inicialização:

```ts
import {
  DdSdkReactNative,
  DdSdkReactNativeConfiguration,
  TrackingConsent,
} from "@datadog/mobile-react-native"

export async function initializeDatadog() {
  const config = new DdSdkReactNativeConfiguration(
    "<CLIENT_TOKEN>",
    "<ENVIRONMENT_NAME>",
    "<RUM_APPLICATION_ID>",
    true,
    true,
    true,
  )

  config.site = "US1"
  config.version = "1.0.0"
  config.sessionSamplingRate = 100
  config.nativeCrashReportEnabled = true
  config.nativeViewTracking = true
  config.trackingConsent = TrackingConsent.GRANTED

  await DdSdkReactNative.initialize(config)
}
```

Use variáveis de ambiente e configuração por ambiente para tokens, env e versão. Não grave segredo em código.

## Enviar uma duração customizada

Uma estratégia simples é enviar ações RUM com atributos numéricos.

```ts
import { DdRum, RumActionType } from "@datadog/mobile-react-native"

export function trackTiming(
  name: string,
  durationMs: number,
  context: Record<string, string | number | boolean | undefined>,
) {
  DdRum.addAction(RumActionType.CUSTOM, name, {
    duration_ms: Math.round(durationMs),
    ...context,
  })
}
```

Uso:

```ts
const start = performance.now()

await loadHomeData()

trackTiming("screen_loaded", performance.now() - start, {
  screen: "Home",
  result: "success",
})
```

Depois, no RUM Explorer, o atributo `duration_ms` pode virar uma medida e alimentar uma métrica de distribuição.

## Criar métrica RUM no Datadog

No Datadog:

1. Abra `Digital Experience > Application Management > Generate Metrics`.
2. Clique em `+ New Metric`.
3. Escolha o tipo de evento, como `Actions`, `Views`, `Resources`, `Errors`, `Sessions` ou `Long Tasks`.
4. Filtre o recorte.
5. Escolha o campo numérico, como `@duration_ms`.
6. Configure agrupamentos de baixa cardinalidade.
7. Habilite percentis para a métrica de distribuição.

Exemplo para tela:

```text
@type:action @action.target.name:screen_loaded @screen:Home
```

Campo:

```text
@duration_ms
```

Agrupamentos:

```text
env, app.version, platform, device_model, screen
```

Percentis:

```text
p50, p95, p99
```

## Métrica de endpoint

Para um endpoint crítico:

```ts
trackTiming("resource_loaded", durationMs, {
  endpoint: "/checkout",
  method: "POST",
  status: 200,
  result: "success",
})
```

Filtro:

```text
@type:action @action.target.name:resource_loaded @endpoint:/checkout
```

Agrupamentos úteis:

- `endpoint`;
- `method`;
- `status`;
- `env`;
- `app.version`;
- `platform`.

Se o backend ou a rede oscilarem só em parte da base, p95 e p99 tendem a mostrar antes da média.

## Tags e cardinalidade

Boas dimensões:

- `env`;
- `app.version`;
- `platform`;
- `device_model`;
- `os_version`;
- `screen`;
- `endpoint`;
- `flow`;
- `result`.

Evite:

- `user_id`;
- `session_id`;
- `request_id`;
- token;
- CPF, CNPJ, e-mail ou telefone;
- URL completa com IDs;
- payload de request ou response.

## Validação

Depois de publicar:

1. Faça uma sessão real no app.
2. Abra o RUM Explorer.
3. Procure a ação customizada.
4. Confirme se `duration_ms` aparece como atributo numérico.
5. Crie uma faceta ou medida quando necessário.
6. Gere a métrica.
7. Coloque p95 e p99 em um dashboard.

## Referências

- Datadog React Native Monitoring Setup: https://docs.datadoghq.com/real_user_monitoring/application_monitoring/react_native/setup/
- Datadog Generate Custom Metrics From RUM Events: https://docs.datadoghq.com/real_user_monitoring/platform/generate_metrics/
- Datadog Send RUM Custom Actions: https://docs.datadoghq.com/real_user_monitoring/guide/send-rum-custom-actions/

---

Série: [[index|Métricas p95 e p99 em React Native e Expo]]  
Anterior: [[Instrumentando métricas no React Native e Expo]]  
Próximo: [[Monitorando TTI e FPS em React Native]]
