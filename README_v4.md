# Hardware specifications

- RAM: 512MB
- Hard disk: 8GB USB stick
- Architecture: x86 (32 bits)
- CPU: Intel Pentium 4

These are the specifications of the real hardware I used. **The important part is the RAM size**, Arch Linux won't run with less than 512MB for RAM, even on VirtualBox.

# Linux specifications

- Distribution: Arch Linux
- Kernel version: 5.2.9
- Release: 2019.08.01
- URL: https://archlinux32.org/
- Filesystem type: EXT4


> - ``frankenstein_8GB_v1.iso`` is a raw copy of the "hard disk" needed to run *frankenstein*. It contains the operative system along with user data on it as unique EXT4 partition. Another special consideration is the ISO was created to be installed on a 4GB USB stick. It means you won't take advantage of having a large USB stick.
> - It don't have any dekstop environment, it can be controlled through SSH.
> - *frankenstein* IP address can be obtained by sending a ping to the broadcast address of your local network and waiting for its response (config file ``/etc/sysctl.d/90-frankenstein.conf`` allows Arch to reply to broadcast ping, more information [here](https://superuser.com/questions/306065/what-is-icmp-broadcast-good-for)).
> - Username and password to connect through SSH is "alarm".

Section below describe special configurations made to the Arch contained in *frankenstein_8GB_v1.iso*.

## Installing Arch Linux from scratch on an USB stick

To install Arch Linux from scrach follow instruccions on the [Arch wiki](https://wiki.archlinux.org/index.php/Installation_guide). **[Special considerations](https://wiki.archlinux.org/index.php/Installing_Arch_Linux_on_a_USB_key). must be taken to install it on an USB stick**. For simplicity I chose [GRUB](https://wiki.archlinux.org/index.php/GRUB#GUID_Partition_Table_(GPT)_specific_instructions) bootloader and I installed it on the MBR (Master Boot Record).

One of the most important considerations if not the most important is to reduce the I/O operations on the USB stick as much as possible. To achive this, the Arch wiki recommends disabling the [journaling](https://es.wikipedia.org/wiki/Journaling) on the EXT4 filesystem. So as a side effect of disabling journaling, I/O errors are more likely to occur. **In case of some I/O error just write a fresh *frankenstein_8GB_v1.iso* to the USB stick with ``dd`` (remember ``dd`` will overwrite every byte on the USB stick)**.

### User basic configuration

1. Create the ``alarm`` user:
	1. Create the user (this will create the home directory and set the zsh interpreter for the user):
		```
		useradd -s /bin/zsh -m alarm
		```
	2. Set its password with the ``passwd`` command.
2. Add it to the ``sudoers`` file (the package ``sudo`` must be installed):
3. Edit it with ``visudo`` and add this line :
	 	```
		alarm = ALL=(ALL) ALL
		```

> - You can use ``sudo`` with the user ``alarm`` to run any command as root.
> - Username and password is "alarm"

### Shell configuration

It has the [zsh shell](https://wiki.archlinux.org/index.php/Zsh) configured to get the same setup as the monthly ISO releases. A way of To customize zsh with the same setup as the monthly ISO releases:

1. Install the ``grml-zsh-config`` and ``wget`` packages (if you don't have it yet).
3. Download this ``.zshrc`` to the home folder of your user:
```
wget -O .zshrc https://git.grml.org/f/grml-etc-core/etc/zsh/zshrc
```


For more info go to [this link](https://wiki.archlinux.org/index.php/Zsh#Sample_.zshrc_files) and to the [grml site](https://grml.org/zsh/).

### Yay package manager

Yay (Yet another Yaourt) is an interface to ``pacman`` like ``yaourt`` to get packages from the [AUR](https://wiki.archlinux.org/index.php/Arch_User_Repository) (Arch User Repository). To install and use it, log in with the ``alarm`` user and follow tlhis instruccions:

1. Install ``git`` and ``base-devel`` packages with ``pacman``.
2. Clone the ``yay`` repository and ``cd`` into it:
	```
	git clone https://aur.archlinux.org/yay.git
	```
3. Replace the line that starts with ``arch=`` on the ``PKGBUILD`` file with this:
	```
	arch=('i686' 'pentium4')
	```
4. Edit the ``/etc/makepkg.conf`` with ``sudo``:
	Comment this line:
	```
	#CFLAGS="-march=pentium4 -mtune=generic -O2 -pipe -fno-plt"
	```
	and add this one:
	```
	CFLAGS="-march=i686 -mtune=generic -O2 -pipe -fno-plt"etc
	```

5. Run ``makepkg``:
	```
	makepkg -si
	```

More info:
- [Installing yay](https://blog.desdelinux.net/yay-una-excelente-ayudante-para-aur-y-una-alternativa-a-yaourt/)
- [Configuring makepkg to install packages on pentium4 architectures](https://bbs.archlinux32.org/viewtopic.php?pid=5920)

### Beep beep

Because *frankenstein* won't use monitor, one way to know Arch Linux booted ok is by making the motherboard sound like "beep beep". For that purpose *frankenstein_8GB_v1.iso* is configured with a service called ``frankenstein_beep.service`` (systemctl service) which runs a script called ``frankenstein_beep`` located on the ``/usr/bin/`` directory. Basically the scripts invokes the ``beep`` command with some particular parameters to make a nice sound. The ``beep`` command requires the ``beep`` package to be installed with the ``pacman`` package manager. For more information about how to run a script during Arch Linux boot process, refer to [this](https://arashmilani.com/post?id=86) link.

### Network wireless configuration

Frankenstein is configured to automatically connect to a wired/wireless network. If it has an ethernet wire plugged in, it will try to connect to that wired network. On the other hand, if it has an USB WiFi adapter, it will try to connect to a WiFi router. For a wired or wireless connection, the system will try to get its IP address through DHCP. For this purpose, the OS is configured to run ``dhcpcd`` after booting:

```
systemctl enable dhcpcd.service
```


The ``/etc/dhcpcd.conf`` file on the ISO was configured to send the frankenstein hostname to your DNS server on your local network so it gives a way of identifying the system. For more information refer to [this](https://wiki.archlinux.org/index.php/Dhcpcd#DHCP_static_route(s)) page on teh Arch Wiki.

#### Configuring a wireless connection

First of all, you have to check you have the proper drivers for your WiFi adapter. Usually you just have to connect the adapter and Arch will automatically enable the proper kernel modules (drivers) to manage your adapter but sometimes you will have to look and install manually. To configure the connection you have two options :

- Using WPA supplicant
- Using Network Manager

To make configurations connect with SSH through a wired connection or do it with VirtualBox and then write all to an USB stick.

##### Using NetworkManager

First, you have to configure to which networks you want the system to connect. This is done with ``nmcli`` as its said on the [Arch wiki](https://wiki.archlinux.org/index.php/NetworkManager#Usage). Additional configurations must be done if you want the system to connect automatically after booting:

1. Create the file ``network-wireless.service`` on ``/etc/systemd/system`` with the following content:
```
[Unit]
Description=Wireless network connectivity (%i)

[Service]
Type=oneshot
RemainAfterExit=yes

ExecStart=/usr/bin/NetworkManager

[Install]
WantedBy=multi-user.target
```
2. Enable the service:
```
systemctl enable network-wireless.service
```

> ``nmcli`` requires the Network Manager process to be running.

##### Using WPA supplicant

1. Configuring WPA supplicant:

Install the ``wpa_supplicant`` package and create the file ``wpa_supplicant.conf`` file on the ``/etc/wpa_supplicant/`` directory with this content:

```
ctrl_interface=/run/wpa_supplicant
update_config=1
```

2. Configure *frankenstein* to automatically connect to a wireless network:

Create the file ``network-wireless@.service`` on ``/etc/systemd/system`` with the following content:

```
[Unit]
Description=Wireless network connectivity (%i)
Wants=network.target
Before=network.target
BindsTo=sys-subsystem-net-devices-%i.device
After=sys-subsystem-net-devices-%i.device

[Service]
Type=oneshot
RemainAfterExit=yes

ExecStart=/usr/bin/ip link set dev %i up
ExecStart=/usr/bin/wpa_supplicant -B -i %i -c /etc/wpa_supplicant/wpa_supplicant.conf
ExecStart=/usr/bin/dhcpcd %i

ExecStop=/usr/bin/ip link set dev %i down

[Install]
WantedBy=multi-user.target
```

Then enable the service replacing ``interface`` with the wireless interface you want to use to connect through :

```
systemctl enable network-wireless@interface.service
```

> interfaces can be obtained with the ``ifconfig`` command


3. Setting known networks:

One way of doing this is with the ``wpa_cli`` command following instructions [here](https://wiki.archlinux.org/index.php/WPA_supplicant#Connecting_with_wpa_cli).

> - You can configure more than one network.
> - Don't forget to enable all configured networks (with the ``enable_network X`` command where ``X`` is the number of the network you want to enable)
> - Don't forget to save the configurations with the ``save`` command

The other way is editing the ``/etc/wpa_supplicant/wpa_supplicant.conf`` file. For example I have this configurations :

```
ctrl_interface=/run/wpa_supplicant
update_config=1

network={
	ssid="TP-LINK_A38C"
	psk="xxxxxx"
	disabled=1
}

network={
	ssid="AndroidAP"
	key_mgmt=NONE
	disabled=1
}
```

> When ejecuting ``save`` on the ``wpa_cli`` configurations for known networks will be saved to ``/etc/wpa_supplicant/wpa_supplicant.conf``.

For more information about how to configure networks with WPA supplicant and ``dhcpcd`` follow  [this](https://wiki.archlinux.org/index.php/WPA_supplicant#Connecting_with_wpa_cli) link.

### Connection through SSH

As *frankenstein* won't run with any peripheral device (screen, keyboard and mouse), it's accessed through SSH (it has an SSH server installed) :

```
ssh alarm@192.168.0.120
```

Basically, the package ``openssh`` must be installed with ``pacman `` and the following service must be enabled to run after booting:

```
systemctl enable sshd.service
```


For more information to install and configure SSH client and server follow instructions on the [Arch wiki](https://wiki.archlinux.org/index.php/OpenSSH).


# Running frankenstein with VirtualBox

Before creating a virtual hard disk and booting it on a VirtualBox virtual machine, the ISO file must be converted to VDI format.

For MacOS this can be acomplished with this command:

```
VBoxManage convertfromraw file.iso file.vdi --format VDI
```

And in almost any Linux distribution :

```
VBoxManage converdd file.iso file.vdi --format VDI
```

And in MacOS, the opossite operation to convert from a VDI file to an ISO file can be achived with this command:

```
VBoxManage clonehd file.vdi file.iso --format RAW
```
More information:
- [How to Convert ISO to VDI Virtual Box Image]( http://osxdaily.com/2018/06/17/convert-iso-to-vdi-virtualbox-image/)
- [booting dd image on virtualbox
](https://askubuntu.com/questions/1023741/booting-dd-image-on-virtualbox)

> Remember to configure the virtual network adapter as bridge on VirtualBox to simuilate an ethernet connection.

# Useful commands

## ``pacman``

Pacman is the default package manager for Arch Linux. Some useful commands are these:

- To install any package:
```
pacman -S package_name
```
- Sometimes it fails so its a good a idea to sync your database:
```
pacman -Sy
```
- Deleting packages completely:
```
pacman -R package_name
```
- Search packages:
```
pacman -Ss package_name
```
- Upgrade all packages (without ``package_name`` it will upgrade all packages)
```
pacman -Syu package_name
```

> ``yay`` commands are similar to ``pacman`` commands

## ``df``

``df`` is useful to know the state of your filesystem, specifically stats about the usage of your disc.

- Simple usage:
```
df
```
- Showing size in MB, GB, etc. (replacing the ``M`` with the size unit):
```
df -BM
```
