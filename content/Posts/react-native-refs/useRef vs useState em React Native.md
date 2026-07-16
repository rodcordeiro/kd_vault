---
title: useRef vs useState em React Native
draft: false
tags:
  - dev
  - react-native
  - react
socialDescription: Como decidir entre useRef e useState sem esconder estado de UI em uma caixa mutável.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# TL;DR;

Use `useState` quando a UI precisa reagir. Use `useRef` quando você precisa guardar um valor imperativo que não deve redesenhar a tela.

## Modelo mental

```tsx
const attemptsRef = useRef(0);

function retry() {
  attemptsRef.current += 1;
  requestAgain();
}
```

`attemptsRef` é o mesmo objeto durante a vida do componente. Mudar `current` não agenda renderização.

## Quando usar state

```tsx
const [isLoading, setIsLoading] = useState(false);

async function submit() {
  setIsLoading(true);
  try {
    await api.post("/orders");
  } finally {
    setIsLoading(false);
  }
}
```

`isLoading` altera a tela: botão desabilitado, spinner, acessibilidade e testes. Ele deve ser reativo.

## Quando usar ref

```tsx
const requestIdRef = useRef(0);

async function loadItems() {
  const requestId = ++requestIdRef.current;
  const response = await api.get("/items");

  if (requestId !== requestIdRef.current) return;

  setItems(response.data);
}
```

Aqui a ref guarda a requisição mais recente. A tela não precisa renderizar a cada incremento; ela só precisa evitar aplicar resposta antiga.

## Regra prática

Se o usuário deve ver a mudança, use estado. Se o valor é apoio operacional para foco, timer, listener, concorrência ou API imperativa, use ref.

