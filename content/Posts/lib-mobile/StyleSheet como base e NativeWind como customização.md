---
title: StyleSheet como base e NativeWind como customização
draft: false
description: Como dividir StyleSheet e NativeWind em uma biblioteca de componentes React Native.
tags:
  - dev
  - mobile
  - react-native
  - nativewind
  - lib-mobile
  - serie
socialDescription: Use StyleSheet para a base estrutural dos componentes e NativeWind para layout e customizações do consumidor.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# TL;DR;

Em biblioteca de componentes React Native, `StyleSheet` é a melhor base para estrutura interna. NativeWind continua útil, mas como camada de layout, composição e customização exposta ao consumidor.

## A divisão recomendada

Use `StyleSheet` para:

- dimensões mínimas;
- alinhamento estrutural;
- bordas e cores essenciais do contrato;
- estados críticos como `disabled`, `loading` e erro;
- estilos usados por APIs nativas, SVGs, modais e medições;
- qualquer estilo que não pode depender do tema compilado pelo app consumidor.

Use NativeWind para:

- layout externo;
- espaçamento de tela;
- ajustes visuais do consumidor;
- composição via `className`;
- classes utilitárias em componentes locais do app;
- customizações que podem variar entre produtos.

## Por que isso importa

Uma lib compartilhada roda dentro de apps consumidores diferentes. O app pode ter tema NativeWind, aliases, build, cache e configuração próprios. Se a estrutura base do componente depende demais de classes customizadas, o componente fica mais sensível ao ambiente onde é instalado.

Com `StyleSheet`, a base viaja junto com o componente.

## Exemplo de Button

```tsx
import { Pressable, PressableProps, StyleSheet, Text } from "react-native"
import { cn } from "@acme/mobile-ui"

type ButtonProps = PressableProps & {
  title: string
  className?: string
}

export function Button({ title, className, style, ...props }: ButtonProps) {
  return (
    <Pressable
      {...props}
      accessibilityRole="button"
      style={[styles.base, props.disabled && styles.disabled, style]}
      className={cn("self-stretch", className)}
    >
      <Text style={[styles.label, props.disabled && styles.disabledLabel]}>
        {title}
      </Text>
    </Pressable>
  )
}

const styles = StyleSheet.create({
  base: {
    minHeight: 52,
    alignItems: "center",
    justifyContent: "center",
    borderRadius: 4,
    paddingHorizontal: 16,
    paddingVertical: 12,
    backgroundColor: "#2563eb",
  },
  disabled: {
    backgroundColor: "#9ca3af",
  },
  label: {
    color: "#ffffff",
    fontSize: 16,
    fontWeight: "600",
  },
  disabledLabel: {
    color: "#f9fafb",
  },
})
```

O consumidor ainda pode ajustar layout:

```tsx
<Button title="Salvar" className="mt-4 self-center" />
```

Mas a altura mínima, alinhamento, cor base e estado desabilitado não dependem de uma classe customizada existir no app.

## Onde entra o cn

`cn` continua sendo o helper certo para compor `className`:

```tsx
className={cn(
  "self-stretch",
  compact && "mx-2",
  selected && "opacity-90",
  className,
)}
```

Ele resolve condicionais e conflitos da camada NativeWind. Só não deve ser tratado como substituto da base estrutural do componente público.

## Regra prática

Se quebrar a classe NativeWind quebra o componente, o estilo deveria estar em `StyleSheet`. Se quebrar a classe NativeWind apenas perde um ajuste de layout/customização, ela está no lugar certo.

---

Série: [[index|Guia da lib mobile]]  
Anterior: [[Helper cn - compondo classes NativeWind]]
