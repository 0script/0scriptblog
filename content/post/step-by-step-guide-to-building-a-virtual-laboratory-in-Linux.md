---
title: "Step by Step Guide to Building a Virtual Laboratory in Linux"
date: 2023-04-09T09:53:33+02:00
draft: false
---

![Virtual Lab](https://benisnous.com/wp-content/uploads/2021/03/how-to-build-a-HACKING-lab-to-become-a-hacker.jpg)

# Content 

* [Overview](#overview)
* [What you will need](#what-you-will-need) 
* [Link to installation media](#link-to-installation-media) 
* [Installing a hypervisor](#installing-a-hypervisor) 
* [Setting up virtual switch](#setting-up-virtual-switch)
* [Installation of vyOs](#installation-of-vyos)
* [Installation of ubuntu desktop](#installation-of-ubuntu-desktop)
* [Installation of Kali Linux](#installation-of-kali-linux)
* [Installation of Metasploitable2](#installation-of-metasploitable2)
* [Conclusion](#conclusion)

# Overview

A virtual laboratory is a simulated environment that allows students, researchers, or professionals to perform laboratory experiments. Unlike traditional laboratory settings, virtual laboratories don't require any physical equipment or materials.

In the case of computing, a virtual laboratory allows you to have access to hardware components on which you can test  software applications and network configuration without worrying about the cost or the risk of damaging newly acquired equipment.

Overall, virtual laboratories offer a flexible and accessible way to explore new network concepts, conduct experiments, and try and develop malware with more flexibility and less risk.


# What You Will Need

![My Computer Specs](https://i.redd.it/nzsn80mofzha1.png)

__Before you start this tutorial__

* __Software requirements :__ A computer with a __Linux__ desktop, preferably a _Debian-based_ distribution. 

* __Hardware requirements :__ A computer manufactured after 2014 with at least 4 cores of CPU, at least 8 GB of RAM, and at least 100 GB of storage for the Linux partition.

* __Prerquise__ : Familiarity with the Linux operating system, knowing how to install applications via the terminal and how to use a text editor such as __vim__ or __nano__ .
Don't be the type to give up at the first obstacle. read carefully, use google, and don't let the documentation scare you. Good luck!


# Link to installation media

We will install a set of operating systems , they can be downloaded from the links below:

* __VyOs__ a open source router and firewall :  [Download here](https://s3-us.vyos.io/rolling/current/vyos-1.4-rolling-202302110324-amd64.iso).

* Virtual image for qemu of __kali linux__  linux distribution for penetration testing [Download link](https://cdimage.kali.org/kali-2022.4/kali-linux-2022.4-qemu-amd64.7z).

* A __ubuntu__ desktop : [Download here](https://sourceforge.net/projects/osboxes/files/v/vb/55-U-u/22.10/64bit.7z/download).

* __Metasploitable2__ a Linux server with security flaws : [Download here](https://sourceforge.net/projects/metasploitable/files/Metasploitable2/).  



# Installing A Hypervisor

## What Is A Hypervisor

A hypervisor, also called a virtual machine manager, is a software layer that allows you to run multiple virtual machines (VMs) on a single physical host computer. A hypervisor creates a virtualized environment in which multiple operating systems can run simultaneously on the same physical hardware. 

Linux-based operating systems support all mainstream hypervisors such as Virtualbox, VMware, etc., but for better performance, KVM (Kernel Based Virtual Machine) is the recommended software. 


## Verify Virtualisation Support

To make use of virtualization it is required that the virtualization module for the cpu have been activated , on your linux host machin open a terminal and enter `egrep -c '(vmx|svm)' /proc/cpuinfo` to verify if virtualization is enabled , if the command return 0 that will mean that virtualization is not enabled in your [BIOS setting](#https://helpdeskgeek.com/how-to/how-to-enable-virtualization-in-bios-for-intel-and-amd/) 

```shell
$egrep -c '(svm|vmx)' /proc/cpuinfo
16
```
* We ask the number of line that contain _'svm'_ or _'vmx'_ in the file __/proc/cpuinfo__


## Install KVM with package manager 

We need to install the packages `qemu-kvm`, `libvirt-daemon-system`, `libvirt-clients`,  `bridge-utils` and `virt-manager` using the package manager , __apt__ for debian
```shell
$sudo apt-get install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager
```

If the installation is done we can check if the libvirt deamon is running `sudo systemctl status libvirtd |grep Active`
```shell
$sudo systemctl status libvirtd | grep Active
    Active: active (running) since Sat 2023-04-08 07:06:44 CAT; 6h ago
```
* If the command return __inactive (dead)__ you need to start the process and check again : `sudo systemctl start libvirtd && systemctl status libvirtd |grep Active` .

Now we will add our current user to the __libvirt__ group
```shell
$sudo adduser 'username' libvirt
```
* Replace __'username'__ with the appropriate user for your computer and reboot or log out and log back in for the changes to take effect. 
* Now verify that you are in libvirt group with `groups` :
```shell
$groups
0script cdrom floppy sudo audio dip video plugdev netdev bluetooth libvirt lpadmin scanner docker
```
* If the output do not contain __libvirt__ make sure that you logged out and relogged in after adding user to the group .


# Setting Up Virtual Switch

Libvirt uses the concept of a virtual network switch : a software construction on the host machine, that your virtual machines "plug in" to, and direct their traffic through.

![virt-switch.png](https://wiki.libvirt.org/images/Host_with_a_virtual_network_switch_and_two_guests.png)


In our network, we will have 2 virtual switches: one in NAT mode that will have access to the internet, and one in isolated mode that will create a private LAN.

The VyOs router will be connected to both the NAT and the private network and will allow the machine on the private network to access the internet.

[![virt-lab.png](https://i.postimg.cc/fT1cdvbT/virt-lab.png)](https://postimg.cc/HV48DX5f)


## Activating Default NAT Interface 

The __libvirt__ package comme with a default NAT configuration that we can activate with the following step :

1. Load the xml configuration with `sudo virsh net-define `  
```shell
$sudo virsh net-define /usr/share/libvirt/network/default.xml
```
2. Than activate the interface  using : `sudo virsh net-start default`
```shell
$sudo virsh net-start default
[sudo] password for 0script:                            
Network default started     
```

3. Check for libvirt the interfaces with  `virs net-list`  , this command shows only activated interfaces : 
```shell
$sudo virsh net-list
[sudo] password for 0script: 
 Name      State    Autostart   Persistent
--------------------------------------------
 default   active   no          yes
```


## Creation Of  Virtual Switch In Isolated Mode 

In __libvirt__, virtual switches are defined using XML files. Create a file named `private.xml` to set up the virtual switch, open it with a text editor, and put the following inside:  
```xml
<network>
    <name>private</name>
    <bridge name="virbr1" />
    <ip address="192.168.152.1" netmask="255.255.255.0">
        <dhcp>
            <range start="192.168.152.2" end="192.168.152.254" />
        </dhcp>
    </ip>
    <ip family="ipv6" address="2001:db8:ca2:3::1" prefix="64" />
</network>
```
* _name_ : Interface name
* _bridge_ : Virtual switch name libvirt create a default name for libvirt start with __virbr__ ending with an interger .
* _ip_ : Define ip address type (v4/v6) addresses range and netmask
* _dhcp_ : Enable and set dhcp for the network .

Define and activate the virtual switch with : `virsh net-create private.xml`
```shell
$sudo virsh net-create private.xml
[sudo] password for 0script:
Network private created from private.xml
```

If everything went as expected, you should see both of our virtual interfaces with `sudo virsh net-list`
```shell
$sudo virsh net-list 
[sudo] password for 0script: 
 Name      State    Autostart   Persistent
--------------------------------------------
 default   active   no          yes
 private   active   no          no
```

# Installation Of VyOs

If you have not downloaded VyOs use the [link](https://vyos.io/subscriptions/software) to download .

* If you completed downloading vyos router ,on your computer you can search for the virt manager application and run it .

![Openning virtual manager](https://i.postimg.cc/7Zb2cBgj/0001.png)

* Start to create a new virtual machine using the top left button
[![cvm1.png](https://i.postimg.cc/3Jcnbmrs/cvm1.png)](https://postimg.cc/JGjNsykx)

* Here you chose the type of installation media for the operating system; for VyOS you downloaded an ISO file, so you can just click forward on this step. 
![Create New Virtual Machine](https://i.postimg.cc/8Pv5rqmp/0002.png)

* Use the browse button to access the installation media. 
[![cvyos3.png](https://i.postimg.cc/brW8Jd8s/cvyos3.png)](https://postimg.cc/64rFbW8X)

* To access the file on your local computer, select the "Browse Local" button, then navigate to the folder containing the VyOs ISO file.

[![browse444.png](https://i.postimg.cc/pVFnq0y5/browse444.png)](https://postimg.cc/6TB3QfrB)

[![browse.png](https://i.postimg.cc/TPgx5WCS/browse.png)](https://postimg.cc/qtJS9gfX)

* Uncheck the box next to __Automatic OS Selection__ after choosing the ISO file, then type __Generic OS__ into the search box. When finished, it should look like this :

[![01.png](https://i.postimg.cc/V6D02N27/01.png)](https://postimg.cc/hzQGdKDx)

* In this part we set the virtual CPU and RAM, for or use case 2 core for CPU and 1GB of RAM should be enough, and just after for the virtual hard drive I put 8GB.

[![02.png](https://i.postimg.cc/XvG651kQ/02.png)](https://postimg.cc/HJgND2Ky)

[![03.png](https://i.postimg.cc/597BjPjX/03.png)](https://postimg.cc/Z0NBMx8m)
   
* Giving the machine a name and choosing a virtual switch are the final steps of installation. I give my machine the name VyOs and utilize the default switch.

[provided by the kvm packages](https://www.ibm.com/docs/en/linux-on-systems?topic=choices-kvm-default-nat-based-networking)

[![04.png](https://i.postimg.cc/NG5bcyxq/04.png)](https://postimg.cc/qgH2cvYj)

* When the installation on the Vyos system is complete, hit Enter to launch live mode.
    
[![04.png](https://www.linuxcompatible.org/data/publish/201/d89a50e839b4319a6a279bcb82b354573aff2b/7vyos.jpg)](https://www.linuxcompatible.org/data/publish/201/d89a50e839b4319a6a279bcb82b354573aff2b/)


* Use default user "vyos" and password  "vyos" to log in 

[![07.png](https://i.postimg.cc/zXbYQXQ0/07.png)](https://postimg.cc/8jSYJDN6)


## Setting up VyOs

![07.png](https://www.expertnetworkconsultant.com/wp-content/uploads/2018/07/ip-nat-inside-source-static.png)


### Install image 

Run the commande "install image" on the shell in order to use the installation wizard to complete the installation.
```shell
vyos@vyos:~$ install image
Welcome to the VyOS install program.  This script
will walk you through the process of installing the
VyOS image to a local hard drive.
Would you like to continue? (Yes/No) [Yes]: Yes  # Pressing Enter i.e to Yes
Probing drives: OK
Looking for pre-existing RAID groups...none found.
The VyOS image will require a minimum 2000MB root.
Would you like me to try to partition a drive automatically
or would you rather partition it manually with parted?  If
you have already setup your partitions, you may skip this step

Partition (Auto/Parted/Skip) [Auto]:    # Press Enter i.e to the default option here [Auto]

I found the following drives on your system:sda    4294MB

Install the image on? [sda]:    # Press Enter to select default option

This will destroy all data on /dev/sda. Continue? (Yes/No) [No]: Yes    # The instalation has started

How big of a root partition should I create? (2000MB - 4294MB) [4294]MB: # Press Enter to select default option

Creating filesystem on /dev/sda1: OK
Done!
Mounting /dev/sda1...
What would you like to name this image? [1.2.0-rolling+201809210337]:
OK.  This image will be named: 1.2.0-rolling+201809210337
Copying squashfs image...
Copying kernel and initrd images...
Done!
I found the following configuration files:/opt/vyatta/etc/config.boot.default
Which one should I copy to sda? [/opt/vyatta/etc/config.boot.default]:  # Press Enteror Yes to select default option

Copying /opt/vyatta/etc/config.boot.default to sda.
Enter password for administrator account  # setting up new  password
Enter password for user 'vyos':
Retype password for user 'vyos':
I need to install the GRUB boot loader.
I found the following drives on your system:
sda    4294MB

Which drive should GRUB modify the boot partition on? [sda]:    # Press Enter or Yes to select default option

Setting up grub: OK
Done!
```

* Restart the machine with the command `reboot` to make the change take effect.
```shell
vyos@vyos:~$ reboot
Proceed with reboot? (Yes/No) [No] Yes
```


### Adding private network interface 

We need to add a second network interface to __VyOs__ .

If your VyOS machine is not running, you can start it with the run button of the virt-manager interface.

[![01run.png](https://i.postimg.cc/43V7kFqH/01run.png)](https://postimg.cc/7CPYSXxq)

* On the grub, you can press Enter to avoid wasting time. 
[![vyos-v-1-4-grub.png](https://i.postimg.cc/DzG5ZXxJ/vyos-v-1-4-grub.png)](https://postimg.cc/gwY3tnFm)

We use virt manager to add a network interface to VyOs .

`Show Virtual Hardware detail > Add Hardware > Network > Network Source > Select the private Network Interface > Finish `

* Enter virtual machine hardware configuration .

[![00-vyos-in-qemu.png](https://i.postimg.cc/KzH1zBbX/00-vyos-in-qemu.png)](https://postimg.cc/WF0NYDtS)

* Hit '`Add Hardware Button`' button .

[![01-virt-manager-hardware-detail.png](https://i.postimg.cc/tgCq0JxL/01-virt-manager-hardware-detail.png)](https://postimg.cc/w1Z8L6FV)

* Select '`Network`' than in '`Network source`' select your __private__ virtual switch .

[![02-virt-manager-setting.png](https://i.postimg.cc/prBctW86/02-virt-manager-setting.png)](https://postimg.cc/1VfrGZvG)

* After finishing, the window should look something like this:

[![03-virt-man-hardware-setting.png](https://i.postimg.cc/J4vYV2Bm/03-virt-man-hardware-setting.png)](https://postimg.cc/svP45Kc0)

* Return to the VyOs shell with top left button `Show The Graphical Console`

[![04-virt-manager-hardware-detail.png](https://i.postimg.cc/QxC5kNCx/04-virt-manager-hardware-detail.png)](https://postimg.cc/GH0HbCvZ)

* Reboot the machine to make the change effective.
```shell
vyos@vyos:~$ reboot
Proceed with reboot? (Yes/No) [No] Yes
```


### VyOs Configuration of the LAN

Check to see if there are two network interfaces on the VyOs machine with `ip a` or `ip link show` or `show interfaces`:

[![ip-a-command.png](https://i.postimg.cc/8Pyt4NX2/ip-a-command.png)](https://postimg.cc/kBbQX3ms)

In order to connect with the outside world, our network we will be using eth0 interface, which will receive its address via DHCP. And eth1 will serve for the LAN with a static address set to `192.168.152.10`.

To make those changes in VYOS, enter configuration mode with the command `configure`. You know you are in configuration mode when the shell displays __#__
```shell
vyos@vyos$ configure
vyos@vyos#
```


#### DHCP/DNS setting

In configuration mode, we first set dhcp for eth0 and after a static IP for eth1, each setting is followed by a single line ending with __[edit]__ to indicate that each command was successfully run.
```shell
set interfaces ethernet eth0 address dhcp
set interfaces ethernet eth0 description 'OUTSIDE'
set interfaces ethernet eth1 address '192.168.152.10/24'
set interfaces ethernet eth1 description 'INSIDE'
```

[![1-vyos-set-dhcp-dns.png](https://i.postimg.cc/SK44KVP4/1-vyos-set-dhcp-dns.png)](https://postimg.cc/xX6hxGYp)

##### Address setting

Still in configuration mode we set the DHCP and DNS services for the internal/LAN network, where VyOS will act as the default gateway and DNS server.

* `192.168.152.10/24` is the default gateway .
* The address range 192.168.152.2/24 - 192.168.152.49/24 will be reserved for static assignments
* DHCP clients will be assigned IP addresses within the range of 192.168.152.50 - 192.168.152.254 and have a domain name of vyos-router
* DHCP address will be leased for 24 hours i.e __86400__ secondes
* VyOS will serve as a full DNS server, replacing the need to utilize Google, Cloudflare, or other public DNS servers (which is good for privacy)
* Only hosts from your internal/LAN network can use the DNS server
```shell
set service dhcp-server shared-network-name LAN subnet 192.168.152.0/24 option default-router '192.168.152.10'
set service dhcp-server shared-network-name LAN subnet 192.168.152.0/24 option name-server '192.168.152.10'
set service dhcp-server shared-network-name LAN subnet 192.168.152.0/24 option domain-name 'vyos.router'
set service dhcp-server shared-network-name LAN subnet 192.168.152.0/24 lease '86400'
set service dhcp-server shared-network-name LAN subnet 192.168.152.0/24 range 0 start 192.168.152.50
set service dhcp-server shared-network-name LAN subnet 192.168.152.0/24 range 0 stop '192.168.152.254'

set service dns forwarding listen-address 192.168.152.10
set service dns forwarding allow-from 192.168.152.0/24
set service dns forwarding cache-size 10240
```


[![2vyos-address-setting.png](https://i.postimg.cc/NGZCkFkC/2vyos-address-setting.png)](https://postimg.cc/qgXGkkY2)

[![3vyos-set-nat-rule.png](https://i.postimg.cc/dVVWqCs1/3vyos-set-nat-rule.png)](https://postimg.cc/TpBj0pWM)


##### Setting SNAT rule

The following settings will configure SNAT rules for our internal network(LAN), allowing hosts to communicate through the outside network (WAN) via ip masquerade , after the setting use `commit` than `save` commandes . 
```shell
set nat source rule 100 outbound-interface name 'eth0'
set nat source rule 100 source address '192.168.152.0/24'
set nat source rule 100 translation address masquerade
commit
save
```

[![4-vyos-set-dns-forwarding.png](https://i.postimg.cc/YSYGmX8C/4-vyos-set-dns-forwarding.png)](https://postimg.cc/B8ZQrBdr)



##### Adding DNS server

The following command will use Cloudfare, Google, and Quad9 as DNS relays.
```shell
set service dns forwarding name-server 1.1.1.1
set service dns forwarding name-server 8.8.8.8
set service dns forwarding name-server 9.9.9.9
set system name-server 1.1.1.1
```

##### Testing network 

* Check for interfaces : `show interface` .

[![vyos-show-interfaces.png](https://i.postimg.cc/436XXzL0/vyos-show-interfaces.png)](https://postimg.cc/rdwBJtLG)


* Verify network connection with `host vyos.org` .

[![host-cmd.png](https://i.postimg.cc/KjRX3Lfx/host-cmd.png)](https://postimg.cc/SYF1wnHv)


* Verify nat rule : `show nat source rule`.

[![show-nat-source.png](https://i.postimg.cc/mrNmLDQj/show-nat-source.png)](https://postimg.cc/gwjV4GtL)


* Verify the dns setting configuration in configuration mode : `configure` and then `show service dns`.

[![show-service-dns.png](https://i.postimg.cc/RFtKSb5B/show-service-dns.png)](https://postimg.cc/ftwJ5CR2)


# Installation Of Ubuntu Desktop

I use a virtual box image for the installation that is avalaible on [osboxes](https://www.osboxes.org/virtualbox-images/) use this [link](https://sourceforge.net/projects/osboxes/files/v/vb/55-U-u/22.10/64bit.7z/download) to dowload the ubuntu virtual image .

If you downloaded the file, extract it, and you can start the installation using the __Virtual Manager__ user interface.


* Use the create machine button on virt manager.

[![cvm1.png](https://i.postimg.cc/3Jcnbmrs/cvm1.png)](https://postimg.cc/JGjNsykx)


* Check to import an existing disk image to use the virtual box virtual image; for me, I'm using a virtual box image __(.vdi)__.

[![import-image-kvm.png](https://i.postimg.cc/kGMqHkKV/import-image-kvm.png)](https://postimg.cc/2btsb9Hm)


* You hit browse to access your file manager, then you locate the virtual image. The steps are the same as for the VYOS installation. 

[![cvyos3.png](https://i.postimg.cc/brW8Jd8s/cvyos3.png)](https://postimg.cc/64rFbW8X)

[![browse444.png](https://i.postimg.cc/pVFnq0y5/browse444.png)](https://postimg.cc/6TB3QfrB)

[![browse-ubuntu.png](https://i.postimg.cc/QtDd2XDc/browse-ubuntu.png)](https://postimg.cc/2qHYvNV5)


* If you selected the correct file you can proceed with '`Forward`'.

[![kvm-ubuntu01.png](https://i.postimg.cc/d1CSyYZN/kvm-ubuntu01.png)](https://postimg.cc/XGVgRt89)


* The cpu and memory setting .

[![kvm-ubuntu-cpu.png](https://i.postimg.cc/pd1Ld3g6/kvm-ubuntu-cpu.png)](https://postimg.cc/nsBJRSV4)


* Because we used a virtual image as installation media , there is no hard drive setting so we end up by setting the name for our ubuntu desktop and we set the network to the private switch so it will be part of the LAN . 

[![kvm-ubuntu-network.png](https://i.postimg.cc/kGCPsdzG/kvm-ubuntu-network.png)](https://postimg.cc/ZBwQYXTz)


* If you downloaded the virtual image from osboxes the default username and password are 'osboxes.org' .

[![kvm-ubuntu-login.png](https://i.postimg.cc/J0WV4WQm/kvm-ubuntu-login.png)](https://postimg.cc/9R1NJKSN)


* In the ubuntu desktop use `Crtl+Alt+T` to open a terminal .

[![ubuntu-kvm-shell.png](https://i.postimg.cc/G2f0gbJx/ubuntu-kvm-shell.png)](https://postimg.cc/HcbPVDNr)


* Check the name of your ethernet interface with `ip link show` this name is very important for the next part .

[![ip-link-show1.png](https://i.postimg.cc/TwjGkK3M/ip-link-show1.png)](https://postimg.cc/r0pvsF4Q)


* Set a static ip to the ubuntu desktop , to do so open the file `/etc/netplan/01-network-manger-all.yaml` as root with text editor and you make the following change
    * Replace __enp1s0__ with the name of your own interface
    * In this configuration we set a static ip : __192.168.152.15__ . The  default gateway and the dns relay .

```yaml
# Let NetworkManager manage all devices on this system
network:
    version: 2
    renderer: NetworkManager
    
    ethernets:
        enp1s0:
            addresses:
                - 192.168.152.15
            routes:
                - to: default
                  via: 192.168.152.10
            nameserver:
                addresses: [192.168.152.10]
```


* Run `sudo netplan apply` in a shell to make the change take effect, then `sudo systemctl restart NetworkManager` to restart the __NetworkManager__ service followed by `host google.com` to test hostname resolution.

[![default-gateway.png](https://i.postimg.cc/GmCcD0PS/default-gateway.png)](https://postimg.cc/CzcTyQQj)


* If the command `netplan apply` returns errors, most likely the issue is in the `/etc/netplan/01-network-manager-all.yaml` configuration file. A __(.yaml)__ config file can be tricky, so if you have a hard time writing it, here is the same configuration in a __JSON__ format that you can convert to __(.yaml)__ format using an online tool like [jsontoyaml](https://json2yaml.com/) .

```JSON
// Network configuration in JSON format , need to be converted into YAML file
//Let NetworkManager manage all devices on this system
{
    "network": {
        "version": 2,
        "renderer": "NetworkManager",
        "ethernets": {
            "enp1s0": {
                "addresses": [
                    "192.168.152.15"
                ],
                "routes": [
                    {
                        "to": "default",
                        "via": "192.168.152.10"
                    }
                ],
                "nameserver": {
                    "addresses": [
                        "192.168.152.10"
                    ]
                }
            }
        }
    }
}
```



## Installation of kali linux 

* Download the installation file for Kali Linux,   in my case I use a __qemu__ vitual image , but you can use any Kali installation media that you have.
Here the [link](https://cdimage.kali.org/kali-2023.1/kali-linux-2023.1-qemu-amd64.7z)  to download Kali .


* If you have a Kali Linux setup to install, you can start the installation with the `Virtual Manager` application.
* Use the `Create New Virtual Machine` button .

[![cvm1.png](https://i.postimg.cc/3Jcnbmrs/cvm1.png)](https://postimg.cc/JGjNsykx)


* Check `Import Existing disk image` if you are using a virtual image like me.

[![import-image-kvm.png](https://i.postimg.cc/kGMqHkKV/import-image-kvm.png)](https://postimg.cc/2btsb9Hm)


* You hit Browse to access your file manager, then you locate the pre-built image. The steps are the same for the VYOS installation .

[![cvyos3.png](https://i.postimg.cc/brW8Jd8s/cvyos3.png)](https://postimg.cc/64rFbW8X)

[![browse444.png](https://i.postimg.cc/pVFnq0y5/browse444.png)](https://postimg.cc/6TB3QfrB)

[![qemu-kali-install.png](https://i.postimg.cc/2jwP98NZ/qemu-kali-install.png)](https://postimg.cc/CdRPz01h)


* After the selection of the installation media it should look like this. 

[![qemu-intall-kali-browse.png](https://i.postimg.cc/ZK7Y2zyw/qemu-intall-kali-browse.png)](https://postimg.cc/QH7rKPm7)


* The CPU and RAM setting .

[![kvm-ubuntu-cpu.png](https://i.postimg.cc/pd1Ld3g6/kvm-ubuntu-cpu.png)](https://postimg.cc/nsBJRSV4)


* To finish you set name and you put this machine in the LAN by using `private` as network switch .

[![qemu-kali-install7.png](https://i.postimg.cc/Yqz4nPp9/qemu-kali-install7.png)](https://postimg.cc/ZC0K0Vvt)


* A successfull installation will redirect you to the Kali Linux Grub .

[![kali-grub.png](https://i.postimg.cc/CKrBSntf/kali-grub.png)](https://postimg.cc/zLWXkvqJ)


* The default user is 'kali' with passwork 'kali' .

[![kali-login.png](https://i.postimg.cc/0QDscrTw/kali-login.png)](https://postimg.cc/348VwrJ8)


* Open a terminal in the Kali machine; on the terminal, use `ip link show` to see the name of your network interface. My interface is named __eth0__, and knowing that, I can start to set a static IP. With sudo, open the file `/etc/network/interface` and edit it as follows:
```shell
# This file describe the network interfaces availabel on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static 
    address 192.168.152.21
    netmask 255.255.255.0
    network 192.168.152.0
    broadcast 192.168.152.255
    gateway 192.168.152.10
    dns-nameservers 192.168.152.10

```

[![kali-net-setting.png](https://i.postimg.cc/L64KcWnN/kali-net-setting.png)](https://postimg.cc/w3rGJ05m)


* Save the file, then run `sudo systemctl restart networking` to apply the change.You can now see the change with `ip a`.

[![kali-static-ip.png](https://i.postimg.cc/sgwrWS56/kali-static-ip.png)](https://postimg.cc/18n2h8Cw)


* Check for hostname resolution with `nslookup google.com` you should see the address of the vyos router in the output .

[![kali-nslookup.png](https://i.postimg.cc/MGYmTx4V/kali-nslookup.png)](https://postimg.cc/nsCDS8nh)



# Installation of Metasploitable2

Metasploitable2 (Linux) Metasploitable is an intentionally vulnerable Linux virtual machine. This VM can be used to conduct security training, test security tools, and practice common penetration testing techniques. 

First, you have to [download](https://sourceforge.net/projects/metasploitable/files/latest/download) the operating system if it is not already done. After extracting the file, you can use the Virtual Manager user interface to install the machine.

[![cvm1.png](https://i.postimg.cc/3Jcnbmrs/cvm1.png)](https://postimg.cc/JGjNsykx)


* The installation media is a virtual image .

[![import-image-kvm.png](https://i.postimg.cc/kGMqHkKV/import-image-kvm.png)](https://postimg.cc/2btsb9Hm)


[![browse444.png](https://i.postimg.cc/pVFnq0y5/browse444.png)](https://postimg.cc/6TB3QfrB)

[![browse-msf.png](https://i.postimg.cc/76Yts5H9/browse-msf.png)](https://postimg.cc/Z92L0Kyv)


* If you are done with the operating system selection you can proceed .

[![dvwa-kvm-install.png](https://i.postimg.cc/FF8vGJFW/dvwa-kvm-install.png)](https://postimg.cc/Sj6HxRMC)


* Setting CPU and RAM according to the host machin specifications .

[![dvwa-cpu-kvm.png](https://i.postimg.cc/C1mhFF8R/dvwa-cpu-kvm.png)](https://postimg.cc/qh6VwHjr)


* Finally, give a name to the machine and include it in the LAN by using the __private__ isolated network as an interface.

[![metasploitable-kvm-install.png](https://i.postimg.cc/q7ZrYmHK/metasploitable-kvm-install.png)](https://postimg.cc/B8DRPB6Z)


* The installation should be ok. Use the default username 'msfadmin' and password 'msfadmin' to log in.

[![msf-login.png](https://i.postimg.cc/Dzbg1Z6X/msf-login.png)](https://postimg.cc/sBs7r3tj)


* Use `sudo vim /etc/network/interfaces` to set a static ip for the Metasploitatble2 machine. In vim, you have to enter insert mode with __i__ to edit ; to save and quit, you use __Esc__; then __:qw__, make the following change into the file :

[![debian-static-net-setting.png](https://i.postimg.cc/jS6Q007J/debian-static-net-setting.png)](https://postimg.cc/v4B6fKXQ)


* Apply the change with `sudo /etc/inid.d/networking restart`

[![debian-restart-networking.png](https://i.postimg.cc/j2zQsGF9/debian-restart-networking.png)](https://postimg.cc/GTmsKSrJ)

* Verify the setting by doing nslookup and making sure that the vyos address `192.168.152.10` is present in the output 

[![nslookup.png](https://i.postimg.cc/wxcmjcBS/nslookup.png)](https://postimg.cc/hhvjCmTr)



## Conclusion 

If you followed all the steps, you should have up to five operating systems on your machine, counting your guests.

[![final-setup.png](https://i.postimg.cc/VkpypBr6/final-setup.png)](https://postimg.cc/jDHkJfh0)


In conclusion, creating a virtual laboratory for penetration testing  is a great way to ensure a safe and efficient environment for carrying out various testing procedures. With KVM, you can easily create and manage virtual machines and provide yourself with a complete isolated environment.  

It is important to note that penetration testing should only be done with the proper authorization and consent of the target organization. Otherwise, it can be considered illegal and unethical. This emphasizes the importance of owning your own laboratory as an ethical hacker to avoid complications with the law.

If you're interested in learning more about cybersecurity and related topics, check out our other articles on this blog. In particular, you may want to read our article on the implementation of a botnet, which can be used for various purposes, including distributed denial-of-service (DDoS) attacks and spam campaigns, the perfect exercice to start experimenting with your virtual laboratory .  Click here to read the article: [simple-botnet](https://0script.netlify.app/simple-botnet/).

Thank you for reading, and we hope this article has been informative and helpful. If you want to propose any improvement or if you notice any mistakes, contact me via <a href="mailto:z5r00script.com">Email</a>.
