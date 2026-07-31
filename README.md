# Sobre
Este documento tem como objetivo mostrar como ajustar ``MANUALMENTE`` uma configuração padrão do CachyOS para extrair o máximo de desempenho da placa AsRock BC-250.
Um dos objetivos aqui é municiar alguém que foi atraído pelo linux por conta dessa plaquinha maravilhosa com mais controle e conhecimento sobre o que acontece "sob o capô".

`Obs.: Existem vários scripts prontos, desenvolvidos por muita gente boa, que fazem todos esses passos de forma automática.`

A demonstração e explicação de cada um desses pontos pode ser assistida nesse link [Setup MANUAL do CachyOS/Arch na BC-250](https://www.youtube.com/)

# Importante

A comunidade em torno da BC-250 é extremamente unida e produtiva e alguns passos descritos aqui podem se tornar obsoletos muito rapidamente. Portanto, lembre-se sempre de consultar a página [AMD BC250 Documentation](https://elektricm.github.io/amd-bc250-docs/)

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
- [Habilitandos as 40 unidades computacionais](#Habilitandos as 40 unidades computacionais)
- [Configurando a VRAM](#Configurando a VRAM)
- [Omitindo a mensagem RSEED no boot](#Omitindo a mensagem RSEED no boot)
- [Configurando o Overclock/Undervolt da GPU](#Configurando o Overclock/Undervolt da GPU)
- [Configurando o Overclock/Undervolt da CPU](#Configurando o Overclock/Undervolt da CPU)
- [Convertendo a zram para zswap](#Convertendo a zram para zswap)
- [Habilitando entrada automática no modo gaming](#Habilitando entrada automática no modo gaming)

## Instalando o yay ou o paru
O gerenciador oficial de pacotes do CachyOS e do Arch é o pacman, mas para poder acessar os pacotes do AUR existem algumas ferramentas, como por exemplo o `paru` e `yay`.
Pessoalmente eu prefiro a forma como o `yay` trabalha, mas o resultado de ambos é o mesmo. Nesse tópico eu mostro como instalar ambos.

**Paru**
```
sudo pacman -S paru
```

**Yay**
```
cd /tmp && git clone https://aur.archlinux.org/yay.git && cd yay && makepkg -si --noconfirm
```

Obs.: pacman, paru e yay aceitam os mesmos parâmetros e opções. Dois que eu costumo usar são `--noconfirm` (elimina a necessidade confirmação) e o `--needed` (se o componente/dependência já existir no ambiente ele não reinstala).

**Exemplo**
```
sudo pacman -S --noconfirm --needed paru
```


## Instalando as dependências/pré-requisitos

## Instalando o ACPI Fix

## Habilitando as 40 unidades computacionais

## Configurando a VRAM

## Omitindo a mensagem RSEED no boot

## Configurando o Overclock/Undervolt da GPU

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
