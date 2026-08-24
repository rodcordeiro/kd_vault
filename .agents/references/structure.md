# Estrutura

| Path | Responsabilidade observada |
| --- | --- |
| `content/` | Vault publicável: notas Markdown, índices, scripts documentados e modelos |
| `content/index.md` | Página inicial e índice das séries |
| `quartz/` | Runtime do Quartz: CLI, componentes, plugins, processadores, estilos e utilitários |
| `quartz.config.ts` | Metadados do site e pipeline de transformers, filters e emitters |
| `quartz.layout.ts` | Composição visual de páginas de conteúdo e listagens |
| `docs/` | Documentação vendorizada do Quartz |
| `.github/workflows/` | CI, previews e deploy no GitHub Pages |
| `.obsidian/` | Configuração local do Obsidian, ignorada pelo Git |

O repositório usa TypeScript/Preact no runtime Quartz e Markdown como fonte editorial. Não invente camadas genéricas de frontend: preserve as extensões e os pontos de configuração do Quartz.
