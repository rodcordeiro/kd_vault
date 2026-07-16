---
title: Correlacionando logcat, Sentry e métricas mobile
draft: false
tags:
  - dev
  - mobile
  - observability
  - logcat
socialDescription: Como conectar logs locais, eventos de erro e métricas de performance durante uma investigação mobile.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# TL;DR;

Cada ferramenta responde uma pergunta diferente. Logcat mostra o dispositivo; Sentry mostra a falha; métricas mostram a escala.

## Perguntas

- Logcat: o que aconteceu naquele aparelho naquele momento?
- Sentry: qual exceção, contexto e sequência de breadcrumbs?
- Métricas: quantos usuários foram afetados e em quais versões?

## Campos de correlação

Use os mesmos nomes sempre que possível:

```ts
const commonContext = {
  appVersion,
  buildNumber,
  platform,
  screen: "OrderList",
  flow: "sync",
};
```

Em eventos remotos:

```ts
Sentry.setTags(commonContext);
```

Em logs locais de desenvolvimento:

```ts
console.info("[sync]", commonContext, { pending: 12, failed: 1 });
```

## Roteiro de investigação

1. Identifique versão e tela no monitoramento.
2. Veja se o erro tem breadcrumbs suficientes.
3. Reproduza em build semelhante.
4. Capture logcat filtrado por tags relevantes.
5. Compare duração, p95/p99 e taxa de erro antes/depois da versão.

## Cuidados

Não envie logcat bruto para ferramenta remota. Ele pode conter dados demais. Extraia sinais, normalize rotas e remova identificadores.

