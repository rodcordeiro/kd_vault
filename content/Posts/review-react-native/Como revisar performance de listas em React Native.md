---
title: Como revisar performance de listas em React Native
draft: false
tags:
  - dev
  - mobile
  - react-native
  - performance
socialDescription: Critérios práticos para revisar listas, evitar ScrollView indevido e reduzir renderizações caras.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# TL;DR;

Se o conteúdo pode crescer, comece com lista virtualizada. `ScrollView` com `map` é aceitável para conteúdo pequeno e limitado.

## Exemplo ruim para dados variáveis

```tsx
<ScrollView>
  {orders.map((order) => (
    <OrderCard key={order.id} order={order} />
  ))}
</ScrollView>
```

Esse padrão renderiza tudo de uma vez. Com centenas de itens, aumenta uso de memória e piora o tempo até interação.

## FlatList como padrão seguro

```tsx
import { FlatList } from "react-native";

export function OrderList({ orders }: { orders: Order[] }) {
  return (
    <FlatList
      data={orders}
      keyExtractor={(item) => item.id}
      renderItem={({ item }) => <OrderCard order={item} />}
      initialNumToRender={12}
      windowSize={7}
      removeClippedSubviews
    />
  );
}
```

Não ajuste propriedades no escuro. Meça FPS, tempo de renderização e experiência real no dispositivo alvo.

## Reduzindo renderizações

```tsx
const OrderCard = React.memo(function OrderCard({ order }: { order: Order }) {
  return (
    <Pressable>
      <Text>{order.customerName}</Text>
      <Text>{formatCurrency(order.totalCents)}</Text>
    </Pressable>
  );
});
```

`React.memo` só ajuda quando a lista recebe referências estáveis. Se o pai recria objetos a cada render, o memo não resolve a causa.

