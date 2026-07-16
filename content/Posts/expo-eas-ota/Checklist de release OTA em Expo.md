---
title: Checklist de release OTA em Expo
draft: false
tags:
  - dev
  - expo
  - release
  - ota
socialDescription: Um checklist objetivo para publicar update OTA com validação e rollback planejado.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# TL;DR;

Antes de publicar OTA, confirme runtime, fluxo afetado, smoke test e plano de rollback.

## Checklist

- A mudança não exige código nativo.
- O app foi testado em build de release ou preview, não apenas no Expo Go.
- O update foi publicado primeiro em canal interno.
- O fluxo afetado tem smoke test manual claro.
- O monitoramento tem versão, canal e runtime nos eventos.
- Existe comando de rollback documentado.

## Marcando versão nos eventos

```ts
import Constants from "expo-constants";
import * as Sentry from "@sentry/react-native";

Sentry.setTags({
  appVersion: Constants.expoConfig?.version ?? "unknown",
  runtimeVersion: String(Constants.expoConfig?.runtimeVersion ?? "unknown"),
  updateChannel: Constants.expoConfig?.updates?.url ? "ota-enabled" : "local",
});
```

Em produção, prefira ler metadados reais do update quando disponíveis na versão do SDK usada. O ponto é garantir que toda falha diga qual pacote JS estava rodando.

## Smoke test mínimo

```md
## Smoke OTA

- Abrir app frio
- Fazer login
- Abrir tela alterada
- Executar ação principal
- Fechar e abrir app novamente
- Conferir evento de versão no monitoramento
```

## Rollback

```bash
eas update:republish --branch production --group <previous-update-group-id>
```

Rollback não substitui teste. Ele reduz o tempo de exposição quando uma falha passa pelo processo.

