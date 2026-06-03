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
