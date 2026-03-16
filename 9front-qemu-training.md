# 9front in QEMU — Comprehensive Agent Training Document

> Aggregated from multiple community sources (2017–2025).  
> Sources: thorjhanson.com, philipbohun.com, samhza.com, offbeatpursuit.com, bsandro.tech, jacobgw.com, mitjafelicijan.com, gist.github.com/99z, gist.github.com/baryluk, fqa.9front.org (FQA 3 / FQA 4 / FQA 7), 9front mailing list (vuxu.org)

---

## Table of Contents

1. [Background and Vocabulary](#1-background-and-vocabulary)
2. [Choosing a 9front Image](#2-choosing-a-9front-image)
3. [Disk Image Creation](#3-disk-image-creation)
4. [QEMU Boot Commands — Installation Phase](#4-qemu-boot-commands--installation-phase)
   - 4.1 [Linux / KVM (amd64)](#41-linux--kvm-amd64)
   - 4.2 [macOS (x86_64 and arm64)](#42-macos-x86_64-and-arm64)
   - 4.3 [Windows with TAP networking](#43-windows-with-tap-networking)
   - 4.4 [Headless / VNC mode](#44-headless--vnc-mode)
5. [Installation Walkthrough](#5-installation-walkthrough)
   - 5.1 [inst/start steps in order](#51-inststart-steps-in-order)
   - 5.2 [Filesystem choice (hjfs vs cwfs vs gefs)](#52-filesystem-choice-hjfs-vs-cwfs-vs-gefs)
   - 5.3 [VGA size and mouseport](#53-vga-size-and-mouseport)
6. [Post-Installation Boot Commands](#6-post-installation-boot-commands)
   - 6.1 [Standard terminal mode](#61-standard-terminal-mode)
   - 6.2 [With port forwarding (drawterm)](#62-with-port-forwarding-drawterm)
   - 6.3 [Multiple disk images](#63-multiple-disk-images)
   - 6.4 [q35 machine type (modern QEMU)](#64-q35-machine-type-modern-qemu)
   - 6.5 [With audio (AC97)](#65-with-audio-ac97)
   - 6.6 [USB tablet for mouse accuracy](#66-usb-tablet-for-mouse-accuracy)
7. [Networking Inside 9front](#7-networking-inside-9front)
   - 7.1 [ip/ipconfig — getting a DHCP address](#71-ipipconfig--getting-a-dhcp-address)
   - 7.2 [Persistent networking on boot (termrc / cpurc)](#72-persistent-networking-on-boot-termrc--cpurc)
   - 7.3 [/lib/ndb/local configuration](#73-libndblocal-configuration)
   - 7.4 [netaudit — verifying the configuration](#74-netaudit--verifying-the-configuration)
8. [CPU and Auth Server Setup](#8-cpu-and-auth-server-setup)
   - 8.1 [Switching service=cpu in plan9.ini](#81-switching-servicecpu-in-plan9ini)
   - 8.2 [The boot prompt after first cpu reboot](#82-the-boot-prompt-after-first-cpu-reboot)
   - 8.3 [bootargs for hjfs and cwfs](#83-bootargs-for-hjfs-and-cwfs)
   - 8.4 [auth/changeuser — setting passwords](#84-authchangeuser--setting-passwords)
   - 8.5 [cpurc and cpurc.local customization](#85-cpurc-and-cpurclocal-customization)
   - 8.6 [Secstore setup](#86-secstore-setup)
   - 8.7 [Adding non-glenda users](#87-adding-non-glenda-users)
9. [Drawterm Connection](#9-drawterm-connection)
   - 9.1 [Which drawterm to use](#91-which-drawterm-to-use)
   - 9.2 [QEMU port forwarding for drawterm](#92-qemu-port-forwarding-for-drawterm)
   - 9.3 [drawterm command syntax](#93-drawterm-command-syntax)
   - 9.4 [Non-privileged port workaround](#94-non-privileged-port-workaround)
   - 9.5 [Minimal rcpu/drawterm without full auth server](#95-minimal-rcpudrawterm-without-full-auth-server)
10. [Automatic Boot (nobootprompt / unattended)](#10-automatic-boot-nobootprompt--unattended)
11. [rio and Profile Setup for Remote Sessions](#11-rio-and-profile-setup-for-remote-sessions)
12. [arm64 QEMU (Apple Silicon / aarch64)](#12-arm64-qemu-apple-silicon--aarch64)
13. [Known Issues and Troubleshooting](#13-known-issues-and-troubleshooting)
    - 13.1 [Mouse problems](#131-mouse-problems)
    - 13.2 [Networking not working after boot](#132-networking-not-working-after-boot)
    - 13.3 [Drawterm timeout / slow login](#133-drawterm-timeout--slow-login)
    - 13.4 [bad nvram key / bad authentication domain](#134-bad-nvram-key--bad-authentication-domain)
    - 13.5 [secstore factotum error on login](#135-secstore-factotum-error-on-login)
    - 13.6 [16-bit color glitches on macOS QEMU](#136-16-bit-color-glitches-on-macos-qemu)
    - 13.7 [VirtIO disk device names](#137-virtio-disk-device-names)
    - 13.8 [Low ports (567, 564) requiring root](#138-low-ports-567-564-requiring-root)
    - 13.9 [Key format migration warning in FQA (can ignore on new installs)](#139-key-format-migration-warning-in-fqa-can-ignore-on-new-installs)
14. [Quick-Reference Command Cheatsheet](#14-quick-reference-command-cheatsheet)
15. [Port Reference Table](#15-port-reference-table)
16. [File Reference](#16-file-reference)
17. [External Resources](#17-external-resources)

---

## 1. Background and Vocabulary

| Term | Meaning |
|---|---|
| **9front** | A modern, actively maintained fork of Plan 9 from Bell Labs |
| **FQA** | 9front's primary documentation: http://fqa.9front.org |
| **rio** | 9front's window manager / GUI |
| **rc** | 9front's shell (similar to but distinct from bash) |
| **glenda** | Default user / hostowner on a fresh 9front install |
| **drawterm** | A client program (runs on Linux/macOS/Windows) that connects to a 9front CPU server and presents a graphical rio session |
| **cpu server** | A 9front machine running `service=cpu`; listens on port 17019 for remote execution |
| **auth server** | Handles authentication; listens on port 567 |
| **factotum** | 9front's credential/key management agent |
| **secstore** | Encrypted server-side credential store |
| **hjfs** | Hjfs filesystem (recommended for new installs; listed as "experimental" in older FQA but widely used) |
| **cwfs** | Older cache-based filesystem (cwfs64x) |
| **gefs** | Newer log-structured filesystem (available in recent 9front releases) |
| **plan9.ini** | Boot configuration file, lives on the 9FAT partition (`/n/9fat/plan9.ini`) |
| **9fat** | The FAT boot partition; mount with `9fs 9fat` |
| **virtio** | Paravirtualized I/O for QEMU (preferred for both disk and network in 9front) |
| **qcow2** | QEMU's copy-on-write disk image format |
| **KVM** | Linux kernel-based virtualisation; enabled with `-enable-kvm` |
| **hvf** | macOS Hypervisor.framework; equivalent of KVM on Apple hardware; enabled with `-accel hvf` |

---

## 2. Choosing a 9front Image

Download from: **http://9front.org/releases/** or http://9front.org/iso/

**Available image types:**

| Filename pattern | Architecture | Notes |
|---|---|---|
| `9front-NNNN.386.iso.gz` | i386 (32-bit x86) | Widest compatibility |
| `9front-NNNN.amd64.iso.gz` | x86-64 | Recommended for modern PCs |
| `9front-NNNN.amd64.qcow2.gz` | x86-64 | Pre-built QEMU image, skip install |
| `9front-NNNN.arm64.qcow2.gz` | ARM 64-bit | For Apple Silicon or aarch64 hosts |
| `9front-NNNN.pi.img.gz` | Raspberry Pi 1/2/3 | |
| `9front-NNNN.pi3.img.gz` | Raspberry Pi 3/4 | |

**Recommendation:** Unless you are on ARM hardware, download the `amd64.iso` and install to a qcow2 disk image.

---

## 3. Disk Image Creation

```sh
# Create a sparse qcow2 image (30G recommended minimum)
qemu-img create -f qcow2 9front.qcow2.img 30G

# Alternative: raw image (some users prefer for performance)
qemu-img create 9front.raw 32G
```

The qcow2 format is sparse (only uses disk space that is actually written). A 30G image is typical; 10G is the documented minimum. Use a dedicated ZFS dataset if running on a NAS.

---

## 4. QEMU Boot Commands — Installation Phase

### 4.1 Linux / KVM (amd64)

**Minimal working command:**
```sh
qemu-system-x86_64 -cpu host -enable-kvm -m 1024 \
  -net nic,model=virtio,macaddr=52:54:00:00:EE:03 -net user \
  -device virtio-scsi-pci,id=scsi \
  -drive if=none,id=vd0,file=9front.qcow2.img \
  -device scsi-hd,drive=vd0 \
  -drive if=none,id=vd1,file=9front.amd64.iso \
  -device scsi-cd,drive=vd1,bootindex=0
```

**With more RAM and audio:**
```sh
qemu-system-x86_64 -enable-kvm -cpu host -m 2048M \
  -audiodev id=alsa,driver=alsa \
  -device AC97,audiodev=alsa \
  -net nic,model=virtio,macaddr=00:20:91:37:33:77 -net user \
  -device virtio-scsi-pci,id=scsi \
  -drive if=none,id=vd0,file=9front.qcow2.img \
  -device scsi-hd,drive=vd0 \
  -drive if=none,id=vd1,file=9front.amd64.iso \
  -device scsi-cd,drive=vd1,bootindex=0
```

**With q35 machine type (modern QEMU, 2024+):**
```sh
qemu-system-x86_64 -enable-kvm -smp $(nproc) -m 8192 \
  -cpu host -M q35 \
  -cdrom ./9front-11321.amd64.iso \
  -net nic \
  -drive file=9front.raw,format=raw,media=disk,index=0,cache=writeback
```

> **Notes:**
> - Always specify a unique `macaddr` per VM to avoid collisions on a shared network.
> - `-cpu host` passes the host CPU features through and is required for KVM.
> - `-enable-kvm` is required for hardware acceleration; may need to be run as root or with the `kvm` group.
> - The virtio-scsi disk device shows up as `/dev/sd00` inside 9front.
> - The virtio-blk disk device (older method) shows up as `/dev/sdF0`.

### 4.2 macOS (x86_64 and arm64)

**Intel Mac (x86_64), using HAXM or software emulation:**
```sh
qemu-system-x86_64 -cpu host -m 2048 \
  -net nic,model=virtio,macaddr=52:54:00:00:EE:03 -net user \
  -device virtio-scsi-pci,id=scsi \
  -drive if=none,id=vd0,file=9front.qcow2.img \
  -device scsi-hd,drive=vd0 \
  -drive if=none,id=vd1,file=9front.amd64.iso \
  -device scsi-cd,drive=vd1,bootindex=0
```

> On macOS, use `-display cocoa` or `-display sdl` as appropriate.  
> Some QEMU versions on macOS exhibit graphical glitches in 16-bit color mode — use 32-bit (e.g., `1920x1080x32`) instead.

**Apple Silicon (arm64) — see dedicated section 12.**

### 4.3 Windows with TAP networking

```sh
# Requires OpenVPN TAP driver installed first
# Rename the TAP interface to "qemu-tap" in Network Manager

qemu.exe -net nic -net tap,ifname="tap-qemu" \
  -device virtio-scsi-pci,id=scsi \
  -drive if=none,id=vd0,file=9front.qcow2.img \
  -device scsi-hd,drive=vd0 \
  -drive if=none,id=vd1,file=9front.iso \
  -device scsi-cd,drive=vd1,bootindex=0
```

After configuring: `ping 10.0.0.2` from Windows host, `ip/ping 10.0.0.1` from inside 9front.

To enable internet NAT on a Linux bridge host:
```sh
iptables -t nat -A POSTROUTING -s 10.0.0.0/24 -o eth0 -j MASQUERADE
echo 1 > /proc/sys/net/ipv4/ip_forward
```

### 4.4 Headless / VNC mode

When running 9front on a headless server (SSH-only NAS, etc.):

```sh
qemu-system-x86_64 -cpu host -enable-kvm -m 4096 \
  -vnc :2 \
  -net nic,model=virtio,macaddr=52:54:00:00:EE:03 -net user \
  -device virtio-scsi-pci,id=scsi \
  -drive if=none,id=vd0,file=my9front.qcow2.img \
  -device scsi-hd,drive=vd0 \
  -drive if=none,id=vd1,file=9front.iso \
  -device scsi-cd,drive=vd1,bootindex=0
```

`-vnc :2` exposes the session on port `5902`. Connect with any VNC client (e.g., Remmina). Punch a hole in the firewall for port 5902 as needed. Once drawterm is configured, VNC can be removed.

---

## 5. Installation Walkthrough

Once the QEMU command boots the ISO, you will see 9front's boot prompt in the graphical/serial window.

### 5.1 inst/start steps in order

At the boot prompt:
- Press `Enter` to accept the default boot device.
- Press `Enter` to choose `glenda` as the user.

Then at the shell, run:
```
inst/start
```

The installer presents steps. Work through them **in order**:

| Step | Action |
|---|---|
| `partdisk` | Select your qcow2 disk (usually `sd00`). Choose `mbr` partition type, then `w` (write), `q` (quit). |
| `prepdisk` | Accept default partition, `w`, `q`. |
| `mountfs` | Choose default partition, default cache, then name the partition. |
| `configdist` | Choose `local`. |
| `confignet` | Leave as DHCP (apprehension about networking after reboot is normal — see section 7). |
| `mountdist` | Use defaults. |
| `copydist` | Press Enter and wait up to an hour for the copy. |
| `ndbsetup` | Set the system name (e.g., `my9front`). |
| `tzsetup` | Choose your timezone. |
| `bootsetup` | Keep defaults. |
| Reboot | Follow installer prompt. |

### 5.2 Filesystem choice (hjfs vs cwfs vs gefs)

| Filesystem | Recommendation |
|---|---|
| **hjfs** | Recommended for new installs. Despite "experimental" label in older FQA, it is the community default. Simpler, no separate cache partition. |
| **cwfs** | Older, more battle-tested. Requires separate `fscache` partition. Bootargs syntax differs (see section 8.3). |
| **gefs** | Newest log-structured filesystem. Available in recent 9front releases. |

### 5.3 VGA size and mouseport

During the boot/install phase you will be asked for VGA and input settings.

**VGA size recommendations:**

| Monitor | Recommended vgasize |
|---|---|
| 1080p | `1920x1080x32` |
| 1440p | `1920x1080x32` or `2560x1440x32` |
| 4K | `2560x1440x32` |
| Default (FQA) | `1024x768x16` — too small for modern monitors |

> Note: On macOS QEMU, 16-bit modes (`x16`) may produce graphical glitches. Always use `x32`.

**Mouseport:**
```
ps2intellimouse
```
This enables scroll wheel support, which is important for navigating 9front's UI. The default `ps2` works but has no scroll wheel.

Post-install, you can also set the mouse type at runtime:
```
aux/mouse ps2intellimouse
# or
echo intellimouse > /dev/mousectl
```

---

## 6. Post-Installation Boot Commands

### 6.1 Standard terminal mode

Remove the ISO drive from the QEMU command:
```sh
qemu-system-x86_64 -cpu host -enable-kvm -m 1024 \
  -net nic,model=virtio,macaddr=52:54:00:00:EE:03 -net user \
  -device virtio-scsi-pci,id=scsi \
  -drive if=none,id=vd0,file=9front.qcow2.img \
  -device scsi-hd,drive=vd0
```

### 6.2 With port forwarding (drawterm)

Required ports for full cpu+auth+drawterm access: `564`, `567`, `17019`, `17020`.

**Running as root (using low ports directly):**
```sh
qemu-system-x86_64 -cpu host -enable-kvm -m 1024 \
  -net nic,model=virtio,macaddr=52:54:00:00:EE:03 \
  -net user,hostfwd=tcp::17007-:17007,hostfwd=tcp::17010-:17010 \
            ,hostfwd=tcp::17019-:17019,hostfwd=tcp::17020-:17020 \
            ,hostfwd=tcp::564-:564,hostfwd=tcp::567-:567 \
  -device virtio-scsi-pci,id=scsi \
  -drive if=none,id=vd0,file=9front.qcow2.img \
  -device scsi-hd,drive=vd0
```

**Non-root workaround (remapping to high ports):**
```sh
#!/bin/sh
qemu-system-x86_64 -cpu host -enable-kvm -m 1024 \
  -net nic,model=virtio,macaddr=52:54:00:00:EE:03 \
  -net user \
    ,hostfwd=tcp::5640-:564 \
    ,hostfwd=tcp::17019-:17019 \
    ,hostfwd=tcp::5670-:567 \
    ,hostfwd=tcp::17020-:17020 \
  -device virtio-scsi-pci,id=scsi \
  -drive if=none,id=vd0,file=$HOME/vm/9front.qcow2.img \
  -device scsi-hd,drive=vd0
```
Then drawterm with: `drawterm -a 'tcp!localhost!5640' -s localhost -h localhost -u glenda`

**Alternative high-port remap (port 12567 → 567):**
```sh
qemu-system-x86_64 -cpu host -enable-kvm -m 1024 \
  -net nic,model=virtio,macaddr=52:54:00:00:EE:03 \
  -net user,hostfwd=tcp::17019-:17019,hostfwd=tcp::17020-:17020 \
           ,hostfwd=tcp::12567-:567 \
  -device virtio-scsi-pci,id=scsi \
  -drive if=none,id=vd0,file=9front.qcow2.img \
  -device scsi-hd,drive=vd0
```
Drawterm: `drawterm -a 'tcp!localhost!12567' -s localhost -h localhost -u glenda`

**With rebind on port 17567 (offbeatpursuit.com pattern):**
```sh
qemu-system-x86_64 -cpu host -enable-kvm -m 4096 \
  -net nic,model=virtio,macaddr=52:54:00:00:EE:03 \
  -net user,hostfwd=tcp::17019-:17019,hostfwd=tcp::17020-:17020 \
           ,hostfwd=tcp::17567-:567 \
  -device virtio-scsi-pci,id=scsi \
  -drive if=none,id=vd0,file=9front_terminal.qcow2 \
  -device scsi-hd,drive=vd0
```
Drawterm: `drawterm -a 'tcp!localhost!17567' -h localhost -s localhost -u glenda`

### 6.3 Multiple disk images

Mount multiple qcow2 images (useful for data transfer, partition resizing):
```sh
qemu-system-x86_64 -cpu host -enable-kvm -m 4096 \
  -net nic,model=virtio,macaddr=52:54:00:00:EE:03 -net user \
  -device virtio-scsi-pci,id=scsi \
  -drive if=none,id=vd0,file=9front_main.qcow2 \
  -device scsi-hd,drive=vd0 \
  -drive if=none,id=vd1,file=9front_extra.qcow2 \
  -device scsi-hd,drive=vd1 \
  -drive if=none,id=vd2,file=other_backup.img \
  -device scsi-hd,drive=vd2
```

### 6.4 q35 machine type (modern QEMU)

Preferred for QEMU 7+ on modern Linux hosts:
```sh
# Install from ISO:
qemu-system-x86_64 -enable-kvm -smp $(nproc) -m 8192 \
  -cpu host -M q35 \
  -cdrom ./9front-11321.amd64.iso \
  -net nic \
  -drive file=9front.raw,format=raw,media=disk,index=0,cache=writeback

# Normal boot:
qemu-system-x86_64 -enable-kvm -smp $(nproc) -m 8192 \
  -cpu host -M q35 \
  -net nic -net user,hostfwd=tcp:127.0.0.1:17019-:17019 \
  -drive file=9front.raw,format=raw,media=disk,index=0,cache=writeback
```

### 6.5 With audio (AC97)

```sh
qemu-system-x86_64 -enable-kvm -cpu host -m 2048M \
  -audiodev id=alsa,driver=alsa \
  -device AC97,audiodev=alsa \
  -net nic,model=virtio,macaddr=00:20:91:37:33:77 -net user \
  -device virtio-scsi-pci,id=scsi \
  -drive if=none,id=vd0,file=9front.qcow2.img \
  -device scsi-hd,drive=vd0
```

### 6.6 USB tablet for mouse accuracy

Improves absolute mouse positioning (eliminates cursor drift in QEMU):
```sh
# Append to any QEMU command:
-device qemu-xhci,id=xhci -device usb-tablet,bus=xhci.0
```

---

## 7. Networking Inside 9front

### 7.1 ip/ipconfig — getting a DHCP address

After booting, if networking is not available:
```
ip/ipconfig
```

This configures both IPv4 and IPv6 using QEMU's built-in DHCP/DNS proxy. Test with:
```
ip/ping google.com
```

Note: With user-mode QEMU networking (`-net user`), you **cannot ping remote hosts** from the guest (ICMP is filtered). Use `hget` for HTTP checks instead:
```
hget http://example.com > /dev/null
```

### 7.2 Persistent networking on boot (termrc / cpurc)

**For terminal mode** — add `ip/ipconfig` to `/rc/bin/termrc` just before the DNS section:
```rc
# in /rc/bin/termrc
ip/ipconfig
```

**For CPU server mode** — add `ip/ipconfig` to `/rc/bin/cpurc`:
```rc
# in /rc/bin/cpurc
ip/ipconfig
```

### 7.3 /lib/ndb/local configuration

This is the network database file that 9front uses to locate auth, cpu, and fileserver services.

**Get current network info first:**
```
ndb/query sys <your-sysname>
# or
netaudit
```

Example output:
```
ip=10.0.2.15 ipmask=255.255.255.0 ipgw=10.0.2.2 sys=QemuTerm dns=8.8.8.8
sys=QemuTerm ether=52540000ee03
```

**Edit `/lib/ndb/local`** to add a complete network stanza. The indentation (tab character) under `ipnet=` lines is **mandatory**:

```
auth=QemuTerm authdom=9front
ipnet=9front ip=10.0.2.0 ipmask=255.255.255.0
	ipgw=10.0.2.2
	dns=8.8.8.8
	auth=QemuTerm
	dnsdom=9front
	cpu=QemuTerm
	fs=QemuTerm
sys=QemuTerm ether=52540000ee03 ip=10.0.2.15
```

**Rules:**
- `auth` and `cpu` must be set to the system name (sysname).
- `authdom` can be anything but must match what you entered at the boot prompt.
- `ipnet`'s `ip` field must be the **subnet** address (last octet = 0), not the host IP.
- There must be an empty line at the end of the file.

**Example for a real network (system named "cirno"):**
```
#  files comprising the database
database=
	file=/net/ndb
	file=/lib/ndb/local
	file=/lib/ndb/common

# DNS root entries
dom=
	ns=A.ROOT-SERVERS.NET
	...

ip=127.0.0.1 sys=localhost dom=localhost

auth=cirno authdom=9front
ipnet=9front ip=192.168.1.0 ipmask=255.255.255.0
	ipgw=192.168.1.1
	dns=192.168.1.1
	auth=cirno
	dnsdom=9front
	cpu=cirno
	fs=cirno

sys=cirno ether=000c29e16a29
```

### 7.4 netaudit — verifying the configuration

```
netaudit
```

A healthy output looks like:
```
checking this host's tuple:
  ip=192.168.1.23 looks ok
  no dom= entry
  ether=000c29e16a29 looks ok
checking the network tuple:
  we are in ipnet=9front
  ipgw=192.168.1.1 looks ok
  dns=192.168.1.1 look ok
  auth=cirno looks ok
checking auth server configuration:
  we are the auth server
  auth/keyfs is running
  someone is listening on port 567
  run auth/debug to test the auth server
  run auth/asaudit to verify auth server configuration
```

If `auth/keyfs is running` and `someone is listening on port 567` appear, auth is configured correctly.

---

## 8. CPU and Auth Server Setup

### 8.1 Switching service=cpu in plan9.ini

Mount the 9FAT partition and edit `plan9.ini`:
```
9fs 9fat
acme /n/9fat/plan9.ini
# or
ed /n/9fat/plan9.ini
```

Add/change:
```ini
service=cpu
```

Full example `plan9.ini` for a QEMU VM:
```ini
bootfile=9pc64
nobootprompt=local!/dev/sd00/fs -m 256 -A -a tcp!*!564
mouseport=ps2intellimouse
monitor=vesa
vgasize=1280x720x16
user=glenda
tiltscreen=none
service=cpu
```

Reboot:
```
fshalt -r
```

### 8.2 The boot prompt after first cpu reboot

On first reboot in CPU mode, you will see (some or all of):
```
bad nvram key
bad authentication id
bad authentication domain
authid: <glenda>
authdom: <9front>
secstore key:
password: [glenda's password]
```

**Correct responses:**
- `authid:` → type `glenda`
- `authdom:` → type `9front` (or your chosen domain — must match `/lib/ndb/local`)
- `secstore key:` → press Enter (empty, unless you set one)
- `password:` → type glenda's password (the one you'll set with `auth/changeuser`)

You will then be in a raw `rc` shell with no `rio`. To get graphics manually:
```
termrc
rio
```
Or start rio with:
```
rio -i riostart
```

### 8.3 bootargs for hjfs and cwfs

**For hjfs** (substitute your actual disk identifier and cache size):
```ini
bootargs=local!/dev/sd00/fs -m 256 -A -a tcp!*!564
```

For unattended boot (no prompt):
```ini
nobootprompt=local!/dev/sd00/fs -m 256 -A -a tcp!*!564
```

**For cwfs**:
```ini
bootargs=local!/dev/sd00/fscache -a tcp!*!564
```

First cwfs cpu-mode boot workaround (if it asks for config):
```
config: noauth
auth is now disabled
config: noauth
auth is now enabled
config: end
```

### 8.4 auth/changeuser — setting passwords

```
auth/changeuser glenda
```
Prompts:
```
Password:           # enter password (will not echo)
Confirm password:   # confirm
assign Inferno/POP secret? (y/n) n
Expiration date (YYYYMMDD or never)[return = never]: <Enter>
Post id: <Enter>
User's full name: <Enter>
...
```

**Important:** The password entered here must match what you type at the CPU server boot prompt.

To also set glenda's password in factotum interactively:
```
auth/factotum -g 'proto=p9sk1 dom=9front user=glenda !password=YOUR_PASSWORD'
```

### 8.5 cpurc and cpurc.local customization

The main CPU server startup script is `/rc/bin/cpurc`. You can add a local override file `/rc/bin/cpurc.local`.

**Enabling services in cpurc:**
```rc
# /rc/bin/cpurc (or cpurc.local)
ip/ipconfig

serviced=/rc/bin/service
auth/keyfs -wp -m /mnt/keys /adm/keys
auth/cron >>/sys/log/cron >[2=1] &
aux/listen -q -t /rc/bin/service.auth -d $serviced tcp
```

**Service files for CPU and auth:**
```
/rc/bin/service/tcp17019        # CPU server
/rc/bin/service.auth/tcp567     # Auth server
```

**cpurc.local — optional start of graphics in QEMU:**
```rc
# /rc/bin/cpurc.local
termrc
. /usr/$user/lib/profile &
#rio -i /usr/$user/bin/rc/riostart  # uncomment to start rio automatically
```

### 8.6 Secstore setup

Secstore is an encrypted per-user key storage server.

**Create secstore directories:**
```
mkdir /adm/secstore
chmod 770 /adm/secstore
```

**Add to cpurc:**
```rc
auth/secstored
```

**Add a user to secstore:**
```
auth/secstored
auth/secuser glenda
```

**Initialize the factotum file in secstore** (prevents login error `secstore: remote file factotum does not exist`):
```
touch factotum
auth/secstore -p factotum
```

### 8.7 Adding non-glenda users

```
auth/keyfs
auth/changeuser <username>
auth/secuser <username>
echo newuser <username> >> /srv/hjfs.cmd
```

Then log in as the new user and run:
```
/sys/lib/newuser
```

If you see the secstore factotum error on login:
```
touch factotum
auth/secstore -p factotum
```

---

## 9. Drawterm Connection

### 9.1 Which drawterm to use

**Use 9front's own fork of drawterm**: http://drawterm.9front.org  
**Do NOT use** Russ Cox's version at `swtch.com/drawterm` — it will not work with 9front due to changed security protocols.

**Building drawterm from source:**
```sh
git clone git://git.9front.org/plan9front/drawterm
cd drawterm
# On Linux:
CONF=unix make
# libX11-devel and libXt-devel required

# On macOS:
CONF=osx make
```

### 9.2 QEMU port forwarding for drawterm

The following ports must be forwarded from QEMU to the host:

| Port | Protocol | Service |
|---|---|---|
| 564 | TCP | Auth server (IL/9p, secstore) |
| 567 | TCP | Auth server (p9sk1) |
| 17019 | TCP | CPU server (rcpu/drawterm) |
| 17020 | TCP | Filesystem export (optional) |
| 17007 | TCP | Additional Plan 9 service |
| 17010 | TCP | Additional Plan 9 service |

Minimum required for drawterm: **567 + 17019**  
For full functionality: **564 + 567 + 17019 + 17020**

### 9.3 drawterm command syntax

**Basic connection (LAN, low ports forwarded directly):**
```sh
drawterm -h 192.168.0.105 -u glenda
# Or with explicit auth server:
drawterm -a 192.168.0.105 -h 192.168.0.105 -u glenda
```

**Local QEMU with direct port forwarding:**
```sh
drawterm -h 127.0.0.1 -u glenda
```

**Local QEMU with non-privileged port remapping:**
```sh
# Auth forwarded to port 5640, secstore also at 5640:
drawterm -a 'tcp!localhost!5640' -s localhost -h localhost -u glenda

# Auth forwarded to port 12567:
drawterm -a 'tcp!localhost!12567' -s localhost -h localhost -u glenda

# Auth forwarded to port 17567:
drawterm -a 'tcp!localhost!17567' -h localhost -s localhost -u glenda
```

> **Note on `-s` flag:** When using remapped ports, `-s` (secstore) must also be specified to the correct address/port. Without it, drawterm defaults to using the auth address for secstore, which may fail.

**After connecting:** At the `auth` prompt, press Enter (empty). Then type the password for `glenda`. Once in the shell, run `rio` to start the graphical interface.

### 9.4 Non-privileged port workaround

Ports below 1024 require root on Linux. Remap 567 to a high port on the host side:

```sh
# In QEMU command:
-net user,hostfwd=tcp::17567-:567,hostfwd=tcp::17019-:17019

# In drawterm:
drawterm -a 'tcp!localhost!17567' -h localhost -s localhost -u glenda
```

Alternatively, use `authsrv` address in drawterm with the full `tcp!host!port` syntax.

### 9.5 Minimal rcpu/drawterm without full auth server

For quick testing without persistent auth setup (live ISO session or fresh install):

```sh
# Inside 9front:
touch $home/lib/keys
auth/keyfs -p $home/lib/keys
auth/changeuser -p glenda         # set password
auth/factotum -n
echo 'key proto=dp9ik dom=9front user='$user' !password=mypassw' >/mnt/factotum/ctl
aux/listen1 -t 'tcp!*!rcpu' /rc/bin/service/tcp17019
# (runs in foreground; add & for background)
```

Then from host:
```sh
drawterm -h 127.0.0.1 -u glenda
# Press Enter at auth prompt, type password
```

Then inside drawterm:
```
rio
```

---

## 10. Automatic Boot (nobootprompt / unattended)

To boot 9front fully unattended (no VNC session required), edit `/n/9fat/plan9.ini`:

```ini
user=glenda
```

Change `bootargs` to `nobootprompt`:
```ini
nobootprompt=local!/dev/sd00/fs -m 256 -A -a tcp!*!564
```

The `user=glenda` line pre-selects the user. The `nobootprompt` line skips the interactive boot menu entirely.

Reboot to test:
```
fshalt -r
```

---

## 11. rio and Profile Setup for Remote Sessions

When connecting via drawterm, you often want `rio` to start automatically and have web/filesystem access.

**Edit `/usr/glenda/lib/profile`** — add the following block after `fn cpu% { $* }`:

```rc
# For drawterm sessions: web, filesystem binding, and auto-start rio
if(! webcookies >[2]/dev/null)
    webcookies -f /tmp/webcookies
webfs
plumber
rio -i riostart
bind -ac /mnt/term $home/term
```

The `bind -ac /mnt/term $home/term` line makes the drawterm host's files accessible at `$home/term` inside the session.

**Creating a custom remote riostart script:**

```
cp /usr/glenda/bin/rc/riostart /usr/glenda/bin/rc/riostart-remote
# Edit riostart-remote as desired
```

Then in the profile's cpu section:
```rc
if(! webcookies >[2]/dev/null)
    webcookies -f /tmp/webcookies
webfs
plumber
rio -i riostart-remote
bind -ac /mnt/term $home/term
```

**Profile additions for cpu instances that also need graphics locally:**

Add after `fn cpu%{ $* }` in `/usr/$user/lib/profile`:
```rc
# custom addition to work with /bin/cpurc.local
if(! webcookies >[2]/dev/null)
    webcookies -f /tmp/webcookies
webfs
plumber
rio -i riostart -s
```

---

## 12. arm64 QEMU (Apple Silicon / aarch64)

9front provides a dedicated arm64 QCOW2 image for ARM hosts (Apple M-series, etc.).

**Key characteristics of the arm64 9front QEMU image:**
- Supports XHCI USB and PCIe devices (used for VirtIO).
- **No graphical interface** — must be driven via serial console.
- After initial setup, connect via drawterm for GUI.
- Requires U-Boot (must be built from source — no prebuilt binaries available).
- The arm64 kernel was specifically written to work under macOS `Hypervisor.framework`.

**Building U-Boot for arm64 QEMU:**
```sh
git clone https://source.denx.de/u-boot/u-boot.git
cd u-boot
make qemu_arm64_defconfig
make -j$(nproc) CROSS_COMPILE=aarch64-linux-gnu-
# Produces u-boot.bin
```

**Running arm64 9front in QEMU (macOS with HVF acceleration):**
```sh
qemu-system-aarch64 \
  -M virt,accel=hvf \
  -cpu host \
  -smp 2 -m 4096 \
  -bios u-boot.bin \
  -device virtio-net-pci-non-transitional,netdev=net0 \
  -netdev user,id=net0,hostfwd=tcp::17019-:17019,hostfwd=tcp::17567-:567 \
  -drive if=none,id=vd0,file=9front.arm64.qcow2 \
  -device virtio-blk-pci-non-transitional,drive=vd0 \
  -nographic \
  -serial stdio
```

**Notes for virt machine version compatibility:**
- `virt-2.12` may not be included in modern `qemu-system-aarch64`.
- `virt-10.1` (modern) works, but requires placing ECAM in lower memory:
```sh
  -M virt-10.1,accel=hvf,highmem=off
```
- If using `gic-version=2` with custom kernel (because `-accel kvm` doesn't work with GICv3 unless host has GICv3):
```sh
  -M virt,accel=hvf,gic-version=2
```

**Networking:**
Use `virtio-net-pci-non-transitional` for arm64 (not the standard `virtio` shorthand).

**Drawterm on Apple Silicon connecting to arm64 QEMU:**
Same as connecting to amd64 — drawterm runs natively on macOS, the architecture of the QEMU guest doesn't matter from drawterm's perspective.

Alternatively, running x86_64 QEMU on Apple Silicon (emulated, not accelerated) is also reportedly usable for 9front since it is not a demanding OS.

---

## 13. Known Issues and Troubleshooting

### 13.1 Mouse problems

**Symptom:** Mouse sensitivity is extremely wonky inside QEMU/VNC, cursor doesn't track correctly.

**Causes and fixes:**
- VNC mouse is always problematic due to relative vs absolute positioning.
- Fix: Use USB tablet device for absolute positioning: `-device qemu-xhci,id=xhci -device usb-tablet,bus=xhci.0`
- Fix: Enable `ps2intellimouse` at the install prompt or post-boot: `aux/mouse ps2intellimouse` or `echo intellimouse > /dev/mousectl`
- The scroll wheel requires `ps2intellimouse` (not available with default `ps2`).
- Setting resolution via `aux/vga` does NOT fix mouse sensitivity in VNC.

**Adjusting resolution at runtime:**
```
@{rfork n; aux/realemu; aux/vga -p}
@{rfork n; aux/realemu; aux/vga -m vesa -l 1920x1080x32}
```

### 13.2 Networking not working after boot

**Symptom:** `ip/ping google.com` fails after fresh install and reboot.

**Fix:** Run `ip/ipconfig` manually, then add it to the appropriate startup script (see section 7.2).

**Note on QEMU user-mode networking:** The default QEMU `-net user` provides:
- DHCP for the guest (default: 10.0.2.15)
- DNS via QEMU proxy
- NAT internet access
- **No** ICMP ping to external hosts (filtered by SLIRP)
- Port forwarding with `hostfwd=`

**Ping times may be "flaky"** in user-mode QEMU networking — this is normal and a known QEMU behaviour.

### 13.3 Drawterm timeout / slow login

**Symptom:** Drawterm hangs for ~1 minute before proceeding when connecting on LAN.

**Possible causes:**
- A firewall is blocking an intermediate port that drawterm tries.
- The auth server is trying to look up a hostname via a DNS server that doesn't respond.
- Port 564 (secstore) may be blocked — if you only forwarded 567 and 17019.

**Fixes:**
- Temporarily disable the host firewall to diagnose.
- Ensure all four ports are forwarded: 564, 567, 17019, 17020.
- Check that `/lib/ndb/local` has the correct IP/DNS/gateway.
- Run `netaudit` inside 9front to verify auth configuration.

### 13.4 bad nvram key / bad authentication domain

**Symptom:** On first boot into cpu mode, you see these messages.

**This is normal** for a new CPU mode installation. The NVRAM just hasn't been initialized yet. Respond to each prompt:
```
authid: glenda
authdom: 9front
secstore key: <Enter>
password: <your password>
```

These prompts will disappear on subsequent boots once the NVRAM is written.

### 13.5 secstore factotum error on login

**Symptom:**
```
secstore
secstore: remote file factotum does not exist
secstore: cmd failed
```

**Cause:** The secstore for this user has no `factotum` file initialized.

**Fix:**
```
touch factotum
auth/secstore -p factotum
```

Reboot or re-login to confirm the error is gone.

### 13.6 16-bit color glitches on macOS QEMU

**Symptom:** Graphical corruption, color banding, or display artifacts when running QEMU on macOS.

**Cause:** Some macOS QEMU builds have a bug with 16-bit (`x16`) color modes.

**Fix:** Use 32-bit color mode in `plan9.ini`:
```ini
vgasize=1920x1080x32
```
Replace any `x16` suffix with `x32`.

### 13.7 VirtIO disk device names

When using VirtIO disk devices, 9front assigns them these names:

| QEMU device | 9front device path |
|---|---|
| `virtio-blk` (older `-drive if=virtio`) | `/dev/sdF0` |
| `virtio-scsi` (`-device virtio-scsi-pci` + `-device scsi-hd`) | `/dev/sd00` |

Use the correct device name when partitioning (`inst/partdisk`) or setting `bootargs`.

### 13.8 Low ports (567, 564) requiring root

**Symptom:** QEMU fails to bind ports 567 or 564 without root privileges.

**Options:**
1. Run QEMU as root (acceptable in isolated environments).
2. Use firewall rules to redirect high ports to low ports on the host.
3. Remap in QEMU: `hostfwd=tcp::17567-:567` and specify the new port in drawterm's `-a` flag.

### 13.9 Key format migration warning in FQA (can ignore on new installs)

FQA section 7.4.3.2 mentions converting to a newer key format. **This can be ignored on new installations.** It applies only to existing installs that predate the key format change. New 9front installs already use the correct key format.

---

## 14. Quick-Reference Command Cheatsheet

```sh
# --- QEMU: Create disk image ---
qemu-img create -f qcow2 9front.qcow2.img 30G

# --- QEMU: Boot from ISO (Linux KVM) ---
qemu-system-x86_64 -cpu host -enable-kvm -m 2048 \
  -net nic,model=virtio,macaddr=52:54:00:00:EE:03 -net user \
  -device virtio-scsi-pci,id=scsi \
  -drive if=none,id=vd0,file=9front.qcow2.img -device scsi-hd,drive=vd0 \
  -drive if=none,id=vd1,file=9front.amd64.iso -device scsi-cd,drive=vd1,bootindex=0

# --- QEMU: Boot installed VM with drawterm ports ---
qemu-system-x86_64 -cpu host -enable-kvm -m 2048 \
  -net nic,model=virtio,macaddr=52:54:00:00:EE:03 \
  -net user,hostfwd=tcp::17019-:17019,hostfwd=tcp::17567-:567 \
  -device virtio-scsi-pci,id=scsi \
  -drive if=none,id=vd0,file=9front.qcow2.img -device scsi-hd,drive=vd0
```

```rc
# --- 9front: Start installer ---
inst/start

# --- 9front: Get DHCP address ---
ip/ipconfig

# --- 9front: Ping test ---
ip/ping google.com

# --- 9front: Mount 9FAT (boot config partition) ---
9fs 9fat

# --- 9front: Edit plan9.ini ---
acme /n/9fat/plan9.ini

# --- 9front: Reboot ---
fshalt -r

# --- 9front: Shutdown ---
fshalt

# --- 9front: Check network config ---
netaudit
ndb/query sys <sysname>

# --- 9front: Set CPU mode ---
# Edit /n/9fat/plan9.ini, add: service=cpu

# --- 9front: Set password ---
auth/changeuser glenda

# --- 9front: Start graphics manually (after cpu reboot) ---
termrc
rio

# --- 9front: Init secstore factotum file ---
touch factotum
auth/secstore -p factotum

# --- 9front: Add new user ---
auth/keyfs
auth/changeuser <username>
auth/secuser <username>
echo newuser <username> >> /srv/hjfs.cmd

# --- 9front: Set mouse to intellimouse (scroll wheel) ---
aux/mouse ps2intellimouse
# or
echo intellimouse > /dev/mousectl

# --- 9front: Change VGA resolution at runtime ---
@{rfork n; aux/realemu; aux/vga -m vesa -l 1920x1080x32}

# --- Drawterm: Connect (low ports forwarded) ---
drawterm -h 127.0.0.1 -u glenda

# --- Drawterm: Connect (high port remapping) ---
drawterm -a 'tcp!localhost!17567' -h localhost -s localhost -u glenda

# --- Drawterm: Connect to LAN 9front ---
drawterm -a 192.168.0.105 -h 192.168.0.105 -u glenda

# --- 9front: Minimal drawterm-ready setup (no persistent cpu) ---
touch $home/lib/keys
auth/keyfs -p $home/lib/keys
auth/changeuser -p glenda
auth/factotum -n
echo 'key proto=dp9ik dom=9front user=glenda !password=PASS' >/mnt/factotum/ctl
aux/listen1 -t 'tcp!*!rcpu' /rc/bin/service/tcp17019 &
```

---

## 15. Port Reference Table

| Port | Service | Notes |
|---|---|---|
| **564** | Secstore / IL 9P fileserver | Required for secstore access; sometimes maps to auth address |
| **567** | Auth server (p9sk1) | Primary auth port; needs `tcp567` service file |
| **17007** | Plan 9 additional service | Forwarded by some setups |
| **17010** | Plan 9 additional service | |
| **17019** | CPU server (rcpu) | Primary drawterm/rcpu port; needs `tcp17019` service file |
| **17020** | Filesystem export | Optional; for mounting 9front's filesystem remotely |
| **5902** | VNC (`:2`) | QEMU VNC display :2 → port 5902 |

---

## 16. File Reference

| File | Purpose |
|---|---|
| `/n/9fat/plan9.ini` | Boot config (accessed via `9fs 9fat`) |
| `/lib/ndb/local` | Network database; auth/cpu/dns server entries |
| `/rc/bin/termrc` | Terminal mode startup script |
| `/rc/bin/cpurc` | CPU server startup script |
| `/rc/bin/cpurc.local` | Local CPU startup overrides |
| `/rc/bin/service/tcp17019` | CPU server service handler |
| `/rc/bin/service.auth/tcp567` | Auth server service handler |
| `/usr/glenda/lib/profile` | User profile (shell config, rio auto-start) |
| `/usr/glenda/bin/rc/riostart` | Default rio startup script |
| `/adm/keys` | Authentication key database |
| `/adm/secstore` | Secstore directory |
| `/srv/hjfs.cmd` | hjfs command pipe (for adding users) |
| `/dev/mousectl` | Mouse control device |
| `/mnt/factotum/ctl` | Factotum control; write keys here |
| `$home/lib/keys` | Per-user key file (used with `auth/keyfs -p`) |

---

## 17. External Resources

| Resource | URL |
|---|---|
| 9front FQA (main docs) | http://fqa.9front.org |
| FQA 3 — Hardware / QEMU | http://fqa.9front.org/fqa3.html |
| FQA 4 — Installation | http://fqa.9front.org/fqa4.html |
| FQA 7 — CPU/Auth server | http://fqa.9front.org/fqa7.html |
| 9front releases | http://9front.org/releases/ |
| 9front drawterm | http://drawterm.9front.org |
| Thor Hanson — 9front in QEMU (2022) | https://thorjhanson.com/blog/20220324-9front-in-qemu |
| Philip Bohun — Plan9 Survival Guide (2024) | https://philipbohun.com/blog/0004.html |
| samhza — Basic 9front QEMU Setup (2021) | https://samhza.com/2021/9front-qemu/ |
| offbeatpursuit — QEMU+drawterm notes | https://offbeatpursuit.com/notes/plan9/qemu |
| bsandro — 9front on QEMU with drawterm (2025) | https://bsandro.tech/posts/9front-on-qemu-with-drawterm-on-linux/ |
| Jacob GW — Remote CPU Server On 9front | https://jacobgw.com/blog/plan9/2021/05/19/cpuserv.html |
| 99z — 9front tutorials gist | https://gist.github.com/99z/1ff7c64f700a18e4025aa3c36ae78df0 |
| baryluk — Plan 9 in QEMU gist | https://gist.github.com/baryluk/e9ba5af68d67e14b8a5716a70656d550 |
| Mitja Felicijan — Run 9front in QEMU (2023) | https://mitjafelicijan.com/run-9front-in-qemu.html |
| majiru — 9front-in-a-box (Nix flake) | https://github.com/majiru/9front-in-a-box |
| 9front mailing list archive | https://inbox.vuxu.org/9front/ |
| Configuring a Standalone CPU Server (9p.io) | https://9p.io/wiki/plan9/Configuring_a_Standalone_CPU_Server/index.html |
| Peter Mikkelsen — TLS / Let's Encrypt for 9front | http://pmikkelsen.com/plan9/lets_encrypt |
| Netsurf for 9front | https://github.com/netsurf-plan9/nsport |
