---
title: Criando uma biblioteca de componentes mobile
draft: false
description: Guia prático para criar uma biblioteca mobile compartilhada com React Native, Expo, TypeScript, NativeWind e react-native-builder-bob.
tags:
  - dev
  - mobile
  - react-native
  - expo
  - lib-mobile
  - serie
socialDescription: Guia prático para criar uma biblioteca mobile compartilhada com React Native, Expo, TypeScript, NativeWind e react-native-builder-bob.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# Criando uma biblioteca de componentes mobile

Este guia mostra como criar do zero uma biblioteca compartilhada para aplicações React Native e Expo. O caminho principal usa Expo 53+, React 19+, React Native 0.79+, TypeScript, NativeWind e `react-native-builder-bob`.

A `Torra.Components.Mobile` aparece como estudo de caso porque já validou, em aplicações reais, a separação entre tema, componentes, funções, hooks e integrações nativas. Os nomes e exemplos são genéricos para que o desenho possa ser reutilizado fora da Torra.

Série: parte 1 de 3. Próximo: [[Roteiro verificável - Biblioteca mobile]].

Veja também:

- [[index|Guia da lib mobile]]
- [[Roteiro verificável - Biblioteca mobile]]
- [[Glossário e decisões - Biblioteca mobile]]

## 1. Defina a fronteira antes do código

Uma biblioteca mobile compartilhada deve concentrar contratos reutilizáveis, não partes aleatórias dos aplicativos.

Inclua no primeiro MVP:

- tokens de tema e helpers de classe;
- componentes visuais puros;
- funções genéricas sem regra de negócio;
- hooks genéricos sem infraestrutura específica do app.

Deixe para depois:

- câmera, áudio e outros wrappers nativos;
- componentes compostos ainda não comprovados em mais de um app.

Não inclua:

- autenticação, API, SQLite, sincronização ou telemetria;
- navegação e estado de uma tela específica;
- vocabulário de um fluxo de negócio no contrato público.

Regra prática: algo entra na biblioteca quando é compartilhável, parametrizável e livre de conhecimento do app consumidor.

## 2. Crie o projeto

Use o gerador do Builder Bob ou configure o pacote manualmente. O resultado precisa ter TypeScript como fonte e gerar CommonJS, ESM e declarações de tipos.

```bash
npx create-react-native-library@latest mobile-ui
cd mobile-ui
pnpm install
```

Estrutura inicial recomendada:

```text
mobile-ui/
  src/
    components/
    functions/
    hooks/
    theme/
    index.ts
  example/
  package.json
  tailwind.config.js
  tsconfig.json
```

O app `example` é útil durante o desenvolvimento, mas não substitui a validação em um app consumidor real.

## 3. Configure o contrato do pacote

O `package.json` deve declarar as entradas geradas pelo Bob:

```json
{
  "source": "./src/index.ts",
  "main": "./lib/commonjs/index.js",
  "module": "./lib/module/index.js",
  "types": "./lib/typescript/index.d.ts",
  "react-native": "./src/index.ts",
  "files": ["src", "lib"]
}
```

Configure os alvos do Builder Bob:

```json
{
  "react-native-builder-bob": {
    "source": "src",
    "output": "lib",
    "targets": ["commonjs", "module", "typescript"]
  }
}
```

## 4. Modele dependências corretamente

Declare como `peerDependencies` tudo que precisa ser uma única instância compartilhada com o app:

```json
{
  "peerDependencies": {
    "expo": ">=53",
    "react": ">=19",
    "react-native": ">=0.79",
    "nativewind": ">=4"
  }
}
```

Dependências nativas também são contratos do consumidor. Se a biblioteca expuser câmera, o app ainda precisa instalar `expo-camera` e configurar seu plugin. Não esconda esse requisito.

Mantenha em `dependencies` apenas pacotes realmente incorporados à implementação da biblioteca.

## 5. Estabilize o tema primeiro

