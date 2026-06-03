# Linux From Scratch
### Conceitos
**Sistema Operacional**: Os Sistemas Operacionais são softwares que gerenciam os aplicativos e os recursos de um dispositivo. Eles realizam tarefas para garantir o bom funcionamento do aparelho. Apesar de cumprirem os mesmos papaies, cada sistema operacional fornece seus próprios recurso, interface e política de segurança.

**Kernel**: Kernel é o núcleo do Sistema Operacional, ele é o responsável por gerenciar os recursos de Hardware do computador e garantir que cada processo tenha acesso aos recursos necessários para funcionar corretamente. O Kernel é a unica parte do Sistema Operacional que tem acesso direto e irrestrito aos Hardwares do computador. Ele gerencia a memória, escalona os processos na CPU, controla a rede e conversa com os discos rígidos, além de executar programas e se conectar com os periféricos. Todo o resto (shell, comandos, interface gráfica) são apenas "espaço de usuário" (user space) pedindo favores ao Kernel através de chamadas de sistema (syscalls).

**Shell**: Shell significa casca ou concha, esta metafóra é a chave para entender o conceito. Sua função primordial é fornecer uma interface gráfica que permite o usuário acessar os serviços do Sistema Operacional. Ele é a camada mais externa de um OS, o Shell é quem envolve o Kernel e serve como interface primária através da qual o usuário interage com a máquina.

**Firmware**: Firmware é uma camada intermediária crucial entre hardware e software. Diferente do Software, que pode ser alterado e modificado pelo usuário final, o fimware é gravado em memória de acesso somente leitura (ROM, EPROM ou PROM) durante o processo de fabricação do dispositivo. A principal função do firmware é apresentar instruções específicas para o Hardware a fim de realizar determinadas tarefas. Ao ligar a máquina, ele é o responsável pela inicialização do dispositivo, por carregar a BIOS ou o UEFI e preparar o Sistema Operacional para a inicialização. Após a inicialização do OS, o firmware atua como uma camada intermediária entre o Hardware e o Sistema Operacional.

**Bootloader**: Bootloader é um software que permite a inicialização do Sistema Operacional de todos os dispositivos. Sempre que o dispositivo é ligado, ele acionará o bootloader para carregar o OS. Além disso, o sftware também funciona com garantia, caso ocorra alguma falha crítica com esse sistema.

### Opções de configuração
y - (Yes/Built-in): O código será compilado diretamente para dentro do arquivo executável final do kernel (`vmlinuz`). O recurso estará disponível no exato milissegundo em que o kernel for carregado na RAM pelo bootloader.

m - (Module): O código será compilado com um arquivo separado, com extensão `.ko` (Kernel Object). Esses arquivos ficarão armazenados no seu disco rígido. O kernel só carregará esse módulo na RAM se detectar o hardware correspondente.

n - (No): O código é ignorado pelo compilador.
