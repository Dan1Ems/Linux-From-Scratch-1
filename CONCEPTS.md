# Linux From Scratch

## Introdução

Linux From Scratch é um livro e projeto que ensina a construir um sistema Linux inteiro a partir do código-fonte, incluindo ferramentas, bibliotecas e kernel. Ele fornece o passo a passo para compilar tudo manualmente, gerando uma distribuição sua, do zero.

Para criar uma nova distro, você precisa entender cada componente. LFS é o roteiro definitivo - você vai usá-lo para saber a ordem correta de compilar binutils, GCC, glibc, e finalmente o kernel.

---

### Conceitos

**Sistema Operacional**: Os Sistemas Operacionais são softwares que gerenciam os aplicativos e os recursos de um dispositivo. Eles realizam tarefas para garantir o bom funcionamento do aparelho. Apesar de cumprirem os mesmos papéis, cada sistema operacional fornece seus próprios recursos, interface e política de segurança.

**Kernel**: Kernel é o núcleo do Sistema Operacional, ele é o responsável por gerenciar os recursos de Hardware do computador e garantir que cada processo tenha acesso aos recursos necessários para funcionar corretamente. O Kernel é a única parte do Sistema Operacional que tem acesso direto e irrestrito aos Hardwares do computador. Ele gerencia a memória, escalona os processos na CPU, controla a rede e conversa com os discos rígidos, além de executar programas e se conectar com os periféricos. Todo o resto (shell, comandos, interface gráfica) são apenas "espaço de usuário" (user space) pedindo favores ao Kernel através de chamadas de sistema (syscalls).

**Shell**: Shell significa casca ou concha, esta metáfora é a chave para entender o conceito. Sua função primordial é fornecer uma interface gráfica (normalmente textual por linha de comando) que permite o usuário acessar os serviços do Sistema Operacional. Ele é a camada mais externa de um OS, o Shell é quem envolve o Kernel e serve como interface primária através da qual o usuário interage com a máquina.

**Firmware**: Firmware é uma camada intermediária crucial entre hardware e software. Diferente do Software, que pode ser alterado e modificado pelo usuário final, o firmware é gravado em memória de acesso somente leitura (ROM, EPROM ou PROM) durante o processo de fabricação do dispositivo. A principal função do firmware é apresentar instruções específicas para o Hardware a fim de realizar determinadas tarefas. Ao ligar a máquina, ele é o responsável pela inicialização do dispositivo - no caso de PCs, isso significa executar a BIOS ou o UEFI, que então preparam o hardware e carregam o bootloader. Após a inicialização do OS, o firmware atua como uma camada intermediária entre o Hardware e o Sistema Operacional.

**Bootloader**: Bootloader é um software que permite a inicialização do Sistema Operacional de todos os dispositivos. Sempre que o dispositivo é ligado, ele acionará o bootloader para carregar o OS. Além disso, o bootloader pode oferecer redundância (Ex: carregar um kernel reserva se o principal falhar) ou permitir escolher entre múltiplos sistemas operacionais.

---

### Código Fonte

É o código em C (e assembly) que forma o kernel. Sem o código-fonte, não há kernel. Você precisa dele para personalizar.

