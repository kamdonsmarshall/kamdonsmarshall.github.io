# kamdonsmarshall.github.io
Downloaded ISO from HTTP from MIT mirror site
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
		1) cat /sys/firmware/efi/fw_platform_size
		%%2) Output: No such file or directory. So, that means my system is booted in BIOS mode %%
		%% 3) Second time around: Changed vmx file's second line to "firmware = "efi", reloaded my VM, tried boot again, 64 was the output (UEFI now) %%
	4) 1.7) Connect to internet
		1) ip link 
		2) Connect to network:
			1) systemctl start systemd-networked # enables the network management
			2) system start systemd-resolved # enables the dns resolution 
			3) ip addr show ens33
			4) ping ping.archlinux.org & recieved 9 packets (good)
		3) # Initially had trouble getting the network to load as I kept getting "<NO-RECIEVER>" errors on the ens33 section which means that my virtual network was detected but wouldn't connect. Restarted multiple times, checked VM settings "Network Adapter" for NAT connectivity, verified it, and then rebooted to get the ping to show.
	5) 1.8) Update the system clock
		1) timedatectl 
		2) timedatectl list-timezones
		3) timedatectl set-timezone America/Chicago
	6) 1.9: Partition the disks
		1) fdisk -l
		2) fdisk /dev/sda
			1) g (creates a new gpt partition table)
			2) creating efi system partition
				1) n, 1, Enter, +1G, t, 1
			3) swap partition 
				1) n, 2, enter, +4G, t, 2, 19
			4) root partition
				1) n, 3, enter, enter, t, 3, 23
			5) w (write & exit)
			6) fdisk -l /dev/sda
	7) 1.10: Format the partitions
		1) EFI partition: 
			1) mkfs.fat -F 32 /dev/sda1
		2) Swap: 
			1) mkswap /dev/sda2  # initalizes the swap
		3) Root
			1) mkfs.ext4 /dev/sda3
	8) 1.11: Mount
		1) mount /dev/sda3 /mnt
		2) mount --mkdir /dev/sda1 /mnt/boot
		3) swapon /dev/sda2
2) STEP 2
	1) 2.1
		1) reflector --latest 20 --protocol https --sort rate --save /etc/pacman.d/mirrorlist
		2) head /etc/pacman.d/mirrorlist
			1) # Before, I kept getting a "bad mirror" message from pacstrap. But I found that regenerating the list would help that issue
	2) 2.2
		1) pacstrap -K /mnt base linux linux-firmware
			1) # the -K area makes it so that it automatically copies the pacman keyring into the new root
3) STEP 3
	1) 3.1: Generating the fstab) genfstab -U /mnt >> /mnt/etc/fstab 
		1) #had to verify the output using cat /mnt/etc/fstab to make sure the partitions appeared
	2) 3.2 :Chroot into System) arch-chroot /mnt 
		1) #helps boot into the new arch system instead of the iso
	3) 3.3: Time Config)
		1) ln -sf /usr/share/zoneinfo/_Region_/_City_ /etc/localtime
		2) hwclock --systohc
	4) 3.4: Localization) # configures the sys language and keyboard layout
		1) locale-gen
		2) echo "LANG=en_US.UTF-8" > /etc/locale.conf
		3) echo "KEYMAP=us"
		4) /etc/vconsole.conf
	5) 3.5: Setting hostname and networking) #host name and setting the default service for handling the wifi aspect
		1) echo "kam-arch-vm" > /etc/hostname
		2) pacman -S networkmanager
		3) systemctl enable NetworkManager
	6) 3.6: Initramfs)  mkinitcpio -P
	7) 3.7: Setting root passwd)  passwd (kamdon46)
	8) 3.8 Bootloader: bootctl)
		1) install nano: pacman -Sy nano
		2) bootctl install
		3) nano /boot/loader/loader.conf
			1) default arch # tells the system to boot the entry arch 
				timeout 3 # bootloader waits 3 secs
				editor yes #allows me to edit boot options at the boot scree 
		4) nano /boot/loader/entries/arch.conf
			1) title   Arch Linux
				linux   /vmlinuz-linux #points to the kernal image to boot
				initrd  /initramfs-linux.img #specifcies the initial ram disk
				initrd  /initramfs-linux-fallback.img # fallback image if the first fails
				options root=UUID=aa1252ba-a0b8-470c-9b7b-8542658027cd rw #kernel command line
		5) bootctl list
	9) 3.9: exit and reboot (aka pray, holy))
		1) umount -R /mnt
		2) reboot
DONE!

Arch Config
1) adding a user and sudo perms)
	1) useradd -m -G wheel -s /bin/bash evilkamdon
	2) passwd evilkamdon (kamdon46)
	3) pacman -S nano sudo (installing nano and sudo editor tools)
	4) EDITOR=nano visudo
	5) remove the # from %wheel ALL=(ALL:ALL) ALL to give sudo perms to user
	6) su -evilkamdon
	7) sudo ls /root
2) installing a DE (LXQt)
	1) sudo pacman -Syu
	2) sudo pacman -S xorg sddm lxqt breeze-icons
	3) sudo pacman -Syy
	4) sudo pacman -S network-manager-applet brightnessctl pipewire pipewire-pulse pavucontrol
	5) sudo systemctl enable NetworkManager
	6) sudo systemctl start NetworkManager
	7) sudo systemctl enable sddm
	8) sudo reboot

3) install a browser
# using yay because it will automatically handle the dependencies and PKGBUILDs.
	1) sudo pacman -S git base-devel
	2) git clone https://aur.archlinux.org/yay.git
	3) cd yay
	4) makepkg -si
	5) cd ~
	6) yay -S google-chrome
	7) google-chrome-stable &
4) Customizations
	1) yay -S fortune mod cowsay spotify discord # fun startup quotes, applications
	2) yay -S macos-icon-theme capitaine-cursors whitesur-gtk-theme sf-pro-font # mac-style theme
5) SSH
	1) sudo pacman -S openssh
	2) sudo systemctl enable --now sshd # initally recieved "preset: disabled", so i fixed it with the now flag
6) Shell and Aliases (Zsh)
	1) yay -S zsh oh-my-zsh-git
	2) chsh -s /bin/zsh
	3) alias ls='ls --color=auto'
	4) alias ll='ls -la'
	5) alias update='yay -Syu'
	6) neofetch && fortune || cowsay lolcat
	7) # i had an error sourcing .zshrc saying "autoload command not found", which was because i was still in bash and not zsh. i just had to install it fully again, and then log back in and out, rebooting the system











