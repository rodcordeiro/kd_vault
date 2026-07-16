---
title: Pirâmide de testes para apps React Native
draft: false
tags:
  - dev
  - react-native
  - tests
socialDescription: Como distribuir testes de unidade, integração e E2E em um app React Native.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# TL;DR;

Tenha muitos testes baratos, alguns testes de fluxo local e poucos E2E realmente críticos.

## Distribuição

- Unidade: funções puras, formatadores, validações, reducers e regras.
- Integração: componentes com hooks, loading, erro, vazio e sucesso.
- E2E: login, fluxo principal, pagamento, sincronização ou qualquer jornada de alto risco.

## Exemplo de unidade

```ts
export function canSubmitOrder(items: OrderItem[]) {
  return items.length > 0 && items.every((item) => item.quantity > 0);
}

it("rejects an empty order", () => {
  expect(canSubmitOrder([])).toBe(false);
});
```

## Exemplo de integração

```tsx
it("shows an empty state", async () => {
  render(<OrderListScreen repository={fakeRepository.withOrders([])} />);

  expect(await screen.findByText("Nenhum pedido encontrado")).toBeTruthy();
});
```

## Exemplo de E2E

```ts
it("creates an order", async () => {
  await element(by.id("orders.create.button")).tap();
  await element(by.id("order.customer.input")).typeText("Cliente Exemplo");
  await element(by.id("order.submit.button")).tap();

  await expect(element(by.id("order.success.message"))).toBeVisible();
});
```

E2E deve provar que a jornada crítica funciona. Ele não deve carregar todo detalhe de regra que poderia estar em teste menor.

