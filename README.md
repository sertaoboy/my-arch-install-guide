## my-arch-install-guide
Repositorio para armezenar comandos e algumas dicas na instalacao do ArchLinux para o formato de arquivo B-tree FileSystem (btrfs).


> `loadkeys br-abnt2`
- Comando para colocar o teclado em portugues
<br>

# Networking
> `ping archlinux.org`
- Testar a conexao wifi
- Caso nao esteja vizualiando as respotas, configurar a conexao via CLI:
- `iwctl`
- Uma vez executado, use `device list` na execucao do iwctl.
- Comando para se scanear as redes disponiveis: `station <seu-dispositivo-de-wireless> scan`
- No meu caso: `station wlan0 scan`
- Depois: `station wlan0 get-networks`
- E finalmente para se conectar: `station wlan0 connect <nome-da-rede-listada>`
- Certifique-se que esta conectado: `ping google.com`
<br>

# Formating
- Primeiro, identificar qual o dispositivo ira formatar: `lsblk`
- Apos identificar: `cfdisk /dev/<dispositivo_escolhido>`
- No meu caso: `cfdisk /dev/sda`
- Configurar no particionador 3 particoes (no minimo), "boot", "/", "home"; Podera optar por acrescentar mais uma sendo a SWAP
- Exemplo para os tamanhos das particoes : 576MB para /boot/efi; 2GB para SWAP; 44GB para /; 209GB para /home
- Confirme as alteracoes no cfdisk
## Formatando e montando as particoes
- Particao boot:
> `mkfs.fat -F32 /dev/sda1`
- Particao swap:
> `mkswap /dev/sda2`
- Particao raiz (/):
> `mk.btrfs /dev/sda3`
- Particao home:
> `mkfs.btrfs /dev/sda4`
- Exemplos no meu caso, atencao aos dispositivos escolhidos para a formatacao. Tip: `lsblk` every moment :)
- Obs: Caso prefira usar outro formato de arquivo, basta trocar o comando `mkfs.btrfs ...` por `mkfs.ext4` ou `mkfs.f2fs` por exemplo.
- Montando a raiz:
> `mount /dev/sda3 /mnt`
- Criando os diretorios:
> `mkdir /mnt/home` Diretorio home
> `mkdir /mnt/boot` Diretorio boot
> `mkdir /mnt/boot/efi` Diretorio /boot/efi
- Uma vez montados, hora de persistir esses diretorios em cada particao formatada:
> `mount /dev/sda4 /mnt/home` Particao home
> `mount /dev/sda1 /mnt/boot` Particao boot
> `mount /dev/sda1 /mnt/boot/efi` Particao /boot/efi
> `swapon /dev/sda2` Ativando a particao SWAP

## Pacstrap - download do sistema
- Dica para essa etapa: sincronizar o melhor mirror para sua conexao;
- Comente todos os mirros que nao sao brasileiros (no meu caso) ou suba o desejado para o topo da lista em: `nano /etc/pacman.d/mirrorlist`
- `pacstrap /mnt base base-devel linux linux-firmware nano dhcpcd` instalando o sistema (few minutes to complete)

## Tabela FSTAB
- Agora vamos mostrar para o sistema onde estao montada as particoes;
> `genfstab -U -p /mnt >> /mnt/etc/fstab`
- Verificar se o arquivo foi gerado de fato: `cat /mnt/etc/fstab`

## Chroot
- `arch-chroot /mnt` Para entrar no sistema
- ` ln -sf /usr/share/zoneinfo/America/Sao_Paulo /etc/localtime` para configurar a data e horario
- `hwclock --systohc` Para sincronizar o relogio da BIOS
- `date` Para conferir a data
- `nano /etc/locale.gen` para mudar o idioma do sistema, no caso so tirar o "#" antes de: pt_BR.UTF-8 UTF-8 pt_BR ISO-8859-1
- `localegen` para gerar o arquivo
- `echo KEYMAP=br-abnt2 >> /etc/vconsole.conf` para configurar a variavel da linguagem
- `passwd` para configurar a senha de root
- `useradd -m -g users -G wheel,storage,power -s /bin/bash <NOME_DO_SEU_USUARIO_ESCOLHIDO>` para adicionar o seu usuario
- `<NOME_DO_SEU_USUARIO_ESCOLHIDO> passwd` para configurar a senha do usuario escolhido
- `pacman -S dosfstools os-prober mtools network-manager-applet networkmanager wpa_supplicant wireless_tools dialog` Pacotes necessarios antes do uso do sistema

## GRUB
> `pacman -S grub efibootmgr` para instalar <br>
> `grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=arch_grub --reecheck` um dos comandos mais errados durante a instalacao <br>
> Se nao deu nenhum erro, basta gerar o arquivo do GRUB com: `grub-mkconfig -o /boot/grub/grub.cfg`<br>

## Sudoers
- Para adicionar o seu usuario ao sudoers: `EDITOR=nano visudo` e procurar a linha `"%wheel ALL=(ALL:ALL) ALL"`. Descomente a linha e salve o arquivo.

## DEs
- `sudo pacman -S xorg-server xorg-xinit xorg-apps mesa`
- AMD: `sudo pacman -S xf86-video-amdgpu`
- Intel: `sudo pacman -S xf86-video-intel`
- Nvidia: `sudo pacman -S nvidia nvidia-settings`
- Agora, para instalar o Plasma, KDE e SDDM: `sudo pacman -S plasma konsole firefox sddm` 
> Atualizando o servico para o login no boot pelo systemd: `systemctl enable sddm` para o SDDM <br>
> Faca o mesmo para o NetworkManager: `systemctl enable NetworkManager` <br>
> O mesmo para bluetooth: `systemctl enable bluetooth.service`


* <strong>Parabens!</strong> Voce acabou de instalar o arch!
* `exit` caso nao tenha mais nenhuma configuracao para acrescentar enquanto chroot!



