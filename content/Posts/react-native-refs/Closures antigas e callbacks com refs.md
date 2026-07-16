---
title: Closures antigas e callbacks com refs
draft: false
tags:
  - dev
  - react
  - react-native
socialDescription: Como evitar callbacks lendo valores antigos em headers, listeners e integrações nativas.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# TL;DR;

Cada render cria novas variáveis locais. Um callback pode fechar sobre valores antigos se ele for registrado uma vez e executado depois.

## O problema

```tsx
function SearchScreen() {
  const [query, setQuery] = useState("");

  useEffect(() => {
    navigation.setOptions({
      headerRight: () => <Button title="Buscar" onPress={() => search(query)} />,
    });
  }, []);

  return <TextInput value={query} onChangeText={setQuery} />;
}
```

O `onPress` pode ficar preso ao `query` inicial porque o header foi configurado apenas uma vez.

## Ref com valor mais recente

```tsx
function SearchScreen() {
  const [query, setQuery] = useState("");
  const queryRef = useRef(query);

  useEffect(() => {
    queryRef.current = query;
  }, [query]);

  useEffect(() => {
    navigation.setOptions({
      headerRight: () => (
        <Button title="Buscar" onPress={() => search(queryRef.current)} />
      ),
    });
  }, [navigation]);

  return <TextInput value={query} onChangeText={setQuery} />;
}
```

A UI continua usando `useState`. A ref só resolve a ponte imperativa com um callback que pode rodar fora do ciclo normal da tela.

## Onde isso aparece

- Header de navegação.
- `AppState` e listeners nativos.
- Callbacks de câmera, scanner e geolocalização.
- WebSocket, fila local e eventos externos.
- Timers e debounce.

Não use ref para esconder regra de negócio. Use ref para manter callback externo lendo o valor atual.

