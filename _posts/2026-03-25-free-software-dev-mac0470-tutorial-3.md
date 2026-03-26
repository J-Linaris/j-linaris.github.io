---
layout: post
title: Free Software Development (MAC0470) Tutorial 3
date: 2026-03-25 21:52 -0300
description: Comments and difficulties regarding my experience in Tutorial 2 of the discipline "Free and Open Source Software Development" (MAC0470) of IME-USP
image: 
category: academic
tags: 
published: true
sitemap: true
---

# Tutorial 3

## Introduction

Now that we have set the envrionment and learned how to configure, build and boot a custom kernel, given that we will work on specific IIO subsystem's modules, the logic next step is to learn how to work with modules, exactly what we will do in this tutorial. More specifically, we will learn how to create and load modules, as well as how to configure the kernel for it to recognize the module.

Once again, I followed the series of tutorials created by FLOSS at USP ( [FLUSP](https://flusp.ime.usp.br/) ) regarding the development for the IIO subsytem. The tutorial is available [here](https://flusp.ime.usp.br/kernel/modules-intro/).

## Experience in the tutorial

Chockingly, in this tutorial, I didn't face any major error that occupied several hours to fix/understand it! However, it would be great to register, briefly, what is the process of creating and implementing a module in the linux kernel, since it is so vital to what I'll be doing next (participating on the development of a module).

### Module creation and Kernel configuration

Basically, first, we actually create and write the file in the kernel's module directory (such as `/drivers/misc`). Then, we have to create what we call the configuration symbols, which consist, in an abstract way, of an object containing several attributes of the module, that we also define. The configuration symbols are all set in the `Kconfig` file. 

In order to build the module, we also add it to the `Makefile` of the directory it was created in. Now, we run the `make` command using `menuconfig` in order to enable the module we just created. This means that we are adding the `Kconfig` settings we did to the `.config` file, which `kbuild` `Makefiles` will use to compile the code and decide which objects must be included in the monolithic kernel image and which objects should be modules. Therefore, in this `menuconfig` part, we signal that we want our module to actually be a module.

Finally, we build the kernel in order to build the modules and kernel all together. 

### Installing and Booting the modules

After this compilation, we install the modules directly into the image of the kernel and, then, boot the VM using this new kernel (this is one way to do it). The other way to install the modules into the VM is to simply, after compiling them with `kw build` and doing the whole configuration part, copy and paste them into the correct directory in the VM, connected to the VM, and include them into the module tracking files for easily managing dependencies (with `depmod`). Then, we can use `modprrobe` to easily run and stop our modules.


### Minor dependencie issue

Although I didn't have major problems, this doesn't mean it all worked on the first try (although it got really close to). When we included the module `simple_mod_part`, a new module after already including the first one, eventhough I ran `depmod --quick`, when I ran `modinfo simple_mod_part`, I received the message:

```bash
modinfo: ERROR: Module simple_mod_part not found.
```

Which, basically, meant that this newer module wasn't mapped as a new module to `depmod`. Hence, to fix this, I simply ran `depmod --all` in order to include the module into the module tracking files, and it worked!

## Conclusion

Finally, although this tutorial was almost a walk in the park, I learned a lot about how modules work and what is their actual relation to the linux kernel. Moreover, I got in contact with the process of going from creating a module to deploying it to the kernel of a VM, which I saw is an awsome task!

