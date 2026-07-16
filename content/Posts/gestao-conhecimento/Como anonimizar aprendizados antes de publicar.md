---
title: Como anonimizar aprendizados antes de publicar
draft: false
tags:
  - dev
  - writing
  - security
socialDescription: Checklist para remover dados sensíveis de posts técnicos sem perder valor prático.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# TL;DR;

Anonimizar não é trocar o nome da empresa. É remover tudo que permita reconstruir sistema, cliente, incidente, volume, endpoint ou regra interna.

## Checklist

- Troque nomes reais por domínios fictícios.
- Substitua endpoints por `api.example.com`.
- Remova tokens, ids, documentos, telefones e e-mails.
- Generalize regras de negócio específicas.
- Arredonde métricas quando elas revelarem escala.
- Troque nomes de filas, bancos, tabelas e serviços.
- Reescreva logs com dados sintéticos.

## Exemplo

```txt
Antes:
POST /internal/customer/123456/sync
status=500 tenant=acme-prod document=123.456.789-00

Depois:
POST /customers/:id/sync
status=500 tenant=<tenant> document=<redacted>
```

## Código sanitizador

```ts
export function sanitizeLog(input: string) {
  return input
    .replace(/[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}/gi, "<email>")
    .replace(/\b\d{3}\.\d{3}\.\d{3}-\d{2}\b/g, "<document>")
    .replace(/Bearer\s+[A-Za-z0-9._-]+/g, "Bearer <token>")
    .replace(/\/\d+\b/g, "/:id");
}
```

Use sanitização como última barreira, não como permissão para copiar logs reais sem revisão.

