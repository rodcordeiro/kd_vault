---
title: Função Open - PowerShell
draft: false
tags:
  - dev
  - ps1
  - script
socialDescription: Função PowerShell para abrir um caminho especificado no editor ou no explorador de arquivos.
socialImage: https://rodcordeiro.github.io/shares/img/IMG-20180223-WA0036.jpg
---

## Intuito
Função para abrir um caminho especificado, que pode ser um arquivo ou diretório, utilizando o explorador de arquivos, VSCode ou o editor padrão conforme o contexto. Aceita uma flag para forçar a abertura como pasta no explorador. 

**Parâmetros:**
- `$Path` (string): Caminho do arquivo ou diretório a ser aberto. Se não especificado, abre o diretório atual.
- `$AsFolder` (switch): Quando presente, abre o caminho sempre como pasta no explorador de arquivos.

**Retorno:**
- Nenhum valor retornado. A função executa a abertura do caminho solicitado no programa apropriado.

## Script
```powershell
function Open {
    Param (
        [string]
        [parameter(ValueFromPipelineByPropertyName, Mandatory = $false)]
        $Path,
        [switch]
        [parameter(ValueFromPipelineByPropertyName, Mandatory = $false)]
        $AsFolder
    )

    Begin {
        if (!$Path) {
            $Path = Get-Location
        }
    }
    Process {
        if ($(Test-Path $Path) -eq $false) { throw [System.IO.FileNotFoundException]::new("Invalid path or file not exists.") }
        $Path = Resolve-Path $Path

        if ($AsFolder) {
            explorer.exe $Path
            return
        }

        if (Test-Path -Path $Path -Include *.sln ) {
            Invoke-Item $Path
            return
        }

        if ( $(Resolve-Path -Path $Path\*.sln  -ErrorAction SilentlyContinue) ) {
            Invoke-Item $(Resolve-Path -Path $Path\*.sln )
            return
        }
        if ( $(Resolve-Path -Path $Path\*.csproj  -ErrorAction SilentlyContinue) ) {
            Invoke-Item $(Resolve-Path -Path $Path\*.csproj )
            return
        }

        if ($(Resolve-Path -Path $Path\package.json -ErrorAction SilentlyContinue)) {
            code $Path
            return
        }

        if (Test-Path -Path $Path -PathType Container) {
            explorer.exe $Path
            return
        }

        code $Path
    }
}
```
## Dependências
- VSCode (`code` CLI) deve estar instalado e disponível no PATH para abrir projetos Node.js e arquivos via VSCode.
- `explorer.exe` disponível (Windows).
- Sistema com PowerShell e permissões para executar os comandos.

## Exemplos de uso

```powershell
# Abre a pasta `desktop` no explorador de arquivos.
Open ./desktop

# Abre o arquivo de solução `.sln` no editor padrão (geralmente Visual Studio).
Open Project/Project.sln

# Abre a pasta `Project` diretamente no explorador de arquivos, mesmo que contenha arquivos de solução ou projetos.
Open Project -AsFolder
```

