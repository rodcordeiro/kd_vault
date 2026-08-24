# Runtime e entrega

## Fluxo

1. O autor mantém notas em `content/`, normalmente com Obsidian.
2. `quartz/bootstrap-cli.mjs` inicia o build.
3. Os transformers de `quartz.config.ts` processam frontmatter, datas, sintaxe, Markdown Obsidian/GitHub, links, descrições e LaTeX.
4. `Plugin.RemoveDrafts()` exclui notas marcadas como draft.
5. Emitters geram páginas, índices, sitemap, RSS e assets em `public/`.
6. `.github/workflows/deploy.yml` executa `npm ci`, `npx quartz build` e publica o artefato no GitHub Pages em pushes para `v4`.

## Comandos comprovados

- `npm run check`: TypeScript sem emissão e verificação Prettier.
- `npm test`: testes com `tsx --test`.
- `npx quartz build`: build estático de produção.
- `npx quartz build --serve`: preview local, não destinado à produção.

O lockfile presente é `package-lock.json`; use npm para manter instalação e CI coerentes.
