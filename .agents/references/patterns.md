# Padrões locais

## Série temática

Cada série observada fica em `content/Posts/<slug>/`, possui um `index.md` com apresentação e lista de artigos, e é ligada por `content/index.md`.

## Fonte e publicação separadas

Markdown em `content/` contém o conhecimento editorial. Configuração e runtime Quartz transformam essa fonte em site estático; mudanças editoriais não devem acoplar-se ao código do gerador.

## Configuração antes de extensão

Metadados, pipeline e layout são centralizados em `quartz.config.ts` e `quartz.layout.ts`. Prefira esses pontos antes de criar ou modificar plugins/componentes internos.

## Entrega reprodutível

O workflow de deploy usa Node 22, `npm ci` e `npx quartz build`, alinhado a `.node-version`, `package-lock.json` e aos engines do `package.json`.
