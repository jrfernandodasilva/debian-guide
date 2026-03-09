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

## 7. Install programs
```bash
sudo apt install -y terminator htop xclip snapd chromium flameshot pngquant curl wget tree deluge btop rfkill fastfetch ffmpeg
```

### 7.1 Terminator Themes
See instructions: https://github.com/EliverLara/terminator-themes

### 7.2 Gnome extension manager + extensions
Install Extension Manager:
```bash
sudo apt install gnome-shell-extension-manager
```
Open Extension Manager -> Browser. <br/>
Search and install:
- Blur my Shell
- Clipboard Indicator
- Coverflow Alt-Tab
- GNOME Fuzzy App Search
- Forge
- AppIndicator and KStatusNotifierItem Support
- Media Controls
- Dash to Dock  # similar OSx apps bar
    - Hide the dock in overview # can combine with "Dash to Dock" extension
- Dash to Panel # similar Windows system bar
    - App Icons Taskbar # can combine with "Dash to Panel" extension
    - ArcMenu # can combine with "Dash to Panel" extension
- V-Shell (Vertial Workspaces)
- User Themes
  > Download themes from Gnome Look, like: https://www.gnome-look.org/p/1687249 and follow install instructions <br/>
  > Dracula Theme for GTK full instructions: https://draculatheme.com/gtk
- Search Light 
  > _needs: sudo apt install imagemagick_

> Videos with extensions configs: <br/>
> https://www.youtube.com/watch?v=IedxEbwpjfs <br/>
> https://www.youtube.com/watch?v=AE1-W2bMVEs

## 8. Install VPN Plugin
```bash
sudo apt install openvpn
sudo apt install network-manager-openvpn
```

