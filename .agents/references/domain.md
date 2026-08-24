# Domínio editorial

O `kd_vault` é um vault Obsidian e jardim digital técnico publicado com Quartz. No Nero, pertence ao domínio `vault`; `front` é apenas o guideline da superfície de renderização web.

## Áreas observadas

- `content/Posts/`: artigos e séries agrupados por tema.
- `content/Scripts/`: documentação de scripts e automações.
- `content/_models/`: modelos de autoria.
- `content/index.md`: entrada pública e catálogo de séries.

## Contrato editorial observado

- Notas publicáveis usam frontmatter YAML com campos como `title`, `draft`, `tags`, `socialDescription` e `socialImage`.
- Índices de série ligam artigos com wikilinks Obsidian.
- A home liga índices de séries por caminhos relativos ao vault.
- `quartz.config.ts` habilita Obsidian Flavored Markdown, GitHub Flavored Markdown, tabela de conteúdo, wikilinks e remoção de drafts.

Não presuma que todo Markdown é público: `private`, `templates` e `.obsidian` estão nos padrões ignorados pelo Quartz, e `draft` participa do filtro de publicação.
