---
title: Refs para timers, scroll e duplo submit
draft: false
tags:
  - dev
  - react-native
  - react
socialDescription: Exemplos práticos de refs para debounce, listas e proteção contra submit duplicado em React Native.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# TL;DR;

Refs são boas para guardar identificadores e instâncias: timeout, lista, input, flag de execução e controller de requisição.

## Debounce

```tsx
const timeoutRef = useRef<ReturnType<typeof setTimeout> | null>(null);

function onSearchChange(value: string) {
  setQuery(value);

  if (timeoutRef.current) clearTimeout(timeoutRef.current);

  timeoutRef.current = setTimeout(() => {
    search(value);
  }, 400);
}

useEffect(() => {
  return () => {
    if (timeoutRef.current) clearTimeout(timeoutRef.current);
  };
}, []);
```

O identificador do timer não precisa aparecer na UI. Ref é suficiente.

## Scroll imperativo

```tsx
const listRef = useRef<FlatList<Order>>(null);

function scrollToTop() {
  listRef.current?.scrollToOffset({ offset: 0, animated: true });
}

return <FlatList ref={listRef} data={orders} renderItem={renderItem} />;
```

Esse é o uso clássico: chamar uma API imperativa de um componente nativo.

## Evitar duplo submit

```tsx
const submittingRef = useRef(false);

async function submit() {
  if (submittingRef.current) return;

  submittingRef.current = true;
  try {
    await api.post("/orders");
  } finally {
    submittingRef.current = false;
  }
}
```

Isso reduz duplicidade no cliente, mas não substitui idempotência no backend. A API ainda deve rejeitar ou consolidar chamadas duplicadas.