Comece por cores, tipografia, espaçamento e composição de classes. Exponha tokens para situações em que classes NativeWind não sejam suficientes:

```ts
export const COLORS = {
  primary: "#ff5101",
  success: "#498d60",
} as const
```

Use um helper `cn` para combinar classes condicionais e resolver conflitos:

```tsx
<Pressable className={cn("px-4 py-3", disabled && "opacity-50", className)} />
```

Classes customizadas dependem do tema compilado pelo app consumidor. Para APIs nativas, SVGs ou estilos que não podem depender desse CSS, use tokens com `StyleSheet`.

## 6. Crie a primeira API pública

Todo consumidor deve importar da raiz do pacote. `src/index.ts` é a única porta pública:

```ts
export { Button } from "./components/Button"
export type { ButtonProps } from "./components/Button"
export { formatCurrency } from "./functions"
export { useOrientation } from "./hooks/useOrientation"
export { COLORS, cn } from "./theme"
```

Não publique caminhos internos como `mobile-ui/src/components/Button`. A implementação interna precisa poder mudar sem quebrar os apps.

## 7. Extraia componentes puros

Comece com contratos pequenos como `Button`, `Divider`, `Container` e `TextRow`.

Um componente público deve:

- receber comportamento por props;
- preservar `className` quando houver customização NativeWind;
- não importar aliases ou serviços do app;
- evitar efeitos colaterais de domínio;
- expor props e tipos coerentes;
- tratar acessibilidade e estados como `disabled` e `loading`.

Exemplo:

```tsx
type ButtonProps = PressableProps & {
  title: string
  loading?: boolean
}

export function Button({ title, loading = false, className, ...props }: ButtonProps) {
  return (
    <Pressable
      {...props}
      accessibilityRole="button"
      disabled={props.disabled || loading}
      className={cn("items-center px-4 py-3", className)}
    >
      {loading ? <ActivityIndicator /> : <Text>{title}</Text>}
    </Pressable>
  )
}
```

## 8. Adicione funções e hooks genéricos

Funções compartilhadas devem ser puras e independentes de domínio. Hooks não devem conhecer endpoints, stores, telemetry providers ou rotas específicas.

Boas candidatas:

- formatação genérica;
- normalização segura de dados;
- orientação da tela;
- estado de rede exposto por provider genérico.

Se a função contém uma regra que só faz sentido para um produto ou processo, ela pertence ao app ou à camada de domínio.

## 9. Valide antes de publicar

Validação mínima:

```bash
pnpm lint
pnpm test
pnpm build
```

Além dos comandos, valide em um app piloto:

1. instale o pacote como o consumidor real fará;
2. importe apenas da raiz;
3. renderize tema e componentes;
4. confirme tipos, refs, foco e classes customizadas;
5. teste Android e iOS quando houver código ou dependência nativa.

## 10. Publique de forma incremental

Use versionamento semântico:

- patch: correção compatível;
- minor: novo contrato compatível;
- major: quebra de contrato público.

O pipeline deve instalar com lockfile, executar lint, testes e build, e só então publicar no registro escolhido.

## Estudo de caso: Torra.Components.Mobile

A biblioteca Torra validou estas práticas:

- `src/index.ts` como única porta pública;
- separação entre `components`, `functions`, `hooks` e `theme`;
- `react`, `react-native`, Expo e módulos nativos como peers;
- `cn` para composição de classes;
- `COLORS` e `StyleSheet` quando uma classe customizada não está disponível no CSS do consumidor;
- remoção de aliases e regras dos apps durante a migração;
- build CommonJS, ESM e TypeScript;
- testes e validação em apps piloto antes de ampliar o catálogo.

O principal aprendizado não é copiar todos os componentes. É começar com uma fronteira pequena e estável, provar o consumo e expandir somente quando o reuso estiver demonstrado.

---

Série: [[index|Guia da lib mobile]]  
Próximo: [[Roteiro verificável - Biblioteca mobile]]
