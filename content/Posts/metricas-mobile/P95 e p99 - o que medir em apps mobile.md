---
title: P95 e p99 - o que medir em apps mobile
draft: false
description: Conceitos práticos de p95, p99 e métricas de cauda para aplicações React Native e Expo.
tags:
  - dev
  - mobile
  - react-native
  - expo
  - performance
  - observabilidade
  - serie
socialDescription: Entenda p95 e p99 em apps mobile e escolha métricas úteis para React Native e Expo.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# P95 e p99 - o que medir em apps mobile

`p95` e `p99` são percentis. Eles respondem uma pergunta melhor do que a média: quão ruim está a experiência para a cauda dos usuários?

Série: parte 1 de 6. Próximo: [[Instrumentando métricas no React Native e Expo]]. Voltar para [[index|Métricas p95 e p99 em React Native e Expo]].

## Como ler

Se o p95 de abertura da tela Home é `2.4 s`, então 95% das medições ficaram em até `2.4 s` e 5% ficaram acima disso.

Se o p99 é `6.8 s`, então 1% das medições ficou acima desse valor. Esse 1% pode parecer pequeno, mas em app com muito uso ele representa usuários reais sofrendo.

Exemplo:

| Métrica | Valor | Leitura |
| --- | ---: | --- |
| Média | `1.8 s` | parece aceitável |
| p95 | `2.4 s` | a maioria já está perto do limite |
| p99 | `6.8 s` | poucos usuários estão sofrendo muito |

Quando média e p95 parecem boas, mas p99 explode, investigue casos extremos: aparelho fraco, rede instável, cache frio, endpoint lento, lock de storage, tela muito pesada ou bug por versão.

## Por que média engana

A média mistura usuários rápidos e lentos em um único número. Em mobile, isso é perigoso porque a base é naturalmente heterogênea:

- aparelhos de entrada e topo de linha;
- Android e iOS;
- versões diferentes do sistema operacional;
- redes 4G, 5G, Wi-Fi ruim e offline intermitente;
- builds e versões do app coexistindo;
- sessões frias, mornas e quentes.

Se 95 usuários carregam uma tela em `1 s` e 5 carregam em `10 s`, a média pode parecer administrável. O p95 e o p99 mostram a dor.

## O que medir primeiro

Comece por métricas que descrevem experiência real:

- `app_start_tti_ms`: tempo até a primeira tela útil ficar interativa;
- `screen_load_ms`: tempo para uma tela crítica carregar conteúdo útil;
- `resource_duration_ms`: duração de recursos de rede importantes;
- `flow_duration_ms`: duração de jornadas como login, checkout, sincronização ou busca;
- `js_error_count`: erros JavaScript por versão e tela;
- `native_crash_count`: crashes nativos;
- `long_task_count`: sinais de trabalho longo bloqueando a experiência;
- `fps_average` e quedas de FPS em fluxos com scroll ou animação.

Não tente medir tudo no começo. Escolha as jornadas que mais doem para o usuário e para o negócio.

## Dimensões obrigatórias

Para p95 e p99 serem úteis, cada métrica precisa de dimensões. No mínimo:

- `env`;
- `app.version`;
- `platform`;
- `device_model`;
- `os_version`;
- `screen`;
- `flow`;
- `network_type`, quando disponível.

Evite dimensões de alta cardinalidade:

- `user_id`;
- `session_id`;
- `request_id`;
- timestamp;
- payload completo;
- URL com IDs variáveis.

Alta cardinalidade aumenta custo, piora dashboards e dificulta leitura.

## P95 ou p99?

Use `p95` como métrica operacional principal. Ele mostra regressão relevante sem ser tão sensível a poucos outliers.

Use `p99` para jornadas críticas e para investigar cauda extrema. Ele é ótimo para revelar casos raros, mas pode oscilar quando há pouco volume.

Regra prática:

- `p50`: experiência típica;
- `p95`: experiência ruim para uma parte relevante da base;
- `p99`: experiência extrema que precisa de recorte e investigação.

## Não compare sem recorte

Comparar p95 global de uma versão com outra pode mentir. Antes de concluir, quebre por:

- versão do app;
- plataforma;
- modelo ou tier de aparelho;
- tela;
- endpoint;
- região ou tipo de rede;
- cold start versus warm/hot start.

Para startup, meça apenas cold starts quando o objetivo for TTI. Warm e hot starts têm natureza diferente.

## Resumo

`p95` e `p99` são métricas de cauda. Elas mostram o que a média esconde e ajudam a priorizar correções que afetam usuários reais. Em React Native e Expo, elas ficam mais fortes quando combinadas com tela, versão, plataforma, aparelho, rede e tipo de fluxo.

---

Série: [[index|Métricas p95 e p99 em React Native e Expo]]  
Próximo: [[Instrumentando métricas no React Native e Expo]]
