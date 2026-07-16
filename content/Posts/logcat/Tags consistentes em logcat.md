---
title: Tags consistentes em logcat
draft: false
description: Como criar uma estratégia de tags para tornar logs Android e React Native mais filtráveis.
tags:
  - dev
  - mobile
  - android
  - react-native
  - logcat
  - serie
socialDescription: Estratégia prática de tags para reduzir ruído e acelerar investigação com logcat.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# Tags consistentes em logcat

Em `logcat`, a tag é o nome que identifica a origem da mensagem. Ela funciona como uma etiqueta de contexto para descobrir rapidamente quem falou aquilo.

Série: parte 4 de 5. Anterior: [[Modificadores do logcat]]. Próximo: [[Mapeando logs Android com logcat]].

## O que uma boa tag responde

Uma boa tag ajuda a responder:

- de onde o log veio;
- qual fluxo gerou a mensagem;
- qual parte do app está se comportando diferente;
- o que filtrar quando o log inteiro está barulhento.

Ela deve responder à pergunta: quem falou isso?

## Tags por responsabilidade

Use tags por responsabilidade, não por frase aleatória.

Exemplos bons:

- `Auth`;
- `Network`;
- `Storage`;
- `Navigation`;
- `HomeScreen`;
- `Sync`;
- `ApiClient`.

Evite tags genéricas:

- `Debug`;
- `App`;
- `Teste`;
- `Foo`.

Uma tag genérica quase sempre vira ruído. Ela não ajuda a isolar a origem do problema.

## Helper simples para React Native

No React Native, é comum os logs JavaScript aparecerem na tag `ReactNativeJS`. Por isso, além da tag nativa, vale padronizar um prefixo dentro da mensagem.

Exemplo:

```ts
// logger.ts
type LogScope = "Auth" | "Network" | "Storage" | "Navigation" | "Home" | "Sync"

export function log(scope: LogScope, message: string) {
  console.log(`[${scope}] ${message}`)
}

export function logWarn(scope: LogScope, message: string, detail?: unknown) {
  console.warn(`[${scope}] ${message}`, detail)
}

export function logError(scope: LogScope, message: string, error?: unknown) {
  console.error(`[${scope}] ${message}`, error)
}
```

Uso:

```ts
import { log, logError } from "./logger"

log("Auth", "login iniciado")
log("Network", "enviando requisicao para /login")
logError("Storage", "falha ao salvar cache", error)
```

O ideal é manter o mesmo padrão em todo o app. Se o módulo chama `Auth`, não use depois `LoginService` sem motivo claro.

## Como decidir uma tag

Pergunte:

1. Esta mensagem representa uma responsabilidade do sistema?
2. Eu vou querer filtrar esse tipo de log no futuro?
3. Essa tag ajuda a achar o ponto do fluxo mais rápido?

Se a resposta for sim, a tag faz sentido.

## Filtro direto no logcat

O formato mais comum é:

```bash
adb logcat Auth:D *:S
```

Isso significa:

- mostrar `Auth` em nível `Debug` ou acima;
- silenciar todo o resto com `*:S`.

Outros exemplos:

```bash
adb logcat Network:I *:S
adb logcat Auth:D Network:D *:S
adb logcat *:W
```

## Filtro comum em React Native

Quando os logs vêm do JavaScript, é normal enxergar tags do runtime, como `ReactNativeJS` e `ReactNative`.

Nesse caso, uma combinação útil é:

```bash
adb logcat ReactNativeJS:V ReactNative:V *:S
```

Se você padronizou prefixos como `[Auth]`, `[Network]` e `[Storage]`, muitas vezes o filtro mais eficaz é por texto:

```powershell
adb logcat -v time | Select-String -Pattern "\[Auth\]|\[Network\]|\[Storage\]"
```

Esse filtro por texto é útil, mas é pior do que tag real porque depende do conteúdo da mensagem. Ainda assim, para React Native, costuma ser uma solução pragmática.

## Estratégia inicial

Para um app mobile, um conjunto simples pode ser:

- `Auth` para autenticação;
- `Home` para tela inicial;
- `Sync` para sincronização;
- `Api` para chamadas HTTP;
- `Cache` para persistência local;
- `Navigation` para troca de telas.

Não comece com dezenas de tags. Comece pelas responsabilidades que você realmente precisa investigar.

## Regra prática

Se você controla o app:

- crie uma tag por responsabilidade importante;
- use a mesma tag em todos os logs daquele fluxo;
- mantenha o nível de verbosidade sob controle;
- não misture mensagem de negócio com ruído técnico;
- não registre tokens, senhas ou payloads sensíveis.

Se você está investigando um bug:

- comece com `*:W`;
- abra uma tag por vez;
- compare o fluxo com e sem falha;
- procure a primeira tag que muda o comportamento.

## Resumo

Tags consistentes organizam o log por responsabilidade. Elas reduzem ruído, tornam o `adb logcat` filtrável e ajudam a ler o fluxo real da aplicação com muito mais rapidez.

---

Série: [[index|Guia prático de adb logcat]]  
Anterior: [[Modificadores do logcat]]  
Próximo: [[Mapeando logs Android com logcat]]