## 9. Git
See [Git Settings and Essential Commands](https://github.com/jrfernandodasilva/debian-guide/blob/main/Tips/Git.md) document instructions.

## 10. Enable syntax highlighting in nano
```bash
find /usr/share/nano/ -iname "*.nanorc" -exec echo include {} \; >> ~/.nanorc
```

## 11. Complementary softwares
- Docker + Docker Compose - [Install instructions](https://docs.docker.com/engine/install/debian/#set-up-the-repository) - [Post install instructions](https://docs.docker.com/engine/install/linux-postinstall/)
- Lazy Docker - [https://github.com/jesseduffield/lazydocker](https://github.com/jesseduffield/lazydocker)
- VSCode - [Download page](https://code.visualstudio.com/download)
- Discord - [Download page](https://discord.com/download)
- Chrome - [Home to download](https://www.google.com/intl/pt-BR/chrome/)
- MySQL Workbench via snapd - [2 ways to Install Mysql Workbench](https://linux.how2shout.com/2-ways-to-install-mysql-workbench-on-debian-11-bullseye-linux/#5_To_Uninstall_run)
- Pinta via snapd - [Install Pinta on Debian](https://snapcraft.io/install/pinta/debian)
- Postman via snapd - [Install Postman on Debian](https://snapcraft.io/install/postman/debian)
- kubectl - [Install and Set Up kubectl on Linux](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux)
- Spotify - [Spotify for Linux](https://www.spotify.com/de-en/download/linux)
- ZSH, Oh My ZSH and Starship - [Install Guide](https://github.com/jrfernandodasilva/debian-guide/blob/main/Tips/ZSH-and-Oh-My-Zsh.md)
- Gemini CLI - [Docs - Installation](https://github.com/google-gemini/gemini-cli)
- Claude Code - [Docs - Native Install](https://code.claude.com/docs/en/overview)
- `uv`for run local serena mcp - [Installing uv](https://docs.astral.sh/uv/getting-started/installation)

### 11.1 Install bash-completion
```bash
sudo apt install bash-completion
```

#### Permanently enable
##### To bash:
```bash
echo "source <(kubectl completion bash)" >> ~/.bashrc
source ~/.bashrc
```

##### To Zsh:
```bash
echo "source <(kubectl completion zsh)" >> ~/.bashrc
source ~/.zshrc
```

## 12. Configure snap apps to show in xorg (If Necessary)
```bash
sudo ln -s /etc/profile.d/apps-bin-path.sh /etc/X11/Xsession.d/99snap
sudo nano /etc/login.defs

# Paste the following at the end of the file and save:
ENV_PATH PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
```

## 13. Install nvm

See repository instructions to install: [https://github.com/nvm-sh/nvm](https://github.com/nvm-sh/nvm)

## 14. Chrome Flags
Access chrome flags with this url [chrome://flags/](chrome://flags/) and enable this:
- GPU rasterization
- Overscroll history navigation
- Preferred Ozone platform (set `Wayland`)
![image](https://github.com/jrfernandodasilva/debian-guide/assets/27747005/077bdec7-3bc2-41cd-96d5-df7f6a06df30)

Run this commant to force enable `Overscroll history navigation`:
```bash
sudo sed -ie 's/Exec=\/usr\/bin\/google-chrome-stable %U/Exec=\/usr\/bin\/google-chrome-stable %U --enable-features=TouchpadOverscrollHistoryNavigation/g' /usr/share/applications/google-chrome.desktop
```
> then restart Chrome browser to enable this config

## 15. Bash Aliases

Follow the instructions in [this repository](https://github.com/jrfernandodasilva/bash-aliases)

## 16. Emby

[See instructions here](https://emby.media/linux-server.html)

After install, run this command to add your system user to emby group:
```sh
sudo usermod -aG emby $USER 
```

## 17. Zram
[See instructions here](https://linuxdicasesuporte.blogspot.com/2022/03/zram-no-debian-gnu-linux-e-derivados.html) or [here](https://fosspost.org/enable-zram-on-linux-better-system-performance/)

## 18. Youtube Download (`yt-dlp`)

Download and install from [official repository](https://github.com/yt-dlp/yt-dlp?tab=readme-ov-file#installation)

The `yt-dlp` is an advanced CLI tool based on youtube-dl that allows you to download videos and audio from various platforms. Here are some useful combinations of options available:

#### Download Videos
1. **Specific quality:**
    - `-f bestvideo+bestaudio/best`: Downloads the best available video and audio, or the best combined format if the former is not available.
    - `-f 720p`: Downloads video in 720p, if available.

2. **Extract audio:**
    - `--extract-audio --audio-format mp3`: Extracts audio in MP3 format.
    - `--audio-quality 0`: Sets the audio quality (0 is the best quality).

3. **Playlist:**
    - `--yes-playlist`: Downloads all videos in the playlist.
    - `--playlist-start 10 --playlist-end 20`: Downloads videos from a specific position.

#### Output Settings
4. **Output Templates**:
    - `-o "%(title)s.%(ext)s"`: Sets the file name to the video title.
    - `-o "~/Downloads/%(uploader)s/%(title)s.%(ext)s"`: Saves files organized in folders based on the channel name.

5. **Prevent Duplicate Downloads**:
    - `--download-archive archive.txt`: Creates an archive to track previously downloaded videos and avoid duplication.

#### Integrations and Control

6. **Cookies and authentication**:
    - `--cookies-from-browser chrome`: Uses browser cookies to access restricted content.
    - `--username USER --password PASS`: Direct authentication.

7. **Video part manipulation (SponsorBlock)**:
    - `--sponsorblock-mark intro`: Marks intros using chapters.
    - `--sponsorblock-remove sponsor`: Removes specific parts as sponsors.

8. **Error resolution**:
    - `--ignore-errors`: Ignores errors and continues downloading.
    - `--retries infinite`: Retries downloading infinitely if there are failures.

These are just a few options. The `yt-dlp` offers a lot of flexibility and is highly configurable. <br/> 
For more details, you can refer to the [official documentation](https://github.com/yt-dlp/yt-dlp) and [related gists​](https://gist.github.com)

> Usage Example
> ```sh
> yt-dlp --audio-quality 0 --extract-audio --audio-format mp3 https://youtu.be/xxxxxx
> ```
