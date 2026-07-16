---
title: Monitorando TTI e FPS em React Native
draft: false
description: Tutorial para medir TTI, FPS, JS thread e UI thread em aplicações React Native e Expo.
tags:
  - dev
  - mobile
  - react-native
  - expo
  - performance
  - observabilidade
  - serie
socialDescription: Como medir startup, TTI, FPS e quedas de frame em React Native e Expo.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# Monitorando TTI e FPS em React Native

P95 e p99 de tela e API mostram latência. Para entender sensação de fluidez, monitore também startup, TTI e FPS.

Série: parte 4 de 6. Anterior: [[Datadog RUM - p95 e p99 em React Native]]. Próximo: [[Dashboards e alertas para p95 e p99 mobile]].

## Métricas principais

- `app_start_tti_ms`: tempo do início do app até a primeira tela útil ficar interativa;
- `screen_interactive_ms`: tempo até a tela atual ficar interativa;
- `js_fps`: frames por segundo percebidos pela thread JavaScript;
- `ui_fps`: frames por segundo da thread nativa de UI;
- `dropped_frames`: frames perdidos durante uma interação;
- `long_task_count`: trabalho longo que bloqueia a experiência.

## Medir TTI

Use `react-native-performance` para marcadores de startup e tela.

```bash
npm install react-native-performance
```

Exemplo:

```tsx
import { useEffect } from "react"
import performance from "react-native-performance"

export function HomeScreen() {
  useEffect(() => {
    performance.mark("home_screen_interactive")
  }, [])

  return null
}
```

Colete os marks e envie para sua camada de métricas:

```ts
import performance from "react-native-performance"
import { metrics } from "./metrics"

function durationBetween(startName: string, endName: string) {
  const start = performance.getEntriesByName(startName).at(-1)
  const end = performance.getEntriesByName(endName).at(-1)

  if (!start || !end) return null

  return end.startTime - start.startTime
}

export function reportStartupMetrics() {
  const tti = durationBetween("nativeLaunchStart", "home_screen_interactive")

  if (tti == null) return

  metrics.timing("app_start_tti_ms", tti, {
    startup_type: "cold",
    screen: "Home",
  })
}
```

Meça cold start separadamente. Warm e hot starts têm outro significado e não devem entrar no mesmo p95.

## Medir FPS rapidamente

Para triagem local:

1. Abra o Dev Menu.
2. Ative `Perf Monitor`.
3. Observe `UI FPS`, `JS FPS` e RAM.
4. Reproduza scroll, animação ou fluxo suspeito.

Leitura:

| FPS | Percepção |
| ---: | --- |
| `55-60` | fluido em telas 60 Hz |
| `45-55` | pequenas travadas |
| `30-45` | jank perceptível |
| `< 30` | crítico |

Em aparelhos 120 Hz, o orçamento por frame é menor. A meta pode ser mais exigente dependendo do público.

## Medir com build confiável

Não tome decisão de performance com dev mode ligado.

React Native CLI:

```bash
npx react-native run-android --mode release
```

Expo:

```bash
npx expo start --no-dev --minify
```

Para validação real em Expo, prefira EAS Build ou development build com configuração próxima de produção.

## Interpretar JS FPS e UI FPS

Se `JS FPS` cai e `UI FPS` fica bem:

- há trabalho pesado no JavaScript;
- pode haver renderização React excessiva;
- procure loops, serialização grande, parsing pesado, estado global amplo ou listas mal virtualizadas.

Se `UI FPS` cai e `JS FPS` fica bem:

- o problema pode estar na renderização nativa;
- investigue layout pesado, muitas views, imagens, sombras, animações nativas ou módulos nativos.

Se ambos caem:

- comece pelo profiler React e depois valide com profiler nativo.

## Perfil React

Para re-render e commits lentos, use React Native DevTools Profiler. O objetivo é capturar a interação exata.

Fluxo:

1. Conecte DevTools.
2. Inicie o profiling.
3. Faça a interação lenta.
4. Pare o profiling.
5. Veja componentes mais lentos, número de renders e commit timeline.

Não aplique `useMemo`, `useCallback` ou troca de estado por instinto. Primeiro confirme que o problema é renderização.

## Perfil nativo

Quando o JS parece saudável, use ferramentas nativas:

- iOS: Xcode Instruments, Time Profiler, Hangs e Leaks;
- Android: Android Studio Profiler, CPU Profiler, Memory Profiler e Perfetto.

Em Expo Go, profiling nativo é limitado. Para esse tipo de investigação, use dev client, prebuild ou build nativo.

## Resumo

TTI mostra quanto tempo o app demora para ficar útil. FPS mostra fluidez durante interação. Combine essas métricas com p95 e p99 para separar latência de rede, renderização React, bloqueio de JS thread e gargalo nativo.

---

Série: [[index|Métricas p95 e p99 em React Native e Expo]]  
Anterior: [[Datadog RUM - p95 e p99 em React Native]]  
Próximo: [[Dashboards e alertas para p95 e p99 mobile]]
