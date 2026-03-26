---
layout: post
title: Free Software Development (MAC0470) Tutorial 1
date: 2026-03-17 20:52 -0300
description: Comments and difficulties regarding my experience in Tutorial 1 of the discipline "Free and Open Source Software Development" (MAC0470) of IME-USP
image: 
category: academic
tags: 
published: true
sitemap: true
---

# Tutorial 1

## Introduction

One of the objective of the discipline is to make a contribution to the Linux Kernel. For this, we will focus on contributing to the IIO subsystem. Notice that, although computers in general (besides from MacBooks), are equiped with the amd64 architecture. However, the IIO subsystem uses the arm64 architecture, an important detail as we will see later in this post.

This tutorial has the goal of setting up the environment for developing Linux Kernel. Hence, we will:
1. Create a Debian arm64 virtual machine (VM) using libvirt to create and manage it
2. Configure the SSH access from the local machine (host) to the VM (guest) (this will bring a valuable learning ;) )

I followed the tutorial created by FLOSS at USP ( [FLUSP](https://flusp.ime.usp.br/) ) available [here](https://flusp.ime.usp.br/kernel/qemu-libvirt-setup/).

## Experience in the tutorial

### Step 1: Creating the VM

As mentioned earlier, I used libvirt to create and manage the VM. It didn't bring as much problem as the next step, however, I managed to encounter two minor errors:

1. The default password for the root user in the VM I created wasn't actually empty, it was something else that only linus himself would be able to find out. Therefore, I had to use the commands: 

```bash
sudo apt update && sudo apt install libguestfs-tools
sudo virt-customize -a "${VM_DIR_ARM}/arm64_img.qcow2" --root-password password:<my_new_password>
```

in order to reset the password from the local machine using libvirt.

2. cloud-init trying to communicate with a metadata server: Everytime I booted the VM, it kept displaying multiple messages about cloud-init not being able to connect to a server. Later, with the help of Google Gemini this time, I figured it out that this command had to do with the expectation brought when we upload VMs in actual production servers, that try to communicate with metadata servers. This increased considerably my VM's booting time and, therefore, I shut this service down by simply running the command after booting into the VM: `touch /etc/cloud/cloud-init.disabled`


These two errors were great for me to get to know more about how the VM is set up and the possible bugs when uploading a local VM.

### Step 2: Configuring SSH access

As I was trying to set the ssh access from my local machine to the VM, I kept seeing the bug "client loop: send disconnect: Broken pipe". This message was the reason I figured out that the detail of me developing an arm64 machine from an amd64 machine is **really important**. I say this because, basically, what happens is, since my local machine can't understand arm64 commands, there exists a program that translates the instructions from the VM to my local. This process increases a lot the time that commands take to run. After **REALLY LONG HOURS**, I discovered that, due to this longer execution time, a security service called PAM was breaking the pipe of execution of ssh. 

The a little bit more detailed explanation for this is that PAM rely on several random numbers generations in order to generate session keys. Hence, if this translation from arm64 to amd64 is too slow, operations envolved in the communication of ssh and systemd-login (linux login program) that normaly PAM would predict to take 0.01 second, may take 2 seconds or more, which will indicate that the service is dead to PAM. This will lead PAM to kill the connection attempt for safety.

To fix this bug, I simply deactivated PAM by changing the configuration file on `/etc/ssh/sshd_config`. 

The only way out I found was this, since the cause of the problem is inherent to the kind of development I'm doing. Also, I can safely do this, since I'm jusst using this VM in my computer and to develop the linux kernel, so there must be no worry regarding the state of this VM.

As we can see, I learned valuable lessons regarding the functioning of developing an arm64 machine from an amd64 machine.

## Conclusion

Having completed this tutorial, I look back to it and see that it was really fun to learn these things about the communication between my local machine and my VM. Obviously there's much more to learn about this and I'm eager to see what more I'll learn on this discipline!
