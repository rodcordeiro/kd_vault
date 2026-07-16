---
title: Canais, branches e runtimeVersion no Expo EAS
draft: false
tags:
  - dev
  - expo
  - eas
  - ota
socialDescription: Como pensar em canal, branch e runtimeVersion para reduzir risco em atualizações OTA.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# TL;DR;

Use canais para público, branches para linha de entrega e `runtimeVersion` para compatibilidade nativa.

## Configuração base

```json
{
  "expo": {
    "name": "ExampleApp",
    "slug": "example-app",
    "runtimeVersion": {
      "policy": "appVersion"
    },
    "updates": {
      "url": "https://u.expo.dev/project-id"
    }
  }
}
```

Com `policy: appVersion`, uma mudança em `version` cria uma nova runtime. Isso é fácil de operar porque o mesmo número comunica versão de loja e compatibilidade OTA.

## Perfis no EAS

```json
{
  "build": {
    "preview": {
      "channel": "preview",
      "distribution": "internal"
    },
    "production": {
      "channel": "production"
    }
  }
}
```

Um build instalado no canal `preview` recebe updates publicados para esse canal. Um build de produção não deve receber update de preview.

## Publicando update

```bash
eas update --branch preview --message "Fix order list empty state"
```

Depois de validar:

```bash
eas update --branch production --message "Fix order list empty state"
```

## Regra de segurança

Se adicionou permissão nativa, SDK nativo, plugin de config, ícone adaptativo, splash nativa ou alteração em `ios/` ou `android/`, faça build novo. OTA é para JavaScript, assets e configurações compatíveis com a runtime já instalada.

