---
title: Helper cn - compondo classes NativeWind
draft: false
description: Post prático para criar e usar um helper cn em uma biblioteca React Native com NativeWind, incluindo conflitos de classes, condicionais e exemplos reais.
tags:
  - dev
  - mobile
  - react-native
  - expo
  - nativewind
  - lib-mobile
  - serie
socialDescription: Como criar e usar um helper cn para compor classes NativeWind em uma biblioteca mobile compartilhada.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# Helper cn - compondo classes NativeWind

Em uma biblioteca de componentes React Native com NativeWind, quase todo componente público acaba precisando combinar classes internas com classes recebidas por props.

O problema parece pequeno:

```tsx
<Pressable className={`${base} ${disabled ? disabledStyle : ""} ${className}`} />
```

Mas esse formato escala mal. Ele deixa strings vazias no meio, dificulta condicionais, não aceita mapas de classes e não resolve conflitos como `bg-color_neutral_04` contra `bg-color_primary_01`.

Um helper `cn` resolve esse trabalho em um ponto único.

```tsx
import { cn } from "@ti_torra/mobile"

<Pressable
  className={cn(
    "items-center rounded-none border p-4",
    disabled && "!bg-color_neutral_04 !border-color_neutral_04",
    variant === "contained" && "bg-color_primary_01 border-color_primary_01",
    className,
  )}
/>
```

Neste exemplo, o componente declara suas classes base, aplica estados e variantes por condição, e ainda permite que o consumidor envie `className`.

## O contrato público

O helper deve ser exportado pela raiz do pacote:

```ts
export { cn } from "./theme/cn"
```

Assim, o app consumidor usa apenas a API pública:

```tsx
import { Button, cn } from "@ti_torra/mobile"
```

Evite importar de caminhos internos como `@ti_torra/mobile/src/theme/cn`. A biblioteca precisa poder reorganizar arquivos sem quebrar quem consome.

## Implementação completa

Esta é a implementação usada na lib:

