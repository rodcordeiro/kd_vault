---
title: Reiniciar contâineres parados em massa
draft: true
tags:
  - dev
  - script
  - bash
socialDescription:
  "{ description }":
socialImage: https://rodcordeiro.github.io/shares/img/IMG-20180223-WA0036.jpg
---

## Intuito
Explicação simples, ao estilo tldr, de qual o objetivo da função, parâmetros recebidos e retorno, caso aplicável.

## Script
```powershell
#!/bin/bash  
echo "Verificando containers parados..."

# Lista IDs dos containers com status exited
STOPPED_CONTAINERS=$(sudo docker ps -aq -f status=exited)
  
if [ -z "$STOPPED_CONTAINERS" ]; then
    echo "Nenhum container parado encontrado."
else
    echo "Iniciando containers parados..."
    sudo docker start $STOPPED_CONTAINERS
    echo "Containers iniciados com sucesso."
fi
```

## Dependências
N/A

## Exemplos de uso
 
 
