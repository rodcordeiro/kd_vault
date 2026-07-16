---
title: Convertendo tarefas manuais em funções PowerShell
draft: false
tags:
  - dev
  - powershell
  - scripts
socialDescription: Como sair de comandos soltos para uma função reutilizável com pipeline e saída estruturada.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# TL;DR;

Quando uma sequência de comandos vira hábito, transforme em função com entrada, saída e nome claros.

## Antes

```powershell
Get-ChildItem .\content -Recurse -Filter *.md |
  Select-String -Pattern 'TODO'
```

Funciona, mas é difícil padronizar e reutilizar.

## Depois

```powershell
function Find-MarkdownTodo {
    [CmdletBinding()]
    param(
        [Parameter(ValueFromPipeline)]
        [string]$Path = '.'
    )

    process {
        Get-ChildItem -LiteralPath $Path -Recurse -Filter '*.md' |
            Select-String -Pattern 'TODO' |
            ForEach-Object {
                [pscustomobject]@{
                    Path = $_.Path
                    Line = $_.LineNumber
                    Text = $_.Line.Trim()
                }
            }
    }
}
```

Saída estruturada permite filtrar, exportar e testar.

## Uso

```powershell
Find-MarkdownTodo -Path .\content |
    Sort-Object Path, Line |
    Format-Table
```

Evite funções que só imprimem texto. Objetos deixam a automação componível.

