# Criando a distro

## Explicação do código

**ATENÇÃO**: Utilize `make mrproper` para resetar o .config do arquivo caso algo dê errado.

### Instalação

Para que possamos criar a nossa distro, é necessário a instalação de alguns pacotes essenciais. Eles forncem os recursos básicos para compilar do código, preparar imagens de sistema, instalar/configurar bootloaders e testar em VMs. Formando a cadeia completa de `build -> empacotamento -> boot -> teste`.

``` bash
sudo apt update && sudo apt install -y build-essential libncurses-dev bison flex libssl-dev libelf-dev bc cpio wget xorriso grub-pc-bin grub-efi-amd64-bin grub-common mtools squashfs-tools qemu-system-x86 tar xz-utils  
```

Agora, temos que baixar o tarball (conjunto de ficheiros e diretórios num arquivo TAR) do código fonte do kernel Linux versão 6.1.60. Ele será necessário para a compilação, gerar o initramfs e construir a imagem do sistema.

``` bash
sudo wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.1.60.tar.xz  
```

Descompacte o arquivo instalado:

``` bash
sudo tar -xvf linux-6.1.60.tar.xz
```

Muda para o diretório do arquivo descompactado:

``` bash
cd linux-6.1.60
```

Agora temos que fazer a configutação do kernel, mas não faremos isso na mão. Vamos utilizar um comando que gera um arquivo de configuração padrão, é muito prático para que não escreva em torno de 15 mil variáveis.

``` bash
sudo make defconfig
```

Entretanto, o arquivo de configurção padrão vem com algumas opções desativadas. Elas são importantes para a comunicação do entre kernel, CPU e disco ser possível. Nesse caso, será necessário utilizar o `menuconfig`.

``` bash
make menuconfig
```

Ao digitar esse comando, uma interface aparecerá na sua tela, é por meio dela que você vai alterar as opções de configuraçao. Utilizar o `menuconfig` é necessário para lidar com as dependencias de cada opção.

No `menuconfig` procure pelas opções de configuração descritas abaixo. Para facilitar, utilize o mecanismo de busca.

Caso alguma das opções não esteja como YES na sua configuração, altere manualmente utilizando o `menuconfig`.

|Opção|Caminho no menuconfig|Porquê|
|-----|---------------------|------|
|Suporte a ext4|File systems > Ext4|Nosso rootfs será ext4|
|Suporte a SquashFS|File systems > Miscellaneous > SquashFS|Para o Live CD ler o filesystem compactado|
|OverlayFS|File systems > Overlay filesystem|Para a camada de escrita do Live CD|
|Suporte a ISO 9660|File systems > ISO 9660|Para o GRUB ler o CD|
|Suporte a FAT/VFAT|File systems > DOS/FAT/VFAT|Para a partição EFI|
|Suporte a tmpfs|File systems > Pseudo filesystems > tmpfs|Para o OverlayFS na RAM|
|Suporte a disco SATA/AHCI|Device Drivers > Serial ATA/ATAPI > AHCI|Para ler discos SATA|
|Suporte a NVMe|Device Drivers > NVM Express block device|Para ler SSDs modernos|
|Suporte a USB Storage|Device Drivers > USB > USB Mass Storage|Para ler pendrives|
|Suporte a loop device|Device Drivers > Block devices > Loopback|Para montar imagens|

Agora precisamos garantir que os módulos que precisaremos estarão carregados. Para isso, vamos usar o `localmodconfig`, ele desativa opções do kernel que não serão necessárias e mantém as opções de drivers utilizados pelo sistema atual, perguntando interativamente sobre os itens opcionais. Para verificar isso vamos rodar o comandos a seguir.

``` bash
sudo make localmodconfig 
# Aperte Yes para tudo
```

**REFAZER DAQUI**

Gera imagem + móulos:

``` bash
sudo make -j$(nproc)
```

Volta para a home:

``` bash
cd $HOME
```

Faz o download o BusyBox:

``` bash
sudo wget https://busybox.net/downloads/busybox-1.37.0.tar.bz2
```

Descompactando o arquivo instalado:

``` bash
tar -xvf busybox-1.37.0.tar.bz2
```

Muda para o diretório do arquivo descompactado

``` bash
cd busybox-1.37.0
```

Cria o arquivo de configuração:

``` bash
sudo make defconfig
```

