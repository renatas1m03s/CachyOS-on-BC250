# Sobre
Este documento tem como objetivo mostrar como ajustar ``MANUALMENTE`` uma instalação padrão do CachyOS para extrair o máximo de desempenho da placa AsRock BC-250.
Um dos objetivos aqui é municiar de conhecimento alguém que foi atraído pelo linux por conta dessa plaquinha maravilhosa e que não quer executar simplesmente um conjunto de scripts sem saber o que está acontecendo "sob o capô".

`Obs.: Existem vários scripts prontos, desenvolvidos por muita gente boa, que fazem todos esses passos de forma automática.`

Um vídeo com a demonstração e explicação de cada um dos pontos deste documento pode ser assistido nesse link [Setup MANUAL do CachyOS/Arch na BC-250](https://www.youtube.com/)

# Importante

A comunidade em torno da BC-250 é extremamente unida e produtiva e alguns passos descritos aqui podem se tornar obsoletos muito rapidamente, portanto lembre-se sempre de consultar a página [AMD BC250 Documentation](https://elektricm.github.io/amd-bc250-docs/)

Procedimentos atualizados para o dia `31/07/2026`

# Premissas
- Sistema Operacional: Linux CachyOS com KDE/Plasma
- Sitemas de arquivos BTRFS
- Limine como bootloader
- Não é necessário ter atualizado a BIOS, mas não interfere se o tiver feito. A BIOS que libera 2 cores e 4 threads a mais é totalmente compatível com esses procedimentos.
- Instalação limpa do Linux CachyOS sem ter executado nenhum dos scripts automatizados
- Usuário com privilégio de sudo

## Conceitos básicos

- Ao copiar os comandos para execução, tente não copiar o bloco todo, copie linha a linha para ter controle sobre a execução e verificar se a mesma foi bem sucedida.
- Uma das maravilhas da base Arch (CachyOS é baseado no Arch Linux) é a possibilidade de se usar uma base de pacotes mantida pela própria comunidade que é o AUR (Arch User Repository).
- As Wikis do [CachyOS](https://wiki.cachyos.org/pt/) e do [Arch](https://wiki.archlinux.org/title/Main_page) são suas amigas, não hesite em consultá-las.

## Tópicos abordados

- [Instalando o yay ou o paru](#instalando-o-yay-ou-o-paru)
- [Instalando as dependências/pré-requisitos](#instalando-as-dependências)
- [Instalando ACPI Fix](#instalando-o-acpi-fix)
- [Habilitandos as 40 unidades computacionais](#habilitandos-as-40-unidades-computacionais)
- [Configurando a VRAM](#configurando-a-vram)
- [Omitindo a mensagem RDSEED no boot](#omitindo-a-mensagem-rdseed-no-boot)
- [Configurando o Overclock/Undervolt da GPU](#configurando-o-overclock/undervolt-da-gpu)
- [Configurando o Overclock/Undervolt da CPU](#configurando-o-overclock/undervolt-da-cpu)
- [Convertendo a zram para zswap](#convertendo-a-zram-para-zswap)
- [Habilitando entrada automática no modo gaming](#habilitando-entrada-automática-no-modo-gaming)

## Instalando o yay ou o paru
O gerenciador oficial de pacotes do CachyOS e do Arch é o pacman, mas para poder acessar os pacotes do AUR existem algumas ferramentas, como por exemplo o `paru` e `yay`, que podem susbtituir o pacman.
Pessoalmente eu prefiro a forma como o `yay` trabalha, mas o resultado de ambos é o mesmo. Nesse tópico eu mostro como instalar ambos.

**Paru**
```
sudo pacman -S paru
```

**Yay**
```
cd /tmp && git clone https://aur.archlinux.org/yay.git && cd yay && makepkg -si --noconfirm
```

Obs1.: pacman, paru e yay aceitam os mesmos parâmetros e opções. Dois que eu costumo usar são `--noconfirm` (elimina a necessidade confirmação) e o `--needed` (se o componente/dependência já existir no ambiente ele não reinstala).

Obs2.: Depois de instalar algum deles podemos usá-los no lugar do pacman.

**Exemplo**
```
sudo pacman -S --noconfirm --needed paru
```

## Instalando as dependências/pré-requisitos
Alguns dos próximos passos necessitam da instalação de pré-requisitos, são eles:

- **rocm-smi-lib** que habilita o btop ler as informações da GPU;
- **stress** necessário para o procedimento de overclock/undervolt da CPU;
- **umr** necessário para o script que libera as unidades computacionais (CUs) adicionais.

Podemos fazer tudo em uma linha de comando única:
```
yay -S --noconfirm --needed rocm-smi-lib stress umr
```

`Obs: Não se usa o sudo antes do yay ou do paru - Como eles invocam o pacman, no momento certo a senha será solicitada.`

## Instalando o ACPI Fix

Por padrão o CachyOS ou o Arch Linux não conseguem enxergar todos os estados da CPU e por isso o funcionamento fica prejudicado, pois, principalmente em idle, as frequências possíveis não são alcançadas. Para corrigir isso é necessário um sobrescrever (override) as instruções padrão do ACPI. Esse procedimento é o que está descrito a seguir.

Obs1.: No linux existem muitas formas de se alcançar um mesmo resultado e algumas vezes a escolha é baseada única e exclusivamente na preferência pessoal. A seguir o método que eu acho mais simples para se aplicar o fix do `ACPI`.

Obs2.: Para ficar organizado eu gosto de concentrar todos os scripts e apps relacionados a BC-250 em uma pasta no meu `home` chamada `bc250`.

**Comandos para criar e acessar uma pasta no home do usuário chamada bc250**
```
mkdir -pv ~/bc250/acpi-fix && cd ~/bc250
```
`Obs.: O comando mkdir (abreviação para make directory) cria a pasta e o comando cd (abreviação de change directory) vai para a pasta criada, além disso o "&&" permite encadear mais de um comando e basicamente significa que quando terminar de executar o mkdir, se ele terminar com sucesso, executar o cd.`

Obs.: No linux o "\~" é um alias para o home do usuário. Ex. um usuário de nome palmeiras teria o home igual a "/home/palmeiras", nesse caso "\~" = "/home/palmeiras"

**Comando para baixar o fix do github** 

Se já tiver atualizado a **BIOS para 8 cores**: 
```
git clone https://github.com/mendesrr/bc250-acpi-fix-updated-8c ~/bc250/acpi-fix
```
Se ainda estiver na **BIOS com 6 cores**:
```
git clone https://github.com/bc250-collective/bc250-acpi-fix ~/bc250/acpi-fix
```
**Criando uma pasta no /etc para receber o fix e o copiando para lá** 
```
sudo mkdir -pv /etc/initcpio/acpi_override && sudo cp -v ~/bc250/acpi-fix/*.aml /etc/initcpio/acpi_override
```
**Editando o arquivo mkinitcpio.conf para adicionar um HOOK no initramfs**
```
sudo nano /etc/mkinitcpio.conf
```
localizar a linha que começa com **"HOOKS=(base... "** - linha que não começa com "#".
Após o **fsck** adicionar **acpi_override** ficando **fsck acpi_override)**

Use a combinção de teclas **ctrl + s** para salvar o arquivo e **ctrl + x** para encerrar o editor nano.

**Gerando o initramfs novamente**
```
sudo limine-update
```
Reinicie o CachyOS e após isso a CPU conseguirá ficar em 800 MHz quando em iddle.

## Habilitando as 40 unidades computacionais
Assumindo que o umr já está instalado (vide tópico [Instalando as dependências/pré-requisitos](#instalando-as-dependências)) o procedimento para liberar as unidades computacionais adicionais é relativamente simples.

**Baixando o script que libera as unidade computacionais adicionais**
```
cd ~/bc250 && curl -L -o bc250-cu-live-manager.sh https://raw.githubusercontent.com/WinnieLV/bc250-cu-live-manager/refs/heads/main/bc250-cu-live-manager.sh
```
**Executando o script**
```
sudo ~/bc250/bc250-cu-live-manager.sh
```
Uma vez o script estando em execução a sequência mais comum de procedimentos é: 

- tecla "f" para habilitar os 40 CUs;

- tecla "i" para instalar o serviço;

- tecla "w" para escrever a tabela com os CUs adicionais habilitados;

- tecla "q" para encerrar o script.

Obs.: A versão do script de **30/07/2026** tem uma opção para habilitar os dois cores adicionais desabilitados de fábrica.

**Referência:**

[WinnieLV/bc250-cu-live-manager#cpu-core-unlock](https://github.com/WinnieLV/bc250-cu-live-manager#cpu-core-unlock)

## Configurando a VRAM
Recentemente a atualização da BIOS para configurar a alocação dinâmica da VRAM deixou de ser necessária e isso pode ser conseguido com uma aplicação.

Pessoalmente eu tenho conseguido bons resultados com a alocação imediata de 6GB e a possibilidade de alocar mais 5GB, totalizando 11GB de VRAM máxima. Pelas minhas observações quando você começa com a VRAM em 512MB ela "gasta" um tempinho requisitando da RAM e liberando a VRAM depois que ela não é mais necessária. No meu caso, os 6GB são um ponto de equilíbrio bom.

Esse resultado pode ser obtido com a aplicação **bc250memcfg** e um parâmetro do kernel adicional no boot.

Para alcançarmos essa combinação devemos seguir os seguintes passos:

**Baixando a aplicação e descompactando na pasta ~/bc250**
```
cd ~/bc250 && wget https://github.com/fanoush/bc250_memcfg/releases/download/v0.1/bc250_memcfg.zip && unzip bc250_memcfg.zip && cd ~/bc250/bc250_memcfg
```

**Configurando o UMA_SIZE para 6144MB (6GB)**
```
sudo ./bc250memcfg UMA_SIZE 6144
```

**Configurando o parâmetro do kernel para alocar mais 5GB de VRAM se for necessário**

O parâmetro em questão é o **ttm.pages_limit**.

Para calcular o valor a ser passado para o **ttm.pages_limit** fazemos a seguinte conta:
```
Valor dinâmico possível x, no caso 5G
((x * 1024) * 1024) / 4 
((5 * 1024) * 1024) / 4 = 1310720
```
Após obter o valor a ser passado vamos configurar o limine para passar esse valor na inicialização do kernel
```
sudo nano /etc/default/limine
```
Localizar a linha **"KERNEL_CMDLINE[default]"** e adicionar **ttm.pages_limit=1310720** ao final.

Use a combinação de teclas **ctrl + s** para salvar o arquivo e **ctrl + x** para encerrar o editor nano

`Obs.: O próximo passo (omitir a mensagem do RDSEED) também envolve passar um parâmetro para o kernel, por isso, se quiser otimizar as atividades, mantenha o arquivo aberto no nano e pule para a próxima etapa.`

**Após isso atualizar o bootloader com o comando:**
```
sudo limine-update
```
Reinicie o CachyOS e sua VRAM estará configurada para 6GB e podendo chegar a 11GB.

**Referências:**

[AMD BC250 Documentation/VRAM Configuration Guide](https://elektricm.github.io/amd-bc250-docs/bios/vram/)

[fanoush/bc250_memcfg](https://github.com/fanoush/bc250_memcfg)

## Omitindo a mensagem RDSEED no boot
Os processadores baseados na APU Cyan Skillfish (Zen 2) não são compatíveis com a instrução RDSEED e no boot do linux aparece uma mensagem informando que isso está sendo desabilitado. Não há qualquer problema nessa mensagem e isso não tem maiores efeitos além dos estéticos.

Apesar de atualmente não gerar qualquer problema, além do incômodo estético, é possível omitir essa mensagem no boot, bastando para isso adicionar mais um parâmetro ao kernel.

**Novamente vamos editar as opções de boot do limine**
```
sudo nano /etc/default/limine
```
Localizar a linha **"KERNEL_CMDLINE[default]"** e adicionar depois de **quiet** o parâmetro **loglevel=0**

Use a combinação de teclas **ctrl + s** para salvar o arquivo e **ctrl + x** para encerrar o editor nano

`Obs.: O próximo passo (omitir a mensagem do RDSEED) também envolve passar um parâmetro para o kernel, por isso, se quiser otimizar as atividades, mantenha o arquivo aberto no nano e pule para a próxima etapa.`

**Após isso atualizar o bootloader com o comando:**
```
sudo limine-update
```

## Configurando o Overclock/Undervolt da GPU
Por padrão a GPU da BC-250 opera em 1500 MHz constantes e isso não é eficiente em consumo, além de limitar o potencial dessa plaquinha tão maravilhosa.

Essa operação padrão pode ser subvertida com a instalação do **Cyan Skillfish GPU Governor** habilitando frequências de 350 MHz até 2230 MHz e é esse o próximo passo da nossa jornada.

`Obs.: Frequências de 350 MHz só são possíveis com um kernel com patch aplicado e isso já é padrão no CachyOS e no Arch Linux`

**Primeiro passo é instalação do serviço**
```
yay -S --noconfirm --needed cyan-skillfish-governor-smu
```
Durante a instalação o serviço criará um arquivo de configuração **(config.toml)** na pasta **/etc/cyan-skillfish-governor-smu**

Esse arquivo virá configurado com parâmetros seguros de operação variando a frequência de **1000 MHz** a **1850 Mhz**.

Para testar a estabilidade a sugestão é iniciar o serviço sem habilitá-lo, ou seja, se ocorrer alguma instabilidade, com um simples reboot a placa voltará para a operação padrão a 1500 MHz.

**Iniciando o serviço e verificando a estabilidade**
```
sudo systemctl start cyan-skillfish-governor-smu
```
Após iniciar o serviço é recomendado executar algum benchmark da GPU para verificar a estabilidade. Sugestão, testar com o [Unigine Superposition](https://benchmark.unigine.com/superposition)

Confirmada a estabilidade, pode-se habilitar o serviço para iniciar com o boot do sistema.

**Habilitando o GPU Governor para iniciar com o boot**
```
sudo systemctl enable cyan-skillfish-governor-smu
```

Com o tempo pode-se brincar com as frequências e voltagens, para isso recomendo a leitura da documentação do desenvolvedor.

**Referência:**

[filippor/cyan-skillfish-governor](https://github.com/filippor/cyan-skillfish-governor)


## Configurando o Overclock/Undervolt da CPU

## Convertendo a zram para zswap

## Habilitando entrada automática no modo gaming

### Power

`J1000` is a standard 8-pin 12V PCIe power connector.

`J2000` and `J2001` are compatible with 8-pin Molex Micro-Fit connectors and are pinned as below:

```
        J2000                J2001
   v                     v
[ LED1 12V 12V 12V ]  [ 12V 12V 12V PGD ]
[ LED2 GND GND GND ]  [ GND GND GND GND ]
```

For more detail on the non-power pins, check [their section of the hardware page](./hardware.md#j2000-and-j2001).

Keep in mind the BC-250 has a TDP of 220W.

Power and reset buttons are located on the rear I/O.

### Fans

`CPU_FAN1` is a normal 4-pin PWM-capable fan header. `J4003` exposes `CPU_FAN1` as `F1*` and provides four additional PWM fan control signals as follows, though no power is provided from this connector.

```
[ GND F1T F2T F3T F4T F5T DET     ]
[ GND F1P F2P F3P F4P F5P GND GND ]
   ^
```

The `F*T` pins are the tachometer outputs from each respective fan, and the `F*P` pins are the PWM outputs that can be sued to control their speeds. Note that the `F1*` pins are electically connected to `CPU_FAN1`.

# Memory
- 16GB GDDR6 shared between the GPU and CPU. By default, this will be set to either 8GB/8GB (CPU/GPU) or 4GB/12GB, depending on your firmware revision, and requires flashing modified firmware to change. 
- I've seen people mention using Smokeless_UMAF to try and expose these settings; Don't try it, you may cause permanent damage.
- If you are using these boards for gaming, make sure that you set the VRAM allocation to 512MB for the best experience (After flashing firmware).

# OS Support
- Linux:
  - At this point, Linux support is almost perfect. Pretty much any distro shipping a modern kernel + mesa should work fine.
  - Don't run LTS distros on this hardware.
- Windows:
  - No
  - It will boot, but the GPU is not supported by any drivers and is unlikely to ever be. Everything else seems to work alright, so I guess if you've been kicked in the head recently you could use it for non-GPU focused workloads.
- MacOS:
  - Next person to ask this will be asked to find out if the PCIe bracket counts as a flared base.
  
# Making it work
It should all just work with any recent release from Fedora/Bazzite etc. However, HW encode/decode *will not work* because we are missing the required firmware for the VCN. This probably won't change any time soon, as Sony are the ones blocking this. 
## Mesa
- Upstream support [landed](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/33116) in Mesa 25.1. This should be shipped by most big distros at this point
- You may also need to set ``ttm.pages_limit=3959290`` and ``ttm.page_pool_size=3959290`` as kernel options to access more than 8GB of the shared memory. Thanks Magnap :)
## Modified firmware
## ***ANY DAMAGE OR ISSUES CAUSED BY FLASHING THIS MODIFIED IMAGE IS YOUR RESPONSIBILITY ENTIRELY***
- A modified firmware image is available at [this repo](https://gitlab.com/TuxThePenguin0/bc250-bios/) (Credit and massive thanks to [Segfault](https://github.com/TuxThePenguin0)). He is responsible for most of the information on running these boards. Say thank you.
- Flashing via a hardware programmer is recommended. Get yourself a CH347, or a Raspberry Pi Pico, or anything else capable of recovering from a bad BIOS flash.
- ***DO NOT FLASH ANYTHING WITHOUT HAVING A KNOWN GOOD BACKUP***
  - SPI flash header pinout:
    ```
      [ GND SCLK MOSI    ]
      [ VCC  CS  MISO  ? ]
         ^
      ```
- VRAM allocation is configured within: ``Chipset -> GFX Configuration -> GFX Configuration``. Set ``Integrated Graphics Controller`` to forced, and ``UMA Mode`` to  ``UMA_SPECIFIED``, and set the VRAM limit to your desired size. 512MB is best for general APU use. You will have a worse experience overall with a 4/12 split, outside of specific circumstances. Credit to [Segfault](https://github.com/TuxThePenguin0)
- Many of the newly exposed settings are untested, and could either do nothing, or completely obliterate you and everyone else within a 100km radius. Or maybe they work fine. Be careful, though.
- Note: If your board shipped with P4.00G (or any other BIOS revision that modified the memory split) you may need to fully clear the firmware settings as it can apparently get a little stuck. Removing the coin cell and using the CLR_CMOS header should suffice.

## NCT6686 SuperIO
- In order for ``lm-sensors`` to recognize the chip (ID ``0xd441``), you must load the nct6683 driver. You can so via ``modprobe nct6683 force=true`` or by adding ``options nct6683 force=true`` to ``/etc/modprobe.d/sensors.conf``, and ``nct6683``to ``/etc/modules-load.d/99-sensors.conf`` and regenerate your initramfs.
- Once enabled you should see a bunch more sensor data reported, including important temps :)
- Massive thanks to [yeyus](https://github.com/yeyus) for [this info](https://github.com/mothenjoyer69/bc250-documentation/issues/3).

## Performance
- A GPU governor is available [here](https://gitlab.com/mothenjoyer69/oberon-governor). You should use it. Values are set in /etc/oberon-config.yaml.
- This is also available as a Fedora COPR package [here](https://copr.fedorainfracloud.org/coprs/g/exotic-soc/oberon-governor/).
  - You can also use the following commands to set the clocks manually:
    ```
    echo vc 0 <CLOCK> <VOLTAGE> > /sys/devices/pci0000:00/0000:00:08.1/0000:01:00.0/pp_od_clk_voltage
    echo c > /sys/devices/pci0000:00/0000:00:08.1/0000:01:00.0/pp_od_clk_voltage
    ```

# Additional notes:
- These boards more or less just work now, and all you need to do is install a distro, install the GPU governor, and then go to town. 
- Please don't make issues asking for help with anything *but* these boards.
- A discord server exists [here](https://discord.gg/8eZfFWhczz). This is a community of people running and pushing the limits of these boards. Feel free to say hi.

# Credits
- [Segfault](https://github.com/TuxThePenguin0)
- [neggles](https://github.com/neggles)
- [yeyus](https://github.com/yeyus)

# WALL OF SHAME!!!!!!!
1. ![SHAMEFUL](https://github.com/mothenjoyer69/bc250-documentation/blob/main/images/WALL_OF_SHAME_1.png)
2. ![BOOOOO](https://github.com/mothenjoyer69/bc250-documentation/blob/main/images/WALL_OF_SHAME_2.png)
3. ![????](https://github.com/mothenjoyer69/bc250-documentation/blob/main/images/WALL_OF_SHAME_3.png)
3. ![smh](https://github.com/mothenjoyer69/bc250-documentation/blob/main/images/WALL_OF_SHAME_4.png)
