**Propósito/público-alvo**: Designers / Ricers

**Tamanho**: ~450–900 MB

**Ferramentas**: binutils, gcc, glibc, linux, linux-headers, coreutils, util-linux, make, bash, file, pkg-config, grep, sed, tar, gzip, bzip2, xz, lzip, gettext, tzdata, e2fsprogs, systemd, wayland, wayland-protocols, libffi, libepoxy, mesa, libdrm, libxkbcommon, wlroots, hyprland, mako, swaybg, wl-clipboard, grim, slurp, wf-recorder, pipewire, wireplumber, alsa-lib, zsh, starship, kitty, neovim, quickshell, rofi, bemenu, sxhkd, swaylock, squashfs-tools, grub

**Visual da sua distro**: Mínimo com apenas um wallpaper minimalista de arroz

**Segurança**: 

Login com senha: sim — contas locais exigem senha; desabilitar root login direto e usar sudo para tarefas administrativas.

Firewall: habilitar firewall por padrão com política "negra entrada, permite saída" (nftables): default deny incoming, allow established/related, permitir loopback; fornecer comando simples para administrar (nft list ruleset / nft add rule …) ou usar front-end leve.

Criptografia: oferecer LUKS2 full-disk encryption durante instalação (root + swap). /boot fica separado e não criptografado por simplicidade; instruir usuário a salvar backup da header LUKS.
