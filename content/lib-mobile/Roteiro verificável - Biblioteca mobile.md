---
title: Roteiro verificável da biblioteca mobile
draft: false
description: Checklist por fase para criar, validar e publicar uma biblioteca mobile compartilhada com evidências objetivas.
tags:
  - dev
  - mobile
  - react-native
  - expo
  - lib-mobile
  - checklist
  - serie
socialDescription: Checklist por fase para criar, validar e publicar uma biblioteca mobile compartilhada com evidências objetivas.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# Roteiro verificável - Biblioteca mobile

Roteiro mínimo para criar e publicar uma biblioteca compartilhada de React Native e Expo.

Série: parte 2 de 3. Anterior: [[Guia - Criando uma biblioteca de componentes mobile]]. Próximo: [[Glossário e decisões - Biblioteca mobile]].

Relacionado: [[index|Guia da lib mobile]].

## Fase 1 - Fundação

- [ ] Definir público consumidor e problema de duplicação que a biblioteca resolve.
- [ ] Fixar a baseline de Expo, React, React Native e TypeScript.
- [ ] Criar o pacote com `react-native-builder-bob`.
- [ ] Configurar saídas CommonJS, ESM e TypeScript.

Evidência de saída:

- [ ] `pnpm build` gera os três formatos sem erro.

## Fase 2 - Fronteira e tema

- [ ] Criar `src/components`, `src/functions`, `src/hooks` e `src/theme`.
- [ ] Definir `src/index.ts` como única porta pública.
- [ ] Criar tokens mínimos de cor, tipografia e espaçamento.
- [ ] Criar ou adotar um helper `cn` para NativeWind.
- [ ] Declarar React, React Native, Expo, NativeWind e módulos nativos como peers quando aplicável.

Evidência de saída:

- [ ] Um arquivo externo consegue importar tema e tipos apenas da raiz do pacote.

## Fase 3 - Primeira API pública

- [ ] Implementar um componente de layout puro.
- [ ] Implementar um componente de ação com `disabled`, `loading`, acessibilidade e `className`.
- [ ] Implementar uma função genérica sem regra de negócio.
- [ ] Implementar um hook genérico, somente se existir um caso compartilhado real.
- [ ] Exportar componentes, funções, hooks, tokens e tipos em `src/index.ts`.

Evidência de saída:

- [ ] Busca por imports de aliases do app não retorna resultados.
- [ ] Testes dos contratos públicos passam.

## Fase 4 - App piloto

- [ ] Instalar a biblioteca em um app Expo real.
- [ ] Consumir somente exports da raiz.
- [ ] Validar tema, classes customizadas, refs, foco e estados visuais.
- [ ] Confirmar que não há React ou React Native duplicados.
- [ ] Registrar configuração exigida por qualquer peer nativo.

Evidência de saída:

- [ ] O app piloto compila e executa o fluxo escolhido em pelo menos uma plataforma alvo.

## Fase 5 - Publicação

- [ ] Definir registro, autenticação e política de versionamento semântico.
- [ ] Criar pipeline com instalação por lockfile, lint, testes e build.
- [ ] Publicar uma versão prerelease ou `0.x`.
- [ ] Reinstalar a versão publicada no app piloto, sem link local.

Evidência de saída:

- [ ] A versão publicada instala, tipa e renderiza corretamente no app piloto.

## Critério de MVP concluído

O MVP está pronto somente quando:

- [ ] a API pública é pequena e importada pela raiz;
- [ ] não existe dependência de regra de negócio ou infraestrutura do app;
- [ ] CommonJS, ESM e tipos são gerados;
- [ ] contratos públicos têm testes;
- [ ] tema e `className` funcionam no consumidor;
- [ ] peers e configurações nativas estão documentados;
- [ ] uma versão publicada foi validada em app piloto.

## Depois do MVP

- [ ] Migrar um componente por vez.
- [ ] Adicionar wrapper nativo somente após validar configuração em device.
- [ ] Validar reuso em um segundo app antes de ampliar contratos compostos.
- [ ] Registrar breaking changes e estratégia de migração.

---

Série: [[index|Guia da lib mobile]]  
Anterior: [[Guia - Criando uma biblioteca de componentes mobile]]  
Próximo: [[Glossário e decisões - Biblioteca mobile]]