```ts
type ClassDictionary = Record<string, unknown>;
type ClassArray = ClassValue[];
type ClassValue = string | number | ClassDictionary | ClassArray | false | null | undefined;

const borderStyles = new Set(["solid", "dashed", "dotted", "double", "hidden", "none"]);

const fontWeights = new Set([
  "thin",
  "extralight",
  "light",
  "normal",
  "medium",
  "semibold",
  "bold",
  "extrabold",
  "black",
]);

const textAlignments = new Set(["left", "center", "right", "justify", "start", "end"]);
const textSizes = new Set([
  "xs",
  "sm",
  "base",
  "md",
  "lg",
  "xl",
  "2xl",
  "3xl",
  "4xl",
  "5xl",
  "6xl",
  "7xl",
  "8xl",
  "9xl",
]);
const textWraps = new Set(["wrap", "nowrap", "balance", "pretty"]);

const spacingUtilities = new Set([
  "p",
  "px",
  "py",
  "pt",
  "pr",
  "pb",
  "pl",
  "m",
  "mx",
  "my",
  "mt",
  "mr",
  "mb",
  "ml",
]);

const borderSides = new Set(["t", "r", "b", "l", "x", "y"]);

const toValue = (mix: ClassValue): string => {
  if (typeof mix === "string" || typeof mix === "number") return String(mix);
  if (!mix || typeof mix !== "object") return "";

  let value = "";

  if (Array.isArray(mix)) {
    for (const item of mix) {
      if (!item) continue;

      const itemValue = toValue(item);
      if (!itemValue) continue;

      if (value) value += " ";
      value += itemValue;
    }

    return value;
  }

  for (const key in mix) {
    if (!mix[key]) continue;

    if (value) value += " ";
    value += key;
  }

  return value;
};

const getUtilityParts = (className: string) => {
  const parts = className.split(":");
  const utility = parts.pop() ?? "";

  return {
    modifiers: parts,
    utility,
  };
};

const stripImportant = (utility: string) => (utility.startsWith("!") ? utility.slice(1) : utility);

const isImportantClass = (className: string) => {
  const { utility } = getUtilityParts(className);

  return utility.startsWith("!");
};

const isWidthValue = (value: string) =>
  value === "px" || /^\d+$/.test(value) || value.startsWith("[");

const getTextGroup = (utility: string) => {
  if (!utility.startsWith("text-")) return null;

  const value = utility.slice("text-".length);

  if (textAlignments.has(value)) return "text-align";
  if (textSizes.has(value) || value.startsWith("[")) return "text-size";
  if (textWraps.has(value)) return "text-wrap";

  return "text-color";
};

const getBorderGroup = (utility: string) => {
  if (utility === "border") return "border-width";
  if (!utility.startsWith("border-")) return null;

  const value = utility.slice("border-".length);
  const [firstPart, ...remainingParts] = value.split("-");

  if (borderStyles.has(value)) return "border-style";
  if (isWidthValue(value)) return "border-width";

  if (borderSides.has(firstPart)) {
    const sideValue = remainingParts.join("-");

    if (!sideValue || isWidthValue(sideValue)) return `border-${firstPart}-width`;
    if (borderStyles.has(sideValue)) return `border-${firstPart}-style`;

    return `border-${firstPart}-color`;
  }

  return "border-color";
};

const getSpacingGroup = (utility: string) => {
  const [prefix] = utility.split("-");

  if (!spacingUtilities.has(prefix)) return null;

  return prefix;
};

const getFontGroup = (utility: string) => {
  if (!utility.startsWith("font-")) return null;

  const value = utility.slice("font-".length);

  if (fontWeights.has(value)) return "font-weight";

  return "font-family";
};

const getFlexGroup = (utility: string) => {
  if (utility === "flex-row" || utility === "flex-col" || utility === "flex-column") {
    return "flex-direction";
  }

  if (utility === "flex-wrap" || utility === "flex-nowrap" || utility === "flex-wrap-reverse") {
    return "flex-wrap";
  }

  return null;
};

const getConflictGroup = (className: string) => {
  const { modifiers, utility } = getUtilityParts(className);
  const normalizedUtility = stripImportant(utility);
  let group: string | null = null;

  if (normalizedUtility.startsWith("bg-")) group = "background-color";
  group ??= getBorderGroup(normalizedUtility);
  group ??= getTextGroup(normalizedUtility);
  group ??= getFontGroup(normalizedUtility);
  group ??= getSpacingGroup(normalizedUtility);
  group ??= getFlexGroup(normalizedUtility);

  if (!group) return null;

  return `${modifiers.join(":")}:${group}`;
};

const mergeConflictingClasses = (classNames: string[]) => {
  const resolvedClasses: string[] = [];
  const indexByConflictGroup = new Map<string, number>();

  for (const className of classNames) {
    const conflictGroup = getConflictGroup(className);

    if (!conflictGroup) {
      resolvedClasses.push(className);
      continue;
    }

    const currentIndex = indexByConflictGroup.get(conflictGroup);
    if (currentIndex !== undefined) {
      const currentClass = resolvedClasses[currentIndex];

      if (isImportantClass(currentClass) && !isImportantClass(className)) {
        continue;
      }

      resolvedClasses[currentIndex] = "";
    }

    indexByConflictGroup.set(conflictGroup, resolvedClasses.length);
    resolvedClasses.push(className);
  }

  return resolvedClasses.filter(Boolean).join(" ");
};

/**
 * Composes NativeWind class names from strings, arrays and object maps.
 *
 * Falsy values are ignored and conflicting utilities are merged so the latest
 * class wins, except when an existing class uses Tailwind's important prefix.
 */
export const cn = (...classes: ClassValue[]) => {
  const classNames: string[] = [];

  for (const classValue of classes) {
    if (!classValue) continue;

    const value = toValue(classValue);
    if (!value) continue;

    classNames.push(...value.trim().split(/\s+/).filter(Boolean));
  }

  return mergeConflictingClasses(classNames);
};
```

## Como o código funciona

O tipo `ClassValue` define tudo que o helper aceita:

