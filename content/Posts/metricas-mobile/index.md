---
title: Métricas p95 e p99 em React Native e Expo
draft: false
description: Série prática para medir, publicar e monitorar p95 e p99 em aplicações React Native e Expo.
tags:
  - dev
  - mobile
  - react-native
  - expo
  - performance
  - observabilidade
  - serie
socialDescription: Série prática para medir p95, p99, TTI, FPS e jornadas críticas em apps React Native e Expo.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# Métricas p95 e p99 em React Native e Expo

Esta série organiza um caminho prático para medir a cauda da experiência em aplicações React Native e Expo.

A média costuma esconder o usuário que sofre. `p95` e `p99` mostram onde o app degrada para os usuários mais afetados: aparelho fraco, rede ruim, versão problemática, tela pesada, endpoint instável ou fluxo com trabalho demais na JS thread.

## Para quem é

- Times mobile que precisam sair de percepção subjetiva para evidência.
- Pessoas investigando tela lenta, checkout demorado, login instável, scroll travado ou inicialização pesada.
- Apps React Native ou Expo que já usam, ou pretendem usar, RUM, Datadog, dashboards e monitores.

## Série

1. [[P95 e p99 - o que medir em apps mobile|P95 e p99 - o que medir em apps mobile]]
   Explica percentis, cauda, diferença para média e quais métricas fazem sentido no mobile.

2. [[Instrumentando métricas no React Native e Expo|Instrumentando métricas no React Native e Expo]]
   Mostra helpers para medir duração de telas, recursos, ações e jornadas críticas.

3. [[Datadog RUM - p95 e p99 em React Native|Datadog RUM - p95 e p99 em React Native]]
   Tutorial para enviar eventos, criar métricas RUM e habilitar percentis como `p95` e `p99`.

4. [[Monitorando TTI e FPS em React Native|Monitorando TTI e FPS em React Native]]
   Tutorial para medir startup, TTI, FPS, JS thread e UI thread com build release-like.

5. [[Dashboards e alertas para p95 e p99 mobile|Dashboards e alertas para p95 e p99 mobile]]
   Define painéis, monitores, thresholds iniciais e leitura por versão, plataforma e device tier.

6. [[Playbook de investigação com p95 e p99|Playbook de investigação com p95 e p99]]
   Roteiro para transformar uma regressão de p95/p99 em hipótese, recorte e correção.

7. [[Worklets e JS thread em React Native|Worklets e JS thread em React Native]]
   Explica quando worklets ajudam em animações, gestos e processamento frequente sem depender da JS thread.

8. [[Profilando ANR e travamentos em Android|Profilando ANR e travamentos em Android]]
   Roteiro para investigar travamentos, ANRs, logcat, JS thread, UI thread e profiler nativo.

## Linha editorial

Métrica mobile precisa de contexto. Um número sem `app.version`, `platform`, `device_model`, `os_version`, rede e tela vira ruído.

Comece pequeno: crash-free sessions, erros JS/nativos, p95 de abertura de tela, p95 de recursos críticos e p99 das jornadas mais importantes.
