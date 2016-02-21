Cleaner guide

Prerequirements:
 
    Arch iso
    USB drive or CD
    UEFI hardware
    Keyboard, mouse, eyes
 
First, you need the Arch Linux iso. The same iso file supports both 32 bit and 64 bit installations, so don't worry about it not being the right version. I recommend torrenting it because of the kagillion seeders.
 
Once you've downloaded it, you need to burn it. Do not use Unetbootin for this - it doesn't show you how to do it the right way, and it doesn't actually work with Arch from the iso.
 
Warning - will delete all data on the drive. Please backup if necessary. Formatting is serious shit.
 
If you're on Linux, then you have a really easy job. Simply open the terminal emulator of your choice, sudo su into root perm, and type:
 
    dd if=/path/to/iso of=/dev/sdX;
 
Where the /dev/sdX is your USB drive or location of your CD drive. If you're on Windows, I recommend Win32 Disk Imager. Select the slowest burn option, select your iso file, your USB drive you wish to format, and you're good to go. It's all GUI, so should be pretty easy to figure out. Once you've burned your livedisk, reboot the machine, but boot directly into UEFI (usually F2, F11, or Del to force it). Every BIOS is different based on manufacturer and version, but boot into whatever medium you burnt your iso to in UEFI mode. It must not boot into standard BIOS mode, it must be UEFI boot. There are several options once the livedisk is loaded - default, UEFI shell 1, UEFI shell 2, etc. Just leave it alone and boot into the standard/default selected option. You can boot into UEFI shells 1 or 2, but they pretty much, for all intensive purposes, achieve the same thing.  Once you've booted into UEFI, it's time to get to business. You've been automatically logged into the root account, so don't worry about that. First of all, we're going to need to format and partition some disks to install your Arch onto. In my case, my SSD I used as my boot drive is /dev/sdb, so I will use that in all commands. To find out your disks names, simply issue the command:
 
    fdisk -l
 
 Once you've found the disk(s) you wish to install your system onto, we can proceed to partition them accordingly and format. In my case, I have a 120GB SSD (my /dev/sdb) and a 2TB HDD (my /dev/sda). I use a partition on my 2TB HDD for storage, but keep my /root and /home partitions on my SSD as a boot drive. You can piece your partitions up however you want, but I like to keep things clean, so they can work off of one disk if need be. To partition your drive(s) for an Arch installation, you are best off with simply starting with a clean disk, so let's delete all of the existing partitions, setup our own ones, and write to the disk.Warning, this WILL DELETE ALL DATA ON YOUR DISK(S) / PARTITIONS THAT YOU USE.  To delete the existing partitions, simply issue the command:
 
    gdisk /dev/sdb
    d
 
 Assuming /dev/sdb is your boot drive, simply repeat the d operation with all partitions (starting with 1, increasing onward) until your disk is empty. Now, we need to write new partitions. The most important step of an Arch installation on UEFI is the EFI boot partition. I am going to go ahead and put my /root and /home partitions on here as well.
 
    gdisk /dev/sdb
    N
    First Sector - Just press enter, so it's at the front of your disk
    Last sector - +1024M
    Type - ef00
    Now, to make the root partition
    N
    First sector - Enter
    Last sector - +30000M
    Type - Enter
    And, the last on this disk, the /home partition
    N
    First sector - Enter
    Last sector - Enter
    Type - Enter
    Now, if you are ready to delete your partition table and write the new one, just enter the command:
    w
    Press enter, and wait for the superblocks to be written.
 
  Note that I didn't include a /swap partition. If you have more than 4GB of memory, you really don't need a swap, but if you really, really want one, I still follow the 1.5 - 2 * rule (total system mem * 1.5 or 2 = swap in GB). It really wastes space on the SSD, though, and adds a lot of writes to the drive, so I don't inculde it.  Although you've written your partition table, you need to format them into the appropriate file system. The EFI /boot partition needs to be Fat32, but the rest can be ext4.
 
    mkfs.vfat -s2 -F32 /dev/sdb1
    mkfs.ext4 /dev/sdb2
 
 Now, we need to mount the partitions so we can install the base system.
 
    mount /dev/sdb2 /mnt
    mkdir /mnt/home
    mount /dev/sdb3 /mnt/home
    mkdir -p /mnt/boot
    mount /dev/sdb1 /mnt/boot
 
 If you're like me, and use a secondary disk as a storage drive, then you will want to mount it now. Using my partition (/dev/sda4) as an example of a storage partiton, it is incredibly simple:
 
    mkdir /mnt/data
    mount /dev/sda4 /mnt/data
 
 That's it, you're done with your disks, you're ready to actually install Arch.  The easiest way to do this is with pacstrap.
 
    pacstrap -i /mnt base base-devel
    Now you wait... And wait... Then it's done
    Now is a good time to generate your fstab.
    genfstab -U -p /mnt /mnt/etc/fstab
    Make sure it generated correctly by simply opening in up with a text editor, such as Vi, Vim, or Nano like such:
    nano /mnt/etc/fstab
    If there's shit in there, you're good. CTRL-X to exit.
 
 Now, we need a bootloader. With UEFI on Arch, I do not recommend Grub. Gummiboot and SysLinux are much better options for UEFI, in my honest opinion (opinion based on the fact that they work).  Your efivars should already be mounted, so unmount it using umount.
 
    umount /sys/firmware/efi/efivars
 
 Now, you can get to installing a bootloader; we will use Gummiboot in this example.
 
    First, you need to chroot before you do anything else.
    arch-chroot /mnt
    mount /sys/firmware/efi/efivars
    pacman -S gummiboot
    gummiboot install
    nano /boot/loader/entries/arch.conf
    title    Arch
    linux    /vmlinuz-linux
    initrd   /initramfs-linux.img
    options  root=/dev/sdb2 rw
    CTRL-O, enter
    CTRL-X
    That is assuming your root partition is at /dev/sdb2, of course.
    Go ahead and umount your partitions:
    umount /mnt/{boot,home}
    reboot
 
