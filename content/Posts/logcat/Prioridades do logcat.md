---
title: Prioridades do logcat
draft: false
description: Como usar as prioridades V, D, I, W, E, F e S para reduzir ruído e investigar logs Android.
tags:
  - dev
  - mobile
  - android
  - react-native
  - logcat
  - serie
socialDescription: Entenda as prioridades do logcat e como filtrar logs por nível de gravidade.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# Prioridades do logcat

As prioridades do `logcat` indicam a gravidade ou importância da mensagem registrada. Elas vão da mais detalhada até a mais silenciosa.

Série: parte 2 de 5. Anterior: [[ADB logcat - guia de estudo]]. Próximo: [[Modificadores do logcat]].

## A escala

Ordem das prioridades:

- `V` = `Verbose`;
- `D` = `Debug`;
- `I` = `Info`;
- `W` = `Warning`;
- `E` = `Error`;
- `F` = `Fatal`;
- `S` = `Silent`.

A regra prática é:

- `V` mostra praticamente tudo;
- `D` mostra mensagens de depuração;
- `I` mostra informação normal de execução;
- `W` destaca avisos;
- `E` mostra erros;
- `F` mostra falhas críticas;
- `S` não imprime nada.

Quanto mais alta a prioridade, mais grave a situação.

## Como usar no dia a dia

Em React Native, use prioridades para separar:

- fluxo normal do app;
- depuração de telas, hooks e services;
- falhas de rede;
- erro de persistência local;
- exceções que precisam de correção.

Exemplos em logs da aplicação:

```ts
console.log("[Home][I] tela carregada")
console.log("[Network][D] iniciando requisicao")
console.warn("[Storage][W] cache antigo encontrado")
console.error("[Auth][E] falha ao autenticar")
```

No JavaScript, `console.warn` e `console.error` ajudam a destacar a gravidade visualmente, mas o ponto principal é manter a mensagem padronizada para filtragem.

## Filtrar por nível mínimo

Para mostrar apenas avisos, erros e falhas fatais:

```bash
adb logcat *:W
```

Esse comando é um bom começo quando o log está muito barulhento. Ele remove mensagens de debug e informação, deixando apenas sinais mais fortes.

Para abrir uma tag específica em nível `Debug` e silenciar o resto:

```bash
adb logcat ReactNativeJS:D *:S
adb logcat MyApp:D *:S
```

O `*:S` é importante porque sem ele o Android ainda pode imprimir mensagens de outras tags conforme a configuração padrão.

## Como escolher a prioridade

Use assim:

- `V`: detalhe temporário, investigação fina, normalmente local;
- `D`: depuração de fluxo, parâmetros resumidos, decisões internas;
- `I`: eventos normais que ajudam a entender o ciclo da aplicação;
- `W`: comportamento suspeito, fallback, retry, cache antigo, dado incompleto;
- `E`: falha real que impediu uma operação esperada;
- `F`: falha crítica ou encerramento;
- `S`: filtro para silenciar.

Exemplo de estratégia:

```ts
console.log("[Sync][I] sincronizacao iniciada")
console.log("[Sync][D] enviando 12 itens pendentes")
console.warn("[Sync][W] tentativa 1 falhou, tentando novamente")
console.error("[Sync][E] sincronizacao abortada", error)
```

## O erro comum

O erro mais comum é usar `Error` para qualquer coisa que chamou atenção. Isso torna o log menos útil.

Se tudo é erro, nada é erro. Use `W` para comportamento recuperável e `E` para falha que realmente impediu o fluxo.

## Resumo

As prioridades do `logcat` são o jeito mais rápido de controlar o ruído. Em React Native, elas ajudam a separar depuração, fluxo normal e erros reais sem perder contexto do que está acontecendo no app.

---

Série: [[index|Guia prático de adb logcat]]  
Anterior: [[ADB logcat - guia de estudo]]  
Próximo: [[Modificadores do logcat]]
