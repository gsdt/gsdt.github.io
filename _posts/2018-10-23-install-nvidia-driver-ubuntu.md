---
layout: post
title: "Install NVIDIA driver Ubuntu"
description: "How to install NVIDIA graphic driver on Ubuntu OS"
categories: [install, nvidia]
tags: [tutorial]
redirect_from:
  - /2018/10/23/
---

I have spend several hours to searching for find out "How to install NVIDIA driver on Ubuntu Operating System". So, for saving time of another people like me, I try to write short tutorial about it.

First you must determine version of your physical devide.

Choose suitable driver for your physical graphic devide: [link](https://launchpad.net/~graphics-drivers/+archive/ubuntu/ppa)

For example, my physical devide is NVIDIA Geforce 820M, I will install 'nvidia-390' version.

~~~ bash
sudo add-apt-repository ppa:graphics-drivers
sudo apt-get update
sudo apt-get install nvidia-390
~~~

After you finish, restart your computer.

Good luck