Você baixa do [kernel.org](https://kernel.org/).

---

### Diretórios

#### **arch/**

**O que é**: Subdiretório do kernel que contém a arquitetura do processador (x86, ARM, PowerPC, etc).

**O que faz**: Ele é o responsável por gerenciar a inicialização do processador, a configuração da MMU (Memory Management Unit) e o tratamento de interrupções via APIC (Advanced Programmable Interrupt Controller).

**Por que é necessário**: Cada CPU tem seu próprio jeito de ligar e gerenciar seus recursos. Sem o diretório `arch/`, o kernel não saberia como conversar com o hardware da sua placa mãe.

---

#### **drivers/**

**O que é**: A maior parte do código-fonte do kernel. Contém os drivers de dispositivo.

**O que faz**: Ensina o kernel a interagir com os Hardwares do computador (discos, placas de rede, USB, GPU, etc).

**Por que é necessário**: Sem drivers, o kernel é cego e surdo - não acessa o teclado, não mostra vídeo, não lê o disco onde está a sua distro.

---

#### **fs/ (Filesystem)**

**O que é**: Implementação de sistemas de arquivos (ext4, btrfs, FAT, proc, sysfs) e a camada VFS (Virtual File System).

**O que faz**: Organiza como os dados são acessados no disco e fornece uma interface única para programas acessarem arquivos, independente do formato.

**Por que é necessário**: Sem um sistema de arquivos, o kernel não conseguiria ler nem gravar nada no disco - você não teria como inicializar o sistema nem salvar arquivos.

---

#### **kernel/**

**O que é**: O coração do kernel - escalonador, sinais, temporizadores, mecanismos de trava (locking).

**O que faz**: Decide qual processo executa (scheduler), permite comunicação entre processos (sinais), controla tempo (timers) e evita conflitos de acesso a dados (locking).

**Por que é necessário**: São as funções mínimas que definem um sistema operacional. Sem elas, não há multitarefa, nem resposta a eventos, nem controle de concorrência.

---

#### **mm/ (Memory Management)**

**O que é**: Subsistema de gerenciamento de memória.

**O que faz**: Aloca e libera RAM física, implementa memória virtual (paginamento), gerencia swapping e mantém o Page Cache (dados de disco na RAM).

**Por que é necessário**: Todo programa precisa de memória. O kernel gerencia esse recurso finito, evita que um processo atrapalhe o outro e acelera o acesso a arquivos (page cache).

---

#### **net/**

**O que é**: Implementação das pilhas de rede (IPv4, IPv6, TCP, UDP) e funcionalidades de roteamento, filtragem e QoS (Quality of Service).

**O que faz**: Permite enviar e receber pacotes de rede, montar conexões TCP, transmitir datagramas UDP, aplicar firewall, etc.

**Por que é necessário**:  Sem `net/` o sistema não teria rede - nem cabo, nem WI-FI, nem localhost. Para uma distro moderna, é indispensável.

---

### Configuração

#### **Arquivo .config**

**O que é**: O arquivo `.config` é um arquivo de texto simples gerado por ferramentas como `make menuconfig`, que contém centenas de variáveis (Ex: CONFIG_SMP=y). O sistema *Kbuild* lê esse arquivo para saber o que compilar.

**O que faz**: Cada opção no código fonte (gerada por um arquivo Kconfig) vira uma linha no `.config`.

*y (Yes/Built-in)*: O código será compilado diretamente para dentro do arquivo executável final do kernel (vmlinuz). O recurso estará disponível no exato milissegundo em que o kernel for carregado na RAM pelo bootloader

*m (Module)*: O código será compilado com um arquivo separado, com extensão .ko (Kernel Object). Esses arquivos ficarão armazenados no seu disco rígido. O kernel só carregará esse módulo na RAM se detectar o hardware correspondente

*n (No)*: O código é ignorado pelo compilador.

O Kbuild passa essas variáveis para o GCC e o linker, que geram o binário final.

**Por que é necessário**: O `.config` é o DNA do seu kernel. Sem ele, você não controla o tamanho, os drivers, nem as funcionalidades. Para um distro enxuta, você precisa do `.config` sob medida - nem inchado (como as distros genéricas), nem capado a ponto de não bootar.

---

#### **A trindade: y / m / n**

**O que é**: São os três valores que cada recurso do kernel pode receber no `.config`.

**O que faz**:

*y (Yes)*: O código vira parte do `vmlinuz`. Está sempre em memória desde o boot.

*m (Module)*: O código fica em um arquivo separado (`/lib/modules/.../driver.ko`). Pode ser carregado (`insmod`, `modprobe`) e descarregado (`rmmod`).

*n (No)*: O código nem é compilado.

**Por que é necessário**: O `y` é essencial para drivers de boot; O `m` reduz o tamanho do kernel principal e acelera o boot. Ideal para drivers de hardware que você usa de forma esporádica/irregular; Usar o `n` economiza espaço em disco e memória. Remove dados que seu hardware nunca usará e também reduz possíveis vulnerabilidades.

---

#### **Ferramentas de configuração**

**O que são**: Programas interativos que geram o `.config`:

`make menuconfig` (terminal, menus coloridos)

`make xconfig` (gráfico, Qt)

`make oldconfig` (atualiza um `.config` antigo perguntando só as novidades)

**O que fazem**: Lêem os arquivos `Kconfig` espalhados no código, mostram hierarquias de menus, e gravam suas escolhas no `.config`. Tratam dependências (Ex: se você desliga USB, opções de webcam USB somem).

**Por que são necessários**: Editar o `.config` manualmente é loucura (mais de 10.000 linhas). `menuconfig` é a ferramenta padrão para qualquer construtor de distro. Você vai usá-la dezenas de vezes até acertar a configuração ideal.

---

#### **Drivers e módulos essenciais**

**O que são**: Conjunto mínimos de drivers que seu kernel precisa ter (embutidos ou em initramfs) para bootar e funcionar (disco/SSD, sistema de arquivos, rede, USB).

**O que fazem**: Permitem que o kernel acesse o dispositivo de boot, monte a partição raiz e tenha suporte a entrada/saída básica.

*Driver de disco/SSD* (ex: AHCI, NVMe) → para ler a partição raiz.

*Driver do sistema de arquivos* (ex: ext4, btrfs) → para interpretar os dados no disco.

*Driver de rede* (opcional, mas quase sempre necessário).

*Driver USB* (se usar teclado/mouse USB no boot ou armazenamento USB).

**Por que são necessários**: Se faltar o driver do seu controlador de disco (Ex: NVMe), o kernel não encontra a partição raiz e o boot falha. Esses drivers precisam estar compilados dentro (`y`) ou em um initramfs.

---

### **Compilação**

#### **Códigos de Compilação**

**O que são**: Comandos que acionam o Kbuild para traduzir código-fonte em binário.

**O que fazem**:

`make`: compila kernel (gera `vmlinuz` ou `bzImage`)

`make modules`: compila todos os códigos marcados com `m`.

`make modules_install`: instala os módulos em `/lib/modules/$(uname -r)/`.

**Por que são necessários**: Sem esses comandos, você só tem o código-fonte. Eles geram os arquivos que o bootloader carregará e os módulos que serão inseridos dinâmicamente.

---

#### **Instalação do Kernel**

**O que é**: Processo de copiar o kernel compilado e seus arquivos auxiliares para o diretório `/boot` e registrar a entrada no bootloader (GRUB, systemd-boot, etc).

**O que faz**:

`vmlinuz` (kernel compactado) - o que o bootloader carrega.

`System.map` - tabela de símbolos (útil para debugging, não usado no boot).

`make install` (em muitas configurações) - já executa `update-grub` ou similar.

**Por que é necessário**: Se você não copiar o kernel para o `/boot` e não atualizar o bootloader, o computador não saberá que ele existe e continuará carregando o kernel antigo (ou nenhum).

---

#### **Módulos compiláveis**

**O que é**: Você pode escrever seu próprio código de kernel (um driver, uma funcionalidade) e compilá-lo como módulo `.ko`, fora da árvore do kernel.

**O que faz**: Adiciona funcionalidades ao kernel em tempo de execução, sem precisar recompilar o kernel inteiro.

Você escreve um arquivo `.c` com `#include <linux/module.h>`, funções `init` e `exit`, e uma licença (Ex: `MODULE_LICENSE("GPL")`). Depois, compila usando o sistema de build do kernel (Makefile especial).

**Por que é necessário**: Se sua distro precisar de um driver exclusivo (hardware muito novo ou personalizado), você poderá escrevê-lo e distribuir como módulo, sem precisar recompilar o kernel todo.

---

#### **Debugging**

**O que é**: Conjunto de técnicas e ferramentas para identificar por que o kernel ou um módulo não funciona.

**O que faz**: Mostra erros de driver, panics, falhas de dependências, versões incompatíveis.

`dmesg`: exibe o anel de mensagens do kernel (erros de driver, panic, falhas de alocação)

`journalctl -k`: logs do kernel no systemd

**Por que é necessário**: Durante a criação da distro, você vai enfrentar muitos kernel panics e dispositivos não funcionando. Sem `dmesg`, você estará voando cego. Aprender a ler essas mensagens é fundamental.

---

#### **Otimização do kernel**

**O que é**: Processo de remover recursos desnecessários e ajustar parâmetros para tornar o kernel menor, mais rápido e mais econômico em memória.

**O que faz**: Diminui tamanho da imagem do kernel, reduz tempo de boot e consumo de RAM.

**Por que é necessário**: Uma distro feita sob medida pode ter um kernel de 3-5 MB (contra 10-15 MB das genéricas). O boot é mais rápido, a RAM usada antes de carregar módulos é menor, e a superfície de ataque diminui (segurança).

---

### **Opções de Configuração**

#### **Conceitos**

**Pseudo-filesystem**: É um sistema de arquivos que não existe no disco rígido. Ele é gerado em tempo real pelo kernel e mantido apenas na memória.

**Device node**: É um arquivo especial (normalmente em `/dev`) que representa um dispositivo de hardware.

**kernel panic**: É um erro fatal do kernel que ele não pode se recuperar. O sistema para imediatamente.

**Rootfs (root filesystem)**: É o sistema de arquivos montado no diretório `/`, o ponto de partida de todo o sistema. Contém `/bin`, `/etc`, `/home`, `/usr`, etc.

**Initramfs**: É uma imagem (geralmente compactada com cpio) de um sistema de arquivos muito pequeno, carregada pelo bootloader junto com o kernel.

**Shebang (`#!`)**: Os dois primeiros caracteres de um script executável, seguidos pelo caminho do interpretador. Exemplo: `#!/bin/bash`.

**ELF (Executable and Linkable Format)**: É o formato padrão de binários executáveis, bibliotecas compartilhadas (`.so`) e arquivos objeto no Linux.

**Block device**: Transferências em blocos de tamanho fixo (Ex: discos, SSDs). Acesso aleatório. Ex: `/dev/sda`.

**Character device**: Transferências byte a byte, sem buffers (Ex: teclado, mouse, porta serial). Ex: `/dev/tty`.

**Hotplug**: Capacidade do sistema operacional de detectar e configurar hardware que é conectado ou desconectado enquanto a máquina está ligada (Ex: pendrive USB).

---

#### **EXT4 - CONFIG_EXT4_FS**

**O que é**: Uma opção de configuração do kernel que ativa o suporte ao sistema de arquivos ext4 (Fourth Extended Filesystem), o mais usado no Linux.

**O que faz**: Permite que o kernel leia e grave arquivos em partições formatadas com ext4. Ele traduz as estruturas dos diretórios, inodes e blocos do disco em arquivos que os programas podem entender.

**Por que é necessário**: Se o seu disco raiz (rootfs) estiver formatado em ext4 e o kernel não tiver essa opção ativada (ou estiver como módulo sem initramfs), o kernel vê apenas bytes brutos, não consegue montar a partição e entra em kernel panic. Para uma distro do zero, você precisa de pelo menos um sistema de arquivos com suporte embutido (`=y`).

---

#### **BLOCK DEVICE - CONFIG_BLK_DEV**

**O que é**: A camada de block devices (dispositivos de blocos) do kernel. É o sistema que gerencia discos rígidos, SSDs, pendrives, etc., e os representa como arquivos especiais do tipo `/dev/sda`, `/dev/nvme0n1`.

**O que faz**: Abstrai o hardware de armazenamento, permitindo que o kernel (e os programas) leiam e escrevam blocos de dados (normalmente 512 ou 4096 bytes) sem se preocupar com os detalhes físicos do controlador.

**Por que é necessário**: Sem `CONFIG_BLK_DEV`, o kernel não enxerga nenhum disco como dispositivo. Não existiriam os arquivos `/dev/sda` e `/dev/nvme0n1`, e não haveria nada para montar. É a base de todo armazenamento no Linux.

---

#### **SCSI - CONFIG_SCSI**

**O que é**: O subsistema SCSI (Small Computer System Interface) do kernel. Embora tenha surgido como um protocolo para discos externos, hoje ele funciona como uma camada de abstração padronizada para comunicação com dispositivos de armazenamento.

**O que faz**: Traduz comandos de leitura/gravação vindos da camada block device em comandos SCSI. Discos SATA modernos usam o protocolo ATA, mas o driver AHCI converte esses comandos para SCSI internamente (via libata). Basicamente, o subsistema SCSI é o "idioma" que o kernel usa para falar com a maioria dos discos.

**Por que é necessário**: Sem o `CONFIG_SCSI`, o kernel não entende os comandos que o controlador SATA/AHCI envia. Mesmo com o block device habilitado, o disco permanece invisível. A cadeia completa é: block device -> SCSI -> driver do controlador -> disco físico.

---

#### **SATA - CONFIG_SATA_AHCI**

**O que é**: O driver para controladores AHCI (Advanced Host Controller Interface), o padrão para discos SATA em placas-mãe modernas.

**O que faz**: Controla diretamente o chip que conecta a CPU e a memória aos discos SATA. Esse driver envia e recebe comandos usando o protocolo SCSI (por isso depende de `CONFIG_SCSI`).

**Por que é necessário**: Sem `CONFIG_SATA_AHCI`, o kernel não consegue sequer tocar no disco, mesmo que as camadas block e SCSI estejam presentes. O disco não aparece em `/dev`. Se o seu computador utiliza disco SATA (tradicionais ou SSD SATA), esse driver é obrigatório, e normalmente deve estar compilado como `y`.

---

#### **NVME - CONFIG_BLK_DEV_NVME**

**O que é**: O driver para discos NVMe (Non-Volatile Memory Express), usados em SSDs modernos diretamente conectados ao barramento PCIe (como os formatos M.2 e PCIe add-in cards).

**O que faz**: Gerencia a comunicação com dispositivos NVMe, que é completamente diferente do protocolo SATA/AHCI. O driver envia e recebe comandos diretamente pelo PCIe, sem usar AHCI ou a camada de emulação SCSI.

**Por que é necessário**: Se o seu sistema possui um SSD NVMe e você habilita apenas `CONFIG_SATA_AHCI`, o kernel não vê o disco. É preciso ativar `CONFIG_BLK_DEV_NVME` (normalmente com `y` ou `m` com initramfs). Muitas distros modernas incluem os dois, mas para uma distro personalizada, você deve saber qual hardware irá usar.

---

#### **PROCFS - CONFIG_PROC_FS**

**O que é**: Ativa o pseudo-filesystem /proc (process filesystem). É um sistema de arquivos virtual que não existe no disco, o kernel o gera dinamicamente em memória.

**O que faz**: Fornece informações sobre processos em execução (`/proc/[pid]/`), uso de CPU (`/proc/cpuinfo`), memória (`/proc/meminfo`), e muitos outros detalhes do sistema. Também permite ajustar parâmetros do kernel em runtime (via `/proc/sys/`).

**Por que é necessário**: Ferramentas básicas como `ps`, `top`, `htop`, `free`, `lsmod` e até o próprio shell (para alguns comandos) dependem do `/proc`. Sem ele, o sistema fica praticamente sem diagnóstico e muitos programas falham. Para qualquer distro que não seja trivial, `CONFIG_PROC_FS` deve ser `y`.

---

#### **SYSFS - CONFIG_SYSFS**

**O que é**: Ativa o pseudo-filesystem /sys (sysfs). Também é um sistema virtual, montado normalmente em `/sys`.

**O que faz**: Expõe uma visão hierárquica e detalhada dos dispositivos, drivers, barramentos e classes do kernel. Por exemplo: `/sys/bus/usb/` lista dispositivos USB, `sys/class/net/` mostra as interfaces de rede, `sys/block/` lista discos. O udev (ou mdev) usa o sysfs para configurar e detectar hardware dinamicamente (hotplug).

**Por que é necessário**: Sem `/sys`, a detecção de dispositivos em tempo real (como conectar um pendrive ou um mouse USB) não funciona. O sistema não sabe que novo hardware foi adicionado. Além disso, programas de configuração de rede, áudio e energia dependem do sysfs. Para uma distro moderna, `CONFIG_SYSFS` é essencial.

---

#### **DEVTMPFS - CONFIG_DEVTMPFS + CONFIG_DEVTMPFS_MOUNT**

**O que é**: Duas opções relacionadas que criam e montam automaticamente o pseudo-filesystem devtmpfs no diretório `/dev`.

**O que faz**:

`CONFIG_DEVTMPFS`: faz o kernel criar internamente um sistema de arquivos temporários que contém os device nodes (arquivos especiais como `/dev/sda`, `/dev/tty`, `/dev/null`).

`CONFIG_DEVTMPFS_MOUNT`: instrui o kernel a montar esse sistema de arquivos  em `/dev` logo no início do boot, antes mesmo de qualquer processo de userspace (como `init`) rodar.

**Por que é necessário**: Sem essas opções, o diretório `/dev` estaria vazio ou você precisaria criar manualmente todos os device nodes (centenas deles) usando `mknod`. Isso é impraticável. Com devtmpfs, o kernel popula `/dev` automaticamente com os dispositivos que ele detecta. Para qualquer sistema que não seja um ambiente totalmente controlado, essas duas opções devem estar ativadas (`=y`).

---

#### **INITRAMFS - CONFIG_BLK_DEV_INITRD**

**O que é**: A opção que permite o kernel carregar e usar um initramfs (Initial RAM filesystem) - uma imagem de um pequeno sistema de arquivos que é carregada na memória junto com o kernel pelo bootloader.

**O que faz**: Quando o kernel inicia, se ele detectar que o bootloader forneceu uma imagem initramfs (normalmente via multiboot ou UEFI), ele a descompacta e usa como sistema de arquivos raiz temporário. Esse initramfs geralmente contém módulos essenciais (por exemplo, drivers de disco que foram compilados como `m`), e um programa `init` que carrega esses módulos e depois monta o verdadeiro root filesystem.

**Por que é necessário**: Se você tem drivers de discos compilados como módulos (`m`) e não usa initramfs, o kernel não conseguirá montar o rootfs porque o módulo ainda não está carregado. O initramfs resolve esse ovo-da-galinha. Além disso, muitas distribuições (incluindo o QEMU padrão) usam initramfs para oferecer flexibilidade. Sem o `CONFIG_BLK_DEV_INITRD`, o kernel simplesmente ignora o initramfs, e o boot falha.

---

#### **ELF - CONFIG_BINFMT_ELF**

**O que é**: A opção que adiciona suporte ao formato executável ELF (Executable and Linkable Format), o padrão usado por quase todos os binários do Linux (incluindo `/bin/ls`, `/bin/bash`, o próprio kernel).

**O que faz**: Ensina o kernel a interpretar o cabeçalho de um arquivo ELF, carregar suas seções na memória (códigos, dados, bibliotecas compartilhadas) e iniciar sua execução. É o que acontece quando você chama `execve()`.

**Por que é necessário**: Sem `CONFIG_BINFMT_ELF`, o kernel não reconhece nenhum binário compilado. Ao tentar executar qualquer comando (até mesmo `sh`), o sistema retorna "Exec format error". Nenhum programa roda. Essa opção é obrigatória (`=y`) para qualquer sistema que não seja um brinquedo. É a mais fundamental de todas.

---

#### **SCRIPTS / SHEBANGS - CONFIG_BINFMT_SCRIPT**

**O que é**: A opção que permite que o kernel execute scripts que começam com uma linha shebang (`#!/caminho/para/interpretador`).

**O que faz**: Quando o kernel encontra um arquivo executável que começa com `#!`, ele lê o caminho do interpretador (por exemplo, `/bin/sh`, `/usr/bin/python`) e executa esse interpretador passando o script como argumento. Sem essa opção, o kernel não entende a linha `#!` e trata o script como um binário desconhecido.

**Por que é necessário**: Praticamente todos os scripts de inicialização (init scripts), scripts de configuração e scripts de ferramenta usam shebang. Sem `CONFIG_BINFMT_SCRIPT`, você não consegue rodar nenhum script shell, Python, Perl, etc., diretamente. O sistema ainda funcionará se você chamar explicitamente o interpretador (`sh script.sh`), mas muitos programas e procedimentos de boot assumem a execução direta com o shebang. Para um distro prática, é recomendado ativar como `y`.
