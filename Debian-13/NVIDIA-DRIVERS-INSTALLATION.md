# Installing NVIDIA Proprietary Drivers on Debian 13 "Trixie" with Secure Boot

This guide provides step-by-step instructions to install NVIDIA proprietary drivers (version 550.163.01) on Debian 13 "Trixie" with Secure Boot enabled. It includes commands for verification and Wayland adjustments.

---

## Prerequisites
- **GPU Compatibility**: Ensure your GPU is supported by NVIDIA driver version 550.163.01 (GeForce 700 series or newer). For older GPUs, use driver version 535.216.03 or the open-source `nouveau` driver.
- **Secure Boot**: If enabled, a Machine Owner Key (MOK) is required to sign DKMS modules.
- **Kernel Headers**: Required for DKMS to compile NVIDIA modules.

---

## Installation Steps

### 1. Update the APT Sources
Add `contrib`, `non-free`, and `non-free-firmware` components to `/etc/apt/sources.list.d/debian.sources`:

```bash
sudo nano /etc/apt/sources.list.d/debian.sources
```

Add the following content:

```
Types: deb
URIs: http://deb.debian.org/debian/
Suites: trixie
Components: main contrib non-free non-free-firmware
Signed-By: /usr/share/keyrings/debian-archive-keyring.gpg
```

Save and update the package list:

```bash
sudo apt update
```

### 2. Install Kernel Headers
Install the kernel headers for your current kernel version:

```bash
sudo apt install linux-headers-$(uname -r)
```

### 3. Configure Secure Boot (if enabled)
If Secure Boot is enabled, enroll a Machine Owner Key (MOK) to sign DKMS modules.

1. **Install DKMS**:
   ```bash
   sudo apt install dkms
   ```

2. **Generate a MOK (if not already present)**:
   ```bash
   sudo dkms generate_mok
   ```
   If the above command fails with `Error! Unknown action specified`, skip it; DKMS will generate the key during module compilation.

3. **Import the MOK**:
   ```bash
   sudo mokutil --import /var/lib/dkms/mok.pub
   ```
   - Enter a one-time password (8–16 characters) when prompted.

4. **Verify the key is queued for enrollment**:
   ```bash
   sudo mokutil --list-new
   ```

5. **Reboot and enroll the MOK**:
   ```bash
   sudo reboot
   ```
   - During reboot, the MOK Manager EFI utility will appear.
   - Select **Enroll MOK**, confirm, enter the password, and reboot again.

6. **Verify MOK enrollment**:
   ```bash
   sudo dmesg | grep cert
   ```
   Optionally, check with:
   ```bash
   sudo mokutil --test-key /var/lib/dkms/mok.pub
   ```
   The output should confirm the key is enrolled (`is already enrolled`).

### 4. Install NVIDIA Drivers
Install the proprietary NVIDIA driver and related packages:

```bash
sudo apt install nvidia-kernel-dkms nvidia-driver firmware-misc-nonfree
```

- **`nvidia-kernel-dkms`**: Builds and signs the NVIDIA kernel module.
- **`nvidia-driver`**: Provides NVIDIA utilities and libraries.
- **`firmware-misc-nonfree`**: Includes additional firmware for some NVIDIA GPUs.

**Alternative (open-source NVIDIA driver)**:
```bash
sudo apt install nvidia-open-kernel-dkms nvidia-driver firmware-misc-nonfree
```

### 5. Verify Driver Compilation
Check if the NVIDIA module was compiled successfully:

```bash
sudo dkms status
```

Expected output:
```
nvidia-current, 550.163.01, <your-kernel-version>, x86_64: installed
```

### 6. Configure Wayland (Optional)
To ensure Wayland works correctly (default in GNOME/KDE Plasma):

1. **Enable kernel modesetting**:
   ```bash
   echo 'GRUB_CMDLINE_LINUX="$GRUB_CMDLINE_LINUX nvidia-drm.modeset=1 nvidia-drm.fbdev=1"' | sudo tee /etc/default/grub.d/nvidia-modeset.cfg
   sudo update-grub
   ```

2. **Check `PreserveVideoMemoryAllocations`**:
   ```bash
   cat /proc/driver/nvidia/params | grep PreserveVideoMemoryAllocations
   ```
   If the output shows `PreserveVideoMemoryAllocations: 0`, enable it:
   ```bash
   echo 'options nvidia NVreg_PreserveVideoMemoryAllocations=1' | sudo tee /etc/modprobe.d/nvidia-power-management.conf
   ```
   **Note**: Skip this for hybrid graphics (Optimus) systems.

### 7. Reboot the System
Reboot to load the NVIDIA driver:

```bash
sudo reboot
```

### 8. Verify Driver Functionality
Confirm the NVIDIA driver is active:

```bash
nvidia-smi
```

The output should display GPU information and running processes.

### 9. Troubleshoot Issues
If issues occur (e.g., external monitor not detected), recompile the DKMS module:

```bash
sudo dpkg-reconfigure nvidia-kernel-dkms
sudo reboot
```

---

## Notes
- **Secure Boot**: Skipping MOK enrollment with Secure Boot enabled will prevent NVIDIA modules from loading, causing graphical issues.
- **Nouveau Conflict**: The `nvidia-driver` package automatically blacklists the `nouveau` driver via `/etc/modprobe.d/`.
- **Wayland**: The above configurations ensure Wayland compatibility. Without `NVreg_PreserveVideoMemoryAllocations=1`, GNOME may fall back to X11.
- **VirtualBox**: For VirtualBox compatibility, create MOK keys in `/var/lib/shim-signed/mok/` as needed.

---

**Source**: [Debian Wiki - NvidiaGraphicsDrivers](https://wiki.debian.org/NvidiaGraphicsDrivers#Debian_13_.22Trixie.22)