---
title: Export-OcelotEntry - Combinação de múltiplos Swagger para configuração Ocelot
draft: false
tags:
  - dev
  - ps1
  - script
socialDescription: Função PowerShell que consolida múltiplos Swagger JSON em uma única configuração Ocelot, com suporte a merge e preservação de rotas existentes.
socialImage: https://rodcordeiro.github.io/shares/img/IMG-20180223-WA0036.jpg
---

## Intuito

A função `Export-OcelotEntry` centraliza a geração do arquivo `ocelot.json` para o [Ocelot API Gateway](https://ocelot.readthedocs.io), combinando múltiplas definições Swagger/OpenAPI. Ela suporta:

- Agrupar várias APIs em um único gateway
- Definir ambiente de origem (produção ou homologação)
- Carregar configurações existentes de um arquivo (`SourceFile`)
- Manter rotas anteriores com a flag `-KeepExistingRoutes`

Essa função utiliza internamente `Export-SwaggerAsOcelot` para transformar cada Swagger individual em um bloco de rota Ocelot válido.

## Script
```powershell

# Define uma classe que representa cada entrada Swagger com porta e chave únicas
class OcelotEntryKeys {
    [int] $Porta
    [string]$Chave
}

function Export-OcelotEntry {
    param(
        [OcelotEntryKeys[]]$keys,            # Lista de APIs (porta + chave) a serem exportadas
        [switch]$Prod,                        # Indica se o ambiente é de produção
        [string]$SourceFile,                 # Caminho para arquivo existente de configuração (opcional)
        [switch]$KeepExistingRoutes          # Mantém rotas existentes ao importar de $SourceFile
    )

    begin {
        # Valida se KeepExistingRoutes foi usado corretamente
        if ($KeepExistingRoutes -and -not $SourceFile) {
            throw "Parâmetros inválidos: KeepExistingRoutes requer o uso de SourceFile."
        }

        # Inicializa o objeto de configuração com estrutura esperada pelo Ocelot
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
                    "ClientWhitelist"      = @("internal"); # clientes liberados
                    "ClientIdHeader"       = "client-id-header";
                }
            }
        }

        # Se um arquivo de configuração existente foi informado, tenta carregá-lo
        if ($SourceFile) {
            if (Test-Path $SourceFile) {
                try {
                    Write-Host "Carregando rotas do arquivo de origem: $SourceFile"
                    $existingConfig = Get-Content -Path $SourceFile | ConvertFrom-Json

                    # Filtra rotas: mantém as sem SwaggerKey ou todas, caso KeepExistingRoutes esteja ativo
                    $existingRoutes = $existingConfig.Routes | Where-Object { -not $_.SwaggerKey -or $KeepExistingRoutes }
                    $dictionary.Routes.AddRange($existingRoutes) | Out-Null

                    # Adiciona também os SwaggerEndPoints anteriores, se solicitado
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
        # Itera sobre cada API definida na lista $keys
        foreach ($key in $keys) {
            try {
                # Exporta as definições dessa API para o formato Ocelot
                $dict = $(Export-SwaggerAsOcelot -porta $key.porta -chave $key.chave -ReturnAsObject -Prod:$Prod)

                # Adiciona as rotas e os endpoints Swagger convertidos ao dicionário principal
                $dictionary.Routes.Add($dict.Routes) | Out-Null
                $dictionary.SwaggerEndPoints.Add($dict.SwaggerEndPoints) | Out-Null
            }
            catch {
                Write-Error "
:: Falha ao processar chave $($key.Chave).
$_"
            }
        }

        # Converte lista de listas em uma lista única (caso tenha múltiplos arrays de rotas)
        $dictionary.Routes = ConvertTo-FlattenArray $dictionary.Routes

        # Gera JSON compactado com profundidade adequada
        $jsonOutput = $dictionary | ConvertTo-Json -Depth 10 -Compress

        # Tenta salvar o resultado no arquivo final
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

