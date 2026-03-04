# Arch Linux: Install & Configure

> **Type:** Runbook &nbsp;|&nbsp; **Classification:** Personal

A step-by-step runbook for installing and configuring Arch Linux with Hyprland as the desktop environment.

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [Installation Procedure](#installation-procedure)
- [System Configuration](#system-configuration)
- [Post-Installation & Configuration](#post-installation--configuration)

---

## Prerequisites

### 1. Installation Image

Download the latest Arch Linux ISO from the official site:
[https://archlinux.org/download/](https://archlinux.org/download/)

### 2. Verify Signature

Verify the integrity of the downloaded ISO before proceeding:

| Operating System | Command |
|-----------------|---------|
| Windows | `certutil -hashfile <file_path> SHA256` |
| Linux | `sha256sum <file_path>` |

Compare the output with the official SHA256 signature:
[https://archlinux.org/iso/2025.12.01/sha256sums.txt](https://archlinux.org/iso/2025.12.01/sha256sums.txt)

### 3. Installation Medium

Flash the ISO to a USB drive using one of the following tools:
- [Rufus](https://rufus.ie/en/)
- [Ventoy](https://www.ventoy.net/en/index.html)

> **Reference:** [Arch Linux Official Installation Guide](https://wiki.archlinux.org/title/Installation_guide)

---

## Installation Procedure

### 1. Connect to Network

```bash
$ iwctl
> device list
> station list
> station wlan0 connect <ssid>
```
> Enter passphrase

```bash
$ exit

$ ping archlinux.org
```

### 2. Update System Clock

```bash
$ timedatectl
$ timedatectl set-timezone Asia/Kolkata
```

### 3. Disk Partitioning

```bash
$ lsblk

# Wipe the NVMe drive
$ nvme format --force /dev/nvme0n1 -s 1 -n 0xffffffff
$ nvme sanitize /dev/nvme0n1 -a 2

# Partition with gdisk
$ gdisk /dev/nvme0n1
# o          → create new GPT partition table
# n          → new partition (+1G → EF00 for EFI, ALL → 8304 for root)
# p          → print partition table
# w          → write and exit
```

### 4. Format Partitions

```bash
$ mkfs.fat -F 32 /dev/nvme0n1p1    # EFI partition
$ mkfs.ext4 /dev/nvme0n1p2         # Root partition
```

### 5. Mount Partitions

```bash
$ mount -o defaults,noatime,nodiratime,discard,barrier=1,data=ordered /dev/nvme0n1p2 /mnt

$ mkdir -p /mnt/boot
$ mount -t vfat -o defaults,noatime,nodiratime,umask=000 /dev/nvme0n1p1 /mnt/boot

# Create and enable swapfile (12G)
$ fallocate -l 12G /mnt/swapfile
$ chmod 600 /mnt/swapfile
$ mkswap -L swapfile /mnt/swapfile
$ swapon -o defaults,pri=100,discard=pages /mnt/swapfile
```

### 6. Update Mirrorlist

```bash
$ reflector --verbose --protocol https --sort rate --save /etc/pacman.d/mirrorlist
```

### 7. Install Base & Firmware Packages

```bash
$ pacstrap -K /mnt base

$ echo "LANG=en_US.UTF-8" >> /mnt/etc/locale.conf
$ echo "KEYMAP=us" >> /mnt/etc/vconsole.conf
$ echo "<your-hostname>" >> /mnt/etc/hostname

$ pacstrap -K /mnt linux linux-headers linux-firmware-intel intel-ucode linux-firmware-nvidia nvidia-open linux-firmware-realtek neovim grub efibootmgr wireplumber pipewire networkmanager bluez acpid zsh
```

---

## System Configuration

### 1. Generate fstab

```bash
$ genfstab -U -p /mnt >> /mnt/etc/fstab
$ cat /mnt/etc/fstab
```

### 2. Chroot into the New System

```bash
$ arch-chroot /mnt
```

### 3. Configure Locale & Timezone

```bash
$ pacman -Syyu
$ ln -sf /usr/share/zoneinfo/Asia/Kolkata /etc/localtime
$ hwclock --systohc
$ nvim /etc/locale.gen
```

> Uncomment en_US.UTF-8 UTF-8

```bash
$ locale-gen
```

### 4. Install Kernel & Essential Packages

```bash
$ pacman -S --needed base-devel git neovim sudo reflector pipewire-pulse network-manager-applet blueman vlc curl dolphin firewalld alacritty hyprland hyprpolkitagent hyprpaper waybar rofi sddm mako xdg-desktop-portal xdg-desktop-portal-hyprland python python-pip fastfetch cliphist hyprpwcenter power-profiles-daemon
```

### 5. Configure Initramfs

```bash
$ nvim /etc/mkinitcpio.conf
```

> Set: MODULES=(i915 nvidia nvidia_modeset nvidia_uvm nvidia_drm)

```bash
$ mkinitcpio -P
```
### 6. Set `root` Password

```bash
$ passwd
```

### 7. User Creation & Password

```bash
$ useradd -m -G wheel parshaw
$ passwd parshaw

# Set up sudo access
$ export EDITOR = nvim
```
> Uncomment: %wheel ALL=(ALL:ALL) NOPASSWD:ALL

```bash
$ sudo nvim /etc/sudoers.d/parshaw
```
> parshaw ALL=(root) NOPASSWD:ALL

### 8. Install Bootloader (GRUB)

```bash
$ grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
$ grub-mkconfig -o /boot/grub/grub.cfg
```

### 9. Enable Services

```bash
$ systemctl enable NetworkManager bluetooth firewalld reflector.timer fstrim.timer acpid sddm
```

### 10. Reboot

```bash
$ swapoff /mnt/swapfile
$ umount -R /mnt
$ reboot
```

---

## Post-Installation & Configuration

### 1. Update & Upgrade

```bash
$ pacman -Syyu
```

### 2. Install AUR Helper (paru)

```bash
$ git clone https://aur.archlinux.org/paru.git
$ cd paru
$ makepkg -si
$ cd .. && rm -rf paru
```

### 3. Install Arch Packages

```bash
$ sudo pacman -S <package-name>
```

### 4. Install AUR Packages

```bash
$ paru -Sy brave-bin visual-studio-code-bin
```

### 5. Install Oh My Zsh

```bash
$ sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

### 6. Configure Hyprland

```bash
$ git clone https://github.com/Parshaw-Bhattacharjee/arch-conf.git
```

### 7. Path of the respective settings

> fastfetch: ~/.config

> hypr: ~/.config
<p>Note: Always make a backup of the old setting of hypr.</p>

> mako: ~/.config

> rofi: ~/.config

> sddm: /etc/sddm.d

> waybar: ~/.config

> zsh: ~
<p>Note: Always paste the files inside the zsh folder.</p>
---

<p align="center">Kindly modify the commands & configuration as per your use-case.</p>