That should be all you need to do. It will boot directly into your Arch install without any waiting, but if you wanted, you could change that later on. Topic for another day.  Now, Arch is installed, we can boot into it. Reboot your system, and select your boot drive to boot priority 1 if it isn't already selected. Boot into Arch, and let's configure.  Should be prompted for a login, type in root, press enter, and you're in. There is no root passwd yet, but to set one, type passwd. Enter you desired password, and it is set.  You need to tell Arch a few things. This is a bare-bones installation, so we'll do the bare minimum.
 
    First, set your locale.
    nano /etc/locale.gen
    Scroll down to your locale (in most cases on this forum, I'd guess en_US or en_UK.UTF-8, but choose the one that fits you)
    Delete the # in front of it
    CTRL-O, enter
    CTRL-X
    Now you need a locale.conf, so make one with your locale of choice
    echo LANG=en_US.UTF-8 > /etc/locale.conf
    export LANG=en_US.UTF-8
 
 
 
    If you use a non-standard keymap, like DVORAK, then you nede to tell Arch that too. While in the terminal, at any point, you can simply use "loadkeys *" where * is your layout, but to set it permanently, set it in your vconsole.conf
    nano /etc/vconsole.conf
    KEYMAP=dvorak
    FONT=Lat2-Terminus16
    CTRL-O, enter
    CTRL-X
    This will set it to DVORAK with the font Lat2-Terminus16
 
 You also need Arch to know your timezone.
 
    To see all available timezones, issue the command:
    ls /usr/share/zoneinfo
    When you've determined which zone you are in, you need to specify a sub zone. To view subzones, enter the command:
    ls /usr/share/zoneinfo/*Zone*
    To set your timezone, simply issue the command:
    ln -s /usr/share/zoneinfo/*Zone*/*SubZone* /etc/localtime
    You might as well go ahead and set your hardware clock it UTC while you're at it.
    hwclock --systohc utc
 
 Time to add a user that isn't root.
 
    useradd -m -g users  -G wheel -s /bin/bash *username*
    passwd *username*
    Enter the password.
    To add that user to the sudoers file, simply type:
    nano /etc/sudoers
    Scroll down to where it says root ALL = (ALL)ALL, and add a line below it that reads:
    *username* ALL = (ALL)ALL
    CTRL-O, enter
    CTRL-X
 
 You'll probably want a GUI interface after so much CLI. This is where you start to have lots of options, but for simplicity, let's just assume you want XFCE. Because it's nice 'n stuff.  Before you get a WM or DE, you need X. Good ol annoying X.
 
    pacman -S xorg-server xorg-server-utils xorg-xinit
    pacman -S mesa
    You should go ahead and install your GPU driver as well. I don't know what card you use, but consult the wiki to find out which package to install.
    If you use a non-standard keyboard map, you will, again, have to set that manually in the X settings. It's pretty easy, though.
    For example, to setup your keymap to Dvorak, just nano ~/.xinitrc, and add setxkbmap -layout dvorak BEFORE you initialize your wm/de.;
 
Test if X is working by typing "startx" to start the default test X WM. Type "exit" in one of the terminal emulators to stop X.  You don't want that ugly piece of shiz, though, you want eyecandy. Here is some documentation on WMs and DEs; you can install WMs ontop of DEs, but for this purpose, we'll use a full-blown DE. I personally just use AwesomeWM, but that is my preference.
 
    For demonstrative purposes, let's install XFCE4.
    pacman -S xfce4
    Let that install, then edit your basd_profile to start X at startup:
    nano ~/.basd_profile
    Scroll to the bottom, add this line:
    [[ -z $DISPLAY && $XDG_VTNR -eq 1 ]] && exec startx
    CTRL-O, enter
    CTRL-X
 
Almost done, but you are still going to boot into a CLI login, with no sound, but those are two very easy steps.  For GUI logins, I recommend SLiM because it's lightweight, supports tons of WMs/DEs, is very configurable, and is just pretty.
 
    pacman -S slim
    Then, just tell your ~/.xinit to start xfce4 at upon startup
    nano ~/.xinitrc
    Scroll to the bottom, and add:
    exec startxfce4
    Or whatever WM/DE you're going to use.
    CTRL-O, enter
    CTRL-X
 
 Lastly, just unmute the already install Alsa so you have noise.
 
    pacman -S alsa-utils
    alsamixer
    m
    CTRL-C
 
Now, that's it. Reboot, login, and you have Arch. Yay! Up next, spell-checking, the AUR, and pretties :)  Let me know how I can improve this guide, of spelling errors/grammatical mistakes, or anywhere you get stuck.  Enjoy Arch!
RAW Paste Data

Prerequirements:

    Arch iso
    USB drive or CD
    UEFI hardware
    Keyboard, mouse, eyes

First, you need the Arch Linux iso. The same iso file supports both 32 bit and 64 bit installations, so don't worry about it not being the right version. I recommend torrenting it because of the kagillion seeders.

Once you've downloaded it, you need to burn it. Do not use Unetbootin for this - it doesn't show you how to do it the right way, and it doesn't actually work with Arch from the iso.

Warning - will delete all data on the drive. Please backup if necessary. Formatting is serious shit.

If you're on Linux, then you have a really easy job. Simply open the terminal emulator of your choice, sudo su into root perm, and type:

    dd if=/path/to/iso of=/dev/sdX;

Where the /dev/sdX is your USB drive or location of your CD drive. If you're on Windows, I recommend Win32 Disk Imager. Select the slowest burn option, select your iso file, your USB drive you wish to format, and you're good to go. It's all GUI, so should be pretty easy to figure out. Once you've burned your livedisk, reboot the machine, but boot directly into UEFI (usually F2, F11, or Del to force it). Every BIOS is different based on manufacturer and version, but boot into whatever medium you burnt your iso to in UEFI mode. It must not boot into standard BIOS mode, it must be UEFI boot. There are several options once the livedisk is loaded - default, UEFI shell 1, UEFI shell 2, etc. Just leave it alone and boot into the standard/default selected option. You can boot into UEFI shells 1 or 2, but they pretty much, for all intensive purposes, achieve the same thing.  Once you've booted into UEFI, it's time to get to business. You've been automatically logged into the root account, so don't worry about that. First of all, we're going to need to format and partition some disks to install your Arch onto. In my case, my SSD I used as my boot drive is /dev/sdb, so I will use that in all commands. To find out your disks names, simply issue the command: 

    fdisk -l

 Once you've found the disk(s) you wish to install your system onto, we can proceed to partition them accordingly and format. In my case, I have a 120GB SSD (my /dev/sdb) and a 2TB HDD (my /dev/sda). I use a partition on my 2TB HDD for storage, but keep my /root and /home partitions on my SSD as a boot drive. You can piece your partitions up however you want, but I like to keep things clean, so they can work off of one disk if need be. To partition your drive(s) for an Arch installation, you are best off with simply starting with a clean disk, so let's delete all of the existing partitions, setup our own ones, and write to the disk.Warning, this WILL DELETE ALL DATA ON YOUR DISK(S) / PARTITIONS THAT YOU USE.  To delete the existing partitions, simply issue the command: 

    gdisk /dev/sdb
    d

 Assuming /dev/sdb is your boot drive, simply repeat the d operation with all partitions (starting with 1, increasing onward) until your disk is empty. Now, we need to write new partitions. The most important step of an Arch installation on UEFI is the EFI boot partition. I am going to go ahead and put my /root and /home partitions on here as well. 

    gdisk /dev/sdb
    N
    First Sector - Just press enter, so it's at the front of your disk
    Last sector - +1024M
    Type - ef00
    Now, to make the root partition
    N
    First sector - Enter
    Last sector - +30000M
    Type - Enter
    And, the last on this disk, the /home partition
    N
    First sector - Enter
    Last sector - Enter
    Type - Enter
    Now, if you are ready to delete your partition table and write the new one, just enter the command:
    w
    Press enter, and wait for the superblocks to be written.

  Note that I didn't include a /swap partition. If you have more than 4GB of memory, you really don't need a swap, but if you really, really want one, I still follow the 1.5 - 2 * rule (total system mem * 1.5 or 2 = swap in GB). It really wastes space on the SSD, though, and adds a lot of writes to the drive, so I don't inculde it.  Although you've written your partition table, you need to format them into the appropriate file system. The EFI /boot partition needs to be Fat32, but the rest can be ext4. 

    mkfs.vfat -s2 -F32 /dev/sdb1
    mkfs.ext4 /dev/sdb2

 Now, we need to mount the partitions so we can install the base system. 

    mount /dev/sdb2 /mnt
    mkdir /mnt/home
    mount /dev/sdb3 /mnt/home
    mkdir -p /mnt/boot
    mount /dev/sdb1 /mnt/boot

 If you're like me, and use a secondary disk as a storage drive, then you will want to mount it now. Using my partition (/dev/sda4) as an example of a storage partiton, it is incredibly simple: 

    mkdir /mnt/data
    mount /dev/sda4 /mnt/data

 That's it, you're done with your disks, you're ready to actually install Arch.  The easiest way to do this is with pacstrap. 

    pacstrap -i /mnt base base-devel
    Now you wait... And wait... Then it's done
    Now is a good time to generate your fstab.
    genfstab -U -p /mnt /mnt/etc/fstab
    Make sure it generated correctly by simply opening in up with a text editor, such as Vi, Vim, or Nano like such:
    nano /mnt/etc/fstab
    If there's shit in there, you're good. CTRL-X to exit.

 Now, we need a bootloader. With UEFI on Arch, I do not recommend Grub. Gummiboot and SysLinux are much better options for UEFI, in my honest opinion (opinion based on the fact that they work).  Your efivars should already be mounted, so unmount it using umount. 

 **gummiboot no longer available. use bootctl**

    umount /sys/firmware/efi/efivars

 Now, you can get to installing a bootloader; we will use Gummiboot in this example. 

    First, you need to chroot before you do anything else.
    arch-chroot /mnt
    mount /sys/firmware/efi/efivars
    pacman -S gummiboot
    gummiboot install
    nano /boot/loader/entries/arch.conf
    title    Arch
    linux    /vmlinuz-linux
    initrd   /initramfs-linux.img
    options  root=/dev/sdb2 rw
    CTRL-O, enter
    CTRL-X
    That is assuming your root partition is at /dev/sdb2, of course.
    Go ahead and umount your partitions:
    umount /mnt/{boot,home}
    reboot

That should be all you need to do. It will boot directly into your Arch install without any waiting, but if you wanted, you could change that later on. Topic for another day.  Now, Arch is installed, we can boot into it. Reboot your system, and select your boot drive to boot priority 1 if it isn't already selected. Boot into Arch, and let's configure.  Should be prompted for a login, type in root, press enter, and you're in. There is no root passwd yet, but to set one, type passwd. Enter you desired password, and it is set.  You need to tell Arch a few things. This is a bare-bones installation, so we'll do the bare minimum.

## Locale

try `localedef -f UTF-8 -i en_GB en_GB.UTF-8` first. found it on [here](http://unix.stackexchange.com/questions/43054/why-is-almost-every-program-complaining-about-my-locale)


    First, set your locale.
    nano /etc/locale.gen
    Scroll down to your locale (in most cases on this forum, I'd guess en_US or en_GB.UTF-8, but choose the one that fits you)
    Delete the # in front of it
    CTRL-O, enter
    CTRL-X
    Now you need a locale.conf, so make one with your locale of choice
    echo LANG=en_GB.UTF-8 > /etc/locale.conf
    export LANG=en_GB.UTF-8

### LANGUAGE: fallback locales
Programs which use gettext for translations respect the LANGUAGE option in addition to the usual variables. This allows users to specify a list of locales that will be used in that order. If a translation for the preferred locale is unavailable, another from a similar locale will be used instead of the default. For example, an Australian user might want to fall back to British rather than US spelling:
locale.conf
LANG=en_AU
LANGUAGE=en_AU:en_GB:en

    If you use a non-standard keymap, like DVORAK, then you nede to tell Arch that too. While in the terminal, at any point, you can simply use "loadkeys *" where * is your layout, but to set it permanently, set it in your vconsole.conf
    nano /etc/vconsole.conf
    KEYMAP=dvorak
    FONT=Lat2-Terminus16
    CTRL-O, enter
    CTRL-X
    This will set it to DVORAK with the font Lat2-Terminus16

 You also need Arch to know your timezone. 

    To see all available timezones, issue the command:
    ls /usr/share/zoneinfo
    When you've determined which zone you are in, you need to specify a sub zone. To view subzones, enter the command:
    ls /usr/share/zoneinfo/*Zone*
    To set your timezone, simply issue the command:
    ln -s /usr/share/zoneinfo/*Zone*/*SubZone* /etc/localtime
    You might as well go ahead and set your hardware clock it UTC while you're at it.
    hwclock --systohc utc

 Time to add a user that isn't root. 

    useradd -m -g users  -G wheel -s /bin/bash *username*
    passwd *username*
    Enter the password.
    To add that user to the sudoers file, simply type:
    nano /etc/sudoers
    Scroll down to where it says root ALL = (ALL)ALL, and add a line below it that reads:
    *username* ALL = (ALL)ALL
    CTRL-O, enter
    CTRL-X

## Network
Basic DHCP network
This setup will enable a DHCP IP for host and container. In this case, both systems will share the same IP as they share the same interfaces.
```/etc/systemd/network/MyDhcp.network```

Put this data in there

```
[Match]
Name=en*

[Network]
DHCP=ipv4
```

```systemctl start /usr/lib64/systemd/systemd-networkd ```

If you did not want to configure a DNS in /etc/resolv.conf and want to rely on DHCP for setting it up, you need to enable systemd-resolved.service and symlink /run/systemd/resolve/resolv.conf to /etc/resolv.conf


```ln -sf /run/systemd/resolve/resolv.conf /etc/resolv.conf```

now start the name resolution service

```systemctl start /usr/lib64/systemd/systemd-resolved ```

https://wiki.archlinux.org/index.php/systemd-networkd#Required_services_and_setup <- to set the services to start automatically

`ping google.com` to test, as always

## GUI

 You'll probably want a GUI interface after so much CLI. This is where you start to have lots of options, but for simplicity, let's just assume you want XFCE. Because it's nice 'n stuff.  Before you get a WM or DE, you need X. Good ol annoying X. 

    pacman -S xorg-server xorg-server-utils xorg-xinit
    pacman -S mesa
    You should go ahead and install your GPU driver as well. I don't know what card you use, but consult the wiki to find out which package to install.
    If you use a non-standard keyboard map, you will, again, have to set that manually in the X settings. It's pretty easy, though.
    For example, to setup your keymap to Dvorak, just nano ~/.xinitrc, and add setxkbmap -layout dvorak BEFORE you initialize your wm/de.;

Test if X is working by typing "startx" to start the default test X WM. Type "exit" in one of the terminal emulators to stop X.  You don't want that ugly piece of shiz, though, you want eyecandy. Here is some documentation on WMs. I personally just use i3, but that is my preference.

    For demonstrative purposes, let's install i3.
    pacman -S i3
    Let that install, then edit your basd_profile to start X at startup:
    nano ~/.bash_profile
    Scroll to the bottom, add this line:
    [[ -z $DISPLAY && $XDG_VTNR -eq 1 ]] && exec startx
    CTRL-O, enter
    CTRL-X

Almost done, but you are still going to boot into a CLI login, with no sound, but those are two very easy steps.  For GUI logins, I recommend SLiM because it's lightweight, supports tons of WMs/DEs, is very configurable, and is just pretty.

    pacman -S slim
    Then, just tell your ~/.xinit to start xfce4 at upon startup
    nano ~/.xinitrc
    Scroll to the bottom, and add:
    exec i3
    Or whatever WM/DE you're going to use.
    CTRL-O, enter
    CTRL-X

 Lastly, just unmute the already install Alsa so you have noise.

    pacman -S alsa-utils
    alsamixer
    m
    CTRL-C

## Packages installed so far
- python-pip
- i3wm (don't install vanilla i3 as i'm using i3-gaps)
- rxvt-unicode
- git
- openssh
- xclip
- ttf-ubuntu-font-family
- adobe-source-code-pro-fonts
- firefox
- tlp
- feh
- vim
- htop
- lxappearance
- unzip
- compton
- pcmanfm
- mopidy
- mpc
- wget
- ncmpcpp
- alsa-utils
- python
- python-setuptools
- screen
- jdk8-openjdk

### for mopidy spotify
- python2-cffi

### AUR

1. `git clone <AUR URL> directory`
2. `cd directory`
3. `makepkg PKGBUILD`
4. `sudo pacman -U compiled.tar.xz`

### pip
`sudo pip install <pip_package_name>
- i3-py

- lemonbar-xft-git (lemonbar port that adds xft font support)
- libspotify
- python2-pyspotify
- yaourt https://aur.archlinux.org/yaourt.git
- package-query https://aur.archlinux.org/package-query.git
- mopidy-spotify
- ttf-font-awesome
- python-xlib
- i3-ipc-python-git (for copperbadgers i3 script) https://aur.archlinux.org/i3ipc-python-git.git
- ttf-noto

## Other installation steps
- install numix solarized themes under `~/.themes/`
- install super flat icons under `~/.icons`