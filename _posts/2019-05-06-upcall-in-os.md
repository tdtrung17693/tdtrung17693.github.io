---
layout: post
title: "Upcall in OS"
date: 2019-05-06 11:43 +0700
tags: os
author: Tran Dinh Trung
---

As we've known that a System call is a way the user application using to execute a provided service of the OS. Sometimes, System Call is called a Down Call. So, you will wonder if there is an Up Call? And the answer is yes. An upcall is similar to a system call but in an opposite direction. Instead of calling a kernel space service from User space, an upcall calls an user space service from kernel space.

> An upcall is like a signal, except that the kernel may use it at any time, for any purpose, including in an interrupt handler.
>
> This means that an upcall can potentially destroy the behaviour of the kernel. If an interrupt handler decides to ask a user space function
> for some information, and the function page faults or does blocking
> IO, your kernel is quite likely trashed.
>
> So, upcalls aren't there for everyday software development. But for specific purposes, they can be extremely useful.