```ts
type ClassValue = string | number | ClassDictionary | ClassArray | false | null | undefined
```

Isso permite usar o mesmo estilo mental do `clsx`:

```ts
cn("base", false, null, undefined, "active")
// "base active"

cn("base", ["px-4", ["py-2"]], {
  "opacity-50": true,
  hidden: false,
})
// "base px-4 py-2 opacity-50"
```

A função `toValue` normaliza cada entrada para string. Strings e números entram direto. Valores falsy somem. Arrays são percorridos recursivamente. Objetos viram uma lista das chaves cujo valor é verdadeiro.

Depois disso, o `cn` quebra tudo por espaço:

```ts
classNames.push(...value.trim().split(/\s+/).filter(Boolean))
```

Assim, tanto `cn("px-4 py-2")` quanto `cn("px-4", "py-2")` chegam ao merge como a mesma lista de classes.

## Conflitos

O diferencial deste helper é resolver conflitos comuns de Tailwind/NativeWind. A regra é simples: para o mesmo grupo, a última classe vence.

```ts
cn("bg-color_neutral_04", "bg-color_primary_01")
// "bg-color_primary_01"

cn("font-semibold", "font-bold")
// "font-bold"
```

Para isso, `getConflictGroup` converte uma classe em um grupo lógico:

- `bg-*` vira `background-color`;
- `border`, `border-2`, `border-color_primary_01` e variantes laterais entram em grupos de borda;
- `text-md`, `text-center` e `text-color_neutral_05` ficam em grupos separados;
- `font-semibold` e `font-bold` viram `font-weight`;
- `p-4`, `px-2`, `mt-3` entram em grupos de espaçamento;
- `flex-row` e `flex-col` entram em `flex-direction`.

Separar grupos de texto é importante. Estas classes não devem se cancelar:

```ts
cn("text-md font-semibold text-center text-color_primary_01")
// "text-md font-semibold text-center text-color_primary_01"
```

Elas representam tamanho, peso, alinhamento e cor.

## Estados e modificadores

Classes com modificadores ficam isoladas por estado:

```ts
cn("bg-white", "disabled:bg-gray-300", "disabled:bg-gray-500")
// "bg-white disabled:bg-gray-500"
```

`bg-white` pertence ao grupo `:background-color`. Já `disabled:bg-gray-300` pertence ao grupo `disabled:background-color`. Por isso, uma classe normal não apaga uma classe de estado.

## Prefixo importante

Quando a classe existente usa o prefixo importante do Tailwind, uma classe posterior normal não consegue sobrescrever:

```ts
cn("!bg-color_neutral_04", "bg-color_primary_01")
// "!bg-color_neutral_04"
```

Esse detalhe é útil em componentes com estado desabilitado:

```ts
cn(
  "items-center rounded-none shadow-md p-4 border",
  "!bg-color_neutral_04 !border-color_neutral_04 text-white",
  "bg-color_primary_01 text-white border-color_primary_01",
)
// "items-center rounded-none shadow-md p-4 border !bg-color_neutral_04 !border-color_neutral_04 text-white"
```

Mesmo que a variante venha depois, o estado desabilitado continua mandando no background e na borda.

## Uso em um Button

Em um componente público, prefira guardar estilos em um objeto e compor no JSX:

