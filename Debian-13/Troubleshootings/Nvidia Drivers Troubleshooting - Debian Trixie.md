## Troubleshooting

### Screen Freezing After Driver Installation (Optimus / Hybrid Graphics)

**Applies to**: Systems with Intel + NVIDIA hybrid graphics (e.g., Dell G15 with RTX 3050)

#### Symptoms
- System freezes after some time of use following NVIDIA driver installation
- `nvidia-smi` may work, but the desktop becomes unresponsive
- Kernel logs show `BUG: unable to handle page fault` and RCU stall errors

#### Root Cause
On kernels 6.12+, the `xe` driver (Intel's new experimental GPU driver) loads alongside `i915` and `nvidia`, causing a three-way conflict that results in kernel page faults and system freezes.

Verify with:
```bash
lsmod | grep -E 'nvidia|xe|i915'
sudo journalctl -b -1 -p err | grep -E 'BUG|page fault|rcu'
```

If `xe` appears in `lsmod` output alongside `nvidia` and `i915`, proceed with the fix below.

#### Fix: Blacklist the `xe` Driver

```bash
echo 'blacklist xe' | sudo tee /etc/modprobe.d/blacklist-xe.conf
sudo update-initramfs -u
sudo reboot
```

#### Verification

After reboot, confirm `xe` is no longer loaded and the NVIDIA driver is functional:

```bash
lsmod | grep -E 'nvidia|xe|i915'
nvidia-smi
sudo journalctl -b 0 -p err
```

Expected: `xe` absent from `lsmod`, `nvidia-smi` displaying GPU info, and no page fault errors in the journal.

---

### NVIDIA Module Not Loading After Kernel Update

#### Symptoms
- `nvidia-smi` fails with *couldn't communicate with the NVIDIA driver*
- `dkms status` shows the module installed for a **different kernel version** than the one running

#### Root Cause
DKMS compiles the NVIDIA module for the kernel version present at installation time. After a kernel update, the module must be recompiled for the new version.

Verify the mismatch:
```bash
uname -r                  # currently running kernel
sudo dkms status          # kernel version the module was compiled for
```

#### Fix

```bash
sudo apt install linux-headers-$(uname -r)
sudo dpkg-reconfigure nvidia-kernel-dkms
sudo reboot
```

> **Note**: Always install kernel headers using `$(uname -r)` rather than the `linux-headers-amd64` metapackage. The metapackage targets the latest available version in the repository, which may differ from the running kernel.

---

### Persistent Kernel Crashes with Proprietary NVIDIA Driver (Dell G15 / Optimus)

**Applies to**: Dell G15 5530 (RTX 3050) and similar Optimus laptops running the proprietary `nvidia-kernel-dkms` driver on Debian 13

#### Symptoms
- System freezes intermittently — at the login screen, shortly after login, or after a few minutes of use
- Kernel log shows crash originating from the NVIDIA ACPI handler:
  ```
  rm_acpi_notify+0x123/0x280 [nvidia]
  acpi_ev_notify_dispatch
  ```
- Cascading effects: RCU stalls, CPU soft lockups, USB subsystem failures
- Bluetooth may stop working as a side effect (USB timeout errors)

#### Root Cause
The proprietary NVIDIA kernel module (driver 550) has a bug in its ACPI event handler (`rm_acpi_notify`) that causes a kernel page fault on certain Dell laptops. The crash destabilizes the entire kernel, including the USB subsystem — which explains why Bluetooth (connected via USB internally) also fails.

Confirm the crash signature:
```bash
sudo journalctl -b -1 | grep -A 10 "BUG: unable" | grep -E "rm_acpi_notify|acpi_ev_notify"
```

#### Fix: Switch to the NVIDIA Open Kernel Module

NVIDIA's open-source kernel module (`nvidia-open-kernel-dkms`) rewrites the ACPI handling and resolves this crash. It is fully supported for the RTX 3050 and later GPUs.

```bash
sudo apt install nvidia-open-kernel-dkms
sudo apt remove nvidia-kernel-dkms nvidia-kernel-support
sudo update-initramfs -u
sudo reboot
```

After rebooting, update `/etc/modprobe.d/nvidia.conf` to reference the open module names. Replace occurrences of `nvidia-current` with `nvidia-current-open`:

```bash
sudo sed -i \
  's/modprobe -i nvidia-current /modprobe -i nvidia-current-open /g; s/nvidia-current-modeset/nvidia-current-open-modeset/g; s/nvidia-current-drm/nvidia-current-open-drm/g; s/nvidia-current-uvm/nvidia-current-open-uvm/g; s/nvidia-current-peermem/nvidia-current-open-peermem/g' \
  /etc/modprobe.d/nvidia.conf
sudo update-initramfs -u
sudo reboot
```

#### Verification

```bash
lsmod | grep nvidia   # nvidia module should be ~10MB, not ~60MB
nvidia-smi
sudo journalctl -b 0 | grep -c "rm_acpi_notify"  # should return 0
```

#### Cleanup

Remove the DKMS entry for any old kernel version no longer in use:
```bash
sudo dkms status   # identify outdated entries
sudo dkms remove nvidia-current-open/<version> -k <old-kernel-version>
```