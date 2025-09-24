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

## 3. Install Timeshift for System Backups

Timeshift creates system snapshots for easy restoration on Debian 13 "Trixie".

### Installation
Install Timeshift:

```bash
sudo apt install timeshift
```

### Configuration
1. Launch Timeshift:
   ```bash
   timeshift-gtk
   ```
2. Follow the wizard to select:
   - Snapshot type: **RSYNC** (recommended).
   - Backup location: External drive or separate partition.
   - Schedule: Daily, weekly, or manual snapshots.

### Creating a Snapshot
Create a manual snapshot:

```bash
sudo timeshift --create --comments "Pre-update backup"
```

### Restoring a Snapshot
Restore a snapshot via GUI or CLI:
- **GUI**: Run `timeshift-gtk`, select a snapshot, and restore.
- **CLI**:
   ```bash
   sudo timeshift --restore
   ```
   Follow prompts to select the snapshot.

### Verification
List snapshots to verify:

```bash
sudo timeshift --list
```

### Notes
- Store snapshots on an external drive for safety.
- Exclude user data for faster system backups.

## 4. Install NVIDIA driver
See [NVIDIA-DRIVER-INSTALLATION.md](https://github.com/jrfernandodasilva/debian-guide/blob/main/Debian-13/NVIDIA-DRIVERS-INSTALLATION.md) document instructions.

## 5. Atualizar Firefox ESR

1. **Baixar o Firefox ESR**  
   Acesse [https://www.firefox.com/pt-BR/download/all/desktop-esr/](https://www.firefox.com/pt-BR/download/all/desktop-esr/), selecione "Português (Brasil)" e baixe o arquivo `.tar.bz2` para Linux 64 bits.

2. **Descomprimir e substituir**  
   Extraia o arquivo e substitua a pasta `/lib/firefox-esr`:  
   ```sh
   cd ~/Downloads
   tar -xJf firefox-*.tar.xz
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

## 6. Icon Theme and Background Customization

Customize your GNOME desktop with the **Vivid-Dark-Icons** theme and wallpapers.

### Install Vivid-Dark-Icons
1. **Create icons directory**:
   ```bash
   mkdir -p ~/.icons
   ```

2. **Download files**:
   - Visit [https://www.pling.com/p/2110196](https://www.pling.com/p/2110196) and download:
     - [Vivid-Dark-Icons](https://www.pling.com/p/2110189)
     - [Vivid Wallpaper](https://www.pling.com/p/2110165)

3. **Extract files**:
   ```bash
   tar -xzvf ~/Downloads/Vivid-Dark-Icons.tar.gz -C ~/.icons
   ```

4. **Update icon cache**:
   ```bash
   gtk-update-icon-cache -f -t ~/.icons/Vivid-Dark-Icons/
   ```

5. **Set icon theme**:
   - Open **GNOME Tweaks** (`gnome-tweaks`).
   - In **Appearance** > **Icons**, select **Vivid-Dark-Icons**.

### Customize Application Icons
Customize icons for specific GNOME applications:

- **Nautilus**:
   ```bash
   cp /usr/share/applications/org.gnome.Nautilus.desktop ~/.local/share/applications/
   sed -i 's|Icon=org.gnome.Nautilus|Icon=user-home.svg|g' ~/.local/share/applications/org.gnome.Nautilus.desktop
   ```

- **Firefox ESR**:
   ```bash
   cp /usr/share/applications/firefox-esr.desktop ~/.local/share/applications/
   sed -i 's|Icon=firefox-esr|Icon=firefox|g' ~/.local/share/applications/firefox-esr.desktop
   ```

- **GNOME Terminal**:
   ```bash
   cp /usr/share/applications/org.gnome.Terminal.desktop ~/.local/share/applications/
   sed -i 's|Icon=org.gnome.Terminal|Icon=Terminal.svg|g' ~/.local/share/applications/org.gnome.Terminal.desktop
   ```

- **Evolution Mail**:
   ```bash
   cp /usr/share/applications/org.gnome.Evolution.desktop ~/.local/share/applications/
   sed -i 's|Icon=evolution|Icon=kube-mail.svg|g' ~/.local/share/applications/org.gnome.Evolution.desktop
   ```

- **GNOME Software**:
   ```bash
   cp /usr/share/applications/org.gnome.Software.desktop ~/.local/share/applications/
   sed -i 's|Icon=org.gnome.Software|Icon=yast-software-group.svg|g' ~/.local/share/applications/org.gnome.Software.desktop
   ```

### Create and Customize ~/Apps Folder
1. **Create the folder**:
   ```bash
   mkdir -p ~/Apps
   ```

2. **Set custom icon**:
   ```bash
   gio set ~/Apps -t string metadata::custom-icon file://$HOME/.icons/Vivid-Dark-Icons/places/32/folder-script.svg
   ```

### Download GNOME Backgrounds
1. **Download additional backgrounds**:
   - Visit [https://zebreus.github.io/all-gnome-backgrounds](https://zebreus.github.io/all-gnome-backgrounds).
   - Download desired wallpapers (e.g., Amber or others).

2. **Set GNOME background**:
   - Open **Settings** (`gnome-control-center`).
   - In **Appearance** > **Background**, click **Add Picture**.
   - Select a downloaded GNOME background image and apply.

## 7. Git
See [Git Settings and Essential Commands](https://github.com/jrfernandodasilva/debian-guide/blob/main/Tips/Git.md) document instructions.
