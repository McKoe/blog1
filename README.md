# Projeto firmware EB-01

## Estrutura

O projeto está dividido em três partes:

**Application** - Contém todo o código da aplicação.

**Bootloader** - Projeto do bootloader da placa, colocado em área separada da aplicação.

**SDK** - Kit de desenvolvimento de software versão 16 da Nordic.


## Instalação do Compilador e Make

As instruções atuais estão para ambiente Windows. Mais dicas de uso [neste tutorial](https://embarcados.com.br/c-para-arm-cortex-m-usando-gcc-e-make/) ![ext-icon]


### 1 - Árvore de pastas

Para organizar o projeto, o local das ferramentas deve estar definido. Crie a estrutura de pastas em sua máquina:

```
C:[tools]
  |
  |--[binutils]
  |--[arm-none-eabi-gcc]
```

### 2 - Compilador

Para este projeto é usado o compilador `arm-none-eabi`. Os arquivos podem ser baixados do [GNU Arm Embedded Toolchain](https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm/downloads).

(A página acima está sendo descontinuada, sendo recomendado o [Arm GNU Toolchain](https://developer.arm.com/Tools%20and%20Software/GNU%20Toolchain). Mas por enquanto funcionou.)

Baixe o arquivo `2`, `gcc-arm-none-eabi-10.3-2021.10-win32.zip` e descompacte-o.

Mova as pastas `arm-none-eabi`, `bin` e `lib` para a pasta `C:\tools\arm-none-eabi-gcc\`


### 3 - Make

Baixar os *binaries* `make-3.81-bin.zip` [deste link](https://gnuwin32.sourceforge.net/packages/make.htm) ![ext-icon]

Copiar toda a pasta `bin` (onde está o `make.exe`) para a pasta `C:\tools\binutils\`


### 4 - Path

Inclua estes caminhos no `Path` das `Variáveis do sistema` do Windows. Para fazer isso vá nas `Propriedades do Sistema` e em `Variáveis de Ambiente`:

```
C:\tools\arm-none-eabi-gcc\bin
C:\tools\binutils
```


### 5 - Makefile SKD config

Ao digitar `make` no terminal cmd do Windows (na pasta que contém o `Makefile` desejado) ainda é informado:

```
Cannot find: '-gcc'.
Please set values in: "...../eb01-firmware/sdk/components/toolchain/gcc/Makefile.windows"
```

Então neste arquivo `Makefile.windows` do projeto é necessário colocar:

```
GNU_INSTALL_ROOT := C:/tools/arm-none-eabi-gcc/bin/
GNU_VERSION := 10.3.1
GNU_PREFIX := arm-none-eabi
```

Agora o projeto está corretamente configurado, com compilador e Make em locais definidos.


### 6 - Preparação Nordic

1. Baixe o programa [nrfutil.exe](https://www.nordicsemi.com/Software-and-tools/Development-Tools/nRF-Util). ![ext-icon] e copie para a pasta `C:\tools\binutils\`.

2. Execute na linha de comando: `nrfutil install nrf5sdk-tools` (isso instala o comando `settings` no `nrfutil`)

3. Instale o [nrf-command-line-tools-xxxxx.exe](https://www.nordicsemi.com/Products/Development-tools/nRF-Command-Line-Tools/Download?lang=en#infotabs) ![ext-icon]
 - isso adiciona o comando `mergehex` que será usado para fazer os merges.
 - instale somente o `mergehex` nas opções apresentadas.
 - não instale o gravador que será sugerido na sequência.
 - **copie o arquivo** instalado `mergehex.exe` para a pasta `C:\tools\binutils\`


## Compilando os projetos


As partes `Application` e `Bootloader` possuem `Makefile` separados, dentro da pasta `Build` de cada um. Normalmente só o `Application` é alterado e precisa ser recompilado.

Para compilar, pelo terminal, vá para a pasta desejada (`....\application\build\`), que contém o arquivo `Makefile` e execute `make`.

Se somente o `Application` foi compilado, copie os arquivos `bootloader.hex` e `soft_device.hex` da pasta `boot_softdevice` para a pasta `Binary`.


## Passos para criar imagem única (bootloader + softdevice + aplicação)

### Guia rápido

Após compilar, vá para a subpasta `Binary` e execute esta sequência de comandos:

```
nrfutil settings generate --family NRF52 --application application_buzzao.hex --application-version 1 --bootloader-version 1 --bl-settings-version 2 settings.hex
mergehex -m bootloader.hex settings.hex -o bootloader_settings.hex
mergehex -m bootloader_settings.hex soft_device.hex -o bl_soft_device.hex
mergehex -m bl_soft_device.hex application_buzzao.hex -o FW_EB01_vxx.hex

```

<details>
<summary>explicação dos comandos acima</summary>

### Guia detalhado

#### Gerar o arquivo `settings.hex`

Após compilar, vá para a subpasta `Binary`

A partir do `application_buzzao.hex` gerado na compilação, gere o `settings.hex` com:

```
nrfutil settings generate --family NRF52 --application application_buzzao.hex --application-version 1 --bootloader-version 1 --bl-settings-version 2 settings.hex
```

#### Merge

1. Realizar o merge do `bootloader` com o `settings`:

```
mergehex -m bootloader.hex settings.hex -o bootloader_settings.hex
```

2. Realizar o merge do passo anterior com o `soft_device`:

```
mergehex -m bootloader_settings.hex soft_device.hex -o bl_soft_device.hex
```

3. Por último, fazer o merge com o `application_buzzao` gerado na compilação do firmware:

```
mergehex -m bl_soft_device.hex application_buzzao.hex -o FW_EB01_vxx.hex
```

</details>


## Gravação

Para fazer o flash é necessário usar um gravador JTag ou SWD.


[ext-icon]: http://www.koetzler.com/ext.png "External link"
