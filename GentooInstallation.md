1. [Installing](#Installing)
	1. [Preparing drives](#Drives)
	2. [Installing stage and base configuration](#Stage-and-base)
	3. [Chroot](#Chroot)
	4. [Preparation for bootloader](#Boot-preporation)
	5. [Portage update](#Portage-update)
	6. [Choosing profile](#Profile)
	7. [Add C flags](#C)
	8. [Update world](#World)
	9. [Configure timezone](#Timezone)
	10. [Configure locales](#Locales)
	11. [Kernel](#Dist-kernel)
	12. [Firmware and microcode](#Firmware&Microcode)
	14. [Writing fstab](#Fstab)
	15. [Network](#Network)
		1. [Hostname](#Hostname)
		2. [IP	](#IP)
	16. [System configuration](#System-configuration)
		1. [Root password](#Root-pass)
		2. [Init and boot configuration ( systemd only )](#Init&Boot)
	17. [Tools](#Tools)
		1. [Loggin daemon ( openRC only )](#Loggind)
		2. [Cron daemon ( openRC only )](#Crond)
		3. [File indexing](#File-index)
		4. [SSH](#SSHd)
		5. [Time sync](#Time-sync)
		6. [Filesystems](#File-system)
		7. [Networking tools](#Network-tools)
			1. [Ethernet](#Ethernet)
			2. [Wi-Fi](#Wi-Fi)
		8. [GRUB](#Bootloader)
	18. [Preparing for DE/WM ( openRC only )](#de/wm)
	19. [Usefull packages](#Usefull-packages)
	20. [New user](#New-user)
	21. [Finish](#Reboot)
2. [Post install](#Post-install)
	1. [Disable root login](#Disable-root)
	2. [Cleanup](#Cleanup)
	3. [Other software](#Other-software)
	4. [Bluetooth](#Bluetooth)
	5. [Fonts](#Fonts)
	6. [Guru overlay](#Guru)
	7. [Neovim](#Neovim)
	8. [Flatpak](#Flatpak)
	9. [Browsers](#Browsers)
	    1. [Brave](#Brave)
		2. [Zen](#Zen)
		3. [Qutebrowser](#Qutebrowser)
		4. [Tor](#Tor)
	10. [Zswap](#Zswap)
	11. [Zsh](#Zsh)
	12. [Qemu/KVM](#Qemu/KVM)
	13. [Steam](#Steam)
	14. [Discord](#Discord)
	15. [Lutris](#Lutris)
	16. [Bottles](#Bottles)
	17. [Power managment](#Power)
# Installing
## Drives
change partions with cfdisk utility.
```
cfdisk /dev/{device}
```
My volumes:
    1. /
    2. /boot
    3. /boot/efi
    4. swap
Formating root
```
mkfs.btrfs /dev/nvme0n1p
```
Formating boot
```
mkfs.btrfs /dev/nvme0n1p
```
Formating /efi
```
mkfs.fat -F 32 /dev/nvme0n1p
```
Formating swap
```
mkswap /dev/nvme0n1p && swapon /dev/nvme0n1p
```
Mounting root
```
mount /dev/nvme0n1p /mnt/gentoo
```
Cd into mounted folder and sync time
```
cd /mnt/gentoo && chronyd -q
```
## Stage-and-base
Open terminal browser and download stage3 file from ru mirror
```
links https://www.gentoo.org/downloads/mirrors/ && tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner
```
Change make.conf like mine. Need to set:
    1. ACCEPT_LICENSE
    2. VIDEO_CARD
    3. USE
    4. MAKEOPTS
    5. -march flag

```
vim /mnt/gentoo/etc/portage/make.conf
```
[My make.conf](dotfiles/portage/make.conf)


## Chroot
Copying resolv.conf for internet work
```
cp --dereference /etc/resolv.conf /mnt/gentoo/etc/
```
Mounting the necessary filesystems
```
mount --types proc /proc /mnt/gentoo/proc &&
mount --rbind /sys /mnt/gentoo/sys &&
mount --make-rslave /mnt/gentoo/sys &&
mount --rbind /dev /mnt/gentoo/dev &&
mount --make-rslave /mnt/gentoo/dev &&
mount --bind /run /mnt/gentoo/run &&
mount --make-slave /mnt/gentoo/run
```
Entering the new environment
```
chroot /mnt/gentoo /bin/bash
```
```
source /etc/profile &&
export PS1="(chroot) ${PS1}"
```
## Boot-preporation
Mounting boot partion
```
mount /dev/nvme0n1p /boot --mkdir 
```
Mounting efi partion
```
mount /dev/nvme0n1p /boot/efi --mkdir 
```
## Portage-update
```
emerge-webrsync
```
>[!note]
> Mirror select need for select best mirrors for gentoo repos
> It's not need if you alredy define it in make.conf
#### Choose mirrors
```
emerge --ask --verbose --oneshot app-portage/mirrorselect && mirrorselect -i -o >> /etc/portage/make.conf
```
### Updating the Gentoo ebuild repository
```
emerge --sync --quiet
```
## Profile
```
eselect profile list | less
```
```
eselect profile set 28
```
### C
```
emerge --oneshot app-portage/cpuid2cpuflags && echo "*/* $(cpuid2cpuflags)" > /etc/portage/package.use/00cpu-flags
```
    
## World

```
emerge --ask --verbose --update --deep --newuse @world
```
```
emerge --ask --depclean
```
## Timezone
> [!note]
> For systemd
> ``` ln -sf ../usr/share/zoneinfo/Europe/Moscow /etc/localtime ```

For OpenRC 
```
echo "Europe/Moscow" > /etc/timezone
```
```
emerge --config sys-libs/timezone-data
```
## Locales
```
echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen && echo "ru_RU.UTF-8 UTF-8" >> /etc/locale.gen
```
```
locale-gen
```
```
eselect locale list
```
```
eselect locale set 4
```
```
env-update && source /etc/profile && export PS1="(chroot) ${PS1}"
```
>[!note]
> I install kernel before installing firmware and microcode
> because i set `dist-kernel` flag in global USE flags and linux-firmware or intel-microcode
> require installed dist-kernel
## Distribution kernel
```
echo "sys-kernel/installkernel dracut" >>  /etc/portage/package.use/installkernel
```
I prefer binnary package cause of i dont change any settings in kernel and compile time not necessary
```
emerge --ask sys-kernel/gentoo-kernel-bin
```
## Firmware&Microcode
```
emerge --ask sys-kernel/linux-firmware sys-firmware/intel-microcode sys-firmware/sof-firmware
```
### Dist-kernel
This is not neccesary if you install only one kernel

Just check which kernel you use
```
eselect kernel list
```
```
eselect kernel set 1
```
## Fstab
```
blkid
```
```
vim /etc/fstab
```
write by [example](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/System)

I use this fstab
```
/dev/nvme0n1p1   /efi        vfat    umask=0077     0 2
/dev/nvme0n1p2   none         swap    sw                   0 0
/dev/nvme0n1p3   /            btrfs   subvol=@,noatime,defaults,autodefrag,compress-force=zstd:1              0 1
/dev/nvme0n1p3   /home            btrfs   subvol=@home,noatime,defaults,autodefrag,compress-force=zstd:1              0 2
/dev/nvme0n1p3   /snapshots            btrfs   subvol=@snapshots,noatime,defaults,autodefrag,compress-force=zstd:8              0 2
/dev/nvme0n1p4    /windows        vfat    umask=0077    0    2
```
## Network

### Hostname
>[!note]
> for systemd use(but nothing change)
> ``` hostnamectl hostname NAME ```
```
echo NAME > /etc/hostname
```

### IP
```
emerge --ask net-misc/dhcpcd
```
>[!note]
> systemd
> `systemctl enable dhcpcd`
```
rc-update add dhcpcd default
```
```
rc-service dhcpcd start
```
## System-configuration
### Root-pass
```
passwd
```
### Init&Boot
This one, for systemd, openrc dont need
```
systemd-machine-id-setup
```
```
systemd-firstboot --prompt
```
```
systemctl preset-all --preset-mode=enable-only
```
### [Other configuration](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/System)
## Tools
### Loggind
> [!note]
> for systemd not needed
```
emerge --ask app-admin/sysklogd
```
```
rc-update add sysklogd default
```
### Crond
> [!note]
> for systemd not needed
```
emerge --ask sys-process/cronie
```
```
rc-update add cronie default
```
### File-index
```
emerge --ask sys-apps/mlocate
```
### SSHd
> [!note]
> for systemd use
> ``` systemctl enable sshd ```
```
rc-update add sshd default
```
### Time-sync
> [!note]
> systemd
> ``` systemctl enable systemd-timesyncd.service ```
```
emerge --ask net-misc/chrony
```
```
rc-update add chronyd default
```
### File-system
```
emerge --ask sys-block/io-scheduler-udev-rules sys-fs/xfsprogs 	sys-fs/e2fsprogs 	sys-fs/dosfstools 	sys-fs/btrfs-progs 	sys-fs/zfs 	sys-fs/jfsutils ntfs3g
```
### Network-tools
#### Ethernet
```
emerge --ask net-dialup/ppp
```
#### Wi-Fi
```
echo "net-wireless/wpa_supplicant tkip" >> /etc/portage/package.use/wpa_supplicant
```
```
emerge --ask net-wireless/iw net-wireless/wpa_supplicant
```
## Bootloader
### GRUB
```
echo "sys-kernel/installkernel grub" >> /etc/portage/package.use/installkernel
```
```
emerge --ask sys-kernel/installkernel
```
Use this option if you want to use EFI
```
echo 'GRUB_PLATFORMS="efi-64"' >> /etc/portage/make.conf
```
```
emerge --ask --verbose sys-boot/grub
```
> [!note]
> for bios
> ``` grub-install --target=i386-pc /dev/sda ```
```
grub-install --target=x86_64-efi --efi-directory=/efi --removable
```
```
grub-mkconfig -o /boot/grub/grub.cfg
```
## de/wm
If using open-rc than do this steps
```
emerge elogind
```
```
rc-update add elogind boot
```
```
gpasswd -a root video
```
## Usefull-packages
```
emerge sudo udev pciutils usbutils cfg-update gentoolkit eselect-repository
```
# New-user
```
useradd -m -G wheel,audio,video,usb -s /bin/bash me
```
```
passwd me
```
```
vim /etc/sudoers
```
## Reboot
```
exit
```
```
cd && umount -l /mnt/gentoo/dev{/shm,/pts,} &&  umount -R /mnt/gentoo && reboot
```
Load your new system
# Post-install

## Disable-root
```
sudo passwd -dl root
```
## Cleanup
```
sudo rm /stage3-*.tar.*
```
```
sudo emerge --deepclean
```

## Other-software

```
sudo emerge --ask --quiet --verbose --tree kitty btop dolphin qbittorrent ark vlc gparted
```
## Bluetooth
Set global USE flag "bluetooth" in make.conf
```
emerge --ask --noreplace net-wireless/bluez
```
> [!note]
> for systemd
> ``` sudo systemctl enable bluetooth && sudo systemctl start bluetooth ```
```
rc-service bluetooth start
```
```
rc-update add bluetooth default
```
## Fonts
```
sudo cp ./dotfiles/fonts ~/.fonts && sudo mv ./dotfiles/fonts ~/.local/share/
```
## Guru
```
emerge --ask  app-eselect/eselect-repository --newuse && eselect repository enable guru && emerge --sync guru
```
## Neovim

dependencies
```
sudo emerge --ask app-emacs/rg dev-libs/tree-sitter dev-lua/luarocks dev-python/pynvim dev-python/tree-sitter media-fonts/fontawesome media-fonts/noto-emoji sys-apps/fd sys-apps/ripgrep neovim
```
```
sudo mv -r ./dotfiles/software/dots/soft/nvim ~/.config/
```

## Flatpak

```
emerge --ask sys-apps/flatpak
```

Flatseal let you change application permissions
```
sudo flatpak install com.github.tchx84.Flatseal
```


## Browsers

### Brave
```
sudo eselect repository add brave-overlay git https://gitlab.com/jason.oliveira/brave-overlay.git
```
```
sudo emerge --sync brave-overlay
```
```
sudo emerge --ask www-client/brave-bin::brave-overlay
```

### Zen browser
Install [flatpak](#Flatpak)
download Zen flatpak realese and install https://zen-browser.app/download

### Qutebrowser
```
sudo echo "www-client/qutebrowser adblock" >> /etc/portage/package.use/qutebrowser
```
```
sudo emerge --ask www-client/qutebrowser
```

### Tor
```
sudo eselect repository enable torbrowser
```
```
sudo emerge --sync torbrowser
```
```
sudo emerge --ask www-client/torbrowser-launcher
```
## Zswap

Write this line in /etc/default/grub
```
GRUB_CMDLINE_LINUX="zswap.enabled=1 zswap.compressor=lz4"
```

## Zsh

```
sudo emerge --ask app-shells/zsh
```
```
sudo emerge --ask app-shells/zsh-completions app-shells/gentoo-zsh-completions 
```
```
sudo mv ./dotfiles/software/dots/shells/zsh ~/
```
## Qemu/KVM with virt-manager

Set use flags for qemu and rebuild it
```
sudo echo "app-emulation/qemu opengl alsa gtk keyutils ncurses pipewire plugins spice udev usb usbredir virgl vte zstd" >> /etc/portage/package.use/qemu && sudo emerge qemu
```
Set use flags for libvirt and rebuild it
```
sudo echo "app-emulation/libvirt udev qemu virt-network nfs nbd parted policykit pcap numa fuse macvtap vepa" >> /etc/portage/package.use/libvirt && sudo emerge libvirt
```
Set use flags for virt-manager and rebuild it
```
sudo echo "app-emulation/virt-manager gui policykit" >> /etc/portage/package.use/virt-manager && sudo emerge virt-manager
```
```
gpasswd -a me kvm && gpasswd -a me libvirt
```
> [!note]
> for systemd 
> ```sudo systemctl enable libvirtd && sudo systemctl start libvirtd ```
```
rc-update add libvirtd default
```
```
mkdir -p /etc/polkit-l/localauthority/50-local.d
```
```
echo "[Allow group libvirt management permissions]
Identity=unix-group:libvirt
Action=org.libvirt.unix.manage
ResultAny=yes
ResultInactive=yes
ResultActive=yes" >> /etc/polkit-l/localauthority/50-local.d/org.libvirt.unix.manage.pkla
```
```
reboot
```
## Steam

```
sudo emerge --ask games-util/game-device-udev-rules
```
```
sudo flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo && sudo flatpak install flathub com.valvesoftware.Steam
```
```
sudo flatpak run com.valvesoftware.Steam
```
## Discord
```
sudo flatpak install flathub com.discordapp.Discord
```
## Lutris
```
emerge --ask games-util/lutris
```
```
echo "#Lutris multilib dependencies
media-libs/vulkan-loader abi_x86_32
media-libs/vulkan-layers abi_x86_32
media-libs/freetype abi_x86_32
media-libs/libpng abi_x86_32
net-libs/gnutls abi_x86_32
media-libs/libsdl2 abi_x86_32" >> /etc/portage/package.use/lutris
```
```
emerge --ask --changed-use --deep @world
```

## Bottles
```
flatpak install flathub com.usebottles.bottles
```

## Power managment
```
sudo emerge --ask sys-power/thermald cpupower
```
```
systemctl enable thermald
```
```
emerge --ask sys-power/tlp
```
```
systemctl enable --now tlp
```
```
sudo eselect repository enable gentoo-zh && sudo emerge --sync gentoo-zh
```
```
sudo emerge tlpui
```
For better usage i preffer run some scripts which use cpupower
```
sudo mv ./dotfiles/software/dots/scripts ~/ 
```
