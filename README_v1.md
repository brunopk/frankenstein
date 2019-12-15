# Hardware specifications

- RAM: 256MB
- Hard disk: 4GB USB stick
- Architecture: x86 (32 bits)
- CPU: Intel Pentium 4

These are the specifications of the real hardware I used. **The important part is the RAM size**, Arch Linux won't run with less than 512MB for RAM, even on VirtualBox.

# Linux specifications

- Distribution: Arch Linux
- Kernel version: 5.2.9
- Release: 2019.08.01
- URL: https://archlinux32.org/
- Filesystem type: EXT4


> - ``frankenstein_4GB_v1.iso`` is a raw copy of the "hard disk" needed to run *frankenstein*. It contains the operative system along with user data on it as unique EXT4 partition. Another special consideration is the ISO was created to be installed on a 4GB USB stick. It means you won't take advantage of having a large USB stick.
> - It don't have any dekstop environment, it can be controlled through SSH. You can obtain *frankenstein* IP address by sending a ping to the broadcast address of your local network (its configured to reply to this address). Username to connect through SSH is "root" and password is also "root".

Section below describe special configurations made to the Arch contained in *frankenstein_4GB_v1.iso*.

> To install ant package with ``pacman`` use: ``pacman -S package_name``

## Installing Arch Linux from scratch on an USB stick

To install Arch Linux from scrach follow instruccions on the [Arch wiki](https://wiki.archlinux.org/index.php/Installation_guide). **[Special considerations](https://wiki.archlinux.org/index.php/Installing_Arch_Linux_on_a_USB_key). must be taken to install it on an USB stick**. For simplicity I chose [GRUB](https://wiki.archlinux.org/index.php/GRUB#GUID_Partition_Table_(GPT)_specific_instructions) bootloader and I installed it on the MBR (Master Boot Record).

One of the most important considerations if not the most important is to reduce the I/O operations on the USB stick as much as possible. To achive this, the Arch wiki recommends disabling the [journaling](https://es.wikipedia.org/wiki/Journaling) on the EXT4 filesystem. So as a side effect of disabling journaling, I/O errors are more likely to occur. **In case of some I/O error just write a fresh *frankenstein_4GB_v1.iso* to the USB stick with ``dd`` (remember ``dd`` will overwrite every byte on the USB stick)**.

### Beep beep

Because *frankenstein* won't use monitor, one way to know Arch Linux booted ok is by making the motherboard sound like "beep beep". For that purpose *frankenstein_4GB_v1.iso* is configured with a service called ``frankenstein_beep.service`` (systemctl service) which runs a script called ``frankenstein_beep`` located on the ``/usr/bin/`` directory. Basically the scripts invokes the ``beep`` command with some particular parameters to make a nice sound. The ``beep`` command requires the ``beep`` package to be installed with the ``pacman`` package manager. For more information about how to run a script during Arch Linux boot process, refer to [this](https://arashmilani.com/post?id=86) link.

### Network configuration

Frankenstein is configured to automatically connect to a wired/wireless network. If it has an ethernet wire plugged in, it will try to connect to that wired network. On the other hand, if it has an USB wifi adapter, it will try to connect to the first **public network** (without authentication required). In any case, after connecting to the network it will try to get its IP address through DHCP. For this purpose, the OS is configured to run ``dhcpcd`` after booting:

```
systemctl enable dhcpcd.service
```


The ``/etc/dhcpcd.conf`` file on the ISO was configured to send the frankenstein hostname to your DNS server on your local network so it gives a way of identifying the system. For more information refer to [this](https://wiki.archlinux.org/index.php/Dhcpcd#DHCP_static_route(s)) page on teh Arch Wiki.


### Shell configuration

It has the [zsh shell](https://wiki.archlinux.org/index.php/Zsh) configured to get the same setup as the monthly ISO releases. To customize zsh this way follow instruccions [here](https://wiki.archlinux.org/index.php/Zsh#Sample_.zshrc_files).

### Connection through SSH

As *frankenstein* won't run with any peripheral device (screen, keyboard and mouse), it's accessed through SSH (it has an SSH server installed) :

```
ssh root@192.168.0.120
```

Basically, the package ``openssh`` must be installed with ``pacman `` and the following service must be enabled to run after booting:

```
systemctl enable sshd.service
```

Also, ``frankenstein_4GB_v1.iso`` contains the file ``90-frankenstein.conf`` in ``/etc/sysctl.d/``  which allows Arch to reply to broadcast ping. More information about this [here](https://superuser.com/questions/306065/what-is-icmp-broadcast-good-for).


For more information to install and configure SSH client and server follow instructions on the [Arch wiki](https://wiki.archlinux.org/index.php/OpenSSH).

# Running frankenstein with VirtualBox

Before creating a virtual hard disk and booting it on a VirtualBox virtual machine, the ISO file must be converted to VDI format.

For MacOS this can be acomplished with this command:

```
VBoxManage convertfromraw file.iso file.vdi --format VDI
```

More info: http://osxdaily.com/2018/06/17/convert-iso-to-vdi-virtualbox-image/

And in almost any Linux distribution :

```
VBoxManage converdd file.iso file.vdi --format VDI
```

More info: https://askubuntu.com/questions/1023741/booting-dd-image-on-virtualbox


> Remember to configure the virtual network adapter as bridge on VirtualBox
