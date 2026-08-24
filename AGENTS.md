# kd_vault

Vault Obsidian publicado como jardim digital pelo Quartz v4. O conteúdo canônico fica em `content/`; a geração do site parte de `quartz/bootstrap-cli.mjs` e das configurações `quartz.config.ts` e `quartz.layout.ts`.

## Como usar este contexto

| Quando | Leia |
| --- | --- |
| Navegar pelo contexto local | `.agents/references/index.md` |
| Alterar notas, séries ou metadados | `.agents/references/domain.md` e `.agents/references/conventions.md` |
| Alterar Quartz, build ou deploy | `.agents/references/structure.md` e `.agents/references/runtime.md` |
| Avaliar padrões e riscos conhecidos | `.agents/references/patterns.md` e `.agents/references/tech-debt.md` |
| Trabalhar com knowledge operacional | skill `$nero`; domínio canônico `vault` |
| Alterar a superfície web | `$nero` → `references/guidelines/front-guidelines.md` |

## Regras rápidas

- Preserve frontmatter, wikilinks e compatibilidade com Obsidian/Quartz; não publique material privado ou sensível.
- Faça mudanças localizadas e não altere o runtime Quartz ao editar somente conteúdo.
- Use `npm run check`, `npm test` e `npx quartz build` conforme o risco da mudança.

## Skill condicional

- Use `$nero` para consultar ou registrar conhecimento operacional deste projeto.
