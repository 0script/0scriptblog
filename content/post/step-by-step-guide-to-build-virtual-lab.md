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

# Installation of VyOs router 
Use this [link](https://vyos.io/subscriptions/software) to download the router iso file , take the free version .

1. We will use the libvirt default network interface but first we need to activate it 
    1. check for libvirt network interface : `sudo virsh net-list --all`
    ```shell
    $sudo virsh net-list --all
    Name      State      Autostart   Persistent
    ----------------------------------------------
    default   inactive   no          yes
    private   inactive   no          yes # ignore this line 
    ```
    * If this command do not show you any interface you can load the libvirt default network interface with the followint :
    ```shell
    $sudo sudo virsh net-define /usr/share/libvirt/networks/default.xml
    ```
    * You should see a message confirming the creation of the interface .
    
    2. Activate the interface  using `sudo virsh net-start default`
    ```shell
        $sudo virsh net-start default 
            Network default started
    ```

2. On your computer search for the virt manager and run it  

    ![Openning virtual manager](https://lh3.googleusercontent.com/pw/AMWts8DHnkaqoGpkbwS4IGhE3-KF0tLLG1yBYwz6gXtW6G2CtpQ83s00nIXawvOZHrcDOmeIUCC28ImoNI7CjMOKs84z03QgFewmLqxdY8dDcd9MAlfkEDCKr44d85JVZ3tjQtMmKtnn2nko_RLXSyp8_b9V=w592-h356-no?authuser=0)

    * Select create a new virtual machine on the top left button
    ![Create New Virtual Machine](https://lh3.googleusercontent.com/pw/AMWts8CiQ1RZXdSwlD-ZKFK7NnKl9NDg4g4A_Uq8mTg2VURi-h3_R2KaeQVqNlBHg8JELR_PzkcVWfiKCiHYC-1JkTDYj8MM9VOpZjLZKuJqompm1SU3t7D8aHzNlCKyP7uv7gbaeEEbwbr3CTJYZ6FEDGnj=w550-h580-no?authuser=0)

    * Click forward to select the operating system iso file 
    ![Create New Virtual Machine](https://lh3.googleusercontent.com/pw/AMWts8BUCkNDYePon0aKuK-rFyf-W5lu-r2tOPuk3POEdDS-ZjLxxUu_1nuHz2tt8yrCNBCfOm_7kJsCBZCUMsTTlPeiPke26gRmWhA7pvTsu7T0KHMCNkq3_IUCXfdIh-06e-i2Bo74GRVUHfIiRHN7t800=w500-h530-no?authuser=0)

    * next select browse 
    ![Create New Virtual Machine](https://lh3.googleusercontent.com/pw/AMWts8Ah37IAgAkVVANoApKBu0Zd2H1us9b61nTnycSaL8O6SMCHpAbJaten3VHh3ikoP4n3A2SsG7TzmmkqUuhQHAJzLunNuxNshF02_flpXF0M8GDd7gdDXax-iqpVa0nbXfEBK6A4aJoL-cl1xyHQOLzE=w500-h530-no?authuser=0)

    * click browse local to access your computer file 
    ![Create New Virtual Machine](https://lh3.googleusercontent.com/pw/AMWts8DHYP6PxDL53ogppE1u9h6223pnwPBdjNH7DcH40vyOY-md4pzwhv3jllurP5ToBE73h5TpGOVj2YneK55Vs9vXkYOylLs8j35XxUlP7h4nSV0JgLuDf8vHhNgtmnkKMzQfuwxlmfQKNwni4xGquR-g=w760-h550-no?authuser=0)

    * Go to the directory that contain vyos image that you downloaded and select it 
    ![Create New Virtual Machine](https://lh3.googleusercontent.com/pw/AMWts8BWJdyCZb0z92mUF0aOFO5aD6KZtdoQor5T68SJBf-1VMos3LZDjWscmLjonRWCls9mjFXugGf4_z6vAnaOYfNWH4sfjegyIbAb7tqoluLdDAu6BXwqtaS9vDsV6PnBM1lWoLgkUNJ4x6WImC5C7y5o=w840-h664-no?authuser=0)

3.  