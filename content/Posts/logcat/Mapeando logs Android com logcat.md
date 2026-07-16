---
title: Mapeando logs Android com logcat
draft: false
description: Comandos práticos para capturar, filtrar e salvar logs Android durante investigações mobile.
tags:
  - dev
  - mobile
  - android
  - react-native
  - logcat
  - serie
socialDescription: Roteiro prático para mapear logs Android com adb logcat, PowerShell e filtros de investigação.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# Mapeando logs Android com logcat

Depois de entender prioridades, modificadores e tags, o próximo passo é montar comandos reutilizáveis para investigação.

Série: parte 5 de 5. Anterior: [[Tags consistentes em logcat]]. Voltar para [[index|Guia prático de adb logcat]].

## Quando mapear logs

Mapeie logs quando você precisa responder:

- o fluxo quebrou em qual etapa?
- houve ANR, crash, erro JavaScript ou falha nativa?
- Datadog, SDKs ou observabilidade estão registrando o evento?
- o problema aparece antes ou depois da navegação?
- o JS thread travou, atrasou ou continuou emitindo logs?

Mapear não é só salvar tudo. É capturar o suficiente para comparar causa, contexto e sequência.

## Captura básica ao vivo

Para ler o fluxo com horário:

```bash
adb logcat -v time
```

Para React Native:

```bash
adb logcat -v time ReactNativeJS:V ReactNative:V *:S
```

Para ver apenas avisos e erros:

```bash
adb logcat *:W
```

## Captura com PowerShell

Para filtrar termos de investigação e salvar em arquivo:

```powershell
adb logcat -v time -v color |
  Select-String -Pattern "ANR|ReactNative|DdSdk|Datadog|JS thread" |
  Out-File -Encoding utf8 -FilePath .\logcat_contagem.log
```

Esse comando procura sinais úteis em apps React Native com observabilidade:

- `ANR`;
- `ReactNative`;
- `DdSdk`;
- `Datadog`;
- `JS thread`.

Se preferir sem cor para deixar o arquivo mais limpo:

```powershell
adb logcat -v time |
  Select-String -Pattern "ANR|ReactNative|DdSdk|Datadog|JS thread" |
  Out-File -Encoding utf8 -FilePath .\logcat_contagem.log
```

## Filtro com allowlist

Para deixar passar apenas tags conhecidas:

```bash
adb logcat -v time -v descriptive ReactNative:D ReactNativeJS:D JSThread:D *:S
```

Use `*:S` para silenciar todo o resto. Sem isso, o log pode continuar trazendo mensagens fora do foco.

## Capturar snapshot

Para capturar o log atual e encerrar:

```bash
adb logcat -d > logcat.txt
```

No PowerShell:

```powershell
adb logcat -d | Out-File -Encoding utf8 .\logcat.txt
```

Para incluir todos os buffers e tornar a saída mais segura para arquivo:

```powershell
adb logcat -b all -v printable -d | Out-File -Encoding utf8 .\logcat_all.txt
```

## Limpar antes de reproduzir

Quando você quer reproduzir um bug com menos ruído, limpe o buffer antes:

```bash
adb logcat -c
```

Depois execute o fluxo problemático e capture:

```powershell
adb logcat -v threadtime |
  Select-String -Pattern "ReactNative|ReactNativeJS|ANR|Exception|Error|Datadog" |
  Out-File -Encoding utf8 .\repro.log
```

Essa sequência ajuda a separar o que aconteceu durante a reprodução do que já estava no buffer antes.

## Roteiro de investigação

1. Limpe o buffer com `adb logcat -c`.
2. Inicie uma captura com `threadtime` ou `time`.
3. Reproduza exatamente o fluxo.
4. Pare a captura.
5. Procure a primeira mensagem diferente do cenário feliz.
6. Compare tags, prioridades e timestamps.
7. Se houver crash, capture também `-b crash`.

Exemplo para crash:

```powershell
adb logcat -b main -b crash -v threadtime -d |
  Out-File -Encoding utf8 .\crash_context.log
```

## O que anotar junto do arquivo

Um arquivo de log sozinho envelhece mal. Anote junto:

- aparelho ou emulador usado;
- versão do app;
- horário aproximado da reprodução;
- usuário ou ambiente, sem dados sensíveis;
- fluxo executado;
- comportamento esperado;
- comportamento observado;
- comando usado para capturar.

Essa informação torna o log útil para outra pessoa ou para você mesmo alguns dias depois.

## Cuidado com dados sensíveis

Antes de compartilhar logs, procure e remova:

- tokens;
- cookies;
- senhas;
- CPF, CNPJ, e-mail e telefone;
- payloads completos de requisição;
- headers de autenticação.

Log é evidência técnica, mas também pode carregar dado sensível.

## Resumo

Mapear logs com `adb logcat` é transformar uma falha em uma sequência investigável. Comece com pouco ruído, filtre por tags e termos relevantes, salve o comando usado e compare o fluxo com uma execução saudável.

---

Série: [[index|Guia prático de adb logcat]]  
Anterior: [[Tags consistentes em logcat]]
