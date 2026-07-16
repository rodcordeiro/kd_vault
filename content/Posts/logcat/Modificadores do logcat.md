---
title: Modificadores do logcat
draft: false
description: Como usar formatos e modificadores do adb logcat para melhorar leitura, captura e análise de logs.
tags:
  - dev
  - mobile
  - android
  - react-native
  - logcat
  - serie
socialDescription: Formatos e modificadores úteis do adb logcat para leitura e investigação.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# Modificadores do logcat

Os modificadores de formato do `logcat` mudam a forma como os logs aparecem no terminal. Eles não mudam o que está sendo logado, mas mudam como você enxerga a saída.

Série: parte 3 de 5. Anterior: [[Prioridades do logcat]]. Próximo: [[Tags consistentes em logcat]].

## Formato base

No `adb logcat`, formatos e modificadores são usados com `-v`:

```bash
adb logcat -v threadtime
```

Formatos base comuns:

- `brief`;
- `long`;
- `process`;
- `raw`;
- `tag`;
- `thread`;
- `threadtime`;
- `time`.

O formato decide a estrutura principal da saída. O modificador adiciona informação ou altera a representação.

## Formatos úteis

Use `threadtime` para investigação normal:

```bash
adb logcat -v threadtime
```

Ele mostra data, hora, PID, TID, prioridade, tag e mensagem. É detalhado o suficiente sem virar um bloco difícil de ler.

Use `time` quando você quer algo mais compacto:

```bash
adb logcat -v time
```

Use `raw` quando vai processar apenas a mensagem:

```bash
adb logcat -v raw
```

## Modificadores mais úteis

### `color`

Mostra cada prioridade com uma cor diferente.

Útil quando:

- você está olhando o log ao vivo;
- quer bater o olho e separar `D`, `I`, `W` e `E` mais rápido.

Exemplo:

```bash
adb logcat -v color
```

### `epoch`

Mostra o horário em segundos desde 1 de janeiro de 1970.

Útil quando:

- você quer comparar logs com outros sistemas;
- precisa de timestamp numérico para análise automatizada.

Exemplo:

```bash
adb logcat -v epoch
```

### `monotonic`

Mostra o tempo desde o último boot.

Útil quando:

- você quer medir intervalos entre eventos sem depender do relógio do sistema;
- está comparando sequências de eventos após reinícios.

Exemplo:

```bash
adb logcat -v monotonic
```

### `printable`

Escapa conteúdo binário para evitar caracteres estranhos na saída.

Útil quando:

- o buffer tem conteúdo não textual;
- você quer salvar a saída em arquivo sem bagunçar o terminal.

Exemplo:

```bash
adb logcat -b all -v printable -d
```

### `uid`

Mostra o UID ou Android ID do processo, quando o acesso permitir.

Útil quando:

- você quer diferenciar processos;
- precisa entender quem gerou a mensagem em ambientes com mais de um app.

Exemplo:

```bash
adb logcat -v uid
```

### `usec`

Mostra tempo com precisão em microssegundos.

Útil quando:

- você está comparando logs muito próximos;
- quer mais detalhe que o timestamp normal.

Exemplo:

```bash
adb logcat -v usec
```

### `UTC`, `year` e `zone`

Esses modificadores ajudam quando você vai salvar logs e comparar com servidor, pipeline ou outro fuso.

Exemplos:

```bash
adb logcat -v UTC
adb logcat -v year
adb logcat -v zone
```

### `descriptive`

Adiciona descrições de eventos do buffer `events`.

Útil quando:

- você está analisando logs binários ou eventos do sistema;
- precisa interpretar o buffer `events`.

Exemplo:

```bash
adb logcat -b events -v descriptive
```

## Como usar em React Native

Para app React Native, os formatos e modificadores mais úteis costumam ser:

- `threadtime` para leitura humana;
- `color` para leitura ao vivo;
- `year` e `zone` quando você vai arquivar logs;
- `epoch` ou `monotonic` quando quer medir sequência e tempo com mais rigor;
- `printable` quando vai redirecionar a saída para arquivo.

Exemplos práticos:

```bash
adb logcat -v threadtime ReactNativeJS:V *:S
adb logcat -v color ReactNativeJS:V ReactNative:V *:S
adb logcat -b all -v printable -d > logcat.txt
```

No PowerShell:

```powershell
adb logcat -b all -v printable -d | Out-File -Encoding utf8 .\logcat.txt
```

## Como escolher

Pergunte o que você precisa fazer:

- ler ao vivo: `color` ou `threadtime`;
- comparar tempo de eventos: `epoch` ou `monotonic`;
- salvar para análise posterior: `year`, `zone`, `printable`;
- investigar eventos do sistema: `descriptive`;
- identificar processo: `uid`.

## Resumo

Modificadores não trocam o conteúdo do log, mas mudam como você interpreta a saída. Em React Native, eles ajudam a deixar o `adb logcat` mais legível no dia a dia e mais útil quando você precisa investigar tempo, processo ou evento com mais rigor.

---

Série: [[index|Guia prático de adb logcat]]  
Anterior: [[Prioridades do logcat]]  
Próximo: [[Tags consistentes em logcat]]
