# Installing Gentoo plan

Use [genkernel](https://wiki.gentoo.org/wiki/Genkernel) instead of distribution kernel and highly customize it

[efibootmgr]() instead of grub

[wayland](https://wiki.gentoo.org/wiki/Wayland) with [hyprland](https://wiki.gentoo.org/wiki/Hyprland)

https://www.reddit.com/r/Gentoo/comments/150r74m/guide_hyprland_nvidia_extremely_minimal_gentoo/

power modes: [cpu](https://github.com/AdnanHodzic/auto-cpufreq) + [tlp for battery](https://wiki.gentoo.org/wiki/Power_management/Guide)

[zram](https://wiki.gentoo.org/wiki/Zram) instead of swap

for configuring buttons using [xev](https://packages.gentoo.org/packages/x11-apps/xev), [forum](https://forums.gentoo.org/viewtopic-p-6909782.html)

 [Installing with Handbook](https://wiki.gentoo.org/wiki/Handbook:AMD64)

## Packages
emerge --ask --quiet --verbose git sudo usbutils dosfstools pciutils gentoolkit

## Useradd

    useradd -m username
    usermod -aG wheel,audio,video
    emerge sudo
    nvim /etc/sudoers
    Uncomment line
## Config

https://wiki.gentoo.org/wiki/Cfg-update

## GURU

https://wiki.gentoo.org/wiki/Eselect/Repository

    eselect repository enable guru

## Zsh

https://wiki.gentoo.org/wiki/Zsh

### oh-my-zsh

https://github.com/ohmyzsh/ohmyzsh

Plugins

https://github.com/zsh-users/zsh-autosuggestions/blob/master/INSTALL.md

https://github.com/zsh-users/zsh-syntax-highlighting/blob/master/INSTALL.md

https://github.com/romkatv/powerlevel10k

## i3wm

First install xorg sever

    sudo emerge --ask --quiet --verbose xorg-server

Now try to start with
 
    startx

If it start install i3 and dependencies

    sudo emerge --ask --quiet --verbose i3 i3status i3lock xterm dmenu

And try to start i3 with

    i3

## Display-manager

https://wiki.gentoo.org/wiki/Display_manager

https://wiki.gentoo.org/wiki/SDDM

## Wifi

https://wiki.gentoo.org/wiki/Wi-Fi

### wpa_supplicant

https://wiki.gentoo.org/wiki/Wpa_supplicant

### Connecting

https://wiki.gentoo.org/wiki/Handbook:AMD64/Networking/Wireless

## Use-flag-issue

https://forums.gentoo.org/viewtopic-t-1089708-start-0.html

## fonts

https://forums.gentoo.org/viewtopic-p-7048980.html

https://forums.gentoo.org/viewtopic-t-1114772-start-0.html

## vmbox

https://wiki.gentoo.org/wiki/VirtualBox