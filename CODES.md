### Explicação do código
Atualiza a lista de pacotes, ele não instala nem atualiza nada, mas busca as informações mais recentes. Rodamos ele antes de qualquer atualização do sistema:
```
sudo apt update
```

Por outro lado, esse comando atualiza todos os pacotes para a versão mais recente disponiveis na lista que você acabou de atualizar:
```
sudo apt upgrade
```

Esse comando serve para instalar os pacotes e suas dependencias, a flag "-y" serve para confirmar automaticamente a instalação dos pacotes:
```
sudo apt install -y build-essential libncurses-dev bison flex libssl-dev libelf-dev bc cpio wget xorriso grub-pc-bin grub-efi-amd64-bin grub-common mtools squashfs-tools qemu-system-x86 tar xz-utils
```

Serve para baixar arquivos diretamente da internet:
```
wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.1.60.tar.xz
```

Utilizado para a descompactação de arquivos .tar.gz, as flags "-x" significa "extraction", "-v" significa "verbose" (lista o nome dos arquivos quando são processados), e "-f" específica o nome do arquivo que será lido:
```
sudo tar -xvf linux-6.1.60.tar.xz
```
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
