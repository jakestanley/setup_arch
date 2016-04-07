# VirtualBox setup

This is a brief and work in progress guide to my setup process. To be honest, it's probably just checking out [this article on the AUR](https://wiki.archlinux.org/index.php/VirtualBox#Installation_steps_for_Arch_Linux_guests)

## Packages

Using pacman, install the following packages:

- virtualbox-guest-utils
- linux-headers

## Start the services

Load the modules manually with:

`modprobe -a vboxguest vboxsf vboxvideo`

Permanently set up the modules to load on boot by creating the file `/etc/modules-load.d/virtualbox.conf` and filling it with the following:

```
vboxguest
vboxsf
vboxvideo
```