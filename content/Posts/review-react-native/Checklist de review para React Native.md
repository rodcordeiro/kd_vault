---
title: Checklist de review para React Native
draft: false
tags:
  - dev
  - mobile
  - react-native
  - review
socialDescription: Um checklist prático para revisar apps React Native sem cair em preferências cosméticas.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# TL;DR;

Revise comportamento, risco e custo de manutenção. Estilo importa, mas não deve esconder bugs.

## Checklist

- A tela possui estado de carregamento, vazio, erro e sucesso.
- Efeitos (`useEffect`) têm dependências corretas e cancelamento quando necessário.
- Listas grandes usam `FlatList`, `FlashList` ou equivalente.
- Funções críticas tratam erro e registram contexto sanitizado.
- Componentes não misturam regra de negócio, chamada HTTP e layout complexo no mesmo arquivo.
- Inputs têm validação local e feedback claro.
- Métricas capturam duração das ações principais.

## Exemplo de separação

```tsx
export function OrderListScreen() {
  const { orders, status, refresh } = useOrders();

  if (status === "loading") return <LoadingState />;
  if (status === "error") return <ErrorState onRetry={refresh} />;
  if (orders.length === 0) return <EmptyState />;

  return <OrderList orders={orders} onRefresh={refresh} />;
}
```

A tela orquestra estados. O hook busca dados. A lista renderiza. Essa separação deixa o review mais objetivo.

## Anti-padrão comum

```tsx
useEffect(() => {
  fetch("/orders")
    .then((r) => r.json())
    .then(setOrders);
}, []);
```

Faltam timeout, erro, cancelamento, tipagem, telemetria e estado visual. Em app real, esse trecho vira dívida rapidamente.

