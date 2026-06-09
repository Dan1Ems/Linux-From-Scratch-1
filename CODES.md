### Explicação do código
**ATENÇÃO**: Utilize `make mrproper` para resetar o .config do arquivo caso algo dê errado.

Instalação dos pacotes necessários:
```
sudo apt update && sudo apt install -y build-essential libncurses-dev bison flex libssl-dev libelf-dev bc cpio wget xorriso grub-pc-bin grub-efi-amd64-bin grub-common mtools squashfs-tools qemu-system-x86 tar xz-utils  
```

Baixa arquivos da internet:
```
sudo wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.1.60.tar.xz  
```

Descompacta o arquivo baixado:
```
sudo tar -xvf linux-6.1.60.tar.xz
```

Muda para o diretório do arquivo descompactado:
```
cd linux-6.1.60
```

Cria o arquivo de configuração:
```
sudo make defconfig
```

Entra no arquivo de configuração .config:
```
sudo nvim .config # Ou algum outro editor de texto
```

No arquivo .config procure por (procure pelos nomes):
```
CONFIG_BINFMT_ELF=y
CONFIG_EXT4_FS=y
CONFIG_BLK_DEV=y
CONFIG_SCSI=y
CONFIG_SATA_AHCI=y
CONFIG_BLK_DEV_NVME=y
CONFIG_PROC_FS=y
```

Checa quais módulos o seu computador precisa pra rodar:
```
sudo make localmodconfig # Aperte Yes para tudo
```

Gera imagem + móulos:
```
sudo make -j$(nproc)
```

Volta para a home:
```
cd $HOME
```

Faz o download o BusyBox:
```
sudo wget https://busybox.net/downloads/busybox-1.37.0.tar.bz2
```

Descompactando o arquivo instalado:
```
tar -xvf busybox-1.37.0.tar.bz2
```

Muda para o diretório do arquivo descompactado
```
cd busybox-1.37.0
```

Cria o arquivo de configuração:
```
sudo make defconfig
```

É necessario que o BusyBox esteja no modo estático, para isso, vamos verificar no arquivo de configuração:
```
cat .config | grep "STATIC"
```

Deve aparecer algo como:
```
# CONFIG_STATIC is not set
# CONFIG_FEATURE_LIBBUSYBOX_STATIC is not set
CONFIG_STATIC_LIBGCC=y
```

Nesse caso, é necessário alterar o CONFIG_STATIC dento do arquivo .config:
```
sed ´s/# CONFIG_STATIC is not set/CONFIG_STATIC=y´ .config
```

Troque também o CONFIG_TC no arquivo .config:
```
sed ´s/CONFIG_TC/# CONFIG_TC is not set´ .config
```
Instale o musl:
```
sudo apt install -d musl
```
Faça a compilação:
```
sudo make -j$(nproc)
```

Volta para a home:
```
cd $HOME
```

Cria o diretório initramfs (carregar o file system na ram):
```
mkdir initramfs; cd initramfs; mkdir bin proc sys dev mnt
```

Copia o BusyBox para o bin do novo diretório:
```
cp busybox-1.37.0/busybox initramfs/bin
```

Vá para o diretório bin:
```
cd initramfs/bin
```

Dá a devida permissão para o executável copiado:
```
chmod +755 busybox
```

Crie um arquivo de script:
```
touch script.sh
```

Dentro do arquivo script.sh, coloque:
```
#!/bin/bash

for programa in $(./busybox --list); do
        ln -s busybox ./$programa
done
```

Dê permissão para o arquivo script.sh:
```
chmod +755 script.sh
```

Execute o script:
```
./script.sh
```

Volte para o arquivo initramfs:
```
cd ..
```

Crie um arquivo init:
```
nvim init
```

Dentro do arquivo init, coloque:
```
#!/bin/sh

mount -t proc none /proc
mount -t sysfs none /sys
mount -t devtmpfs none /dev

echo "Inicializando a disto..."
```

Dê a permissão para o arquivo init:
```
chmod +755 init
```
