# ArchLinux Project Documentation 
# kamdonsmarshall.github.io
- Downloaded ISO from HTTP from MIT mirror site
- Validated the checksum using 7Zip and compared the two SHA256 values
- Config:
	- Installed VM on VMWare Workstation, created Guest Operating System using ISO (Linux, Version- Other Linux 6.x kernel 64-bit)
	- Allocated 2GB of RAM and 20GB of HDD Storage
	
1) **STEP 1:**
	1) 1.5) Set the keyboard layout using 
		1) `localectl list-keymaps`
		2) `loadkeys us`
	2) 1.5) Set font using
		1) `set font ter-132b` 
	3) 1.6) Verify the boot mode
		1) `cat /sys/firmware/efi/fw_platform_size`
		2) Output: No such file or directory. So, that means my system is booted in BIOS mode
		3) Second time around: Changed vmx file's second line to "firmware = "efi", reloaded my VM, tried boot again, 64 was the output (UEFI now)
	4) 1.7) Connect to internet
		1) `ip link` 
		2) Connect to network:
			1) `systemctl start systemd-networked` # enables the network management
			2) `system start systemd-resolved` # enables the dns resolution 
			3) `ip addr show ens33`
			4) `ping ping.archlinux.org` & recieved 9 packets (good)
		3) I had trouble getting the network to load as I kept getting "<NO-RECIEVER>" errors on the ens33 section which means that my virtual network was detected but wouldn't connect. Restarted multiple times, checked VM settings "Network Adapter" for NAT connectivity, verified it, and then rebooted to get the ping to show.
	5) 1.8) Update the system clock
		1) `timedatectl` 
		2) `timedatectl list-timezones`
		3) `timedatectl set-timezone America/Chicago`
	6) 1.9: Partition the disks
		1) `fdisk -l`
		2) `fdisk /dev/sda`
			1) g (creates a new gpt partition table)
			2) creating efi system partition
				1) n, 1, Enter, +1G, t, 1
			3) swap partition 
				1) n, 2, enter, +4G, t, 2, 19
			4) root partition
				1) n, 3, enter, enter, t, 3, 23
			5) w (write & exit)
			6) `fdisk -l /dev/sda`
	7) 1.10: Format the partitions
		1) EFI partition: 
			1) `mkfs.fat -F 32 /dev/sda1`
		2) Swap: 
			1) `mkswap /dev/sda2`  # initalizes the swap
		3) Root
			1) `mkfs.ext4 /dev/sda3`
	8) 1.11: Mount
		1) `mount /dev/sda3 /mnt`
		2) `mount --mkdir /dev/sda1 /mnt/boot`
		3) `swapon /dev/sda2`
**2) STEP 2**
	9) 2.1
		1) `reflector --latest 20 --protocol https --sort rate --save /etc/pacman.d/mirrorlist`
		2) `head /etc/pacman.d/mirrorlist`
			1) Before, I kept getting a "bad mirror" message from pacstrap. But I found that regenerating the list would help that issue
	10) 2.2
		1) `pacstrap -K /mnt base linux linux-firmware`
			1) the -K area makes it so that it automatically copies the pacman keyring into the new root
**3) STEP 3**
	11) 3.1: Generating the fstab) `genfstab -U /mnt >> /mnt/etc/fstab` 
		1) had to verify the output using cat /mnt/etc/fstab to make sure the partitions appeared
	12) 3.2 :Chroot into System) `arch-chroot /mnt` 
		1) helps boot into the new arch system instead of the iso
	13) 3.3: Time Config)
		1) ``ln -sf /usr/share/zoneinfo/_Region_/_City_ /etc/localtime`
		2) `hwclock --systohc`
	14) 3.4: Localization) # configures the sys language and keyboard layout
		1) `locale-gen`
		2) `echo "LANG=en_US.UTF-8" > /etc/locale.conf`
		3) `echo "KEYMAP=us"`
		4) `/etc/vconsole.conf`
	15) 3.5: Setting hostname and networking) #host name and setting the default service for handling the wifi aspect
		1) `echo "kam-arch-vm" > /etc/hostname`
		2) `pacman -S networkmanager`
		3) `systemctl enable NetworkManager`
	16) 3.6: Initramfs)  `mkinitcpio -P`
	17) 3.7: Setting root passwd)  `passwd` (kamdon46)
	18) 3.8 Bootloader: bootctl)
		1) install nano: `pacman -Sy nano`
		2) `bootctl install`
		3) `nano /boot/loader/loader.conf`
			1) `default arch # tells the system to boot the  entry arch` 
				`timeout 3 # bootloader waits 3 secs`
				`editor yes #allows me to edit boot options at the boot screen`
		4) nano /boot/loader/entries/arch.conf
			1) `title   Arch Linux`
				`linux   /vmlinuz-linux #points to the kernal image to boot`
				`initrd  /initramfs-linux.img #specifcies the initial ram disk`
				`initrd  /initramfs-linux-fallback.img # fallback image if the first fails`
				`options root=UUID=aa1252ba-a0b8-470c-9b7b-8542658027cd rw #kernel command line`
		5) `bootctl list`
	19) 3.9: exit and reboot (aka pray, holy))
		1) `umount -R /mnt`
		2) `reboot`
DONE!

Arch Config
1) adding a user and sudo perms)
	`useradd -m -G wheel -s /bin/bash evilkamdon`
	`passwd evilkamdon #(kamdon46)`
	`pacman -S nano sudo (installing nano and sudo editor tools)`
	`EDITOR=nano visudo`
	`# remove the hashtag from %wheel ALL=(ALL:ALL) ALL to give sudo perms to user`
	`su -evilkamdon`
	`sudo ls /root`
2) installing a DE (LXQt)
	`sudo pacman -Syu`
	`sudo pacman -S xorg sddm lxqt breeze-icons`
	`sudo pacman -Syy`
	`sudo pacman -S network-manager-applet brightnessctl pipewire pipewire-pulse pavucontrol`
	`sudo systemctl enable NetworkManager`
	`sudo systemctl start NetworkManager`
	`sudo systemctl enable sddm`
	`sudo reboot`

3) install a browser
using yay because it will automatically handle the dependencies and PKGBUILDs.
	`sudo pacman -S git base-devel`
	`git clone https://aur.archlinux.org/yay.git`
	`cd yay`
	`makepkg -si`
	`cd ~`
	`yay -S google-chrome`
	`google-chrome-stable &`
4) Customizations
	`yay -S fortune mod cowsay spotify discord # fun startup quotes, applications`
	`yay -S macos-icon-theme capitaine-cursors whitesur-gtk-theme sf-pro-font # mac-style theme`
5) SSH
	`sudo pacman -S openssh`
	`sudo systemctl enable --now sshd # initally recieved "preset: disabled", so i fixed it with the now flag`
6) Shell and Aliases (Zsh)
	`yay -S zsh oh-my-zsh-git`
	`chsh -s /bin/zsh`
	`alias ls='ls --color=auto'`
	`alias ll='ls -la'`
	`alias update='yay -Syu'`
	`neofetch && fortune || cowsay lolcat`
	i had an error sourcing .zshrc saying "autoload command not found", which was because i was still in bash and not zsh. i just had to install it fully again, and then log back in and out, rebooting the system
