---
title: ADB logcat - guia de estudo
draft: false
description: Guia prático para entender buffers, formatos, filtros e um roteiro de investigação com adb logcat.
tags:
  - dev
  - mobile
  - android
  - react-native
  - logcat
  - serie
socialDescription: Como estudar e usar adb logcat para ler logs Android em tempo real.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# ADB logcat - guia de estudo

`adb logcat` é a forma mais direta de ler os logs do Android pelo terminal. Ele é um atalho para `adb shell logcat` e mostra mensagens do sistema, do runtime e da aplicação em tempo real.

Série: parte 1 de 5. Próximo: [[Prioridades do logcat]]. Voltar para [[index|Guia prático de adb logcat]].

## O que aparece no log

O log no Android é organizado em buffers circulares mantidos pelo sistema. Os mais importantes no uso diário são:

- `main`: onde normalmente aparecem os logs da aplicação;
- `system`: mensagens do sistema Android;
- `crash`: stack traces e falhas;
- `radio`: eventos de telefonia e rede;
- `events`: eventos binários do sistema.

Cada mensagem normalmente traz:

- prioridade, como `V`, `D`, `I`, `W`, `E`, `F` ou `S`;
- tag, que identifica a origem da mensagem;
- PID e, em alguns formatos, TID;
- data e hora, dependendo do formato escolhido;
- mensagem registrada.

No código Android nativo, logs comuns vêm de chamadas como `Log.v`, `Log.d`, `Log.i`, `Log.w` e `Log.e`. Em React Native, é comum enxergar logs JavaScript nas tags `ReactNativeJS` e `ReactNative`.

## Comece pelo help

Use o help quando quiser confirmar as opções disponíveis no aparelho ou na versão do Android Platform Tools instalada:

```bash
adb logcat --help
```

As opções podem variar por versão do Android. O help local é a fonte mais próxima do ambiente que você está investigando.

## Escolha buffers com `-b`

Você pode ler um buffer específico ou combinar vários:

```bash
adb logcat -b main
adb logcat -b crash
adb logcat -b main -b radio -b events
adb logcat -b main,radio,events
```

Para investigação comum de app, comece por `main`. Quando houver crash, combine com `crash` para enxergar a falha e o contexto anterior.

## Filtre por tag e prioridade

O formato básico é:

```bash
adb logcat TAG:PRIORIDADE
```

Exemplos:

```bash
adb logcat ActivityManager:I MyApp:D *:S
adb logcat *:W
```

A lógica é:

- `I` mostra `Info` e acima;
- `D` mostra `Debug` e acima;
- `*:S` silencia todo o resto, transformando o comando em uma allowlist.

Para React Native, uma combinação inicial útil é:

```bash
adb logcat ReactNativeJS:V ReactNative:V *:S
```

## Escolha o formato com `-v`

O formato padrão costuma ser `threadtime`, mas é melhor ser explícito.

Formatos úteis:

- `brief`: prioridade, tag e PID;
- `long`: todos os metadados em blocos separados;
- `process`: mostra só o PID;
- `raw`: só a mensagem bruta;
- `tag`: prioridade e tag;
- `thread`: formato legado com PID e TID;
- `threadtime`: data, hora, prioridade, tag, PID e TID;
- `time`: data, hora, prioridade, tag e PID.

Exemplos:

```bash
adb logcat -v threadtime
adb logcat -v brief
adb logcat -v long
```

Use `threadtime` para investigação humana. Use `raw` quando quiser passar a mensagem para outro processamento e os metadados atrapalharem.

## Um roteiro prático de estudo

1. Abra a aplicação no emulador ou aparelho.
2. Rode `adb logcat -v threadtime`.
3. Aplique um filtro por tag da aplicação.
4. Execute uma ação simples, como login, abertura de tela ou busca de dados.
5. Identifique quais mensagens representam entrada, processamento e saída.
6. Repita com uma falha proposital.
7. Compare onde o fluxo começou a divergir.

Esse contraste entre cenário feliz e cenário com erro costuma revelar causa raiz mais rápido do que olhar apenas a stack trace.

## O que procurar

Durante a leitura, procure:

- entradas do usuário;
- chamadas de serviço;
- respostas de rede;
- persistência local;
- erros de validação;
- mudanças de tela;
- retries, timeouts e cancelamentos;
- exceções no buffer `crash`.

Quando algo quebra, olhe o que aconteceu 5 a 30 segundos antes do erro. Muitas falhas só ficam claras quando você vê a sequência que levou até elas.

## Perguntas úteis

- Qual foi a primeira mensagem útil antes do erro?
- O problema veio do app, da rede, do banco local ou do sistema?
- Houve retry, timeout ou cancelamento?
- A tela navegou para o lugar certo antes de quebrar?
- A falha foi silenciosa ou houve um `E` explícito?

## Boas práticas

- Use `D` para depuração local e reduza ruído com filtros.
- Suba o nível dos logs só quando estiver investigando um bug real.
- Remova logs verbosos antes de enviar build de produção.
- Mantenha nomes de tags consistentes entre módulos.
- Se o log estiver confuso, comece filtrando por `*:W` e vá abrindo aos poucos.

---

Série: [[index|Guia prático de adb logcat]]  
Próximo: [[Prioridades do logcat]]