É necessario que o BusyBox esteja no modo estático, para isso, vamos verificar no arquivo de configuração:

``` bash
cat .config | grep "STATIC"
```

Deve aparecer algo como:

``` bash
# CONFIG_STATIC is not set
# CONFIG_FEATURE_LIBBUSYBOX_STATIC is not set
CONFIG_STATIC_LIBGCC=y
```

Nesse caso, é necessário alterar o CONFIG_STATIC dento do arquivo .config:

``` bash
sed ´s/# CONFIG_STATIC is not set/CONFIG_STATIC=y´ .config
```

Troque também o CONFIG_TC no arquivo .config:

``` bash
sed ´s/CONFIG_TC/# CONFIG_TC is not set´ .config
```

Instale o musl:

``` bash
sudo apt install -d musl
```

Faça a compilação:

``` bash
sudo make -j$(nproc)
```

Volta para a home:

``` bash
cd $HOME
```

Cria o diretório initramfs (carregar o file system na ram):

``` bash
mkdir initramfs; cd initramfs; mkdir bin proc sys dev mnt
```

Copia o BusyBox para o bin do novo diretório:

``` bash
cp busybox-1.37.0/busybox initramfs/bin
```

Vá para o diretório bin:

``` bash
cd initramfs/bin
```

Dá a devida permissão para o executável copiado:

``` bash
chmod +755 busybox
```

Crie um arquivo de script:

``` bash
touch script.sh
```

Dentro do arquivo script.sh, coloque:

``` bash
#!/bin/bash

for programa in $(./busybox --list); do
        ln -s busybox ./$programa
done
```

Dê permissão para o arquivo script.sh:

``` bash
chmod +755 script.sh
```

Execute o script:

``` bash
./script.sh
```

Volte para o arquivo initramfs:

``` bash
cd ..
```

Crie um arquivo init:

``` bash
touch init
```

Dentro do arquivo init, coloque:

``` bash
#!/bin/sh

mount -t proc none /proc
mount -t sysfs none /sys
mount -t devtmpfs none /dev

echo "Inicializando a disto..."
```

Dê a permissão para o arquivo init:

``` bash
chmod +755 init
```

Crie a sua variável da pasta da distro:

``` bash
export DISTRO=/home/administrador/distro
```

Vá para a sua pasta distro:

``` bash
cd distro
```

Crie o diretório rootfs:

``` bash
mkdir rootfs
```

Crie os diretórios:

``` bash
mkdir bin sbin dev proc mnt sys etc run root home lib var temp
```

Copie o busybox pro diretório bin:

``` bash
cp ~/busybox-1.37.0/busybox bin/
```

Entre no diretório bin:

``` bash
cd bin
```

Crie um arquivo temp:

``` bash
nvim temp
```

Dentro, coloque:

``` bash
#!/bin/bash

for prog in $(./busybox --list); do
        ln -s busybox ./$prog
done
```

Dê permissão:

``` bash
chmod +755 temp
```

Execute o script:

``` bash
./temp
```

Remova o script temporário:

``` bash
rm temp
```

Navegue para a pasta etc:

``` bash
cd ../etc
```

Crie o diretório:

``` bash
mkdir init.d
```

Navegue para o diretório var:

``` bash
cd ../var
```

Crie os diretórios:

``` bash
mkdir log run lib
```

Volte para o diretório etc:

``` bash
cd ../etc
```

Crie um arquivo chamado group:

``` bash
nvim group
```

Nele, coloque:

``` bash
#!/bin/sh

root:x:0:0:root:/root:/bin/sh
```

Dê a permissão:

``` bash
chmod +755 group
```

Crie outro arquivo:

``` bash
nvim inittab
```

Nele, coloque:

``` bash
#!/bin/sh

::sysinit:/etc/init.d/rcS

ttyS0::respawn:/bin/sh
tty1::respawn:/bin/sh

::ctrlaltdel:/bin/reboot
::shutdown:/bin/unmount -a -r
```

Dê a permissão:

``` bash
chmod +755 inittab
```

Entre no diretório init.d:

``` bash
cd init.d
```

Crie o arquivo rcS (run command SINGLE):

``` bash
nvim rcS
```

Dentro, coloque:

``` bash
#!/bin/sh

mount -t proc none /proc
mount -t sysfs none /sys
mount -t devtmpfs none /dev

mkdir /dev/pts
mount -t devpts none /dev/pts
```

