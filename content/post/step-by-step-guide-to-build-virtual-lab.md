---
title: "Step by Step Guide to Build Virtual Lab"
date: 2023-02-10T14:19:56+02:00
draft: false
---

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

![Tux, the Linux mascot](https://i.redd.it/nzsn80mofzha1.png)
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

# Installation of VyOs router 
Use this [link](https://vyos.io/subscriptions/software) to download the router setup , take the free version .
