---
layout: post
title: Free Software Development (MAC0470) Tutorial 2
date: 2026-03-24 13:14 -0300
description: Comments and difficulties regarding my experience in Tutorial 2 of the discipline "Free and Open Source Software Development" (MAC0470) of IME-USP
image: 
category: academic
tags:
published: true
sitemap: true
---

# Tutorial 2

## Introduction

After setting the environment up for developing the kernel, the next step in this journey of contributing to the Linux Kernel is to learn how to **configure, compile** and **boot** a customized kernel. Just as a reminder, we will develop for the IIO subsystem kernel tree, one of many Linux Kernel trees (kernel trees are different repositories containing the source code of Linux Kernel). For example, the IIO subsystem is widely applied in the industrial field.

In order to learn how to do these three major tasks, I followed the sequence of steps:

1. Install kw (a system that helps developers of the linux kernel reducing development overhead)
2. Clone a linux tree (IIO subsystem in our case)
3. Configure kw to develop on a local context (set up ssh connection between host and guest)
4. Configure and build the custom linux kernel with kw (20 minutes to compile)
5. Install the modules and boot the customized linux kernel


Once again, I followed the second tutorial created by FLOSS at USP ( [FLUSP](https://flusp.ime.usp.br/) ) from the series of developing for the IIO subsytem. The tutorial is available [here](https://flusp.ime.usp.br/kernel/build-linux-for-arm-kw/).

## Experience in the tutorial

Although this tutorial had 5 steps, I only faced difficulties (a lot of them btw) on the last step, something not really expected. Anyway, let's dive deeper on these challenges:

### `kw` scope

As the first and easily fixable error, after I finished setting up kw inside my `$IIO_TREE` directory, for some reason, I tried to run kw ssh on this iio directory's parent directory. The result:

```
$ kw ssh
We could not reach the remote machine by using:
 IP: localhost
 User: root
 Port: 22
Please ensure that the above info is correct.
Suggestion: Check if your remote machine permits root login via ssh
or check if your public key is in the remote machine.
```
The reason behind this error is a simple concept called "local context". Basically, kw works just like git in this manner, so, if you set it from a directory, most of your settings will only be available in the local environment, hence, the directory you set everything up (somethings like the .config file are considered in a "global" context). 

Being honest, who identified this error was the teacher assistant of this discipline, who helped me a lot with this and one of the following errors. Moreover, if you want to diagnose if is it this that is happening to you, you could run:

```bash
kw remote --list
```

In your current directory and in the `$IIO_TREE` directory. This will list every host your kw system can see. To fix this, I simply went to the correct directory and ran the command `kw ssh`.

### Network issues (starting with no console issues)

As I was doing the last step of this tutorial (Installing modules and booting the custom-built Linux kernel), there's a part in which you delete your current VM and recreates a VM with your custom kernel. This was where it all started. First, when I did this, the major problem was:

- I initialized the custom kernel VM and it simply didn't connect my console to the VM's console.

Instead of asking for help (which I did after the following first attempt), I thought that the best solution would be to recreate the VM with the old kernel just to make sure it's all working. After this recreation, I managed to connect my console to the VM, however, everytime I'd try to connect to my VM using ssh, I would see the following error:

```bash
ssh: connect to host <VMs-IP-ADDR> port <Nr-Port>: No route to host
```

First, I tried to analyse the ssh logs both in my host and in my VM, but it led nowhere. After, with the help of Google Gemini, I managed to identify the problem: Basically, there were two factors:

1. My VM's netwrok interface was with the state DOWN, in other words, it was sleeping
2. The IP address my host was looking for was the previously deleted VM's address, so it was basically looking for a ghost

How I diagnosed these errors:

I checked the status of the VM's internet interface and it's IP address using the command:

```bash
@Guest (VM)
ip addr
```

And I checked the IP my host was looking for through libvirt using:

```bash
@Host
sudo virsh net-dhcp-leases default
```

While the VM was running (important detail).

To fix this first network issue, basically, we have to make the internet interface always boot up (otherwise, we will have to boot it up manually all the time) and create a static IP address for the VM in order to facilitate the access (at least, setting a static IP was the solution I found and it seems good enough, given that I will just run this VM in my local network).

#### Making the VM boot up the network interface on boot:

As the first step, I had to figure out what network service my VM was using (NetworkManager? Netplan? systemd-networkd?). To do this, I manually checked if each of this services was running using `systemctl status <service-name>`, hence, the command that showed me that `systemd-networkd.service` is the one my VM is using was:

```bash
systemctl status systemd-networkd.service
```

Having this information in hands, according to systemd-networkd documentation and a post on the legendary Stack Overflow, I had to create a network configuration file that had priority over other network configuration files. The name of my VM's network interface is en1ps0. So, in order of forcing it to have priority, I had to make it be lexically prior to the other files, so I named it `1-en1ps0.network` and placed it under the directory `/etc/systemd/network/`.

Now, we have all the structure we need for the interface to start on boot. However, we actually need to write on this configuration file, which will also solve the second factor we noticed:

#### Setting the network config file and Making the IP address static

To accomplish the goals of making not only the configuration file work but also the connection issues relating to the update of the IP address of the VM from time to time disappear, we must write the configuration file correctly. For writing on this file, I ran:

```bash
sudo nano /etc/systemd/network/1-enp1s0.network
```

Since I chose to create a static IP address in detriment of using DHCP, the configuration file looked like this:

```bash
[Match]
Name=enp1s0

[Network]
DHCP=no
Gateway=<GATEWAY-ADDR>
Address=<IP-ADDR>
```

To get both the gateway and ip addresses, I ran:

```bash
ip r
```

looking for the part "default via <GATEWAY-ADDR>", and:

```bash
ip addr
```

looking for the part "inet: <IP-ADDR>". 

[This article](https://askubuntu.com/questions/1391256/systemd-networking-basics-cannot-switch-from-dhcp-to-static-ip-address) was extremely helpful during this process.

After that, I just restarted the networking service on my VM with: 

```bash
sudo systemctl restart systemd-networkd.service
```
And it worked!! Now, I was able to ssh to the virtual machine using the ip address I configured on kw. Since it won't change, this problem is solved (I hope).

#### Solution for not connecting the console

As I talked to the teacher assistant of this discipline, he explained to me that this is a common problem, happening to a lot of students. His recommendation was to just keep connecting to the VM using ssh after waiting a while after starting the VM to let it boot properly, which simply solve the problem of lacking access to the VM!

## Conclusion

This tutorial has taught me some things about networking, what makes me really satified with the experience. In the end of the day, now, if I want to test my changes on the IIO subsystem kernel, I "simply" compile it (it took 20 minutes to do this in this tutorial), install the modules on the VM, shutdown and boot again the VM, which is a kind of a simple process now that I've done it once :). Finally, I'm really eager to see what is waiting for me in the next tutorial and what I'll know in the end of this semester.

