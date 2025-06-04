---
title: ConvertTo-FlattenArray
draft: false
tags:
  - dev
  - ps1
  - util
socialDescription: Função PowerShell para transformar arrays aninhados em um único array plano.
socialImage: https://rodcordeiro.github.io/shares/img/IMG-20180223-WA0036.jpg
---

## Intuito

Esta função tem como objetivo "achatar" (flatten) arrays aninhados, ou seja, transformar uma estrutura como `[[1, 2], [3, [4, 5]]]` em um único array plano como `[1, 2, 3, 4, 5]`.

Muito útil quando se manipula listas de objetos compostas por outras listas (como ocorre após agregações ou mapeamentos complexos em JSON).

---

## Script
```powershell
function ConvertTo-FlattenArray {
    param (
        [Parameter(Mandatory)]
        [object[]]$array
    )

    # Inicializa array vazio para armazenar os resultados
    $result = @()

    # Percorre cada item do array de entrada
    foreach ($item in $array) {

        # Verifica se o item é uma coleção (IEnumerable), mas não um Hashtable
        # Isso garante que dicionários (objetos JSON) não sejam desmembrados erroneamente
        if ($item -is [System.Collections.IEnumerable] -and $item -isnot [System.Collections.Hashtable]) {

            # Recursivamente chama a função para processar subarrays
            $result += ConvertTo-FlattenArray -array $item
        }
        else {
            # Caso contrário, adiciona o item diretamente ao resultado final
            $result += $item
        }
    }

    # Retorna o array resultante achatado
    return $result
}
```

## Exemplo de uso
```powershell
$input = @(1, @(2, 3), @(4, @(5, 6)), 7)
$flattened = ConvertTo-FlattenArray -array $input
Write-Output $flattened
# Resultado: 1 2 3 4 5 6 7
```
