---
title: Testando loading, erro e vazio com RNTL
draft: false
tags:
  - dev
  - react-native
  - tests
socialDescription: Como cobrir estados principais de uma tela React Native com React Native Testing Library.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# TL;DR;

Toda tela de dados deveria ter testes para loading, erro, vazio e sucesso.

## Componente testável

```tsx
export function OrderListScreen({ repository }: Props) {
  const { status, orders, reload } = useOrders(repository);

  if (status === "loading") return <Text>Carregando...</Text>;
  if (status === "error") return <ErrorState onRetry={reload} />;
  if (orders.length === 0) return <Text>Nenhum pedido encontrado</Text>;

  return <OrderList orders={orders} />;
}
```

## Loading

```tsx
it("shows loading state", () => {
  render(<OrderListScreen repository={fakeRepository.pending()} />);

  expect(screen.getByText("Carregando...")).toBeTruthy();
});
```

## Erro

```tsx
it("allows retry after error", async () => {
  const repository = fakeRepository.failOnceThenReturn([]);

  render(<OrderListScreen repository={repository} />);

  fireEvent.press(await screen.findByText("Tentar novamente"));

  expect(await screen.findByText("Nenhum pedido encontrado")).toBeTruthy();
});
```

## Sucesso

```tsx
it("renders orders", async () => {
  render(<OrderListScreen repository={fakeRepository.withOrders([
    { id: "1", customerName: "Cliente Exemplo" },
  ])} />);

  expect(await screen.findByText("Cliente Exemplo")).toBeTruthy();
});
```

Teste comportamento percebido pelo usuário. Evite testar state interno, nome de hook ou detalhe de implementação.

