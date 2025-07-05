# Lab 5 — VirtualBox & Ubuntu Guest

## Task 1 · VM Deployment

### VirtualBox version

- **7.1.10 r169112**

### VM configuration

| Setting  | Value                              |
| -------- | ---------------------------------- |
| Name     | `Ubuntu‑Lab5`                      |
| ISO      | `ubuntu‑24.04.2‑desktop‑amd64.iso` |
| RAM      | **3 GB**                           |
| vCPU     | **2 cores**                        |
| Disk     | 20 GB VDI (dynamically allocated)  |
| Graphics | VBoxSVGA, 128 MB VRAM, 3D off      |
| Network  | NAT                                |

### Deployment steps

1. Created VM via *New → Unattended Install*; user **vboxuser**, password set during wizard.
2. Attached the Ubuntu 24.04.2 ISO and started the VM.
3. Automatic installer completed, system booted to GNOME desktop.
4. Guest Additions offered but not required for the lab.



---

## Task 2 · System Information (inside guest)

### 2.1 CPU / RAM / Network

```bash
$ lscpu | head -n 10
Architecture:            x86_64
CPU op‑mode(s):          32‑bit, 64‑bit
Address sizes:           39 bits physical, 48 bits virtual
Byte Order:              Little Endian
CPU(s):                  2
On‑line CPU(s) list:     0,1
Vendor ID:               GenuineIntel
Model name:              13th Gen Intel(R) Core(TM) i5‑13450HX
CPU family:              6
Model:                   183
```

```bash
$ free -h
               total   used   free  shared  buff/cache  available
Mem:           3.8Gi  1.0Gi  2.2Gi    32Mi       863Mi       2.8Gi
Swap:             0B     0B     0B
```

```bash
$ ip -4 addr | grep inet
inet 127.0.0.1/8 scope host lo
inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute enp0s3
```

### 2.2 OS specifications (`neofetch`)

```bash
OS: Ubuntu 24.04.2 LTS x86_64
Host: VirtualBox 1.2
Kernel: 6.11.0‑29‑generic
Uptime: 3 mins
Packages: 1528 (dpkg), 10 (snap)
Shell: bash 5.2.21
Resolution: 1280x800
DE: GNOME 46.0   WM: Mutter
Theme: Yaru  Icons: Yaru
Terminal: gnome‑terminal
CPU: 13th Gen Intel i5‑13450HX (2) @ Virtual 2.4 GHz
GPU: VMware SVGA II Adapter
Memory: 798 MiB / 3915 MiB
```

---

**Submission prepared by:** Zhalil

