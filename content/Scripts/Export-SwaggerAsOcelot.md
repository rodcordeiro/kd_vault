---
title: Export-SwaggerAsOcelot – Geração de rotas Ocelot a partir de Swagger
draft: false
tags:
  - dev
  - ps1
  - script
socialDescription: Função PowerShell que transforma um Swagger JSON em um conjunto de rotas compatível com Ocelot, incluindo opções de rate limit e Swagger endpoints.
socialImage: https://rodcordeiro.github.io/shares/img/IMG-20180223-WA0036.jpg
---

## Intuito
Conecta-se a uma API Swagger via HTTP para extrair a especificação OpenAPI e convertê-la em um objeto de configuração de rotas compatível com o gateway Ocelot.

Parâmetros:
- `Porta`: porta onde o serviço expõe o Swagger (`/swagger/v1/swagger.json`).
- `Chave`: nome identificador da API.
- `Prod`: se ativado, utiliza o domínio de produção para compor a URL base.
- `ReturnAsObject`: se ativado, retorna o dicionário como objeto PowerShell em vez de salvar no disco.

Retorno: um objeto contendo `Routes` e `SwaggerEndPoints`, ou um arquivo `.json` com as rotas, salvo no disco.

## Script
```powershell
function Export-SwaggerAsOcelot {
    param(
        [Parameter(Mandatory, ValueFromPipelineByPropertyName)]
        [int]$Porta,

        [Parameter(Mandatory, ValueFromPipelineByPropertyName)]
        [string]$Chave,

        [switch]$Prod,
        [switch]$ReturnAsObject
    )

    begin {
        # Define a URL base com base no ambiente
        $url = "backend.xxx.com.br"
        if (!$Prod) {
            $url = "hml.backend.xxx.com.br"
        }

        $openApiUrl = "http://$($url):$($Porta)/swagger/v1/swagger.json"
        $outputFile = "$Chave.json"
    }

    process {
        try {
            Write-Host "Fetching OpenAPI schema from $openApiUrl..."
            $openApiSchema = Invoke-RestMethod -Uri $openApiUrl -Method GET
        } catch {
            Throw "Failed to fetch OpenAPI schema: $_"
        }

        $formattedPaths = @()

        # Converte cada rota do Swagger para o formato aceito pelo Ocelot
        foreach ($path in $openApiSchema.paths.PSObject.Properties) {
            $methods = @()
            foreach ($method in $path.Value.PSObject.Properties) {
                $methods += $method.Name.ToUpper()
            }

            $formattedPaths += @{
                DownstreamPathTemplate   = $path.Name;
                DownstreamScheme         = "http";
                DownstreamHostAndPorts   = @(@{
                    Host  = $url;
                    Port  = $Porta
                });
                UpstreamPathTemplate     = $path.Name;
                UpstreamHttpMethod       = $methods;
                SwaggerKey               = $Chave;
                RateLimitOptions         = @{
                    ClientWhitelist      = @("internal");
                    EnableRateLimiting   = $true;
                    Period               = "1s";
                    PeriodTimespan       = 30;
                    Limit                = 15;
                    QuotaExceededMessage = "Limite de consultas por segundo excedida! Aguarde e tente novamente mais tarde.";
                }
            }
        }

        $dictionary = [PSCustomObject]@{
            Routes           = $formattedPaths;
            SwaggerEndPoints = @{
                Key    = $Chave;
                Config = @(@{
                    Name    = $Chave;
                    Version = "v1";
                    Url     = $openApiUrl;
                })
            }
        }
    }

    end {
        $jsonOutput = $dictionary | ConvertTo-Json -Depth 10 -Compress

        if ($ReturnAsObject) {
            return $dictionary
        }

        try {
            Set-Content -Path $outputFile -Value $jsonOutput -Force
            Write-Host "Formatted paths written to $outputFile"
        } catch {
            Throw "Failed to write to file: $_"
        }
    }
}
```
  
## Exemplos de uso
```powershell
# Gera o arquivo "clientes.json" com rotas do Swagger em homologação
Export-SwaggerAsOcelot -Porta 5001 -Chave "clientes"

# Exporta rotas no ambiente de produção e salva no disco
Export-SwaggerAsOcelot -Porta 80 -Chave "gateway" -Prod

# Retorna o objeto sem gravar no disco
Export-SwaggerAsOcelot -Porta 6001 -Chave "pedidos" -ReturnAsObject

```