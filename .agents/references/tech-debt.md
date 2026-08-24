# Dívida técnica e riscos

## Documentação de origem genérica

O `README.md` ainda descreve o Quartz upstream e não documenta o propósito específico do `kd_vault`, seu fluxo editorial ou a entrega em `v4`. Este pacote de contexto reduz a lacuna para agentes, mas não substitui uma decisão humana sobre atualizar o README público.

## Conteúdo e metadados

Há conteúdo legado fora das séries e modelos com convenções possivelmente diferentes. Não normalize frontmatter, nomes ou links em massa sem inventário e validação próprios.

## Superfície vendorizada

Grande parte de `docs/` e `quartz/` deriva do Quartz. Mudanças locais profundas aumentam custo de upgrade; prefira configuração e componentes locais com escopo explícito.

## Validação visual

O checkout oferece build, typecheck/format e testes, mas não evidencia uma suíte e2e específica do blog. Alterações visuais exigem preview e inspeção responsiva manual até existir automação comprovada.
