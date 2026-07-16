---
title: Arquitetura para features offline com IA local
draft: false
tags:
  - dev
  - mobile
  - react-native
  - ai
socialDescription: Um desenho simples para executar IA local em React Native sem misturar UI, modelo e sincronização.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# TL;DR;

Separe UI, serviço de inferência, cache local e sincronização. O modelo não deve conhecer tela, navegação ou API.

## Desenho

```txt
Screen
  -> useLocalAssistant
  -> InferenceService
  -> ModelRuntime
  -> LocalStore
  -> SyncQueue
```

## Contrato

```ts
type InferenceInput = {
  text: string;
  locale: string;
};

type InferenceResult = {
  label: string;
  confidence: number;
  explanation?: string;
};

export interface InferenceService {
  classify(input: InferenceInput): Promise<InferenceResult>;
}
```

## Hook

```tsx
export function useLocalClassification(service: InferenceService) {
  const [status, setStatus] = useState<"idle" | "running" | "done" | "error">("idle");
  const [result, setResult] = useState<InferenceResult | null>(null);

  async function run(text: string) {
    setStatus("running");
    try {
      const output = await service.classify({ text, locale: "pt-BR" });
      setResult(output);
      setStatus("done");
    } catch {
      setStatus("error");
    }
  }

  return { status, result, run };
}
```

## Sincronização

Se o resultado local precisa chegar ao backend depois, trate como offline-first: grave localmente, coloque na fila e envie quando houver rede. Não acople inferência diretamente à chamada HTTP.

