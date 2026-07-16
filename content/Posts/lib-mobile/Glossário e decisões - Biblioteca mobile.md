---
title: Glossário e decisões da biblioteca mobile
draft: false
description: Vocabulário e decisões arquiteturais para orientar contratos públicos de uma biblioteca mobile compartilhada.
tags:
  - dev
  - mobile
  - react-native
  - expo
  - lib-mobile
  - arquitetura
  - serie
socialDescription: Vocabulário e decisões arquiteturais para orientar contratos públicos de uma biblioteca mobile compartilhada.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# Glossário e decisões - Biblioteca mobile

Vocabulário e decisão arquitetural usados em [[Guia - Criando uma biblioteca de componentes mobile]] e [[Roteiro verificável - Biblioteca mobile]].

Série: parte 3 de 5. Anterior: [[Roteiro verificável - Biblioteca mobile]]. Próximo: [[Helper cn - compondo classes NativeWind]].

## Linguagem

**Biblioteca mobile compartilhada**:
Pacote de contratos reutilizáveis por mais de uma aplicação React Native, sem possuir fluxos de negócio dos consumidores.
_Evitar_: app base, pacote de telas, repositório de código comum.

**API pública**:
Conjunto estável de componentes, funções, hooks, tokens e tipos importados exclusivamente da raiz do pacote.
_Evitar_: barrel interno, caminho profundo, arquivo exportado por conveniência.

**Componente puro**:
Componente visual configurado por props e livre de autenticação, API, persistência, telemetria, navegação específica e vocabulário de negócio.
_Evitar_: componente comum, componente genérico sem critérios.

**Função genérica**:
Transformação reutilizável que não depende de estado, infraestrutura ou regras exclusivas de um app.
_Evitar_: helper de negócio, utilitário local promovido sem prova de reuso.

**Hook genérico**:
Hook cujo contrato representa uma capacidade transversal e não conhece stores, endpoints, telas ou providers específicos do consumidor.
_Evitar_: hook compartilhado apenas porque foi copiado.

**Token visual**:
Nome estável para uma decisão de cor, tipografia, espaçamento ou outra propriedade do design system.
_Evitar_: constante de estilo, valor mágico.

**Wrapper nativo**:
Contrato reutilizável sobre uma capacidade nativa cuja instalação e configuração continuam explícitas no app consumidor.
_Evitar_: abstração transparente, dependência interna invisível.

**Peer dependency**:
Dependência que deve ser fornecida pelo app consumidor para manter uma única instância e compatibilidade de runtime.
_Evitar_: dependência opcional quando ela é obrigatória para o contrato.

**App piloto**:
Aplicação real usada para provar instalação, tipagem, renderização e integração da versão publicada da biblioteca.
_Evitar_: playground como única validação, app exemplo como prova de produção.

**Contrato público**:
Comportamento, props, tipos, exports e requisitos de integração nos quais os consumidores podem confiar entre versões compatíveis.
_Evitar_: implementação interna, detalhe de pasta.

## Decisão arquitetural: dependências nativas permanecem explícitas no consumidor

Status: aceita.

Uma biblioteca mobile pode expor wrappers para câmera, áudio e outras capacidades nativas, mas o app consumidor continua responsável por instalar versões compatíveis, declarar plugins e manter configurações de plataforma. A alternativa de esconder essas dependências simplificaria a aparência da API, porém criaria falhas de build difíceis de diagnosticar e acoplamento operacional invisível. Como consequência, todo wrapper nativo deve documentar seus peers, plugins, permissões e validação mínima em device.

## Limites derivados

- A biblioteca não aplica plugins Expo pelo consumidor.
- Um wrapper nativo não entra no MVP sem necessidade comprovada.
- Código de auth, API, sync, SQLite e telemetria permanece no app.
- Componentes públicos usam `StyleSheet` como base estrutural quando o estilo precisa ser estável, nativo ou independente do tema compilado pelo consumidor.
- NativeWind fica reservado para layout geral, customizações via `className` e composição visual não crítica.
- Uma classe NativeWind customizada depende do tema compilado pelo consumidor; tokens com `StyleSheet` são a saída segura quando essa garantia não existe.
- App exemplo acelera desenvolvimento, mas app piloto valida o contrato real de distribuição.

---

Série: [[index|Guia da lib mobile]]
Anterior: [[Roteiro verificável - Biblioteca mobile]]
Próximo: [[Helper cn - compondo classes NativeWind]]