```tsx
type ButtonVariant = "contained" | "outlined" | "text" | "secondary"

type ButtonProps = {
  title: string
  variant?: ButtonVariant
  loading?: boolean
  className?: string
} & PressableProps

export function Button({
  title,
  variant = "contained",
  loading = false,
  className,
  disabled,
  ...props
}: ButtonProps) {
  const isDisabled = disabled || loading

  return (
    <Pressable
      {...props}
      disabled={isDisabled}
      accessibilityRole="button"
      accessibilityState={{ disabled: isDisabled, busy: loading }}
      className={cn(
        styles.base,
        isDisabled && styles.disabled,
        variant === "contained" && styles.contained,
        variant === "secondary" && styles.secondary,
        variant === "outlined" && styles.outlined,
        variant === "text" && styles.text,
        className,
      )}
    >
      <Text
        className={cn(
          styles.textBase,
          isDisabled && styles.textDisabled,
          (variant === "contained" || variant === "secondary") && styles.textContained,
          variant === "outlined" && styles.textOutlined,
          variant === "text" && styles.textOnly,
        )}
      >
        {title}
      </Text>
    </Pressable>
  )
}

const styles = {
  base: "items-center rounded-none shadow-md p-4 border",
  contained: "bg-color_primary_01 text-white border-color_primary_01",
  secondary: "bg-color_primary_02 text-white border-color_primary_02",
  outlined: "border-color_primary_01 border-2",
  text: "bg-transparent shadow-none border-0",
  textBase: "text-md font-semibold text-center",
  disabled: "!bg-color_neutral_04 !border-color_neutral_04 text-white",
  textDisabled: "text-white",
  textContained: "text-white",
  textOutlined: "text-color_primary_01",
  textOnly: "text-color_primary_01 font-semibold",
}
```

O componente fica previsível:

- a base sempre entra;
- estados entram por booleanos;
- variantes entram por comparação explícita;
- `className` fica por último para permitir extensão pelo consumidor;
- classes importantes protegem estados que não devem ser sobrescritos por uma variante comum.

## Uso no app consumidor

O app pode importar o helper para montar classes locais com a mesma regra da biblioteca:

```tsx
import { cn } from "@ti_torra/mobile"

type StatusPillProps = {
  status: "success" | "warning" | "error"
  label: string
}

export function StatusPill({ status, label }: StatusPillProps) {
  return (
    <View
      className={cn(
        "self-start rounded-full border px-3 py-1",
        status === "success" && "border-green-700 bg-green-100",
        status === "warning" && "border-yellow-700 bg-yellow-100",
        status === "error" && "border-red-700 bg-red-100",
      )}
    >
      <Text
        className={cn(
          "text-sm font-semibold",
          status === "success" && "text-green-800",
          status === "warning" && "text-yellow-800",
          status === "error" && "text-red-800",
        )}
      >
        {label}
      </Text>
    </View>
  )
}
```

Esse componente não precisa conhecer a implementação interna da lib. Ele só usa o contrato público.

## Testes mínimos

Teste o helper como unidade pura. Os casos mais importantes são:

```ts
import { cn } from "./cn"

describe("cn", () => {
  it("joins class names with spaces", () => {
    expect(cn("flex", "items-center", "gap-2")).toBe("flex items-center gap-2")
  })

  it("ignores falsy conditional class names", () => {
    expect(cn("base", false, null, undefined, "active")).toBe("base active")
  })

  it("supports arrays and object maps", () => {
    expect(
      cn("base", ["px-4", ["py-2"]], {
        "opacity-50": true,
        hidden: false,
      }),
    ).toBe("base px-4 py-2 opacity-50")
  })

  it("keeps the last background class for the same state", () => {
    expect(cn("bg-color_neutral_04", "bg-color_primary_01")).toBe("bg-color_primary_01")
  })

  it("preserves important classes over later non-important classes", () => {
    expect(cn("!bg-color_neutral_04", "bg-color_primary_01")).toBe("!bg-color_neutral_04")
  })

  it("keeps modifier conflicts isolated by state", () => {
    expect(cn("bg-white", "disabled:bg-gray-300", "disabled:bg-gray-500")).toBe(
      "bg-white disabled:bg-gray-500",
    )
  })
})
```

## Quando usar

Use `cn` quando:

- houver classes condicionais;
- o componente aceitar `className`;
- houver variantes visuais;
- estados como `disabled`, `loading` ou `selected` alterarem classes;
- você precisar evitar conflitos previsíveis de spacing, cor, borda, texto, fonte ou flex.

Não use `cn` para esconder regra de negócio. Ele deve compor classe visual, não decidir fluxo, permissão ou estado de domínio.

---

Série: [[index|Guia da lib mobile]]  
Anterior: [[Glossário e decisões - Biblioteca mobile]]
