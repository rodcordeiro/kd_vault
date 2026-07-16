---
title: Pipeline simples para classificar lançamentos financeiros
draft: false
tags:
  - dev
  - nlp
  - finance
socialDescription: Um pipeline prático para normalizar descrição, aplicar regras e retornar categoria com confiança.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# TL;DR;

Comece com normalização e regras explícitas. Só depois adicione modelo estatístico ou LLM.

## Modelo de entrada

```ts
type Transaction = {
  id: string;
  description: string;
  amountCents: number;
  occurredAt: string;
};

type Classification = {
  category: string;
  confidence: number;
  reason: string;
};
```

## Normalização

```ts
function normalizeDescription(value: string) {
  return value
    .normalize("NFD")
    .replace(/\p{Diacritic}/gu, "")
    .toLowerCase()
    .replace(/\d+/g, " ")
    .replace(/\s+/g, " ")
    .trim();
}
```

Normalizar reduz variação irrelevante e melhora regras simples.

## Regras

```ts
const rules = [
  { pattern: /\b(supermercado|mercado|grocery)\b/, category: "groceries" },
  { pattern: /\b(farmacia|pharmacy|drugstore)\b/, category: "health" },
  { pattern: /\b(uber|taxi|metro|bus)\b/, category: "transport" },
];

export function classify(transaction: Transaction): Classification {
  const text = normalizeDescription(transaction.description);
  const rule = rules.find((item) => item.pattern.test(text));

  if (rule) {
    return {
      category: rule.category,
      confidence: 0.9,
      reason: `matched rule ${rule.pattern}`,
    };
  }

  return {
    category: "uncategorized",
    confidence: 0.1,
    reason: "no rule matched",
  };
}
```

O resultado carrega `reason` para revisão. Sem explicação, fica difícil melhorar o classificador.

