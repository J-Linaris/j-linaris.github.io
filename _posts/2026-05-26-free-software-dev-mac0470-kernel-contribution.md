---
layout: post
title: Free Software Dev (MAC0470) Kernel Contribution
date: 2026-05-26 21:34 -0300
description: 'Comments on my contribution to the linux kernel as part of the discipline "Free and Open Source Software Development" (MAC0470) of IME-USP' 
image: 
category: academic
tags: 
published: true
sitemap: true
---

# Linux Kernel Contribution

## Introduction

After this series of six tutorials which not only taught all the necessary concepts, but also helped us setting the environment for developing the linux kernel, the natural and expected sequence is actually engaging in the development. For this, I worked with a friend, who really helped me throughout this proccess. After having the commit accepted, we wrapped up the first part of the discipline "Free and Open Source Software Development" (MAC0470) of IME-USP.

## The planing phase

First off, one has to decide what he wants to change. Remember we are working on the IIO subsystem of Linux, so any change would be really hard to test beyond successfully compiling the code (this will be really important). As we saw all the topics the teacher assistants recommended, we were really attracted by the one envolving replacing mutex with gurad locks statements (more details on this later). Making the code cleaner seemed really interesting.

Next, we had to decide which files we wanted to take. As if one wasn't already sufficient work, me and my pair really decided to make changes to two separate files, why not?! Indeed, looking back, I agree with our decision, but, by the time we accepted this challenge, it didn't seem much easy to be accomplished, given the phame of extremely rigorous the reviewers of linux kernel have (will also come back to this later).

We split our work, basically, setting, for each one, a file to be modified, and we cooperated whenever any changes were necessary to any of the files. The chosen files for which each one would be the main author were:

* Guilherme (my pair): drivers/iio/gyro/adxrs290.c (lore available [here](https://lore.kernel.org/all/20260504190455.9841-1-guilhermeabreu200105@usp.br/))
* Joao (me): drivers/counter/intel-qep.c (lore  available [here](https://lore.kernel.org/all/20260512173058.14858-1-jplinaris@usp.br/))

Now, I'll dive a little deeper in the experience we had with each one of them.

## The development phase

### What were we changing

According to [this](https://lwn.net/Articles/940117/) website, in 2023, it was determined that guard locks were the new pattern for handling locks, since they don't require you to use goto statements. Guard locks work by simply locking the execution, just like mutex does, while you're in the same scope as `guard(mutex)(&add_of_mutex_variable)` and releasing it when you leave the scope. An alternative to this statement would be `scoped_guard(mutex, &add_of_mutex_variable)`, although this wasn't defined as the pattern for linux kernel (and I figured this out the not so nice way).

A typical situation this change would be applicable would be one in which we lock the mutex for all executions but we use goto statements to unlock mutex considering if an execution enter a switch-case or an if statement. For example, something like:

```C
static int random_function(...){
	(...)
	
	int ret = 0;
	
	(previous function logic)
	
	mutex_lock(&qep->lock);
	
	if (qep->enabled) {
		ret = -EBUSY;
		goto out;
	}
	
	(rest of the function logic)

out:
	mutex_unlock(&qep->lock);
	return ret;
}
```
would be changed to something like:
```C
static int random_function(...){
	(intialization)
	(previous function logic)
	
	guard(mutex, &qep->lock){
		if (qep->enabled) 
			return -EBUSY;

		(rest of the function logic)
	}
	return 0;
}
```

### adxrs290.c modification

An interesting factor about the changes in this file is that we had to change it a decent amount of times before it was accepted: it got up to V5. At first, we simply replaced the mutex statements with guard locks. However, we missed one minor detail: one simple return statement missing, **not indicated by neither VS Code nor the actual compiler when we ran it**.

As previously said, it couldn't be different, this was the gentle comment we received by the reviewer:

"Two people as authors and none of them even compiled it?"

Really inspiring. After fixing the bug and sending it again, there were some style preferences of the reviewers we didn't know existed (such as number of empty spaces for the identation) and incorporated into the code and that was it. Although it was a long proccess, it was much more about getting to know the pattern the reviewers implement in the code than anything else. For getting to know more about the whole development process of this file, I highly recommend you to check out my [pair's post](https://guilherme-dna.github.io/posts/kernel-contr/) about this.

### intel-qep.c modification

The proccess of getting the commit accepted was a bit shorter than the one for the prior file, mainly, because we were able to learn and apply the reviewers recommmendations about code style already in the file. There were three versions of the patch, which I'll briefly describe here:
1. Introduced my changes replacing mutex with *scoped_guard* with *comments* explaining what scoped_guard does. After the changes, the funciton would look something like:
```C
static int intel_random_function(...){
	(intialization)
	(previous function logic)
	
	/* Locks mutex while execution in scoped_guard() scope */
	scoped_guard(mutex, &qep->lock){
		if (qep->enabled) 
			return -EBUSY;

		(rest of the function logic)
	}
	return 0;
}
```
Reviewer response: "Why add dead obvious comments like this" and "Why is using scoped variable of guard necessary in these cases??? Are you just making these changes using some tool without inspecting if the end result makes sense or not??". Again, really inspiring words.
2. Changed scoped_guard() to guard()() and removed comments. Now, the code looks something like:
```C
static int intel_random_function(...){
	(intialization)
	(previous function logic)
	
	guard(mutex)(&qep->lock){
		if (qep->enabled) 
			return -EBUSY;

		(rest of the function logic)
	}
	return 0;
}
```
Reviewer response: "Would recommend putting blank lines between guard(mutex)() and other code, Andy usually treats the guard()()s as separate things.". This reviewer was **much nicer**, a real relief.

3. Added blank lines between guard statements and anything else. The final version of the code looks something like:
```C
static int intel_random_function(...){
	(intialization)
	(previous function logic)
	
	guard(mutex)(&qep->lock){

		if (qep->enabled) 
			return -EBUSY;

		(rest of the function logic)
	}
	return 0;
}
```
Reviewer response: "Applied, thanks!". This is what we call **victory**.

## Conclusion

Jokes aside, I know that reviewers must get dozens of commits every day for a project the size of linux kernel. Therefore, it is comprehensible why some of them may seem kind of rispid in certain messages, it's probably the hurry they are in. However, it would be really good if they had a nicer approach sometimes, specially considering there may be a lot of people, just like us, that are entering the world of contribution to the linux kernel. 

Given this disclaimer, it was a really great time to develop for the kernel. I really learned a lot about linux overall both during setup tutorials and actual development. I would really like to thank [FLUSP](https://flusp.ime.usp.br/) for providing such great materials and support during the proccess.
