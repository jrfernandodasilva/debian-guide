# Debian 13

## 1. Add user to sudores
>save current user in a temp file and log as root user
```bash
echo "$USER" > /tmp/current_user.txt
su - 
```

>then execute this
```bash
ORIGINAL_USER=$(cat /tmp/current_user.txt)
sed -i "/^root.*ALL=(ALL:ALL) ALL$/a $ORIGINAL_USER ALL=(ALL) ALL" /etc/sudoers
rm /tmp/current_user.txt
exit
```

## 2. Update repositories
>update system repositories and run `update` and `upgrade`
```bash
echo "Types: deb deb-src
URIs: https://deb.debian.org/debian
Suites: trixie trixie-updates
Components: main contrib non-free non-free-firmware
Enabled: yes
Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg

Types: deb deb-src
URIs: https://security.debian.org/debian-security
Suites: trixie-security
Components: main contrib non-free non-free-firmware
Enabled: yes
Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg
"| sudo tee /etc/apt/sources.list.d/debian.sources > /dev/null && sudo apt update && apt list --upgradable && sudo apt upgrade
```

## 3. Install NVIDIA driver
See [NVIDIA-DRIVER-INSTALLATION.md](https://github.com/jrfernandodasilva/debian-guide/blob/main/Debian-13/NVIDIA-DRIVER-INSTALLATION.md) document instructions.

## 4. Atualizar Firefox ESR

1. **Baixar o Firefox ESR**  
   Acesse [https://www.firefox.com/pt-BR/download/all/desktop-esr/](https://www.firefox.com/pt-BR/download/all/desktop-esr/), selecione "Português (Brasil)" e baixe o arquivo `.tar.bz2` para Linux 64 bits.

2. **Descomprimir e substituir**  
   Extraia o arquivo e substitua a pasta `/lib/firefox-esr`:  
   ```sh
   cd ~/Downloads
   tar -xjf firefox-*.tar.bz2
   sudo rm -rf /lib/firefox-esr
   sudo mv firefox /lib/firefox-esr
   ```

3. **Criar atalho**  
   Crie um link simbólico `firefox-esr` para `firefox`:  
   ```sh
   cd /lib/firefox-esr
   sudo ln -s firefox firefox-esr
   sudo ln -s /usr/share/firefox-esr/distribution distribution
   ```

4. **Verificar**  
   Teste a instalação:  
   ```sh
   firefox-esr --version
   ```

### 4. Icon Theme

Baixar e configurar **Vivid-Dark-Icons**, a partir do link de Download no repositório [Vivid-Plasma-Themes](https://github.com/L4ki/Vivid-Plasma-Themes)

1. **Criar pasta de ícones**:
   ```bash
   mkdir -p ~/.icons
   ```

2. **Baixar arquivos**:
   - Acesse [https://www.pling.com/p/2110196](https://www.pling.com/p/2110196) e baixe os arquivos: 
      - [Vivid-Dark-Icons](https://www.pling.com/p/2110189)
      - [Vivid Wallpaper](https://www.pling.com/p/2110165)

3. **Descomprimir arquivos**:
   ```bash
   tar -xzvf ~/Downloads/Vivid-Dark-Icons.tar.gz -C ~/.icons
   ```

4. **Configurar ícones**:
   - Abra **GNOME Tweaks** (`gnome-tweaks`).
   - Em **Aparência** > **Ícones**, selecione **Vivid-Dark-Icons**.

5. **Configurar plano de fundo**:
   - Abra **Configurações** (`gnome-control-center`).
   - Em **Aparência** > **Plano de Fundo**, clique em **Adicionar Imagem**.
   - Escolha uma imagem da pasta `Vivid Wallpapers` e aplique.