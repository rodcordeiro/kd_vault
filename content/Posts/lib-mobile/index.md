---
title: Guia da lib mobile
draft: false
description: Série prática para criar, validar e evoluir uma biblioteca mobile compartilhada para React Native e Expo.
tags:
  - dev
  - mobile
  - react-native
  - expo
  - lib-mobile
  - serie
socialDescription: Série prática para criar, validar e evoluir uma biblioteca mobile compartilhada para React Native e Expo.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# Guia da lib mobile

Esta série organiza um caminho pragmático para criar uma biblioteca mobile compartilhada para React Native e Expo. A referência prática é a `Torra.Components.Mobile`, mas os critérios foram escritos para funcionar em qualquer organização que precise reduzir duplicação sem transformar a lib em um app base.

## Para quem é

- Times que precisam compartilhar componentes, funções, hooks e tokens entre apps mobile.
- Pessoas criando uma lib com Expo, React Native, TypeScript, NativeWind e `react-native-builder-bob`.
- Tech leads avaliando o que deve virar contrato público e o que deve continuar no app consumidor.

## Série

1. [[Guia - Criando uma biblioteca de componentes mobile|Criando uma biblioteca de componentes mobile]]
   Define fronteira, estrutura do pacote, dependências, API pública, tema, componentes, validação e publicação.

2. [[Roteiro verificável - Biblioteca mobile|Roteiro verificável]]
   Checklist de execução por fase, com evidências mínimas para considerar o MVP pronto.

3. [[Glossário e decisões - Biblioteca mobile|Glossário e decisões]]
   Vocabulário, decisão arquitetural sobre dependências nativas e limites derivados.

4. [[Helper cn - compondo classes NativeWind|Helper cn - compondo classes NativeWind]]
   Implementa um helper prático para compor classes NativeWind com condicionais, mapas, arrays e resolução de conflitos.

## Linha editorial

O guia parte de uma regra simples: a biblioteca deve publicar contratos pequenos, estáveis e sem regra de negócio. Código de autenticação, API, sync, SQLite, telemetria, navegação de tela e fluxos específicos fica no app consumidor.

O MVP só é confiável quando a versão publicada instala, tipa, renderiza e roda em um app piloto real, consumindo apenas exports da raiz do pacote.
