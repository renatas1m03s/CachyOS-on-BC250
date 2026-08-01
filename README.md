# Sobre
Este documento tem como objetivo mostrar como ajustar ``MANUALMENTE`` uma instalação padrão do CachyOS para extrair o máximo de desempenho da placa AsRock BC-250.

Um dos objetivos aqui é municiar de conhecimento alguém que foi atraído pelo linux, por conta dessa plaquinha maravilhosa, que não quer executar simplesmente um conjunto de scripts sem saber o que está acontecendo "sob o capô".

`Obs.: Existem vários scripts prontos, desenvolvidos por muita gente boa, que fazem todos esses passos de forma automática.`

Um vídeo com a demonstração e explicação de cada um dos pontos deste documento pode ser assistido nesse link [Setup MANUAL do CachyOS/Arch na BC-250](https://www.youtube.com/)

# Importante

A comunidade em torno da BC-250 é extremamente unida e produtiva e alguns passos descritos aqui podem se tornar obsoletos muito rapidamente, portanto lembre-se sempre de consultar a página [AMD BC250 Documentation](https://elektricm.github.io/amd-bc250-docs/)

Procedimentos correntes em `31/07/2026`

# Premissas
- Sistema Operacional: **Linux CachyOS com KDE/Plasma**
- Sitemas de arquivos: **BTRFS**
- Bootloader: **Limine**
- Não é necessário ter atualizado a BIOS, mas não interfere se o tiver feito, além disso,  a BIOS que libera 2 cores e 4 threads a mais é totalmente compatível com esses procedimentos.
- Instalação limpa do Linux CachyOS sem ter executado nenhum dos scripts automatizados
- Usuário com privilégio de sudo

## Conceitos básicos

- Ao copiar os comandos para execução, tente não copiar o bloco todo, copie linha a linha para ter controle sobre a execução e verificar se a mesma foi bem sucedida.
- Uma das maravilhas da base Arch (CachyOS é baseado no Arch Linux) é a possibilidade de se usar uma base de pacotes mantida pela própria comunidade que é o AUR (Arch User Repository).
- As Wikis do [CachyOS](https://wiki.cachyos.org/pt/) e do [Arch](https://wiki.archlinux.org/title/Main_page) são suas amigas, não hesite em consultá-las.

## Tópicos abordados

- [Instalando o yay ou o paru](#instalando-o-yay-ou-o-paru)
- [Instalando as dependências e pré-requisitos](#instalando-as-dependências-e-pré-requisitos)
- [Instalando ACPI Fix](#instalando-o-acpi-fix)
- [Habilitandos as 40 unidades computacionais](#habilitandos-as-40-unidades-computacionais)
- [Configurando a VRAM](#configurando-a-vram)
- [Omitindo a mensagem RDSEED no boot](#omitindo-a-mensagem-rdseed-no-boot)
- [Overclock na GPU](#overclock-na-gpu)
- [Overclock na CPU](#overclock-na-cpu)
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

## Instalando as dependências e pré-requisitos
Alguns dos próximos passos necessitam da instalação de pré-requisitos, são eles:

- **rocm-smi-lib** - habilita o btop ler as informações da GPU;
- **stress** - necessário para o procedimento de overclock/undervolt da CPU;
- **umr** - necessário para o script que libera as unidades computacionais (CUs) adicionais.
- **python-pipx** - será usado para configurar o overclock da CPU

Podemos fazer tudo em uma linha de comando única:
```
yay -S --noconfirm --needed rocm-smi-lib stress umr python-pipx
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
`Obs1.: O comando mkdir (abreviação para make directory) cria a pasta e o comando cd (abreviação de change directory) vai para a pasta criada, além disso o "&&" permite encadear mais de um comando e basicamente significa que quando terminar de executar o mkdir, se ele terminar com sucesso, executa o cd.`

`Obs2.: No linux o "\~" é um alias para o home do usuário. Ex. um usuário de nome palmeiras teria o home igual a "/home/palmeiras", nesse caso "\~" = "/home/palmeiras"`

**Comando para baixar o fix do github** 

**Fix para quando a BIOS estiver atualizada para habilitar os 8 cores**
```
git clone https://github.com/mendesrr/bc250-acpi-fix-updated-8c ~/bc250/acpi-fix
```
**Fix para a BIOS original ou autalizada, mas somente com 6 cores**
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
localizar uma linha parecida com:

**HOOKS=(base systemd autodetect microcode modconf kms keyboard sd-vconsole block filesystems fsck)**

Observar que existem várias linhas parecidas com ela, mas somente uma sem o **"#"** na frente, essa é a que iremos modificar.

Acrescentar o parâmetro **acpi_override** logo após o **fsck**, deixando a linha similar a:

**HOOKS=(base systemd autodetect microcode modconf kms keyboard sd-vconsole block filesystems fsck acpi_override)**

`Obs.: Os parâmetros contidos entre os parênteses podem variar de sistema para sistema, tome cuidado para a única alteração realizada ser o acréscimo do acpi_override.`


Use a combinação de teclas **ctrl + s** para salvar o arquivo e **ctrl + x** para encerrar o editor nano.

**Gere o initramfs novamente**
```
sudo limine-update
```
Reinicie o CachyOS e após isso a CPU conseguirá ficar em 800 MHz quando em iddle.

## Habilitando as 40 unidades computacionais
Assumindo que o **umr** já está instalado (vide tópico [Instalando as dependências/pré-requisitos](#instalando-as-dependências)) o procedimento para liberar as unidades computacionais adicionais é relativamente simples.

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

Aproveitando o momento de editar os parâmetros de boot para incluir o mitigations=off que melhora o desempenho em algumas situações relacionadas a jogos

**Novamente vamos editar as opções de boot do limine**
```
sudo nano /etc/default/limine
```
Localizar a linha **"KERNEL_CMDLINE[default]"** e adicionar depois de **quiet** os parâmetros
```
loglevel=0 mitigations=off
```

Use a combinação de teclas **ctrl + s** para salvar o arquivo e **ctrl + x** para encerrar o editor nano

**Após isso atualizar o bootloader com o comando:**
```
sudo limine-update
```

## Overclock na GPU
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

## Overclock na CPU

A faixa de frequência padrão de operação da CPU depois de aplicado o fix do ACPI é de 800 MHz a 3500 MHz, mas é possível levá-la até 4000 MHz e fazer um undervolt o que resulta em um menor aquecimento quando em altas cargas.

Para habilitar o overclock/unvervolt via SMU existe uma ferramenta chamada **bc250_smu_oc**

A seguir temos os passos para configurar essas possibilidades.

** Fazendo o download e ativando o bc250_smu_oc**
```
cd ~/bc250 && git clone https://github.com/bc250-collective/bc250_smu_oc.git && cd ~/bc250/bc250_smu_oc && pipx install . && chmod +x *.py
```
O comando acima faz o download da ferramenta com o comando **"git clone"**, vai para o diretório da mesma e ativa um ambiente python para rodar uma aplicão em modo isolado via **"pipx"**, após isso finaliza configurando o atributo de execução nos scripts python com o comando **"chmod"** e o parâmetro **"+x"**. Lembrar do encadeamento de comandos usando o **"&&"**

**Testando a capacidade de overclock e undervolt da CPU**
```
sudo ./bc250_detect.py -f 3850 -v 1119 -t 89
```
Esse comando irá testar a frequência de 3850 MHz com 1119 mV, se tudo der certo ele vai concluir o teste com sucesso.

Se o script der erro você pode tentar variar a frequência e a voltagem (variando aos poucos).

O script terminando com sucesso ele gera um arquivo na mesma pasta chamado **overclock.conf** e está na hora de fixar esse parâmetros.

**Tornando os resultados dos testes acima permanente**
```
sudo ./bc250_apply install overclock.conf && sudo systemctl enable --now bc250-smu-oc
```
O comando acima instala o serviço **bc250-smu-oc** e logo após o habilita.

Obs.: Existe um parâmetro para o **bc250_detect.py** que é o **"--keep"** que após o script terminar os parâmetros do teste permanecem aplicados até o reboot.

**Referência:**

[bc250_smu_oc](https://github.com/bc250-collective/bc250_smu_oc)

## Convertendo a zram para zswap
O CachyOS, como muitos sistemas modernos, usa o swap em RAM, mas em um sistema com somente 16GB que é compartilhado com a GPU isso acaba custando muito caro e gerando problemas, por isso, a recomendação é converter a **ZRAM** em **ZSWAP**

Os passos a seguir devem ser executados com cuidado.

**Editar o arquivo mkinitcpio.conf e adicionar o módulo de compressão do swap**
```
sudo nano /etc/mkinitcpio.conf
``` 
Procurar a linha:

**MODULES=()**

Alterá-la para:

**MODULES=(lz4)**

Use a combinação de teclas **ctrl + s** para salvar o arquivo e **ctrl + x** para encerrar o editor nano.

**Adicionar mais um parâmetro ao kernel no limine**
```
sudo nano /etc/default/limine
```
Localizar a linha **"KERNEL_CMDLINE[default]"** e adicionar ao final dela:
```
systemd.zram=0 zswap.enabled=1 zswap.shrinker_enabled=1 zswap.compressor=lz4 zswap.max_pool_percent=30
```
Use a combinação de teclas **ctrl + s** para salvar o arquivo e **ctrl + x** para encerrar o editor nano.

**Após isso atualizar o bootloader e o initramfs com o comando:**
```
sudo limine-update
```

**Execute a seguinte sequência de comandos UM POR UM e tendo a certeza que o último concluiu sem erros**
```
sudo touch /etc/udev/rules.d/30-zram.rules

sudo btrfs subvolume create /swap

sudo btrfs filesystem mkswapfile --size 8g --uuid clear /swap/swapfile

sudo swapon /swap/swapfile

sudo bash

echo "/swap/swapfile none swap defaults 0 0" | sudo tee -a /etc/fstab
```
Reinicie o CachyOS e a troca para ZSWAP estará concluída

## Conclusão
A maior parte desses procedimentos é válida para o Arch Linux, bastando as premissas do Limine e do BTRFS estarem atendidas.
