---
title: Criando um módulo do powershell
draft: 
tags:
  - dev
  - ps1
  - post
description: Como criar um módulo do powershell simples e publicar na PSGallery
socialDescription: Como criar um módulo do powershell simples e publicar na PSGallery
socialImage: https://rodcordeiro.github.io/shares/img/rodcordeiro.png
linkedin: https://www.linkedin.com/pulse/criando-um-m%25C3%25B3dulo-do-powershell-rodrigo-cordeiro-p5kzf
date: 2025-01-15
---


# Vamos do começo!

 Vez ou outra precisamos de uma função que nos ajude a automatizar ou gerenciar tarefas e as vezes, e acabamos precisando de mais de um script para fazer isso de forma organizada. Nestes casos um módulo do powershell permite criar e exportar funções, classes e até mesmo criar funções privadas, disponíveis apenas dentro do módulo. Vamos fazer um hands-on com o MyModule, o hello world dos módulos!. Vamos começar com a estrutura básica do projeto (arquivos explicados posteriormente)

## Estrutura de arquivos básica

```jsx
MyModule
│   MyModule.psm1
│   MyModule.psd1
│   README.md
│   LICENSE
└── Functions
    ├── Public
    │       Get-HelloWorld.ps1
    └── Private
            HelperFunction.ps1

```

A partir desta estrutura, teremos as funções publicas a serem exportadas, as funções privadas que estarão disponíveis apenas no contexto do módulo e é possível copiar a lógica para a implementação de classes e outros fatores

### **Arquivo `.psm1`**

O arquivo `.psm1` é o núcleo do módulo. Ele carrega as funções que você deseja exportar e pode conter lógica adicional.

```powershell
# Importar funções publicas e privadas
$publicFunctions = Get-ChildItem -Path "$PSScriptRoot\Functions\Public" -Filter *.ps1
$privateFunctions = Get-ChildItem -Path "$PSScriptRoot\Functions\Private" -Filter *.ps1

# Dot-source all Public Functions
foreach ($file in $publicFunctions) {
    . $file.FullName
}

# Dot-source all Private Functions
foreach ($file in $privateFunctions) {
    . $file.FullName
}

# Exportar apenas funções públicas
Export-ModuleMember -Function (Get-ChildItem -Path "$PSScriptRoot\Functions\Public" -Filter *.ps1 | ForEach-Object { $_.BaseName })

```

### Arquivo `.psd1`

Este arquivo é o Manifesto do módulo, ou arquivo de *“configurações”*, nele estão as informações sobre o módulo, módulos que devem ser carregados juntos, funções a serem exportadas caso deseje exportar de forma explicita e não automática e os metadados para compartilhamento do módulo na psgallery ou em uma galeria privada.

Este arquivo pode ser gerado através do comando New-ModuleManifest(vide abaixo):

### Readme e License

Estes são arquivos padrões de todo repositório, um bom Readme e o arquivo de licença do projeto. O Arquivo de licença define qual a licença aplicada sobre o código, definindo em que e como terceiros podem utilizar ou até reutilizar seu código. O Readme é a introdução do repositório.

### Functions

Separadas em *public* (como diz o nome, públicas, compartilhadas,exportadas) e *private*, aqui vão as funções existentes no módulo.

Exemplo de `Functions/Public/Get-HelloWorld.ps1`:

```powershell
function Get-HelloWorld {
    [CmdletBinding()]
    param (
        [Parameter(Mandatory)]
        [string]$Name
    )

    Write-Output "Hello, $Name!"
}
```

**Exemplo de `Functions/Private/HelperFunction.ps1`:**

```powershell
function Helper-LogMessage {
    param (
        [string]$Message
    )

    Write-Output "Log: $Message"
}
```

## Adicionar classes

Para adicionar classes, podemos criá-la dentro do arquivo da função a ser exportada ( a classe não será exportada) ou criá-la em uma pasta a parte, mantendo a hierarquia e estrutura das pastas. Por convenção e preferência, seguiremos o segundo caminho!

1. Crie a pasta Classes na raíz do projeto, ficando:
    
    ```markdown
    markdown
    Copiar código
    MyModule
    │   MyModule.psm1
    │   MyModule.psd1
    └── Classes
        ├── MyPublicClass.ps1
        └── MyPrivateClass.ps1
    ```
    
