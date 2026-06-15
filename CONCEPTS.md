# Linux From Scratch

Linux From Scratch é um livro e projeto que ensina a contruir um sistema Linux inteiro a partir do código-fonte, incluindo ferramentas, bibliotecas e kernel. Ele fornece o passo a passo para compilar tudo manualmente, gerando uma distribuição sua, do zero.

Para criar uma nova distro, você precisa entender cada componente. LFS é o roteiro definitivo - você vai usá-lo para saber a ordem correta de compilar binutils, GCC, glibc, e finalmente o kernel.

### Conceitos

**Sistema Operacional**: Os Sistemas Operacionais são softwares que gerenciam os aplicativos e os recursos de um dispositivo. Eles realizam tarefas para garantir o bom funcionamento do aparelho. Apesar de cumprirem os mesmos papaies, cada sistema operacional fornece seus próprios recurso, interface e política de segurança.

**Kernel**: Kernel é o núcleo do Sistema Operacional, ele é o responsável por gerenciar os recursos de Hardware do computador e garantir que cada processo tenha acesso aos recursos necessários para funcionar corretamente. O Kernel é a unica parte do Sistema Operacional que tem acesso direto e irrestrito aos Hardwares do computador. Ele gerencia a memória, escalona os processos na CPU, controla a rede e conversa com os discos rígidos, além de executar programas e se conectar com os periféricos. Todo o resto (shell, comandos, interface gráfica) são apenas "espaço de usuário" (user space) pedindo favores ao Kernel através de chamadas de sistema (syscalls).

**Shell**: Shell significa casca ou concha, esta metafóra é a chave para entender o conceito. Sua função primordial é fornecer uma interface gráfica que permite o usuário acessar os serviços do Sistema Operacional. Ele é a camada mais externa de um OS, o Shell é quem envolve o Kernel e serve como interface primária através da qual o usuário interage com a máquina.

**Firmware**: Firmware é uma camada intermediária crucial entre hardware e software. Diferente do Software, que pode ser alterado e modificado pelo usuário final, o fimware é gravado em memória de acesso somente leitura (ROM, EPROM ou PROM) durante o processo de fabricação do dispositivo. A principal função do firmware é apresentar instruções específicas para o Hardware a fim de realizar determinadas tarefas. Ao ligar a máquina, ele é o responsável pela inicialização do dispositivo, por carregar a BIOS ou o UEFI e preparar o Sistema Operacional para a inicialização. Após a inicialização do OS, o firmware atua como uma camada intermediária entre o Hardware e o Sistema Operacional.

**Bootloader**: Bootloader é um software que permite a inicialização do Sistema Operacional de todos os dispositivos. Sempre que o dispositivo é ligado, ele acionará o bootloader para carregar o OS. Além disso, o sftware também funciona com garantia, caso ocorra alguma falha crítica com esse sistema.

---

### Diretórios

#### **arch/**

**O que é**: Subdiretório do kernel que contém a arquitetura do processador (x86, ARM, PowerPC, etc).

**O que faz**: Ele é o responsável por gerenciar a inicialização do processador, a configuração da MMU (Memory Management Unity) e o tratamento de interrupções via APIC (Advanced Programmable Interrupt Controller).

**Por que é necessário**: Cada CPU tem seu próprio jeito de ligar e gerenciar seus recursos. Sem o diretório `arch/`, o kernel não saberia como conversar com o hardware da sua placa mãe.

---

#### **drivers/**

**O que é**: A maior parte do código-fonte do kernel. Contém os drivers de dispositivo.

**O que faz**: Ensina o kernel a interagir com os Hardwares do computador (discos, placas de rede, USB, GPU, etc).

**Por que é necessário**: Sem drivers, o kernel é cego e surdo - não acessa o teclado, não mostra vídeo, não lê o disco onde está a sua distro.

---

#### **fs/ (Fylesystem)**

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

**O que faz**: Permite enviar e receber pacotes de rede, montar conexões TCP, tansmitir datagramas UDP, aplicar firewall, etc.

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

**Por que é necessário**: O `.config` é o DNA do seu kernel. Sem ele, você não controla o tamanho, os drivers, nem as funcionalidades. Para um dsitro enxuta, você precisa do `.config` sob medida - nem inchado (como as distros genéricas), nem capado a ponto de não bootar.

