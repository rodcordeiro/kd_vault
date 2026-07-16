---
title: Criando scripts PowerShell seguros e reexecutáveis
draft: false
tags:
  - dev
  - powershell
  - automation
socialDescription: Padrões simples para escrever scripts PowerShell com validação, WhatIf e tratamento de erro.
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
---

# TL;DR;

Todo script que altera estado deveria validar entrada, suportar `-WhatIf` e falhar cedo com mensagem útil.

## Template

```powershell
[CmdletBinding(SupportsShouldProcess)]
param(
    [Parameter(Mandatory)]
    [ValidateScript({ Test-Path -LiteralPath $_ })]
    [string]$Path
)

$ErrorActionPreference = 'Stop'

$resolvedPath = Resolve-Path -LiteralPath $Path

if ($PSCmdlet.ShouldProcess($resolvedPath, 'Process files')) {
    Get-ChildItem -LiteralPath $resolvedPath -File | ForEach-Object {
        Write-Information "Processing $($_.FullName)" -InformationAction Continue
    }
}
```

`SupportsShouldProcess` permite rodar com `-WhatIf`. `Resolve-Path` evita operar em caminho ambíguo.

## Execução

```powershell
.\Process-Files.ps1 -Path .\content -WhatIf
.\Process-Files.ps1 -Path .\content
```

Primeiro simule. Depois execute.

## Boas práticas

- Use `-LiteralPath` para caminhos de usuário.
- Defina `$ErrorActionPreference = 'Stop'`.
- Prefira parâmetros nomeados a valores mágicos no corpo do script.
- Não misture enumeração em PowerShell com deleção em outro shell.