2. Crie os arquivos das classes: 
    
    **`Classes/MyPublicClass.ps1`:**
    
    ```powershell
    powershell
    Copiar código
    class MyPublicClass {
        [string]$Name
    
        MyPublicClass([string]$Name) {
            $this.Name = $Name
        }
    
        [string] SayHello() {
            return "Hello, $($this.Name)!"
        }
    }
    
    ```
    
    **`Classes/MyPrivateClass.ps1`:**
    
    ```powershell
    powershell
    Copiar código
    class MyPrivateClass {
        [int]$Id
    
        MyPrivateClass([int]$Id) {
            $this.Id = $Id
        }
    
        [string] GetId() {
            return "ID: $($this.Id)"
        }
    }
    
    ```
    
3. Adicione a importação das classes no `MyModule.psm1`, de forma que as classes estejam disponíveis para todo o módulo.
    
    ```powershell
    # Load class files
    Get-ChildItem -Path "$PSScriptRoot\Classes" -Filter "*.ps1" | ForEach-Object {
        . $_.FullName
    }
    
    ```
    
    > A classe privada permanecerá privada por convenção!
    > 

# Documentando o módulo

A importância de documentar o módulo, tanto o módulo em si quanto as funções, se dá não apenas pela necessidade de uma boa documentação para o módulo, mas também pelo fato de o cmdlet Get-Help ler estes dados ao ser invocado.

## Documentar funções:

Este é um processo muito importante pois, a partir dele podemos gerar a documentação completa do módulo.

```powershell
function Get-SampleData {
<#
.SYNOPSIS
	Retrieves sample data for demonstration purposes.
.DESCRIPTION
	This function generates a list of sample data items.
.PARAMETER Count
	Specifies the number of sample data items to generate.
.EXAMPLE
	Get-SampleData -Count 5
	Generates 5 sample data items.
.INPUTS
	[System.Int32]
		The number of items to generate.
.OUTPUTS
	[System.Object[]]
		Returns an array of sample data.
.NOTES
	Author: Your Name
	Version: 1.0
#>
	param (
		[Parameter(Mandatory)]
		[int]$Count
	)
	for ($i = 1; $i -le $Count; $i++) {
	    [PSCustomObject]@{
	        ID   = $i
	        Name = "Sample$i"
	    }
	}
}
```

## Documentar o módulo em si

Adicionando as informações abaixo no início do arquivo .psm1, podemos passar algums dados do módulo para o get-help.

```powershell
<#
.SYNOPSIS
    A brief description of your module.
.DESCRIPTION
    A detailed description of your module, its purpose, and its functionality.
.EXAMPLE
    Import-Module MyModule
    Use examples of commands provided by the module.
.NOTES
    Additional notes, such as author name, version, or contact information.
#>
```

## PlatyPS para gerar um documento em markdown

Você pode utilizar o https://github.com/PowerShell/platyPS para gerar uma documentação em markdown para complementar seu módulo e também para gerar o ‘External help’, que é um arquivo de documentação. Vide site para melhor explicação (e atualizada) sobre como utilizar este módulo

# Publicar um módulo

## Criando o Manifesto do módulo

Para criar o manifesto do módulo, ou o arquivo `.psd1`, basta executar o comando:

```powershell
New-ModuleManifest -Path "MyModule.psd1" `
    -RootModule "MyModule.psm1" `
    -Author "YourName" `
    -Description "Description of My Module" `
    -ModuleVersion "1.0.0" `
    -FunctionsToExport '*' `
    -CmdletsToExport '*' `
    -VariablesToExport '*' `
    -PrivateData @{
        PSData = @{
            Tags = @("Tag1", "Tag2")
            LicenseUri = "https://opensource.org/licenses/MIT"
            ProjectUri = "https://github.com/yourusername/repo"
        }
    }
```

## Publicar o módulo para a PSGallery

Antes de prosseguir, lembre-se de pegar um [token da PSGallery](https://learn.microsoft.com/pt-br/powershell/gallery/how-to/managing-profile/creating-apikeys?view=powershellget-3.x) para poder publicar:

Para publicar, execute o seguinte comando:

```powershell
Publish-Module -Path "Path\To\MyModule.psd1" -Repository PSGallery -NuGetApiKey "YourAPIKeyHere"
# Caso não tenha gerado as documentações do módulo conforme instruído, o caminho deve ser do arquivo .psm1
```

# **Dicas Extras**

- Use boas práticas como `[CmdletBinding()]` para definir funções.
- Adicione documentação XML para cada função para suportar `Get-Help`.
- Automatize testes com o [Pester](https://pester.dev/)