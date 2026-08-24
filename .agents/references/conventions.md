# Convenções de mudança

## Conteúdo

- Preserve UTF-8, frontmatter YAML válido e o estilo de wikilinks já usado pela série.
- Ao criar um artigo de série, atualize o `index.md` da série e, para uma série nova, `content/index.md`.
- Use `draft: true` enquanto o material não deve ser publicado; não contorne `Plugin.RemoveDrafts()`.
- Não inclua segredos, tokens, dados pessoais, URLs privadas ou conteúdo vindo de `private/`.
- Separe fatos verificados, inferências e opinião; não invente conteúdo ou metadados.

## Quartz

- Preserve a ordem dos plugins: transformers são sequenciais e podem depender da posição.
- Faça composição de layout em `quartz.layout.ts`; altere componentes internos apenas quando a configuração não bastar.
- Preserve acessibilidade, navegação por teclado, contraste, responsividade e estados da interface.
- Não adicione dependências sem justificativa técnica e operacional explícita.

## Validação

- Markdown isolado: revise frontmatter, links e diff.
- Mudança editorial ampla ou de configuração: `npm run check` e `npx quartz build`.
- Runtime Quartz: acrescente `npm test`; faça inspeção visual quando layout ou interação mudar.
