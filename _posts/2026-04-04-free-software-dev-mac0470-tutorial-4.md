---
layout: post
title: Free Software Development (MAC0470) Tutorial 4
date: 2026-04-04 19:59 -0300
description: Comments and difficulties regarding my experience in Tutorial 4 of the discipline "Free and Open Source Software Development" (MAC0470) of IME-USP
image: 
category: academic
tags: 
published: true
sitemap: true
---

# Tutorial 4

## Introduction

After learning how to work with modules, our next stop is to learn what are Character Devices in linux operating systems and how they work. First, I'll go through a brief overview of the concepts regarding linux Character Devices and, then, there'll be a description of my experience on the fourth tutorial of the series of developing for the IIO subsystem linux kernel.

Once again, I followed the series of tutorials created by FLOSS at USP ( [FLUSP](https://flusp.ime.usp.br/) ) regarding the development for the IIO subsytem. The tutorial is available [here](https://flusp.ime.usp.br/kernel/char-drivers-intro/).

## Important concepts about Character Devices

### What are Character Devices

They are an abstraction provided by Linux operating system itself that enables us to support devices that can be read from or written to (I/O). Usually abstracted as files (support normal files system calls): /dev/ttyS0 (first shell terminal), /dev/input/mouse0, /dev/kmsg and /dev/zero.

Basically, they are a concept we use to support these devices in linux. In this tutorial, we can call them: Character Devices or just Devices.

### Character Device files

Character Devices are represented as files in the linux system. Due to this file representation, we can interact with them from the user space using normal syscalls and the Device **drivers** will implement the logic to make this syscalls work for the Device. Basically, each driver will enable the Kernel to support the particular device it is responsible for. Hence, when we make a syscall to a Device file, we are interacting with the driver who manages interactions with the Device, not the Device itself.

### How Character Device files and the actual Device drivers are associated? Device ID

A Device ID identifies the driver for the device so that, when the Kernel needs to redirect the syscall made to a file to the driver, it knows exatcly who to call. The Device ID consists of two parts:

	1- A minor number: Used within the device driver to distinguish between two or more devices of the same type (such as two terminals) or switch between operation modes of a device
	2- A major number: can be specific or dynamically allocated (previously, it was used to identify a specific driver)

Nowadays, the distribution of Device IDs happens by the driver allocating a RANGE of minor numbers, not all of them, within a set of possible minor numbers given a major number. With that, currently, more than one driver can have the same major number as long as their range of minor numbers don't intersect!
	
The Device ID is stored as an unsigned `int` in the linux kernel and is defined by the `dev_t` type (unsigned 32 bit int with the 12 most significant being the major and last 20, the minor).

There's a C library to deal with this DEVICE IDs: `include/linux/kdev_t.h`
	

### Useful commands relating to Devices

1. To See The Character And Block Devices In My Computer:

```bash
cat /proc/devices
```

Will list the pairs: `<major number> <name of the device driver>`

2. How To See The Major And Minor Numbers (And Some Other Info) Of A Character Device File:

```bash
stat <name of character device file>
```

Will show you the inode, minor and major numbers of the device responsible for it (Device type field), last access, the size, number of blocks occupied, etc.

These files can be manually created with `mknod` and are often called "device node", "device file" or "device special file" (files that create this interface to a device)
 
 
### File Operations

Drivers, depending on the device, may implement different functions to handle the syscall for a device file. The list of these implemented syscalls is set into a `struct file_operations` object (interface), defined in `include/linux/fs.h`. In this file, `struct file_operations` holds a list of all entry points a driver can implement. We will focus only on the basic ones in this tutorial: OPEN, CLOSE (release), READ and WRITE -> in () or CAPSLOCK, we have the name of the function in the driver that implements the syscall.

### Read and write operation

Kernel should not de-reference pointers to memory in user space (we separate both types of data, often, even phisically, to prevent one reading from the other). Also, sometimes, the operating system can offer virtualized memmory. Hence, the addresses the user is trying to acces may not be valid at the moment (may not have been allocated yet or the was swapped out) or may not belong to user-space data when trying to access from kernel space. Due to that, there are some methods we use to ensure there's no problem accessing data in linux. To learn more memory management in linux, it is recommended to acccess [this link](https://www.youtube.com/watch?v=7aONIVSXiJ8).

### How It All Comes Together

Basically, `struct cdev` unites everything related to a device, hence, it is the structure that actually represents the device. It is defined in `/include/linux/cdev.h` and has the following fields:

```C
struct cdev {
	struct kobject kobj; // used for reference counting
	struct module *owner; // owner of this resource (we will set it to the character device)
	const struct file_operations *ops; // struct of supported file_operations
	struct list_head list; // Character Devices linked list
	dev_t dev; // Device ID
	unsigned int count; // amount of minor numbers owned by the character device 
} __randomize_layout;
```

It is the structure that ties all info about Character Devices together:
- Device IDs are used to identify the driver the kernel must call to handle syscalls for files/nodes linking to the device

- `struct file_operations`: holds all the operations one can do to a character device.

**`CDEV` TIES THESE ALL TOGETHER**

To register a new `cdev` we can use `*<cdev_name>_alloc()`, `<cdev_name>_init()`, `<cdev_name>_add()` and `<cdev_name>_del()`, after we include `<linux/cdev.h>`



- Obs: `cdev_add` -> adds your cdev to `struct kobj_map *cdev_map` array, kept in `fs/char_dev.c` (array of system's cdevs)

## Experience in the tutorial

In this tutorial, we created a simple driver `simple_char.c`, built the kernel with this driver as a module, loaded it and tested it by writing 256 bytes to it using an executable we wrote. Notice that the steps of this tutorial were a mix of the last tutorial and this one. The processes that involved the current tutorial had the objective of showing how the exposed concepts apply in practice, given that we wrote the driver with handlers for syscalls and for creating the character device object (`cdev`).

There was no major difficulty involving this tutorial, so this section will end here ;)


## Conclusion

Finally, this tutorial was really great to get to know more about how I/O works in Linux, given that this concept of device special files always seemed a little magical to me. These tutorials have already taught me so much and it seems there's a lot to come yet! 


