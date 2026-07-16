---
title: Checklist antes de colocar modelo no dispositivo
draft: false
tags:
  - dev
  - mobile
  - ai
  - react-native
socialDescription: Questões de produto, privacidade, performance e operação antes de distribuir modelo de IA dentro do app.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# TL;DR;

Modelo local vira parte do produto instalado. Antes de publicar, avalie tamanho, bateria, fallback, métricas e atualização.

## Checklist

- O modelo cabe no orçamento de tamanho do app?
- A inferência roda bem nos aparelhos mais fracos suportados?
- Existe fallback quando o runtime falha?
- O resultado tem confiança mínima?
- A feature funciona sem rede?
- Há telemetria sem capturar conteúdo sensível?
- O modelo pode ser atualizado sem quebrar a runtime nativa?
- Há teste em release build?

## Métricas

```ts
const startedAt = performance.now();
const result = await inference.classify(input);

metrics.track("local_ai_inference", {
  durationMs: performance.now() - startedAt,
  model: "text-classifier-v1",
  confidence: result.confidence,
});
```

Não envie o texto do usuário por padrão. Meça duração, sucesso, erro e confiança.

## Fallback

```ts
async function classifyWithFallback(input: InferenceInput) {
  try {
    return await localInference.classify(input);
  } catch {
    if (!network.isOnline) throw new Error("offline_model_unavailable");
    return remoteInference.classify(input);
  }
}
```

Fallback remoto é útil, mas muda o contrato de privacidade. Deixe isso explícito no produto e na arquitetura.

