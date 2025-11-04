# ArchLinux Project Documentation   
### kamdonsmarshall.github.io

- Downloaded ISO from HTTP from MIT mirror site  
- Validated the checksum using 7Zip and compared the two SHA256 values  
- Config:  
  - Installed VM on VMware Workstation, created Guest Operating System using ISO (Linux, Version - Other Linux 6.x kernel 64-bit)  
  - Allocated 2GB of RAM and 20GB of HDD Storage  

---

## 1. **STEP 1**

### 1.5) Set the keyboard layout using
1. `localectl list-keymaps`
2. `loadkeys us`

### 1.5) Set font using
1. `setfont ter-132b`

### 1.6) Verify the boot mode
1. `cat /sys/firmware/efi/fw_platform_size`
2. Output: No such file or directory. So, that means my system is booted in BIOS mode.  
3. Second time around: Changed VMX file's second line to "firmware = efi", reloaded VM, tried boot again, 64 was the output (UEFI now).

### 1.7) Connect to the internet
1. `ip link`  
2. Connect to the network:  
   - `systemctl start systemd-networked` # enables the network management  
   - `systemctl start systemd-resolved` # enables the DNS resolution  
   - `ip addr show ens33`  
   - `ping ping.archlinux.org` & received 9 packets (good)  
3. I had trouble getting the network to load as I kept getting "<NO-RECEIVER>" errors on the ens33 section, which means that my virtual network was detected but wouldn't connect. Restarted multiple times, checked VM settings "Network Adapter" for NAT connectivity, verified it, and then rebooted to get the ping to show.

### 1.8) Update the system clock
1. `timedatectl`  
2. `timedatectl list-timezones`  
3. `timedatectl set-timezone America/Chicago`

### 1.9) Partition the disks
1. `fdisk -l`  
2. `fdisk /dev/sda`  
   - `g` (creates a new GPT partition table)  
   - Creating EFI system partition  
     - `n`, `1`, `Enter`, `+1G`, `t`, `1`  
   - Swap partition  
     - `n`, `2`, `Enter`, `+4G`, `t`, `2`, `19`  
   - Root partition  
     - `n`, `3`, `Enter`, `Enter`, `t`, `3`, `23`  
   - `w` (write & exit)  
   - `fdisk -l /dev/sda`

### 1.10) Format the partitions
- EFI partition:  
  - `mkfs.fat -F 32 /dev/sda1`  
- Swap:  
  - `mkswap /dev/sda2` # Initializes the swap  
- Root:  
  - `mkfs.ext4 /dev/sda3`

### 1.11) Mount
1. `mount /dev/sda3 /mnt`  
2. `mount --mkdir /dev/sda1 /mnt/boot`  
3. `swapon /dev/sda2`

---

## 2. **STEP 2**

### 2.1
1. `reflector --latest 20 --protocol https --sort rate --save /etc/pacman.d/mirrorlist`  
2. `head /etc/pacman.d/mirrorlist`  
   - Before, I kept getting a "bad mirror" message from pacstrap. But I found that regenerating the list would help with that issue.  

### 2.2
1. `pacstrap -K /mnt base linux linux-firmware`  
   - The -K area makes it so that it automatically copies the pacman keyring into the new root.  

---

## 3. **STEP 3**

### 3.1) Generating the fstab
1. `genfstab -U /mnt >> /mnt/etc/fstab`  
   - Had to verify the output using `cat /mnt/etc/fstab` to make sure the partitions appeared.  

### 3.2) Chroot into System
1. `arch-chroot /mnt`  
   - Helps boot into the new Arch system instead of the ISO.  

### 3.3) Time Config
1. `ln -sf /usr/share/zoneinfo/_Region_/_City_ /etc/localtime`  
2. `hwclock --systohc`  

### 3.4) Localization # Configures the system language and keyboard layout
1. `locale-gen`  
2. `echo "LANG=en_US.UTF-8" > /etc/locale.conf`  
3. `echo "KEYMAP=us"`  
4. `/etc/vconsole.conf`  

### 3.5) Setting Hostname and Networking # Hostname and setting the default service for handling the Wi-Fi aspect
1. `echo "kam-arch-vm" > /etc/hostname`  
2. `pacman -S networkmanager`  
3. `systemctl enable NetworkManager`  

### 3.6) Initramfs
- `mkinitcpio -P`

### 3.7) Setting Root Password
- `passwd` (kamdon46)

### 3.8) Bootloader: bootctl
1. Install Nano:  
   - `pacman -Sy nano`  
2. `bootctl install`  
   - `nano /boot/loader/loader.conf`  
     - `default arch # Tells the system to boot the entry Arch`  
     - `timeout 3 # Bootloader waits 3 seconds`  
     - `editor yes # Allows me to edit boot options at the boot screen`  
3. `nano /boot/loader/entries/arch.conf`  
   - `title   Arch Linux`  
   - `linux   /vmlinuz-linux # Points to the kernel image to boot`  
   - `initrd  /initramfs-linux.img # Specifies the initial RAM disk`  
   - `initrd  /initramfs-linux-fallback.img # Fallback image if the first fails`  
   - `options root=UUID=aa1252ba-a0b8-470c-9b7b-8542658027cd rw # Kernel command line`  
4. `bootctl list`

### 3.9) Exit and Reboot (aka Pray, Holy)
1. `umount -R /mnt`  
2. `reboot`

DONE!

---

## Arch Config

### 1) Adding a User and Sudo Permissions
- `useradd -m -G wheel -s /bin/bash evilkamdon`  
- `passwd evilkamdon #(kamdon46)`  
- `pacman -S nano sudo`  
- `EDITOR=nano visudo`  
- Remove the hashtag from `%wheel ALL=(ALL:ALL) ALL` to give sudo permissions to the user  
- `su -evilkamdon`  
- `sudo ls /root`

### 2) Installing a DE (LXQt)
- `sudo pacman -Syu`  
- `sudo pacman -S xorg sddm lxqt breeze-icons`  
- `sudo pacman -Syy`  
- `sudo pacman -S network-manager-applet brightnessctl pipewire pipewire-pulse pavucontrol`  
- `sudo systemctl enable NetworkManager`  
- `sudo systemctl start NetworkManager`  
- `sudo systemctl enable sddm`  
- `sudo reboot`

### 3) Install a Browser
Using yay because it will automatically handle the dependencies and PKGBUILDs.  
- `sudo pacman -S git base-devel`  
- `git clone https://aur.archlinux.org/yay.git`  
- `cd yay`  
- `makepkg -si`  
- `cd ~`  
- `yay -S google-chrome`  
- `google-chrome-stable &`

### 4) Customizations
- `yay -S fortune-mod cowsay spotify discord # Fun startup quotes, applications`  
- `yay -S macos-icon-theme capitaine-cursors whitesur-gtk-theme sf-pro-font # Mac-style theme`

### 5) SSH
- `sudo pacman -S openssh`  
- `sudo systemctl enable --now sshd`  
  - Initially received "preset: disabled", so I fixed it with the now flag.  

### 6) Shell and Aliases (Zsh)
- `yay -S zsh oh-my-zsh-git`  
- `chsh -s /bin/zsh`  
- `alias ls='ls --color=auto'`  
- `alias ll='ls -la'`  
- `alias update='yay -Syu'`  
- `neofetch && fortune || cowsay lolcat`  

I encountered an error sourcing `.zshrc`, which stated "autoload command not found". This was because I was still in Bash, not Zsh. I had to reinstall it fully again, then log back in and out, and reboot the system.
