---
title: Maturidade de observabilidade mobile
draft: false
tags:
  - dev
  - mobile
  - observability
socialDescription: Um checklist por nível para evoluir observabilidade em apps React Native sem capturar dados sensíveis.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# TL;DR;

Observabilidade mobile amadurece em camadas: crash, contexto, jornadas, performance e investigação guiada.

## Nível 1: sobrevivência

- Crash reporting ativo.
- Versão do app nos eventos.
- Plataforma, OS e device model.
- Source maps/symbols configurados.
- Sanitização de dados sensíveis.

## Nível 2: contexto

- Breadcrumbs de navegação.
- Erros HTTP com rota normalizada.
- Estado de rede.
- Usuário anônimo ou hash não reversível.
- Ambiente e canal de release.

## Nível 3: jornadas

- Duração de login, abertura de tela e envio de formulário.
- Resultado de sincronização.
- Quantidade de pendências offline.
- Erro por etapa da jornada.

## Nível 4: performance

- p95/p99 por tela.
- TTI e startup.
- FPS em interações críticas.
- ANR e travamentos por versão.

## Nível 5: investigação

- Playbooks por sintoma.
- Dashboards por versão e device tier.
- Alertas com threshold inicial revisável.
- Correlação entre logcat, RUM, crash e release.

## Critério

Se um alerta não aponta para ação, ele ainda é métrica exploratória. Transforme em monitor só quando houver dono, recorte e resposta operacional.

