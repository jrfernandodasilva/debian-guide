# Instalação dos Drivers Proprietários NVIDIA no Debian 13 "Trixie"

Este guia descreve os passos para instalar os drivers proprietários mais recentes da NVIDIA no Debian 13 "Trixie". Certifique-se de verificar a versão mais recente do driver para sua placa de vídeo NVIDIA em: [https://www.nvidia.com/pt-br/drivers](https://www.nvidia.com/pt-br/drivers).

## Passos

1. **Acessar o terminal TTY**  
   Abra o terminal TTY com o comando:  
   ```
   Ctrl + Alt + F4
   ```  
   Faça login com seu usuário e alterne para o modo root:  
   ```sh
   sudo -i
   ```

2. **Instalar o extrepo**  
   Instale o pacote `extrepo` para gerenciar repositórios externos:  
   ```sh
   apt install extrepo
   ```

3. **Atualizar a lista de repositórios**  
   Atualize a lista de repositórios disponíveis:  
   ```sh
   extrepo update
   ```

4. **Pesquisar repositórios NVIDIA**  
   Busque repositórios relacionados à NVIDIA:  
   ```sh
   extrepo search nvidia | less
   ```

5. **Habilitar o repositório nvidia-cuda**  
   Ative o repositório `nvidia-cuda`:  
   ```sh
   extrepo enable nvidia-cuda
   ```

6. **Atualizar pacotes disponíveis**  
   Atualize a lista de pacotes:  
   ```sh
   apt update
   ```

7. **Instalar pacotes necessários**  
   Instale os pacotes essenciais para compilação e suporte aos drivers:  
   ```sh
   apt install build-essential linux-headers-amd64 dkms
   ```

8. **Instalar os drivers NVIDIA**  
   Instale o pacote do driver NVIDIA (exemplo: versão 580):  
   ```sh
   apt install nvidia-open-580
   ```

9. **Instalar o utilitário MOK e configurar a chave**  
   Instale o `mokutil` e importe a chave MOK para o Secure Boot:  
   ```sh
   apt install mokutil
   mokutil -i /var/lib/dkms/mok.pub
   ```  
   Digite a senha solicitada e confirme.

10. **Reiniciar o sistema**  
    Reinicie o sistema para prosseguir com a configuração do MOK:  
    ```sh
    reboot
    ```

11. **Configurar o MOK na inicialização**  
    Após reiniciar, siga os passos no menu de inicialização:  
    1. Pressione qualquer tecla para acessar o menu.  
    2. Selecione **Enroll MOK**.  
    3. Escolha **Continue**.  
    4. Selecione **Yes**.  
    5. Digite a senha definida anteriormente.  
    6. Selecione **Reboot** para reiniciar novamente.

12. **Verificar o Secure Boot na BIOS**  
    Acesse a BIOS/UEFI do sistema e verifique se o **Secure Boot** está habilitado. Caso não esteja, ative-o e salve as alterações.

## Configuração do Wayland em Ambientes GNOME

Em ambientes GNOME, mesmo com a versão correta do driver NVIDIA instalada, o gerenciador de login (GDM3) pode não oferecer a opção de iniciar uma sessão Wayland. Isso pode ocorrer devido à falta de ativação do kernel modesetting ou porque os scripts de suporte para suspensão/hibernação/retomada não estão instalados ou habilitados.

### Habilitar Kernel Modesetting
Para habilitar o kernel modesetting com o driver NVIDIA, execute os seguintes comandos:  
```sh
echo 'GRUB_CMDLINE_LINUX="$GRUB_CMDLINE_LINUX nvidia-drm.modeset=1 nvidia-drm.fbdev=1"' > /etc/default/grub.d/nvidia-modeset.cfg
update-grub
```

### Instalar e Habilitar Scripts de Suspensão
Para instalar os scripts de suporte à suspensão/hibernação, instale o pacote `nvidia-suspend-common` e habilite os serviços correspondentes:  
```sh
apt install nvidia-suspend-common
systemctl enable nvidia-suspend.service
systemctl enable nvidia-hibernate.service
systemctl enable nvidia-resume.service
```

### Verificar o Parâmetro PreserveVideoMemoryAllocations
Verifique se o parâmetro `PreserveVideoMemoryAllocations` está ativado, pois sem ele, as regras do udev em `/usr/lib/udev/rules.d/61-gdm.rules` forçarão o uso do X11 em vez do Wayland:  
```sh
cat /proc/driver/nvidia/params | grep PreserveVideoMemoryAllocations
```  
A saída deve ser:  
```
PreserveVideoMemoryAllocations: 1
```  
Se o valor for `0`, configure o parâmetro criando um arquivo de configuração no `modprobe.d`:  
```sh
echo 'options nvidia NVreg_PreserveVideoMemoryAllocations=1' > /etc/modprobe.d/nvidia-power-management.conf
```

## Configuração do NVIDIA como Padrão para Aplicativos Gráficos

Para garantir que todos os aplicativos gráficos usem a GPU NVIDIA por padrão, adicione as seguintes linhas ao arquivo `~/.bashrc`:

```sh
echo -e "\n\n# Set NVIDIA as default for all graphic apps\nexport __GLX_VENDOR_LIBRARY_NAME=\"nvidia\"\nexport __EGL_VENDOR_LIBRARY_NAME=\"nvidia_icd\"" >> ~/.bashrc
```

Após adicionar essas linhas, recarregue o arquivo `~/.bashrc` para aplicar as alterações:

```sh
source ~/.bashrc
```

## Para forçar a NVIDIA como GPU primária
```sh
sudo nano /etc/X11/xorg.conf.d/nvidia_as_primary.conf
```

Adicione:
```sh
Section "OutputClass"
    Identifier "nvidia"
    MatchDriver "nvidia-drm"
    Driver "nvidia"
    Option "PrimaryGPU" "yes"
EndSection
```

Reinicie:
```
sudo reboot
```

## Verificação da Configuração

Para confirmar que os drivers NVIDIA estão configurados e funcionando corretamente, execute os seguintes comandos:

1. **Verificar o renderizador OpenGL**  
   Este comando exibe informações sobre o renderizador OpenGL em uso, confirmando se a GPU NVIDIA está sendo utilizada:  
   ```sh
   glxinfo | grep "OpenGL renderer"
   ```  
   A saída deve indicar que o renderizador é da NVIDIA, por exemplo:  
   ```
   OpenGL renderer string: NVIDIA GeForce GTX 960/PCIe/SSE2
   ```

2. **Verificar o status da GPU NVIDIA**  
   Este comando exibe informações sobre a GPU NVIDIA, incluindo a versão do driver e os processos que a estão utilizando:  
   ```sh
   nvidia-smi
   ```  
   A saída deve mostrar uma tabela com detalhes da GPU, como:  
   ```
   +-----------------------------------------------------------------------------+
   | NVIDIA-SMI 580.XX.XX    Driver Version: 580.XX.XX    CUDA Version: XX.X     |
   |-------------------------------+----------------------+----------------------+
   | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
   | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
   |===============================+======================+======================|
   |   0  GeForce GTX 960    Off  | 00000000:07:00.0 Off |                  N/A |
   | 23%   45C    P0    30W / 120W |      0MiB /  4096MiB |      0%      Default |
   +-------------------------------+----------------------+----------------------+
   ```

## Conclusão  
Os drivers NVIDIA foram instalados e configurados com sucesso, e o suporte ao Wayland foi habilitado no ambiente GNOME. Os comandos de verificação acima ajudam a garantir que a GPU está funcionando como esperado.

## Fontes  
- [Vídeo no YouTube](https://www.youtube.com/watch?v=FaDENzwkzys)  
- [Documentação oficial do Debian](https://wiki.debian.org/NvidiaGraphicsDrivers#Debian_13_.22Trixie.22)  
- [Configuração do Wayland no Debian](https://wiki.debian.org/NvidiaGraphicsDrivers#wayland-modesetting)