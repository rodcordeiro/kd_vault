---
title: ConvertTo-FlattenArray
draft: false
tags:
  - dev
  - ps1
  - script
socialDescription: Função PowerShell recursiva para transformar arrays aninhados em um único array plano.
socialImage: https://rodcordeiro.github.io/shares/img/IMG-20180223-WA0036.jpg
---

## Intuito
Função utilitária que transforma arrays aninhados (arrays dentro de arrays) em um único array plano.

### Parâmetros:
- `array` (`object[]`): Array de entrada que pode conter elementos e subarrays.

### Retorno:
- Um array plano com todos os itens do array original, removendo qualquer aninhamento.

Exemplo: `@(1, @(2, 3), @(4, @(5, 6)))` vira `1, 2, 3, 4, 5, 6`.


## Script
```powershell
function ConvertTo-FlattenArray {
    param (
        [Parameter(Mandatory)]
        [object[]]$array
    )

    # Inicializa o array de saída
    $result = @()

    foreach ($item in $array) {
        # Se for uma coleção (IEnumerable), mas não um dicionário (Hashtable)
        if ($item -is [System.Collections.IEnumerable] -and $item -isnot [System.Collections.Hashtable]) {
            # Chamada recursiva para explorar subníveis
            $result += ConvertTo-FlattenArray -array $item
        }
        else {
            # Adiciona o item diretamente
            $result += $item
        }
    }

    return $result
}
```

## Exemplos de uso

```powershell
$input = @(1, @(2, 3), @(4, @(5, 6)), 7)
$flattened = ConvertTo-FlattenArray -array $input
Write-Output $flattened
# Resultado: 1 2 3 4 5 6 7
```
