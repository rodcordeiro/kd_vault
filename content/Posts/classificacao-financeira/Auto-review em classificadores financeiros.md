---
title: Auto-review em classificadores financeiros
draft: false
tags:
  - dev
  - nlp
  - finance
socialDescription: Como separar classificações confiáveis das que precisam de revisão humana.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# TL;DR;

Nem toda classificação deve ser aplicada automaticamente. Use limiar de confiança e fila de revisão.

## Decisão

```ts
function shouldAutoApprove(result: Classification) {
  return result.confidence >= 0.85 && result.category !== "uncategorized";
}
```

Esse limiar deve ser calibrado com dados reais e amostras revisadas.

## Criando fila de revisão

```ts
type ReviewItem = {
  transactionId: string;
  suggestedCategory: string;
  confidence: number;
  reason: string;
  status: "pending" | "approved" | "rejected";
};

export function buildReviewItem(
  transaction: Transaction,
  classification: Classification
): ReviewItem | null {
  if (shouldAutoApprove(classification)) return null;

  return {
    transactionId: transaction.id,
    suggestedCategory: classification.category,
    confidence: classification.confidence,
    reason: classification.reason,
    status: "pending",
  };
}
```

## Métricas

- Taxa de autoaprovação.
- Taxa de correção humana por categoria.
- Categorias com mais baixa confiança.
- Regras que mais geram rejeição.

Essas métricas mostram onde melhorar regras, dados de treino ou interface de revisão.

