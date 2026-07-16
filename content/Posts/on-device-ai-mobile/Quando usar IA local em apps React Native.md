---
title: Quando usar IA local em apps React Native
draft: false
tags:
  - dev
  - mobile
  - react-native
  - ai
socialDescription: Critérios para decidir entre modelo local no dispositivo e chamada remota para uma API de IA.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# TL;DR;

Use IA local quando privacidade, offline ou baixa latência forem mais importantes que flexibilidade e qualidade máxima do modelo.

## Bom caso de uso

- Classificação simples de texto.
- Sugestões locais.
- Extração leve de informação.
- OCR ou visão com modelo pequeno.
- Assistência offline.
- Pré-processamento antes de enviar dados ao backend.

## Melhor usar API remota

- Raciocínio complexo.
- Modelo grande.
- Atualização frequente de comportamento.
- Necessidade de auditoria centralizada.
- Dispositivo alvo fraco.
- Tarefa que exige contexto extenso.

## Matriz rápida

| Critério | Local | Remoto |
|---|---|---|
| Offline | forte | fraco |
| Privacidade | forte | depende |
| Latência | forte | depende da rede |
| Qualidade máxima | limitado | forte |
| Tamanho do app | pior | melhor |
| Atualização do modelo | mais difícil | mais simples |

## Decisão

Comece pela jornada. Se o usuário precisa da resposta no campo, sem rede e com dado sensível, IA local faz sentido. Se a tarefa tolera rede e exige modelo maior, API remota costuma ser mais pragmática.

