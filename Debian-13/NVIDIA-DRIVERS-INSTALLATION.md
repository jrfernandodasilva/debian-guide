Para instalar os drivers proprietários da NVIDIA no Debian 13 "Trixie" com Secure Boot ativado, siga os passos abaixo com base nas informações fornecidas. Esses passos garantem que os drivers sejam instalados corretamente e que os módulos DKMS sejam assinados com uma chave MOK (Machine Owner Key) para compatibilidade com o Secure Boot.

---

### **Passos para instalar os drivers proprietários da NVIDIA no Debian 13 "Trixie"**

#### **1. Verifique os requisitos**
- **GPU compatível**: Certifique-se de que sua GPU é suportada pela versão 550.163.01 do driver NVIDIA (GeForce 700 series ou mais recente). Caso contrário, considere a versão 535.216.03 ou o driver open-source **nouveau**.
- **Secure Boot**: Se o Secure Boot estiver ativado, você precisará configurar uma chave MOK para assinar os módulos DKMS.
- **Kernel headers**: Você precisará dos headers do kernel correspondente à sua versão atual do kernel.

#### **2. Adicione os componentes necessários ao sources.list**
Edite ou crie o arquivo `/etc/apt/sources.list.d/debian.sources` para incluir os componentes `contrib`, `non-free` e `non-free-firmware`. Um exemplo de configuração:

```bash
Types: deb
URIs: http://deb.debian.org/debian/
Suites: trixie
Components: main contrib non-free non-free-firmware
Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg
```

Salve o arquivo e atualize a lista de pacotes:

```bash
sudo apt update
```

#### **3. Instale os headers do kernel**
Certifique-se de que os headers do kernel correspondente à sua versão atual estejam instalados, pois eles são necessários para o DKMS compilar os módulos do driver NVIDIA:

```bash
sudo apt install linux-headers-$(uname -r)
```

#### **4. Configure o Secure Boot (se ativado)**
Se o Secure Boot estiver ativado, você precisa criar e inscrever uma chave MOK para assinar os módulos DKMS. Siga estas etapas:

1. **Verifique se a chave MOK já existe**:
   ```bash
   ls /var/lib/dkms/
   ```
   Se os arquivos `mok.key` e `mok.pub` existirem, você pode pular para a inscrição da chave. Caso contrário, gere uma nova chave:

2. **Gere uma chave MOK (opcional)**:
   ```bash
   sudo dkms generate_mok
   ```
   **Nota**: Se você receber um erro como `Error! Unknown action specified: ""`, ignore este comando. O DKMS gerará automaticamente a chave na primeira compilação do módulo.

3. **Inscreva a chave MOK**:
   ```bash
   sudo mokutil --import /var/lib/dkms/mok.pub
   ```
   - Você será solicitado a criar uma senha temporária (de 8 a 16 caracteres).
   - Verifique se a chave está pronta para inscrição:
     ```bash
     sudo mokutil --list-new
     ```

4. **Reinicie o sistema**:
   Durante a reinicialização, o sistema entrará no utilitário **MOK Manager EFI**:
   - Selecione "Enroll MOK".
   - Confirme a inscrição da chave, insira a senha temporária criada e reinicie novamente.

5. **Verifique se a chave foi inscrita**:
   Após a reinicialização, confirme que a chave MOK foi carregada:
   ```bash
   sudo dmesg | grep cert
   ```

   Opcionalmente, verifique com:
   ```bash
   sudo mokutil --test-key /var/lib/dkms/mok.pub
   ```
   A saída deve indicar que a chave está inscrita (`is already enrolled`).

#### **5. Instale os pacotes do driver NVIDIA**
Para instalar a versão **proprietária** do driver NVIDIA:

```bash
sudo apt install nvidia-kernel-dkms nvidia-driver firmware-misc-nonfree
```

