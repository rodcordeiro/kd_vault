---
title: Export-OcelotEntry – Exportação de rotas Ocelot a partir de múltiplos Swagger
draft: false
tags:
  - dev
  - ps1
  - script
socialDescription: Função PowerShell que gera um arquivo `ocelot.json` a partir de múltiplas definições Swagger, consolidando rotas e configurações para gateway Ocelot.
socialImage: https://rodcordeiro.github.io/shares/img/IMG-20180223-WA0036.jpg
---

## Intuito
Consolida várias definições Swagger de APIs (identificadas por chave e porta) em um único arquivo `ocelot.json`, compatível com o gateway Ocelot.  
Permite reaproveitar rotas de um arquivo de origem e configurar opções como ambiente de produção e controle de rate limit.

Parâmetros:
- `keys`: lista de objetos com `Porta` e `Chave` para cada API.
- `Prod`: indica se o ambiente é de produção.
- `SourceFile`: caminho para JSON de rotas existentes (opcional).
- `KeepExistingRoutes`: se verdadeiro, mantém rotas existentes do `SourceFile`.

Retorna: nenhum valor. Gera/atualiza o arquivo `ocelot.json`.

## Script
```powershell
# Classe que representa cada entrada de Swagger a ser processada
class OcelotEntryKeys {
    [int] $Porta
    [string]$Chave
}

function Export-OcelotEntry {
    param(
        [OcelotEntryKeys[]]$keys,             # Lista de APIs (porta + chave)
        [switch]$Prod,                         # Ambiente de produção
        [string]$SourceFile,                  # Caminho para arquivo de origem (opcional)
        [switch]$KeepExistingRoutes           # Mantém rotas do arquivo anterior
    )

    begin {
        # Validação de parâmetros
        if ($KeepExistingRoutes -and -not $SourceFile) {
            throw "Parâmetros inválidos: KeepExistingRoutes requer SourceFile."
        }

        # Inicializa a estrutura do dicionário de configuração para ocelot.json
        $dictionary = [PSCustomObject]@{
            Routes                = [System.Collections.ArrayList]::new();
            SwaggerEndPoints      = [System.Collections.ArrayList]::new();
            "GlobalConfiguration" = @{
                "AddHeadersToResponse" = @{
                    "X-Paginacao-Total-Itens"   = "X-Paginacao-Total-Itens";
                    "X-Paginacao-Total-Paginas" = "X-Paginacao-Total-Paginas"
                };
                "Cors" = @{
                    "ExposedHeaders" = @(
                        "X-Paginacao-Total-Itens",
                        "X-Paginacao-Total-Paginas"
                    )
                };
                "RateLimitOptions" = @{
                    "QuotaExceededMessage" = "Limite de requisições excedido. Aguarde e tente novamente.";
                    "ClientWhitelist"      = @("internal");  # Lista de clientes permitidos
                    "ClientIdHeader"       = "client-id-header";  # Nome do header de identificação
                }
            }
        }

        # Se um arquivo de origem foi especificado, tenta carregá-lo
        if ($SourceFile) {
            if (Test-Path $SourceFile) {
                try {
                    Write-Host "Carregando rotas do arquivo de origem: $SourceFile"
                    $existingConfig = Get-Content -Path $SourceFile | ConvertFrom-Json
                    
                    # Mantém somente rotas sem SwaggerKey ou todas (se KeepExistingRoutes estiver ativo)
                    $existingRoutes = $existingConfig.Routes | Where-Object { -not $_.SwaggerKey -or $KeepExistingRoutes }
                    $dictionary.Routes.AddRange($existingRoutes) | Out-Null

                    # Também adiciona endpoints Swagger antigos, se for o caso
                    if ($KeepExistingRoutes) {
                        $dictionary.SwaggerEndPoints.AddRange($existingConfig.SwaggerEndPoints) | Out-Null
                    }
                }
                catch {
                    Write-Error "Erro ao carregar o arquivo de origem: $_"
                }
            }
            else {
                Throw "Arquivo de origem não encontrado: $SourceFile"
            }
        }
    }

    process {
        foreach ($key in $keys) {
            try {
                # Chama a função Export-SwaggerAsOcelot para obter as rotas da API
                $dict = $(Export-SwaggerAsOcelot -porta $key.porta -chave $key.chave -ReturnAsObject -Prod:$Prod)

                # Adiciona as rotas e endpoints ao dicionário principal
                $dictionary.Routes.Add($dict.Routes) | Out-Null
                $dictionary.SwaggerEndPoints.Add($dict.SwaggerEndPoints) | Out-Null
            }
            catch {
                Write-Error ":: Falha ao processar chave $($key.Chave). $_"
            }
        }

        # Remove possíveis aninhamentos em $dictionary.Routes
        $dictionary.Routes = ConvertTo-FlattenArray $dictionary.Routes

        # Gera JSON formatado para o Ocelot
        $jsonOutput = $dictionary | ConvertTo-Json -Depth 10 -Compress

        # Escreve o conteúdo no arquivo final
        try {
            Set-Content -Path ocelot.json -Value $jsonOutput -Force
            Write-Host "Arquivo gerado com sucesso: ./ocelot.json"
        }
        catch {
            Throw "Erro ao escrever o arquivo de saída: $_"
        }
    }
}
```
## Dependências

- [[Export-SwaggerAsOcelot]]: função auxiliar responsável por extrair e formatar os dados de uma API.
- [[ConvertTo-FlattenArray]]: função utilitária que "desaninha" arrays para garantir compatibilidade com o formato JSON final.    

## Exemplos de uso

```powershell
# Exporta 2 serviços do ambiente de homologação, sobrescrevendo tudo
Export-OcelotEntry -keys @(
    [OcelotEntryKeys]@{ Porta = 5001; Chave = "clientes" },
    [OcelotEntryKeys]@{ Porta = 5002; Chave = "produtos" }
)

# Atualiza o arquivo existente, mantendo as rotas anteriores
Export-OcelotEntry -keys @(
    [OcelotEntryKeys]@{ Porta = 6001; Chave = "vendas" }
) -SourceFile "./ocelot.json" -KeepExistingRoutes

# Exporta tudo no modo produção
Export-OcelotEntry -keys @(
    [OcelotEntryKeys]@{ Porta = 80; Chave = "gateway" }
) -Prod
```
