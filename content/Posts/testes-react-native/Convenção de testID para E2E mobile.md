---
title: Convenção de testID para E2E mobile
draft: false
tags:
  - dev
  - react-native
  - tests
socialDescription: Como criar testIDs estáveis para E2E sem acoplar testes ao texto, idioma ou posição visual.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# TL;DR;

`testID` deve ser estável, sem depender de texto visível, idioma ou ordem da tela.

## Convenção

```ts
export const orderTestIds = {
  createButton: "orders.create.button",
  customerInput: "order.customer.input",
  submitButton: "order.submit.button",
  successMessage: "order.success.message",
} as const;
```

Use uma hierarquia curta: `tela.elemento.ação` ou `fluxo.elemento.estado`.

## Uso

```tsx
<Pressable testID={orderTestIds.createButton} onPress={openCreateOrder}>
  <Text>Novo pedido</Text>
</Pressable>
```

## Lista dinâmica

```tsx
function getOrderItemTestId(orderId: string) {
  return `orders.item.${orderId}`;
}
```

Use identificador de dado somente quando o teste realmente precisa mirar um item específico. Para lista grande, prefira buscar por texto acessível ou estado visível quando for estável.

## O que evitar

- `button1`, `button2`, `input`.
- IDs baseados em texto traduzido.
- IDs duplicados na mesma tela.
- IDs que mudam quando o layout muda.