Crie a sua variável da pasta da distro:

``` bash
export DISTRO=/home/administrador/distro
```

Vá para a sua pasta distro:

``` bash
cd distro
```

Crie o diretório rootfs:

``` bash
mkdir rootfs
```

Crie os diretórios:

``` bash
mkdir bin sbin dev proc mnt sys etc run root home lib var temp
```

Copie o busybox pro diretório bin:

``` bash
cp ~/busybox-1.37.0/busybox bin/
```

Entre no diretório bin:

``` bash
cd bin
```

Crie um arquivo temp:

``` bash
nvim temp
```

Dentro, coloque:

``` bash
#!/bin/bash

for prog in $(./busybox --list); do
        ln -s busybox ./$prog
done
```

Dê permissão:

``` bash
chmod +755 temp
```

Execute o script:

``` bash
./temp
```

Remova o script temporário:

``` bash
rm temp
```

Navegue para a pasta etc:

``` bash
cd ../etc
```

Crie o diretório:

``` bash
mkdir init.d
```

Navegue para o diretório var:

``` bash
cd ../var
```

Crie os diretórios:

``` bash
mkdir log run lib
```

Volte para o diretório etc:

``` bash
cd ../etc
```

Crie um arquivo chamado group:

``` bash
nvim group
```

Nele, coloque:

``` bash
root:x:0:0:root:/root:/bin/sh
```

Dê a permissão:

``` bash
chmod +755 group
```

Crie outro arquivo:

``` bash
nvim inittab
```

Nele, coloque:

``` bash
::sysinit:/etc/init.d/rcS

ttyS0::respawn:/bin/sh
tty1::respawn:/bin/sh

::ctrlaltdel:/bin/reboot
::shutdown:/bin/unmount -a -r
```

Dê a permissão:

``` bash
chmod +755 inittab
```

Entre no diretório init.d:

``` bash
cd init.d
```

Crie o arquivo rcS (run command SINGLE):

``` bash
nvim rcS
```

Dentro, coloque:

``` bash
!/bin/sh

# pseudofilesystems essenciais
mount -t proc none /proc
mount -t sysfs none /sys
mount -t devtmpfs none /dev

# terminais para os pseudo-terminal
mkdir /dev/pts
mount -t devpts none /dev/pts

# gerenciador de dispositivos
echo /bin/mdev > /proc/sys/kernel/hotplug
mdev -s
```

Dentro do etc, crie o arquivo passwd:

``` bash
touch passwd
```

Dentro, coloque:

``` bash
root:x:0:0:root:/root:/bin/sh
bin:x:1:1:bin:/dev/null:/bin/false
daemon:x:6:6:Daemon User:/dev/null:/bin/false
messagebus:x:18:18:D-Bus Message Daemon User:/run/dbus:/bin/false
nobody:x:65534:65534:Unprivileged User:/dev/null:/bin/false
```

Crie o arquivo group:

``` bash
touch group
```

Nele, coloque:

``` bash
root:x:0:
bin:x:1:daemon
sys:x:2:
kmem:x:3:
tape:x:4:
tty:x:5:
daemon:x:6:
floppy:x:7:
disk:x:8:
lp:x:9:
dialout:x:10:
audio:x:11:
video:x:12:
utmp:x:13:
usb:x:14:
cdrom:x:15:
adm:x:16:
messagebus:x:18:
input:x:24:
mail:x:34:
kvm:x:61:
wheel:x:97:
nogroup:x:65533:
nobody:x:65534:
```

Crie o arquivo hostname:

``` bash
touch hostname
```

No diretório etc, digite o comando:

``` bash
echo "my-distro" > /etc/hostname
```

Para testar, crie o arquivo profile:

``` bash
touch profile
```

Nele, coloque:

``` bash
export FEATURE_SH_STANDALONE=1
export PATH=/bin

echo ""
echo "TESTANDO DISTRO"
echo ""
```

Volte para a pasta rootfs:

``` bash
cd ..
```

Execute o comando:

``` bash
sudo chroot /home/administrador/distro/rootfs /bin/sh -l
```

Caso compilar o kernel tenha dado errado, dentro do busybox digite:

``` bash
make CONFIG_PREFIX=<caminho_para_distro_rootfs> install
```