- **`nvidia-kernel-dkms`**: Compila e assina os módulos do kernel NVIDIA.
- **`nvidia-driver`**: Inclui os utilitários e bibliotecas do driver NVIDIA.
- **`firmware-misc-nonfree`**: Fornece firmwares adicionais necessários para algumas GPUs NVIDIA.

**Alternativa (driver open-source)**:
Se preferir o driver open-source da NVIDIA, substitua `nvidia-kernel-dkms` por `nvidia-open-kernel-dkms`:

```bash
sudo apt install nvidia-open-kernel-dkms nvidia-driver firmware-misc-nonfree
```

#### **6. Configure o Dracut (se necessário)**
Se o seu sistema usa **dracut** como gerador de initramfs, crie um arquivo de configuração para incluir os arquivos de configuração do modprobe:

1. Crie o arquivo `/etc/dracut.conf.d/10-nvidia.conf` com o seguinte conteúdo:
   ```bash
   install_items+=" /etc/modprobe.d/nvidia-blacklists-nouveau.conf /etc/modprobe.d/nvidia.conf /etc/modprobe.d/nvidia-options.conf "
   ```

2. Certifique-se de que há espaços entre as aspas e os caracteres, conforme indicado.

#### **7. Verifique a instalação do driver**
Antes de reiniciar, confirme se o módulo NVIDIA foi compilado corretamente:

```bash
dkms status
```

A saída deve mostrar algo como:

```
nvidia-current, 550.163.01, <sua-versão-do-kernel>, x86_64: installed
```

#### **8. Configurações adicionais para Wayland (opcional)**
Se você usa Wayland (padrão no GNOME e KDE Plasma desde o Debian 12 "Bookworm"), siga estas etapas para garantir suporte adequado:

1. **Habilite a preservação de memória de vídeo**:
   Para evitar problemas gráficos, adicione a seguinte opção ao arquivo `/etc/modprobe.d/nvidia-options.conf`:
   ```bash
   echo "options nvidia NVreg_PreserveVideoMemoryAllocations=1" | sudo tee -a /etc/modprobe.d/nvidia-options.conf
   ```
   **Aviso**: Não faça isso se seu sistema usa gráficos híbridos (Optimus).

2. **Para suspensão/hibernação no KDE**:
   Para evitar problemas gráficos após suspensão ou hibernação no KDE, a opção acima também ajuda.

#### **9. Reinicie o sistema**
Reinicie para carregar o driver NVIDIA:

```bash
sudo reboot
```

#### **10. Verifique se o driver está funcionando**
Após a reinicialização, confirme que o driver NVIDIA está em uso:

```bash
nvidia-smi
```

A saída deve mostrar informações sobre a GPU NVIDIA e os processos que a estão utilizando.

#### **11. Solução de problemas (se necessário)**
Se você encontrar problemas, como falhas na conexão de monitores externos, recompile os módulos DKMS:

```bash
sudo dpkg-reconfigure nvidia-kernel-dkms
```

Depois, reinicie o sistema novamente.

---

### **Notas importantes**
- **Secure Boot**: Se você pular a etapa de inscrição da chave MOK e o Secure Boot estiver ativado, os módulos NVIDIA não serão carregados, resultando em falhas gráficas.
- **Conflitos com o nouveau**: O driver `nvidia-driver` automaticamente desativa o driver open-source `nouveau` ao adicionar configurações de blacklist em `/etc/modprobe.d/`.
- **Wayland**: Certifique-se de que o Wayland está configurado corretamente se você usa ambientes como GNOME ou KDE Plasma. Caso o GNOME inicie com X11, a opção `NVreg_PreserveVideoMemoryAllocations=1` pode habilitar o suporte ao Wayland.
- **VirtualBox**: Se você usa pacotes como o VirtualBox, siga as instruções para criar chaves MOK em `/var/lib/shim-signed/mok/` para compatibilidade, conforme descrito no documento.

---

Debian doc: https://wiki.debian.org/NvidiaGraphicsDrivers#Debian_13_.22Trixie.22
