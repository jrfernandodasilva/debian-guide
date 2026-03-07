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