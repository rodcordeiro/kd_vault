---
title: Profilando ANR e travamentos em Android
draft: false
tags:
  - dev
  - mobile
  - android
  - react-native
socialDescription: Um roteiro para investigar ANR, travamentos e bloqueios de thread em apps React Native no Android.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# TL;DR;

ANR é sintoma de thread bloqueada ou trabalho demorado demais. Em React Native, investigue UI thread, JS thread, chamadas nativas e lifecycle.

## Comece pelo recorte

Registre no bug:

- versão do app;
- dispositivo e Android;
- tela e ação;
- rede;
- tempo até travar;
- se ocorreu em debug, release ou build interno.

Sem recorte, o log vira ruído.

## Logcat inicial

```bash
adb logcat -v threadtime ActivityManager:E AndroidRuntime:E ReactNative:W ReactNativeJS:W *:S
```

Para uma captura mais ampla:

```bash
adb logcat -v threadtime | grep -E "ANR|ReactNative|ReactNativeJS|Choreographer|InputDispatcher"
```

No PowerShell:

```powershell
adb logcat -v threadtime |
  Select-String -Pattern "ANR|ReactNative|ReactNativeJS|Choreographer|InputDispatcher"
```

## O que procurar

- `Input dispatching timed out`.
- `Application Not Responding`.
- Exceções em `ReactNativeJS`.
- Picos de `Choreographer` indicando frames pulados.
- Stack traces envolvendo módulo nativo, storage, banco local ou serialização.

## Próximo passo

Se o log aponta JS thread ocupada, use React DevTools Profiler e medições de FPS. Se aponta main thread ou módulo nativo, grave um perfil no Android Studio Profiler e olhe flame graph, thread e duração.

## Regra prática

Não trate ANR como "erro do Android". Trate como investigação de thread: quem ficou ocupado, por quanto tempo e em qual fluxo.

