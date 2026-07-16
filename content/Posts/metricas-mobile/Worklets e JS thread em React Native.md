---
title: Worklets e JS thread em React Native
draft: false
tags:
  - dev
  - mobile
  - react-native
  - performance
socialDescription: Quando usar worklets para tirar animações e processamento frequente da JS thread em React Native.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# TL;DR;

Worklets ajudam quando uma rotina visual ou frequente não pode depender da JS thread. Eles não são uma saída genérica para qualquer código pesado.

## O problema

Em React Native, a JS thread executa React, handlers, chamadas de API, serialização, lógica de tela e parte da orquestração do app. Se ela fica ocupada demais, interações e animações podem parecer travadas.

Antes de mover código para worklet, meça:

- FPS durante a interação.
- Re-renderizações da tela.
- Tempo na JS thread.
- Uso de CPU no profiler nativo.

## Exemplo com animação

```tsx
import Animated, {
  useAnimatedStyle,
  useSharedValue,
  withTiming,
} from "react-native-reanimated";

export function FadeInCard() {
  const opacity = useSharedValue(0);

  const style = useAnimatedStyle(() => ({
    opacity: opacity.value,
  }));

  useEffect(() => {
    opacity.value = withTiming(1, { duration: 250 });
  }, [opacity]);

  return <Animated.View style={style} />;
}
```

O valor animado vive fora do estado React. A animação não precisa re-renderizar o componente a cada frame.

## Exemplo de worklet pequeno

```ts
const calculateProgress = (current: number, total: number) => {
  "worklet";

  if (total <= 0) return 0;
  return Math.min(1, Math.max(0, current / total));
};
```

Worklet deve ser curto, determinístico e livre de dependências complexas do mundo React.

## O que não colocar em worklet

- Chamada HTTP.
- Acesso direto a store React/Zustand.
- Telemetria executada a cada frame.
- Regra de negócio extensa.
- Serialização pesada sem medição.

## Critério

Use worklet quando o problema medido for frequência, frame, gesto ou animação. Para tela lenta por renderização, comece no Profiler React. Para CPU nativa, use Android Studio Profiler ou Instruments.

