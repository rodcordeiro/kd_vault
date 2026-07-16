---
title: Guia prático de adb logcat
draft: false
description: Série prática para usar adb logcat no diagnóstico de aplicações Android e React Native.
tags:
  - dev
  - mobile
  - android
  - react-native
  - logcat
  - serie
socialDescription: Série prática para ler, filtrar e mapear logs Android com adb logcat.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# Guia prático de adb logcat

Esta série organiza um caminho prático para usar `adb logcat` no diagnóstico de aplicações Android e React Native.

A ideia não é decorar todas as opções do comando. É aprender a ler o fluxo real da aplicação, reduzir ruído, escolher filtros úteis e salvar evidências quando uma falha precisa ser analisada depois.

## Para quem é

- Pessoas desenvolvendo ou mantendo apps Android e React Native.
- Quem precisa investigar crashes, ANRs, falhas de rede, erros de storage ou comportamento estranho em runtime.
- Times que querem padronizar tags e níveis de log para tornar o diagnóstico mais rápido.

## Série

1. [[ADB logcat - guia de estudo|ADB logcat - guia de estudo]]
   Apresenta buffers, formatos, filtros por tag e um roteiro para comparar fluxo feliz com fluxo com erro.

2. [[Prioridades do logcat|Prioridades do logcat]]
   Explica `V`, `D`, `I`, `W`, `E`, `F` e `S`, com exemplos de uso e filtros por nível mínimo.

3. [[Modificadores do logcat|Modificadores do logcat]]
   Mostra formatos e modificadores como `threadtime`, `color`, `epoch`, `monotonic`, `uid`, `year`, `zone` e `descriptive`.

4. [[Tags consistentes em logcat|Tags consistentes em logcat]]
   Define uma estratégia de tags por responsabilidade para deixar logs filtráveis e menos ambíguos.

5. [[Mapeando logs Android com logcat|Mapeando logs Android com logcat]]
   Junta comandos práticos para capturar, filtrar e salvar logs de investigação.

## Linha editorial

Log bom não é log em grande quantidade. Log bom ajuda a responder onde o fluxo começou, qual dependência mudou de comportamento e qual foi a primeira mensagem útil antes do erro.

Use `logcat` como ferramenta de leitura de fluxo, não só como lugar para procurar stack trace.
