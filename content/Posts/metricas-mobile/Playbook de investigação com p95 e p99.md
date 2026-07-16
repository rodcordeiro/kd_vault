---
title: Playbook de investigação com p95 e p99
draft: false
description: Roteiro prático para investigar regressões de p95 e p99 em aplicações React Native e Expo.
tags:
  - dev
  - mobile
  - react-native
  - expo
  - performance
  - observabilidade
  - serie
socialDescription: Como investigar regressões de p95 e p99 em apps React Native e Expo.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# Playbook de investigação com p95 e p99

Quando p95 ou p99 piora, a primeira reação não deve ser otimizar. Primeiro recorte.

Série: parte 6 de 6. Anterior: [[Dashboards e alertas para p95 e p99 mobile]]. Voltar para [[index|Métricas p95 e p99 em React Native e Expo]].

## 1. Confirme o sinal

Pergunte:

- houve volume suficiente?
- o aumento começou em um horário claro?
- afetou p95, p99 ou ambos?
- a média também mudou?
- existe aumento simultâneo de erros?

Se só p99 mudou com baixo volume, trate como investigação de outlier antes de abrir incidente amplo.

## 2. Quebre por versão

Compare `app.version`.

Se uma versão nova piorou:

- veja changelog;
- procure feature flag ligada;
- compare telas alteradas;
- veja erros novos;
- procure aumento de long tasks ou crashes.

Se todas as versões pioraram ao mesmo tempo, olhe backend, rede, serviço externo ou mudança de configuração remota.

## 3. Quebre por plataforma e aparelho

Separe:

- Android;
- iOS;
- modelos específicos;
- device tier;
- versão do sistema operacional.

P99 alto concentrado em Android de entrada pede hipótese diferente de p99 alto em todas as plataformas.

## 4. Quebre por tela e endpoint

Para tela lenta:

- veja quais recursos a tela chamou;
- compare duração dos endpoints;
- procure erro silencioso com retry;
- confira se a tela espera dados demais antes de ficar interativa.

Para endpoint lento:

- compare status HTTP;
- compare região ou tipo de rede;
- veja se o backend também degradou;
- procure payload maior ou cache miss.

## 5. Reproduza com build correto

Não valide performance em dev mode.

Expo:

```bash
npx expo start --no-dev --minify
```

React Native CLI:

```bash
npx react-native run-android --mode release
```

Quando possível, use device real próximo do recorte afetado.

## 6. Use o profiler certo

Se a métrica lenta é rede:

- investigue recurso, endpoint, payload, cache e retries.

Se a métrica lenta é tela:

- use React DevTools Profiler para renderização;
- meça FPS durante a interação;
- use profiler nativo quando UI thread cair sem queda clara na JS thread.

Se a métrica lenta é startup:

- separe cold, warm e hot start;
- veja bundle load;
- veja inicialização nativa;
- veja primeira tela útil.

## 7. Corrija e remeça

O ciclo deve ser:

1. medir baseline;
2. formular hipótese;
3. aplicar uma correção pequena;
4. reexecutar a mesma medição;
5. comparar p50, p95, p99 e volume;
6. manter ou reverter a mudança.

Não comemore melhoria de média se p95 e p99 continuaram ruins.

## 8. Registre a decisão

Para cada regressão relevante, registre:

- métrica afetada;
- intervalo;
- recorte;
- hipótese;
- evidência;
- correção;
- resultado depois da correção.

Exemplo:

```md
## Regressão Home p99

- Métrica: screen_load_ms
- Recorte: Android, app 1.9.0, device tier low
- Sintoma: p95 estável, p99 subiu de 4.2s para 9.8s
- Hipótese: cache frio + endpoint de banners com retry
- Evidência: resource_duration_ms p99 de /banners subiu no mesmo horário
- Correção: timeout menor e renderização parcial da Home
- Resultado: Home p99 caiu para 5.1s
```

## Resumo

P95 e p99 não dizem sozinhos qual é a causa. Eles apontam onde a experiência piorou. O trabalho é recortar por versão, plataforma, aparelho, tela, endpoint e tipo de startup até a hipótese ficar pequena o bastante para corrigir e remedir.

---

Série: [[index|Métricas p95 e p99 em React Native e Expo]]  
Anterior: [[Dashboards e alertas para p95 e p99 mobile]]
