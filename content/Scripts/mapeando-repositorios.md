---
title: Mapear-Repositorios – Criação automática de repositórios no Azure DevOps
draft: false
tags:
  - dev
  - ps1
  - script
socialDescription: Função PowerShell que automatiza a criação de repositórios em lote no Azure DevOps a partir de uma estrutura de diretórios local, com inicialização e push do Git.
socialImage: https://rodcordeiro.github.io/shares/img/IMG-20180223-WA0036.jpg
---

## Intuito
Automatiza a criação de múltiplos repositórios Git no Azure DevOps com base em subpastas locais. Para cada pasta encontrada:
- Cria o repositório no Azure DevOps.
- Gera um `.gitignore` apropriado.
- Inicializa o repositório localmente com Git.
- Conecta ao remoto e envia os arquivos (`git push`).

Parâmetros:
- `Path` (opcional): caminho com as subpastas a serem mapeadas como repositórios. Se não informado, utiliza o diretório atual.

## Script
```powershell
function Mapear-Repositorios {
    param(
        # Caminho onde estão os projetos. Pode passar o caminho para a função ou executar direto na pasta
        [Parameter(ValueFromPipeline)]
        [string]
        $Path
    )

    begin {
        # Configuração da organização, projeto e autenticação
        $organization = "irienu"     # Substitua pelo nome real da sua organização
        $project = "Projetos"        # Substitua pelo nome do seu projeto
        $pat = "PAT"                 # Personal Access Token (substitua por variável segura em produção)

        $base64AuthInfo = [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes(":$($pat)"))
        $url = "https://dev.azure.com/$organization/$project/_apis/git/repositories?api-version=7.1-preview.1"

        if (-not $Path) {
            $Path = $PWD
        }
    }

    process {
        Set-Location (Resolve-Path $Path)
        $repositorios = Get-ChildItem -Directory

        foreach ($repositorio in $repositorios) {
            Write-Host "Acessando pasta: $repositorio"
            Set-Location $repositorio.FullName

            # Cria .gitignore padrão com fallback
            try {
                Invoke-WebRequest `
                    -Uri "https://www.toptal.com/developers/gitignore/api/dotnetcore,csharp,aspnetcore" `
                    -OutFile .gitignore `
                    -TimeoutSec 10 `
                    -ErrorAction Stop
                Write-Host ".gitignore criado com sucesso"
            }
            catch {
                Write-Warning "Falha ao baixar .gitignore: $_"
                New-Item -ItemType File -Name ".gitignore" -Value "# fallback .gitignore" | Out-Null
            }

            # Criação do repositório via API
            $body = @{ name = $repositorio.Name } | ConvertTo-Json -Depth 10

            Write-Host "Criando repositorio $repositorio no Azure DevOps"
            $response = Invoke-RestMethod -Method Post -Uri $url -Headers @{
                Authorization  = "Basic $base64AuthInfo"
                "Content-Type" = "application/json"
            } -Body $body -Verbose

            if ($response.name -eq $repositorio.Name) {
                Write-Host "Repositório '$($repositorio.Name)' criado com sucesso."

                # Inicialização Git e push
                git init
                git add .
                git commit -m 'Criado o repositório'

                $repoUrl = "https://$organization:$pat@dev.azure.com/$organization/$project/_git/$($repositorio.Name)"
                git remote add origin $repoUrl
                git push
            }
            else {
                Write-Host "Falha ao criar o repositório $($repositorio.Name)"
            }

            Set-Location (Resolve-Path $Path)
        }
    }
}
```
## Dependências
- Git instalado e disponível no ambiente.
- Personal Access Token (PAT) do Azure DevOps com permissão de criação de repositórios.
- Requer acesso à internet para baixar `.gitignore`.
## Exemplos de uso
```powershell
# Executa a função no diretório atual
Mapear-Repositorios

# Mapeia uma pasta específica com projetos
Mapear-Repositorios -Path "C:\Projetos\DotNet"

# Encadeado com pipeline
"C:\Projetos\App1", "C:\Projetos\App2" | ForEach-Object { Mapear-Repositorios -Path $_ }
```