---

#### **A trindade: y / m / n**

**O que é**: São os três valores que cada recurso do kernel pode receber no `.config`.

**O que faz**:

*y (Yes)*: O código vira parte do `vmlinuz`. Está sempre em memória desde o boot.

*m (Module)*: O código fica em um arquivo separado (`/lib/modules/.../driver.ko`). Pode ser carregado (`insmod`, `modprobe`) e descarregado (`rmmod`).

*n (No)*: O código nem é compilado.

**Por que é necessário**: O `y` é essencial para drivers de boot; O `m` reduz o tamanho do kernel principal e acelera o boot. Ideal para drivers de hardware que você usa maneira irregular; Usar o `n` economiza espaço em disco e memória. Remove dados que seu hardware nunca usará e também reduz possíveis vulnerabilidades.

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

**Por que são necessários**: Se faltar o diver do seu controlador de disco (Ex: NVMe), o kernel não encontra a partição raiz e o boot falha. Esse divers precisam estar compilados dentro (`y`) ou em um initramfs.

---

### **Compilação**

#### **Códigos de Compilação**

**O que são**: Comandos que acionam o Kbuild para traduzir código-fonte em binário.

**O que fazem**:

`make`: compila kernel (gera `vmlinuz` ou `bzImage`)

`make modules`: compila todos os códigos marcados com `m`.

`make modules_install`: instala os módulos em `/lib/modules/$(uname -r)/`.

**Por que são necessários**: Sem esse comandos, você só tem o código-fonte. Eles geram os arquivos que o bootloader carregará e os módulos que serão inseridos dinâmicamente.

---

#### **Instalação do Kernel**

**O que é**: Processo de copiar o kernel compilado e seus arquivos auxiliares para o diretório `/boot` e registrar a entrada no bootloader (GRUB, systemd-boot, etc).

**O que faz**:

`vmlinuz` (kernel compactado) - o que o bootloader carrega.

`System.map` - tabela de símbolos (útil para debbuging, não usado no boot).

`make install` (em muitas configurações) - já executa `update-grub` ou similar.

**Por que é necessário**: Se você não copiar o kernel para o `/boot` e não atualizar o bootloader, o computador não saberá que ele existe e continuará carregando o kernel antigo (ou nenhum).

---

#### **Módulos compiláveis**

**O que é**: Você pode escrever seu próprios código de kernel (um driver, uma funcionalidade) e compilá-lo como módulo `.ko`, fora da árvore do kernel.

**O que faz**: Adiciona funcionalidades ao kernel em tempo de execução, sem precisar recompilar o kernel inteiro.

Você escreve um arquivo `.c` com `#include <linux/module.h>`, funções `init` e `exit`, e uma licença (Ex: `MODULE_LICENSE("GPL")`). Depois, compila usando o sistema de build do kernel (Makefile especial).

**Por que é necessário**: Se sua distro precisar de um driver exclusivo (hardware muito novo ou personalizado), você poderá escreve-lo e distribuir como módulo, sem precisar recompilar o kernel todo.

---

#### **Debugging**

**O que é**: Conjunto de técnicas e ferramentas para indentificar por que o kernel ou um módulo não funciona.

**O que faz**: Mostra erros de driver, panics, falhas de dependências, versões incompatíveis.

`dmesg`: exibe o anel de mensagens do kernel (erros de driver, panic, falhas de alocação)

`journalctl -k`: logs do kernel no systemd

**Por que é necessário**: Durante a criação da distro, você vai enfrentar muitos kernel panics e dispositivos não funcionando. Sem `dmesg`, você estará voando cego. Aprender a ler essas mensagens é fundamental.

---

#### **Otimização do kernel**

**O que é**: Processo de remover recursos desnecessários e ajustar parâmetros para tornar o kernel menor, mais rápido e mais econômico em memória.

**O que faz**: Diminui tamanho da imagem do kernel, reduz tempo de boot e consumo de RAM.

**Por que é necessário**: Uma distro feita sob medida pode ter um kernel de 3-5 MB (contra 10-15 MB das genéricas). O boot é mais rápido, a RAM usada antes de carregar módulos é menos, e a superfície de ataque diminui (segurança).
