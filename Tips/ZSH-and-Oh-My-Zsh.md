## ZSH and Oh My Zsh

This section covers the installation and customization of ZSH, a powerful shell that serves as a great alternative to Bash, along with Oh My Zsh for plugin and theme management.

### Install ZSH

Install ZSH using the package manager.
```sh
sudo apt install zsh -y
```

### Configure ZSH Autocomplete

Initialize the completion system for ZSH.

```sh
zsh
# Inside ZSH:
    autoload -Uz compinit
compinit
_comp_options+=(globdots)
```

### Set ZSH as Default Shell

Change your system's default shell to ZSH.

```sh
chsh -s $(which zsh)
```
> **Note:** To rollback to Bash, use `chsh -s /bin/bash`. You must restart your terminal for changes to take effect.

### Install Oh My Zsh

Install the Oh My Zsh framework to manage your ZSH configuration.

```sh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

### Install Oh My Zsh Plugins

Plugins add extra functionality to your shell. Clone these popular plugins into your custom directory.

zsh-syntax-highlighting
```sh
git clone https://github.com/zsh-users/zsh-syntax-highlighting ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

zsh-completions
```sh
git clone https://github.com/zsh-users/zsh-completions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-completions
```

zsh-autosuggestions
```sh
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

fzf (Fuzzy Finder)
```sh
git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf && ~/.fzf/install
```

### Enable Plugins

Edit your `.zshrc` file to activate the installed plugins.

```sh
nano ~/.zshrc
```

Update the `plugins` line:
```sh
plugins=(
  git
  zsh-syntax-highlighting
  zsh-autosuggestions
  fzf
)
```

### Install Powerlevel10k Theme

Powerlevel10k is a fast and highly flexible theme for ZSH.

```sh
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/themes/powerlevel10k
```

To enable it, edit `~/.zshrc` and change the theme:
```sh
ZSH_THEME="powerlevel10k/powerlevel10k"

# Powerlevel9k compatibility mode
POWERLEVEL9K_MODE='awesome-fontconfig'
```

Apply the changes:

```sh
source ~/.zshrc
```

## Starship Prompt

Starship is a minimal, blazing-fast, and infinitely customizable prompt for any shell. It is a great alternative if you prefer a cross-shell experience.

### Install Starship

Download and install the latest Starship binary.

```sh
curl -sS https://starship.rs/install.sh | sh
```

### Configure ZSH to use Starship

Add the initialization script to the end of your `~/.zshrc` file.

```sh
echo 'eval "$(starship init zsh)"' >> ~/.zshrc
```

### Customizing Starship

Create the configuration file to start customizing your prompt.

```sh
mkdir -p ~/.config && touch ~/.config/starship.toml
```

### Useful Resources

- [Oh My Zsh Plugins Wiki](https://github.com/ohmyzsh/ohmyzsh/wiki/Plugins)
- [Powerlevel10k Repository](https://github.com/romkatv/powerlevel10k)
- [Starship Official Website](https://starship.rs)
- [Example .zshrc Config](https://github.com/jrfernandodasilva/debian-guide/blob/main/Debian-12/.zshrc)