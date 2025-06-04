---
title: Export-SwaggerAsOcelot - Conversão de Swagger para configuração Ocelot
draft: true
tags:
  - dev
  - ps1
  - script
socialDescription: Função PowerShell que consome um Swagger JSON e exporta rotas no formato esperado pelo Ocelot API Gateway.
socialImage: https://rodcordeiro.github.io/shares/img/IMG-20180223-WA0036.jpg
---
## Intuito

A função `Export-SwaggerAsOcelot` consome um documento Swagger/OpenAPI (`swagger.json`) de uma aplicação backend e converte automaticamente os endpoints descritos nele para o formato de configuração utilizado pelo [Ocelot API Gateway](https://ocelot.readthedocs.io/en/latest/).

Ela permite definir o ambiente (produção ou homologação), a chave identificadora da API (`SwaggerKey`) e opcionalmente retorna o objeto em memória ao invés de salvar em disco. Também inclui configuração básica de *rate limiting* para proteger os endpoints.

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

        if ($ReturnAsObject) { return $dictionary }

        try {
            Set-Content -Path $outputFile -Value $jsonOutput -Force
            Write-Host "Formatted paths written to $outputFile"
        } catch {
            Throw "Failed to write to file: $_"
        }
    }
}
