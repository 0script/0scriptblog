---
title: "Step by Step Guide to Build Virtual Lab In Linux With Kvm"
date: 2023-02-10T14:19:56+02:00
draft: false
---

![Virtual Lab](https://benisnous.com/wp-content/uploads/2021/03/how-to-build-a-HACKING-lab-to-become-a-hacker.jpg)
# Content 
* Overview
* What you'll need 
* Installation of a hypervisor 


# Overview
A virtual lab is an isolated space in which ethical hacker and security researcher can perform experimentation without causing any arm .
In this article I will show you how to create your own virtual lab under a linux system .

# What you'll need 

* A computer with enough power  at least i3 not older than 2015 a minimum of 8gb or ram a swap memory(virtual ram) will be plus . As you can my set up
consist of AMD Ryzen 7 cpu with 16GB of RAM and 22GB of swap memory so I'm good to proceed 

![My Computer Specs](https://i.redd.it/nzsn80mofzha1.png)
* A computer with a linux distribution (Debian based prefered)
* A basic understanding of the linux shell
* How to use text editor like vim or nano
* How to install a package using a package manager like apt, aptitude ,pacman etc.

* You can Download all the operating system needed with the following links :
    * __VyOs__ a open source router and firewall that you can  [download here](https://s3-us.vyos.io/rolling/current/vyos-1.4-rolling-202302110324-amd64.iso)
    * a virtual image for qemu of __kali linux__  linux distribution for ethical hacking you can  [download it here](https://cdimage.kali.org/kali-2022.4/kali-linux-2022.4-qemu-amd64.7z)
    * __metasploitable2__ a linux server with security vulnerability that you can  [download on this website](https://sourceforge.net/projects/metasploitable/files/Metasploitable2/)
    * A additional desktop here i'll use __CrunchBang__ a liightway debian based distribution with great performance and a good looking UI dowload the 64 bit virtual image version for virtual box on [the following site](https://www.osboxes.org/crunchbang/#crunchBang-11-waldorf-vbox) . 

# Installation of hypervisor
A hypervisor is a software program that allows you to create and run virtual machine .
In a virtual machine , the hardware components  will be simulated by your [Hypervisor](https://www.vmware.com/topics/glossary/content/hypervisor.html) . Basically , the hypervisor will run on your main computer (_the host_) and present your virtual machine (_guest_) with some virtual hardware so your _guest_ will believe and act as if it was one computer by its own .
Linux based operating system supports all the mainstream hypervisor like [VirtualBox](#https://www.virtualbox.org/) and [VMWare](#https://www.vmware.com/) , but for better performance it is recommended to use [KVM](#https://www.linux-kvm.org/page/Main_Page) the linux Kernel Based Virtualization  Machine , a virtualization solution for Linux system running on AMD or Intel CPU who have virtualization support  enabled .

1. First let's check if your computer support virtualization with the following command `$egrep -c '(vmx|svm)' /proc/cpuinfo`
    * With the __egrep__ commande you check if your CPU support virtualization , this command work for both intel and amd cpu by checking if one of the virtualisation support is in __/proc/cpuinfo__  

    * If the command return 0 you'll need to [activate virtualisation](#https://helpdeskgeek.com/how-to/how-to-enable-virtualization-in-bios-for-intel-and-amd/) support in the BIOS setting of your computer  

    * as example , the svm flag appears 16 time in my /proc/cpuinfo file   

    ```shell
    $egrep -c '(svm|vmx)' /proc/cpuinfo
    16
    ```
2. Install the required package for kvm  
    ```shell
    $sudo apt-get install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils
    ```

3. Add your current user to the __libvirt__ group
    ```shell
    $sudo adduser 'username' libvirt
    ```
    * Replace 'username' with the corresponding user for your machine then restart or relog in to make the change effective .

4. Check the installation by tipping the command `virsh list --all` , here is a example of the output on my machine :
    ```shell
    $virsh list --all
    Id   Name   State
    --------------------
    ```
    * If your output there is a error , make sure that :
        1. You reloged in or restarted your computer after adding user to __libvirt__ group ,
        2. Check if the __libvirt__ daemon is enabled with `$systemctl status libvirtd.service | grep Active` 
            ```shell
                $sudo systemctl status libvirtd.service |grep Active
                Active: active (running) since Fri 2023-02-17 19:42:15 CAT; 53min ago
            ```
            * If you see __inactive__ in the output you'll need to activate libvirtd with `systemctl start libvirtd.service`

5. As last we install a graphic interface for the KVM hypervisor : `sudo apt-get install virt-manager`

# Setting up virtual network interface 

In our network we will have 2 virtual switch , one in NAT mode that will have access to internet and one in isolated mode.

The VyOs router will be connected to both the NAT network and the private network and will allow the machine on the private network to access the internet .
[![virt-lab.png](https://i.postimg.cc/fT1cdvbT/virt-lab.png)](https://postimg.cc/HV48DX5f)

## Activating default NAT interface 
The __libvirt__ package comme with a default NAT configuration that we can activate with the following step :
* We load the xml configuration  
```shell
$sudo virsh net-define /usr/share/libvirt/network/default.xml
```
* than we can activate the interface 
```shell
$sudo virsh net-start default
[sudo] password for 0script:                            
Network default started     
```
You can check for libvirt interface using `virs net-list`  , this command will show only activated interface : 
```shell
$sudo virsh net-list
[sudo] password for 0script: 
 Name      State    Autostart   Persistent
--------------------------------------------
 default   active   no          yes
```
## Creation and activation of private isolated interface 
We will use the xml format to create a new virtual network configuration for an isolated network , to do so create a file named `private.xml` and open it with a text editor and put the following inside :  
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
* name : Interface name
* bridge : Virtual switch name libvirt create a default one named virbr0
* ip : Define ip address type (v4/v6) address range and netmask
* dhcp : Setting dhcp for the interface

Now we can define and activate the virtual interface with  `virsh net-create private.xml`
```shell
$sudo virsh net-create private.xml 
[sudo] password for 0script:
Network private created from private.xml
```

Now if everything went as expected you should see both of our virtual interface with `sudo virsh net-list`
```shell
$sudo virsh net-list 
[sudo] password for 0script: 
 Name      State    Autostart   Persistent
--------------------------------------------
 default   active   no          yes
 private   active   no          no
```

# Installation of VyOs router

Use this [link](https://vyos.io/subscriptions/software) to download the router iso file , take the free version .

* Once the iso file downloaded we can start the installation  the vyos router ,on 
    your computer search for the virt manager application and run it   

    ![Openning virtual manager](https://i.postimg.cc/7Zb2cBgj/0001.png)

    * Select create a new virtual machine on the top left button
    [![cvm1.png](https://i.postimg.cc/3Jcnbmrs/cvm1.png)](https://postimg.cc/JGjNsykx)

    * Click forward as the media of installation for vyos is an iso file 
    ![Create New Virtual Machine](https://i.postimg.cc/8Pv5rqmp/0002.png)

    * select browse access the installation media   
    [![cvyos3.png](https://i.postimg.cc/brW8Jd8s/cvyos3.png)](https://postimg.cc/64rFbW8X)

    * Click the button `Browse Local` to access file on your local machine 
    * Go to the folder containing the iso file for vyos that you [downloaded](https://s3-us.vyos.io/rolling/current/vyos-1.4-rolling-202302110324-amd64.iso)
    [![browse444.png](https://i.postimg.cc/pVFnq0y5/browse444.png)](https://postimg.cc/6TB3QfrB)

    [![browse.png](https://i.postimg.cc/TPgx5WCS/browse.png)](https://postimg.cc/qtJS9gfX)

    * Once the image selecter , uncheck the automatic selection of os and enter Generic Os in the search bar  once done , it should look like this  , then i click forward to proceed 

    [![01.png](https://i.postimg.cc/V6D02N27/01.png)](https://postimg.cc/hzQGdKDx)

    * Now we are setting the virtual cpu and ram for or use case 2 cpu and 1gb of ram should be enough  and just after for the virtual hard drive I put 8gb 
    
    [![02.png](https://i.postimg.cc/XvG651kQ/02.png)](https://postimg.cc/HJgND2Ky)

    [![03.png](https://i.postimg.cc/597BjPjX/03.png)](https://postimg.cc/Z0NBMx8m)
   
   * Name the machine and select as virtual network interface the  default nat network device [provided by the kvm packages](https://www.ibm.com/docs/en/linux-on-systems?topic=choices-kvm-default-nat-based-networking)

    [![04.png](https://i.postimg.cc/NG5bcyxq/04.png)](https://postimg.cc/qgH2cvYj)

    * Once the installation is finished on the vyos machine press enter to run live mode 
    
    [![04.png](https://www.linuxcompatible.org/data/publish/201/d89a50e839b4319a6a279bcb82b354573aff2b/7vyos.jpg)](https://www.linuxcompatible.org/data/publish/201/d89a50e839b4319a6a279bcb82b354573aff2b/)


    * Use default user "vyos" and password  "vyos" to log in 

    [![07.png](https://i.postimg.cc/zXbYQXQ0/07.png)](https://postimg.cc/8jSYJDN6)

## Setting up VyOs

### Install image 

* To complete the installation you need to run the commande `install image` on the shell using the installation wizard
    
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

* We need to restart the machine with the command `reboot` in order to make the change effective :
```shell
vyos@vyos:~$ reboot
Proceed with reboot? (Yes/No) [No] Yes
```


### Adding private network interface 

On the virt manager interface click on the VyOs machine than use the run bouton on top followed by the open button to have access to your machine .

[![01run.png](https://i.postimg.cc/43V7kFqH/01run.png)](https://postimg.cc/7CPYSXxq)

You type Enter than you log in to access the VyOs shell 
[![vyos-v-1-4-grub.png](https://i.postimg.cc/DzG5ZXxJ/vyos-v-1-4-grub.png)](https://postimg.cc/gwY3tnFm)

Firt we will add the private network interface to the VyOs router , on the vyos window 
`Show Virtual Hardware detail > Add Hardware > Network > Network Source > Select the private Network Interface > Finish `

[![00-vyos-in-qemu.png](https://i.postimg.cc/KzH1zBbX/00-vyos-in-qemu.png)](https://postimg.cc/WF0NYDtS)

[![01-virt-manager-hardware-detail.png](https://i.postimg.cc/tgCq0JxL/01-virt-manager-hardware-detail.png)](https://postimg.cc/w1Z8L6FV)

[![02-virt-manager-setting.png](https://i.postimg.cc/prBctW86/02-virt-manager-setting.png)](https://postimg.cc/1VfrGZvG)

[![03-virt-man-hardware-setting.png](https://i.postimg.cc/J4vYV2Bm/03-virt-man-hardware-setting.png)](https://postimg.cc/svP45Kc0)

Once done return to the VyOs shell with top left button `Show The Graphical Console`

[![04-virt-manager-hardware-detail.png](https://i.postimg.cc/QxC5kNCx/04-virt-manager-hardware-detail.png)](https://postimg.cc/GH0HbCvZ)

On the VyOs console reboot the machine to make the change effective 
```shell
vyos@vyos:~$ reboot
Proceed with reboot? (Yes/No) [No] Yes
```

### VyOs Configuration of the LAN
* On the VyOs machine check if you have 2 network interface installed : 
[![ip-a-command.png](https://i.postimg.cc/8Pyt4NX2/ip-a-command.png)](https://postimg.cc/kBbQX3ms)

We will set up our network as with eth0 as WAN interface that will communicate with the outside word and receive its address via DHCP 

And eth1 internal/LAN  will use a static IP address of 192.168.0.1/24.


Firts of all you enter in configuration mode with the command `configure` you know you are in configuration mode when the shell display __#__ 

```shell
vyos@vyos$ configure
vyos@vyos#
```

#### DHCP/DNS setting

In configure mode we start by setting dhcp for eth0 and a static ip for eth1  after each command a single line with __[edit]__ confirming that the command have been executed succefully 

```shell
set interfaces ethernet eth0 address dhcp
set interfaces ethernet eth0 description 'OUTSIDE'
set interfaces ethernet eth1 address '192.168.152.10/24'
set interfaces ethernet eth1 description 'INSIDE'
```

[![1-vyos-set-dhcp-dns.png](https://i.postimg.cc/SK44KVP4/1-vyos-set-dhcp-dns.png)](https://postimg.cc/xX6hxGYp)

##### Address setting

 Now as we are still in configuration mode we will set  the DHCP and DNS services on your internal/LAN network, where VyOS will act as the default gateway and DNS server.

* The default gateway and DNS recursor address will be 192.168.152.10/24
* The address range 192.168.152.2/24 - 192.168.152.49/24 will be reserved for static assignments
* DHCP clients will be assigned IP addresses within the range of 192.168.152.50 - 192.168.152.254 and have a domain name of internal-network
* DHCP address will be leased for 24 hours i.e __86400__ secondes
* VyOS will serve as a full DNS server, replacing the need to utilize Google, Cloudflare, or other public DNS servers (which is good for privacy)
* Only hosts from your internal/LAN network can use the DNS server


```shell
set service dhcp-server shared-network-name LAN subnet 192.168.152.0/24 default-router '192.168.152.10'
set service dhcp-server shared-network-name LAN subnet 192.168.152.0/24 name-server '192.168.152.10'
set service dhcp-server shared-network-name LAN subnet 192.168.152.0/24 domain-name 'vyos.router'
set service dhcp-server shared-network-name LAN subnet 192.168.152.0/24 lease '86400'
set service dhcp-server shared-network-name LAN subnet 192.168.152.0/24 range 0 start 192.168.152.50
set service dhcp-server shared-network-name LAN subnet 192.168.152.0/24 range 0 stop '192.168.152.254'
```

[![2vyos-address-setting.png](https://i.postimg.cc/NGZCkFkC/2vyos-address-setting.png)](https://postimg.cc/qgXGkkY2)

##### Setting SNAT rule

* The following settings will configure SNAT rules for our internal/LAN network, allowing hosts to communicate through the outside/WAN network via IP masquerade than commit and save 

```shell
set nat source rule 100 outbound-interface 'eth0'
set nat source rule 100 source address '192.168.152.0/24'
set nat source rule 100 translation address masquerade
commit
save
```
[![3vyos-set-nat-rule.png](https://i.postimg.cc/dVVWqCs1/3vyos-set-nat-rule.png)](https://postimg.cc/TpBj0pWM)

##### Setting DNS forwarding 
* We activate DNS service on eth1
```shell
set service dns forwarding listen-address 192.168.152.10

set service dns forwarding allow-from 192.168.152.0/24

set service dns forwarding cache-size 10240
```

[![4-vyos-set-dns-forwarding.png](https://i.postimg.cc/YSYGmX8C/4-vyos-set-dns-forwarding.png)](https://postimg.cc/B8ZQrBdr)

##### Adding DNS server

* Now run the following command to use Cloudfare Google and Quad9 dns server

```shell
set service dns forwarding name-server 1.1.1.1
set service dns forwarding name-server 8.8.8.8
set service dns forwarding name-server 9.9.9.9
```

* Specify a DNS server for the system to be used for DNS lookups than commit save and exit 


```shell
set system name-server 192.168.152.10
commit
save
exit
```

##### Testing network 

* check for interface : `show interface`

[![vyos-show-interfaces.png](https://i.postimg.cc/436XXzL0/vyos-show-interfaces.png)](https://postimg.cc/rdwBJtLG)

* verify network connection with host vyos.org
[![host-cmd.png](https://i.postimg.cc/KjRX3Lfx/host-cmd.png)](https://postimg.cc/SYF1wnHv)

* verify nat rule : `show nat source rule`

[![show-nat-source.png](https://i.postimg.cc/mrNmLDQj/show-nat-source.png)](https://postimg.cc/gwjV4GtL)

* And now we verify the dns setting configuration by entering configuration mode `configure` and then `show service dns`
[![show-service-dns.png](https://i.postimg.cc/RFtKSb5B/show-service-dns.png)](https://postimg.cc/ftwJ5CR2)

# Installation of ubuntu desktop

I use  virtual box image for the installation that is avalaible on [osboxes](https://www.osboxes.org/virtualbox-images/) use this [link](https://sourceforge.net/projects/osboxes/files/v/vb/55-U-u/22.10/64bit.7z/download) to dowload the ubuntu virtual image 

Once the set up downloaded we can start the installation using the __virt manager__ user interface 

* Use the create machine button on virt manager

[![cvm1.png](https://i.postimg.cc/3Jcnbmrs/cvm1.png)](https://postimg.cc/JGjNsykx)

* Check import existing disk image to use the virtual box virtual image as for me I'm using a virtual box image `.vdi`

[![import-image-kvm.png](https://i.postimg.cc/kGMqHkKV/import-image-kvm.png)](https://postimg.cc/2btsb9Hm)

* You hit browse to access your file manager than you locate the virtual image the step are the same then for the vyos installation 

[![cvyos3.png](https://i.postimg.cc/brW8Jd8s/cvyos3.png)](https://postimg.cc/64rFbW8X)

[![browse444.png](https://i.postimg.cc/pVFnq0y5/browse444.png)](https://postimg.cc/6TB3QfrB)

[![browse-ubuntu.png](https://i.postimg.cc/QtDd2XDc/browse-ubuntu.png)](https://postimg.cc/2qHYvNV5)

* once the file selected click forward

[![kvm-ubuntu01.png](https://i.postimg.cc/d1CSyYZN/kvm-ubuntu01.png)](https://postimg.cc/XGVgRt89)

* here you set the memory and cpu

[![kvm-ubuntu-cpu.png](https://i.postimg.cc/pd1Ld3g6/kvm-ubuntu-cpu.png)](https://postimg.cc/nsBJRSV4)

* Because this is a virtual image instead of an iso file there is no hard drive setting so we end up by using the private virtual interface for our ubuntu desktop so it will be part of the LAN 

[![kvm-ubuntu-network.png](https://i.postimg.cc/kGCPsdzG/kvm-ubuntu-network.png)](https://postimg.cc/ZBwQYXTz)

* to login if you downloaded the virtual image from osboxes the default username and password are 'osboxes.org' 

[![kvm-ubuntu-login.png](https://i.postimg.cc/J0WV4WQm/kvm-ubuntu-login.png)](https://postimg.cc/9R1NJKSN)

* once logged in use Crtl+Alt+T to open a terminal 

[![ubuntu-kvm-shell.png](https://i.postimg.cc/G2f0gbJx/ubuntu-kvm-shell.png)](https://postimg.cc/HcbPVDNr)

* check the name of your ethernet interface with `ip link show` this name will be very important for the next part 

[![ip-link-show1.png](https://i.postimg.cc/TwjGkK3M/ip-link-show1.png)](https://postimg.cc/r0pvsF4Q)

* we will set a static route , to do so we open the file `/etc/netplan/01-network-manger-all.yaml` as root in a text editor and you make the following change
    * Replace __enp1s0__ with the name of your own interface
    * In this configuration we set a static ip : __192.168.152.15__ . And  default gateway and dns server 

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

* To apply the change in a shell with run `sudo netplan apply` follower by `sudo systemctl restart NetworkManager` and check hostname resolution with `host google.com`

[![default-gateway.png](https://i.postimg.cc/GmCcD0PS/default-gateway.png)](https://postimg.cc/CzcTyQQj)


## Installation of kali linux 

* We download a virtual image of kali linux made for qemu on the official [website](https://cdimage.kali.org/kali-2023.1/kali-linux-2023.1-qemu-amd64.7z) 

* once the image downloaded and extracted we can follow the same step as for the installation of the ubuntu desktop 

* Use the create machine button on virt manager

[![cvm1.png](https://i.postimg.cc/3Jcnbmrs/cvm1.png)](https://postimg.cc/JGjNsykx)

* Check import existing disk image to use the virtual box virtual image as for me I'm using a qemu virtual image `.qemu`

[![import-image-kvm.png](https://i.postimg.cc/kGMqHkKV/import-image-kvm.png)](https://postimg.cc/2btsb9Hm)

* You hit browse to access your file manager than you locate the virtual image the step are the same then for the vyos installation 

[![cvyos3.png](https://i.postimg.cc/brW8Jd8s/cvyos3.png)](https://postimg.cc/64rFbW8X)

[![browse444.png](https://i.postimg.cc/pVFnq0y5/browse444.png)](https://postimg.cc/6TB3QfrB)


[![qemu-kali-install.png](https://i.postimg.cc/2jwP98NZ/qemu-kali-install.png)](https://postimg.cc/CdRPz01h)

* now that you have selected the media of installation enter genric os as os type and proceed 

[![qemu-intall-kali-browse.png](https://i.postimg.cc/ZK7Y2zyw/qemu-intall-kali-browse.png)](https://postimg.cc/QH7rKPm7)

* Now cpu and virtual ram setting 

[![kvm-ubuntu-cpu.png](https://i.postimg.cc/pd1Ld3g6/kvm-ubuntu-cpu.png)](https://postimg.cc/nsBJRSV4)

* And to finish the virtual machine name and we select the private bridge to add this machine into our LAN

[![qemu-kali-install7.png](https://i.postimg.cc/Yqz4nPp9/qemu-kali-install7.png)](https://postimg.cc/ZC0K0Vvt)

* You should now be in front of the kali grub press Enter to go faster

[![kali-grub.png](https://i.postimg.cc/CKrBSntf/kali-grub.png)](https://postimg.cc/zLWXkvqJ)

* The default user  if you used a virtual image from kali linux official website 'kali' with password 'kali'

[![kali-login.png](https://i.postimg.cc/0QDscrTw/kali-login.png)](https://postimg.cc/348VwrJ8)

* On the kali machin open a terminal , you can use the shortcut Crtl+Alt+T

* On the terminal use `ip link show` to see the name of your interface
My interface is named eth0 knowing it I can start to set a static ip 

* In root mode open the file `/etc/network/interface` and edit it as follow :

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
    gateway 192.168.152.10
```

[![kali-net-setting.png](https://i.postimg.cc/kMkvvp7z/kali-net-setting.png)](https://postimg.cc/s14WVKQ4)

* Save the file than run `sudo systemctl restart networking` to apply the change
* You can now see the change with `ip a`
    insert image here

[![kali-static-ip.png](https://i.postimg.cc/sgwrWS56/kali-static-ip.png)](https://postimg.cc/18n2h8Cw)

* Now open  `/etc/resolv.conf` with sudo and add the dns address in my  case `192.168.152.10` put it at the top of the file 

[![kali-resolv.png](https://i.postimg.cc/D00gHDxs/kali-resolv.png)](https://postimg.cc/WqRgJ9Lb)

* Apply the change with `sudo systemctl restart networking` and check with `nslookup google.com`

[![kali-nslookup.png](https://i.postimg.cc/MGYmTx4V/kali-nslookup.png)](https://postimg.cc/nsCDS8nh)

## Installation of metasploitable 2

Metasploitable2 (Linux) Metasploitable is an intentionally vulnerable Linux virtual machine. This VM can be used to conduct security training, test security tools, and practice common penetration testing techniques. 

First you have to [download](https://sourceforge.net/projects/metasploitable/files/latest/download) installation media if it is not already done after extracting the file you can use virt manager user interface to install the machine

[![cvm1.png](https://i.postimg.cc/3Jcnbmrs/cvm1.png)](https://postimg.cc/JGjNsykx)


* The installation media is still a virtual image 

[![import-image-kvm.png](https://i.postimg.cc/kGMqHkKV/import-image-kvm.png)](https://postimg.cc/2btsb9Hm)


[![browse444.png](https://i.postimg.cc/pVFnq0y5/browse444.png)](https://postimg.cc/6TB3QfrB)

[![browse-msf.png](https://i.postimg.cc/05cxSHhs/browse-msf.png)](https://postimg.cc/K4kdSQZq)

* We define it as a generic Os to go faster 

[![dvwa-kvm-install.png](https://i.postimg.cc/FF8vGJFW/dvwa-kvm-install.png)](https://postimg.cc/Sj6HxRMC)

* Set the cpu ram

[![dvwa-cpu-kvm.png](https://i.postimg.cc/C1mhFF8R/dvwa-cpu-kvm.png)](https://postimg.cc/qh6VwHjr)

* finally we give a name to the machin and we include it in our LAN by using the private isolated network as interface 

[![metasploitable-kvm-install.png](https://i.postimg.cc/q7ZrYmHK/metasploitable-kvm-install.png)](https://postimg.cc/B8DRPB6Z)

* The installation should be ok you can use the default username 'msfadmin' and password 'msfadmin' to login

[![msf-login.png](https://i.postimg.cc/Dzbg1Z6X/msf-login.png)](https://postimg.cc/sBs7r3tj)

* Now with use `sudo vim /etc/network/interfaces` to set the network interface , in vim you have to enter insert mode with __i__ to save and quit you use __Esc__ than __:qw__ , edit the file as follow 

[![debian-static-net-setting.png](https://i.postimg.cc/jS6Q007J/debian-static-net-setting.png)](https://postimg.cc/v4B6fKXQ)

* To apply the change you use `sudo /etc/inid.d/networking restart`

[![debian-restart-networking.png](https://i.postimg.cc/j2zQsGF9/debian-restart-networking.png)](https://postimg.cc/GTmsKSrJ)

* We verify the setting by doing nslookup and making sure that the vyos address `192.168.152.10` is present in the output 

[![nslookup.png](https://i.postimg.cc/wxcmjcBS/nslookup.png)](https://postimg.cc/hhvjCmTr)


## Conclusion 