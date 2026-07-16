---
title: Dashboards e alertas para p95 e p99 mobile
draft: false
description: Como montar dashboards e monitores de p95 e p99 para aplicações React Native e Expo.
tags:
  - dev
  - mobile
  - react-native
  - expo
  - performance
  - observabilidade
  - serie
socialDescription: Estrutura prática de dashboards e alertas para p95 e p99 em apps mobile.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# Dashboards e alertas para p95 e p99 mobile

Dashboard bom responde rápido: está piorando, onde piorou, desde quando e em qual recorte?

Série: parte 5 de 6. Anterior: [[Monitorando TTI e FPS em React Native]]. Próximo: [[Playbook de investigação com p95 e p99]].

## Painel mínimo

Crie um dashboard mobile com estes blocos:

- crash-free sessions;
- erros JS por versão;
- crashes nativos por versão;
- p95 e p99 de `app_start_tti_ms`;
- p95 e p99 de telas críticas;
- p95 e p99 de recursos de rede críticos;
- taxa de erro por endpoint;
- long tasks;
- distribuição por plataforma e versão do app.

Comece com poucas telas:

- Home;
- Login;
- Checkout;
- Sincronização;
- Busca;
- Tela que mais recebe reclamação.

## Quebras essenciais

Todo gráfico de p95/p99 deve permitir quebrar por:

- `env`;
- `app.version`;
- `platform`;
- `device_model` ou `device_tier`;
- `os_version`;
- `screen`;
- `endpoint`;
- `result`.

Sem essas quebras, o dashboard só mostra que algo piorou. Com elas, ele começa a apontar onde.

## Thresholds iniciais

Use valores iniciais como hipótese, não como verdade universal.

Exemplos:

| Métrica | Monitor inicial |
| --- | --- |
| Home p95 | acima de `3 s` por 10 min |
| Home p99 | acima de `8 s` por 10 min |
| Login p95 | acima de `2 s` por 10 min |
| Checkout p95 | acima de `4 s` por 10 min |
| API crítica p95 | acima de `1 s` por 10 min |
| JS errors | aumento por versão nova |
| Crash-free sessions | queda abaixo da meta do time |

Depois de algumas semanas, ajuste os thresholds com base no histórico real.

## Alerta por versão

Monitore regressão por `app.version`.

Exemplo de leitura:

| Versão | Home p95 | Home p99 | Leitura |
| --- | ---: | ---: | --- |
| `1.8.0` | `1.9 s` | `4.2 s` | baseline |
| `1.9.0` | `2.1 s` | `9.8 s` | cauda piorou |

Quando p99 piora muito e p95 quase não muda, procure caso específico: aparelho, rede, cache, feature flag ou fluxo raro.

## Alerta por anomalia

Use monitor de threshold para regressão clara.

Use monitor de anomalia quando:

- a métrica oscila por horário;
- o volume muda muito;
- existe sazonalidade;
- o p99 tem comportamento naturalmente irregular.

Alertas de p99 exigem mais cuidado porque poucos eventos podem deslocar a métrica.

## SLO mobile

Transforme métricas em objetivos simples:

- 95% das aberturas da Home abaixo de `3 s`;
- 95% dos logins abaixo de `2 s`;
- 99% das sessões sem crash;
- p95 dos recursos críticos abaixo de `1 s`.

SLO mobile precisa estar conectado à experiência do usuário. Não crie objetivo só porque a ferramenta permite.

## Checklist de dashboard

- [ ] Mostra p50, p95 e p99.
- [ ] Separa Android e iOS.
- [ ] Separa versão do app.
- [ ] Mostra volume junto da latência.
- [ ] Mostra erro junto da duração.
- [ ] Permite clicar do gráfico para eventos RUM.
- [ ] Evita tags de alta cardinalidade.
- [ ] Tem links para playbook de investigação.

## Resumo

Dashboard de p95 e p99 precisa mostrar cauda, volume e contexto. O objetivo é reduzir o tempo entre detectar regressão e descobrir o recorte responsável.

---

Série: [[index|Métricas p95 e p99 em React Native e Expo]]  
Anterior: [[Monitorando TTI e FPS em React Native]]  
Próximo: [[Playbook de investigação com p95 e p99]]
