---
title: "Step by Step Guide to Build Virtual Lab"
date: 2023-02-10T14:19:56+02:00
draft: false
---

# Content 
* Overview

* What you'll need 
* Installation of a hyperv

# Overview
A virtual lab is an isolated space in which ethical hacker and security researcher can perform experimentation without causing any arm .
In this article I will show you how to create your own virtual lab under a linux system .

# What you'll need 

* A computer with enough power  at least i3 not older than 2015 a minimum of 8gb or ram a swap memory(virtual ram) will be plus . As you can my set up
consist of AMD Ryzen 7 cpu with 16GB of RAM and 22GB of swap memory so I'm good to proceed 

![Tux, the Linux mascot](https://i.redd.it/nzsn80mofzha1.png)

* A basic understanding of linux system
    * A computer with a linux distribution (Debian based prefered)
    * How to install a package from the command line
    * How to use text editor like vim or nano

* You can Download all the operating system needed with the following links :
    * __VyOs__ a open source router and firewall that you can  [download here](https://s3-us.vyos.io/rolling/current/vyos-1.4-rolling-202302110324-amd64.iso)
    * a virtual image for qemu of __kali linux__  linux distribution for ethical hacking you can  [download it here](https://cdimage.kali.org/kali-2022.4/kali-linux-2022.4-qemu-amd64.7z)
    * __metasploitable2__ a linux server with security vulnerability that you can  [download on this website](https://sourceforge.net/projects/metasploitable/files/Metasploitable2/)
    * A additional desktop here i'll use __CrunchBang__ a liightway debian based distribution with great performance and a good looking UI dowload the 64 bit virtual image version for virtual box on [the following site](https://www.osboxes.org/crunchbang/#crunchBang-11-waldorf-vbox) . 

# Installation of hypervisor
An hypervisor a software program that allow you to create and run virtual machine , a virtual machine is a computer who do not have any hardware on its own .
In a virtual machine the hardware components  will be simulated by your [Hypervisor](url_link){:target="_blank"} which is a basicaly a software to create and manage virtual machine. Hypervisor allows us to create a virtual machine that can act as a computer at whole.

Linux based operating system support all the mainstream hypervisor like [VirtualBox](#https://www.virtualbox.org/) and [VMWare](#https://www.vmware.com/) but for better performance it is recommanded to use [KVM](#https://www.linux-kvm.org/page/Main_Page) the linux Kernel Based Virtualisation Machine hypervisor .

1. First let's check if your computer support virtualization with the following command :