# Asus T100 TA with Linux

> This document/repository is inspired by [5bentz's linux-asus-t100 guide](https://github.com/5bentz/linux-asus-t100/blob/master/T100TA_guide.md)

The Asus T100 TA has a 64-bit CPU with a 32-bit UEFI/BIOS and thus needs some extra steps to work with many Linux distributions/their GRUB configuration.

The steps in this guide are for installing Linux Mint 21.3 Cinnamon 64-bit, but may work with other versions and distros.

## Pre requirements

- A USB flash drive (>=3 GB)
- The [Linux Mint 21.3 iso](https://linuxmint.com/edition.php?id=311)
- bootia32.efi [src1](./bootia32.efi), [src2](https://github.com/jfwells/linux-asus-t100ta/raw/master/boot/bootia32.efi) (files might be unsafe, compile them yourself)
- grubia32.efi [src1](./grubia32.efi), [src2](http://www.thinktwisted.com/gradschool/Public/grubia32.efi) (files might be unsafe, compile them yourself)
- An Asus T100 TA (duh!)
- This guide (on a second device/printed or memorized)

### Preparing the installation media

Use a tool like [Rufus](https://rufus.ie/), [balena etcher](https://etcher.balena.io/), [dd](https://man.archlinux.org/man/dd.1) ... or copy the iso to a USB flash drive with [Ventoy](https://ventoy.net) installed.

### Install Linux Mint

> Press/spam `Esc` for the boot-menu and `F2` for the UEFI-Setup

Disable Secure Boot, then boot from the USB flash drive and install the system like you normally would.

> If you run into problems during the installation, try connecting your device to the internet before starting the installer.

After the intaller finishes, open a terminal and type:

```bash
sudo -s
mount /dev/<device name>p<boot partition num> /mnt
pushd /mnt/EFI/Boot
wget -o bootia32.efi <bootia32.efi url> | sha512sum  # sha512:d493701aaa5cdf57eb35a22c198d496fc3d7295bd763f17b7cb2edba3e4b9939391e18b2cd808564bc14e4076d3559f3b598230db42b490cf48d9133febd8bc4
  # or copy the previously downloaded file
```

Take note of the device name (typically `mmcblk0` or `mmcblk1`) and partitions where `root` and `efi` are mounted.

### First boot

#### Get a GRUB shell

Select the USB flash drive again, but when the GRUB menu comes up, press `c`.

You should now be in a GRUB shell.

> During your very first boot you might get dropped into a GRUB shell and don't need the USB flash drive at all.

#### Boot Linux

To start our already installed Linux, type the following into the shell:

```grub
configfile (hd<device num>,gpt<root partition num>)/boot/grub/grub.cfg
```

Or do it manually:

> You can press tab to display all options (e.g. for `hd`, `gpt`, `vmlinuz` and `initrd`)
>
> Mint might also boot when using `vmlinuz` and `initrd.img`. The `video` and `reboot` arguments might be optional as well.

```grub-shell
set prefix=(hd<device num>,gpt<root partition num>)/boot/grub
set root=(hd<device num>,gpt<root partition num>)
linux /boot/vmlinuz-xxx root=/dev/<device name>p<root partition num> video=VGA-1:1368x768e reboot=pci,force
initrd /boot/initrd.img-xxx
boot
```

Linux Mint should now start

#### Switch GRUB to 32-bit

1. Connect to the internet

1. Open a terminal (`CTRL+ALT+T`) and type:
   ```bash
   sudo apt update
   sudo apt install grub-efi-ia32-bin  # might not be necessary
   sudo grub-install --efi-directory /boot/efi  # might not be necessary
     # sources say you also need grub-efi-ia32, but apt reports some dependency error
   sudo -s
   pushd /boot/efi/EFI/ubuntu/
   mv grubx64.efi grubx64.efi.bak
   wget -o grubia32.efi <grubia32.efi url> | sha512sum  # cf83e1357eefb8bdf1542850d66d8007d620e4050b5715dc83f4a921d36ce9ce47d0d13c5d85f2b0ff8318d2877eec2f63b931bd47417a81a538327af927da3e
     # or copy the previously downloaded file
   update-grub
   update-grub2
   reboot
   ```
1. You should now be able to boot Linux Mint without the USB flash drive
   > If it does not work and you see two `ubuntu` entries in the boot menu, try selecting the other one.

## Sources

- https://github.com/5bentz/linux-asus-t100/

- https://unix.stackexchange.com/questions/206274/how-do-i-repair-grub2-not-booting-32-bit-efi-on-a-64-bit-machine

- https://linuxnorth.wordpress.com/2014/12/11/installing-64-bit-linux-on-the-asus-transformer-book-t100/

- http://www.jfwhome.com/2014/03/07/perfect-ubuntu-or-other-linux-on-the-asus-transformer-book-t100/
