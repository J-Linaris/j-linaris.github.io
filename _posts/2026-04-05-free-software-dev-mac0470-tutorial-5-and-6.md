---
layout: post
title: Free Software Development (MAC0470) Tutorial 5 and 6
date: 2026-04-05 13:03 -0300
description: '"Sending patches by email with git" and "Sending patches with git and a USP email", respectively, tutorials 5 and 6, comments and difficulties regarding my experience in these tutorials of the discipline "Free and Open Source Software Development" (MAC0470) of IME-USP' 
image: 
category: academic
tags: 
published: true
sitemap: true
---

# Tutorial 5 and 6

## Introduction

Now that we know more about Linux Character Devices, it is time for us to learn how to upload our contributions to the linux kernel. These two tutorials are more about setting things up for sending contributions to the kernel, hence, I grouped both tutorials in one post given that they both relate to this setup.

Once again, I followed the series of tutorials created by FLOSS at USP ( [FLUSP](https://flusp.ime.usp.br/) ) regarding the development for the IIO subsytem. Tutorial 5 is available [here](https://flusp.ime.usp.br/git/sending-patches-by-email-with-git/) and Tutorial 6, [here](https://flusp.ime.usp.br/git/sending-patches-with-git-and-a-usp-email/).

## Experience in tutorial 5

This tutorial is about configuring git to send emails through git itself (contributions for the kernel (and git) are sent via email instead of "fancy UIs", as called in the tutorial, such as Git Hub or Git Lab). We were told just to read through this part, since we will have to take some additional steps given we are using an USP email. To setup this email sending process through git, the steps would be:

1. Configure an App Password for my personal email account
2. Configure git to use this email to send emails

The idea is that we create a patch of our modifications in a file and we send this git commit file using our email to specific review accounts using `git send-email`. All steps of this tutorial were completed without any concern. It was much more of a conceptual tutorial than anything.


## Experience in tutorial 6

On the other hand, this tutorial was much more practical. We learned how to send contributions through `git send-email` using our USP email account (we overwrote tutorial 5 setup). For this, we had to do some extra configuration after Tutorial 5 since, basically, USP e-mail doesn't accept neither App Password nor *Less Secure Apps* authentications. The solution is to use OAuth2.0.

To implement this solution with OAuth2.0, we will need to set, first, an email proxy to implement OAuth protocol, which git send-email itself doesn't do. Also, we will use `kw` to simplify processes regarding our kernel contributions. Therefore, the steps of these tutorials were:

1. Running the necessary email proxy
2. Configuring `git send-email` with `kw send-patch`
3. Testing the setup with `kw send-patch`


### Running the necessary email proxy

To simplify the process of configuring this proxy, we will use a docker to run the python script email-oauth2-proxy, that implements this proxy for us (the project owner is the creator of the tutorial!). For this, the process of configuring and running the proxy consisted of a really short sequence of commands:

```bash
#------CONFIGURATION-------
# Clone the project to our local machine
git clone https://github.com/davidbtadokoro/emailproxy-container.git

# Open emailproxy.config and change the necessary fields according to the tutorial
nano emailproxy-container/emailproxy.config
```

```bash
#------BUILDING THE EMAIL PROXY CONTAINER-------

# Enter the container directory
cd /home/lk_dev/emailproxy-container/

# Build the docker container
docker compose up --build
```

Inspecting the `Dockerfile` in the container, it is easy to see that the container will sleep infinitely after been built by the command we ran. Hence, in another terminal, we ran the other neecessary commands:

```bash
#------RUNNING THE EMAIL PROXY SERVER-------

# Enter the container directory
cd /home/lk_dev/emailproxy-container/

# Start the container server
docker exec -it emailproxy-container-server-1 /bin/bash

# Start the email proxy server
emailproxy --no-gui --external-auth --config-file /app/emailproxy.config
```

To check if everything went according to the expected, you must check if you received the message 
```bash
<rest of the output>
Initialised Email OAuth 2.0 Proxy - listening for authentication requests. Connect your email client to begin
```

### Configuring `git send-email` with `kw send-patch`

After setting and starting the email proxy, we used kw convenience to configure our information for `git send-email` such as name, email, smtpuser, smtpserver and smtpserverport. ALl these info was stored in `.git/config/`. In case it occurs any errors, refer to this file.

### Testing the setup

We tested our setup sending an email with this email proxy using our USP email to a personal email account! For this, first, we created a mock commit in the repository of `emailproxy-container`. Then, we simulated the whole ideal process of sending a contribution:

#### 1st: Check if `kw send-patch`will do the right thing

Before actually sending the email, it's good practice to check if `kw send-patch` will send the correct message to the correct people. To do this, we simply add the flag `--simulate` to `kw send-patch` command of sending the email. This flag makes `kw send-patch` do everything it would do normally, except for actually sending the email. Hence, we can check if everything is correct in the output of the command (`To:`, `Cc:` and message fields).

```bash
# Mock email sending (... indicates that the list can go on)
kw send-patch --send --private --simulate --to='<EMAIL-ADDRESS-1>','<EMAIL-ADDRESS-2>' ...

# MY TESTING CASE
kw send-patch --send --private --simulate --to='joaolinaris@gmail.com'
```

#### Actually sending the email

To actually send the email, run this command but without the `--simulate` flag: 

```bash
# Mock email sending (... indicates that the list can go on)
kw send-patch --send --private --to='<EMAIL-ADDRESS-1>','<EMAIL-ADDRESS-2>' ...

# MY TESTING CASE
kw send-patch --send --private --to='joaolinaris@gmail.com'
```

The first time we run this command, we will be prompted to enter a password in a message like the following:

```bash
<rest of output>
Password for 'smtp://davidbtadokoro%40ime.usp.br@127.0.0.1:2587':
```

This does NOT require the USP email account password. It is of local usage, hence, you can define any (just remember it for future uses). After setting your password, you must authenticate using the url provided in the container log but it will only work if you belong to the group of acceptable clients. The whole process worked out great, with the only problem that I wasn't in the group when I first tried, which introduced a bit of overhead.

## Conclusion

Finally, these two tutorial were, although quite simple, really interesting since they show how commits in the development of such popular and complex tools like git and linux kernel work and let us set our machine to be able to contribute to such projects!


