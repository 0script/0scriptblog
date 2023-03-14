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
In our network we will have to network interfaces : 

# Installation of VyOs router 
Use this [link](https://vyos.io/subscriptions/software) to download the router iso file , take the free version .

1. We will use the libvirt default network interface but first we need to activate it 
    1. check for libvirt network interface : `sudo virsh net-list --all`
    ```shell
    $sudo virsh net-list --all
    Name      State      Autostart   Persistent
    ----------------------------------------------
    default   inactive   no          yes
    ```
    * If this command do not show you any interface you can load the libvirt default network interface with the followint :
    ```shell
    $sudo sudo virsh net-define /usr/share/libvirt/networks/default.xml
    ```
    * You should see a message confirming the creation of the interface .
    
    2. Create additional virtual interface for private network , this is the network that will be used to contain our virtual laboratory :
        1.  Copy and paste the following in a file named `private.xml` :
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
        * this file help us to define the attribute of a virtual network interface , refers to the [documentation](https://libvirt.org/formatdomain.html) for more understanding  

    2. Activate the interface  using `sudo virsh net-start default`
    ```shell
    $sudo virsh net-start default 
        Network default started
    ```

2. After setting the virtual network we can install the vyos router ,on 
    your computer search for the virt manager application and run it   

    ![Openning virtual manager](https://i.postimg.cc/7Zb2cBgj/0001.png)

    * Select create a new virtual machine on the top left button
    [![cvm1.png](https://i.postimg.cc/3Jcnbmrs/cvm1.png)](https://postimg.cc/JGjNsykx)

    * Click forward as the media of installation for vyos is an iso file 
    ![Create New Virtual Machine](https://i.postimg.cc/8Pv5rqmp/0002.png)

    * select browse access the installation media   
    [![cvyos3.png](https://i.postimg.cc/brW8Jd8s/cvyos3.png)](https://postimg.cc/64rFbW8X)

    * Click the button `Browse Local` to access file on your local machine 
    * Go to the folder containing the iso file for vyos that you [download here](https://s3-us.vyos.io/rolling/current/vyos-1.4-rolling-202302110324-amd64.iso)
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

    I found the following drives on your system:
    sda    4294MB

    Install the image on? [sda]:    # Press Enter to select default option

    This will destroy all data on /dev/sda.
    Continue? (Yes/No) [No]: Yes    # The instalation has started

    How big of a root partition should I create? (2000MB - 4294MB) [4294]MB: # Press Enter to select default option

    Creating filesystem on /dev/sda1: OK
    Done!
    Mounting /dev/sda1...
    What would you like to name this image? [1.2.0-rolling+201809210337]:
    OK.  This image will be named: 1.2.0-rolling+201809210337
    Copying squashfs image...
    Copying kernel and initrd images...
    Done!
    I found the following configuration files:
        /opt/vyatta/etc/config.boot.default
    Which one should I copy to sda? [/opt/vyatta/etc/config.boot.default]:  # Press Enter or Yes to select default option

    Copying /opt/vyatta/etc/config.boot.default to sda.
    Enter password for administrator account
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
