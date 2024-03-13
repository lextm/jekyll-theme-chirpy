---
layout: post
title: How I Set Up Visual Studio Code Remote with Multipass on macOS
tags: Visual-Studio-Code macOS Linux Multipass
permalink: /how-i-set-up-visual-studio-code-remote-with-multipass-on-macos-674ce00956d1
excerpt_separator: <!--more-->
---
![img-description](/images/competitiveness-prosperity.jpg){: width="512" }
_Copyright © Lex Li. Institute for Competitiveness And Prosperity._

[Visual Studio Code Remote Development Extension](https://code.visualstudio.com/blogs/2019/07/25/remote-ssh) is an excellent idea with easy-to-use user experience. You can easily configure if your target machine is an SSH enabled Linux machine, either physical or virtual, in the cloud or on premise.

It works pretty good on Windows 10, as WSL delivers solid Linux experience.

With that in mind, it is natural that you want something similar on macOS, but installing VirtualBox/Parallel can be painful. Well, painful till the Ubuntu guys invented [Multipass](https://multipass.run/).

I tried it out today, and really enjoy the simple experience to get things done in just a few minutes.
<!--more-->

# Set Up Multipass

I always use homebrew, so it takes only a command to install Multipass,

``` bash
brew cask install multipass
```

Once installed, it gives me a default Ubuntu machine instance that I can shell in from the menu bar,

![img-description](/images/multipass-menu.png)
_Figure 1: Multipass menu._

and the shell looks like this,

![img-description](/images/multipass-shell.png)
_Figure 2: Multipass shell to an instance._

# Set Up Visual Studio Code Remote on macOS

> All steps below are performed on macOS.

If you don't have much Linux/macOS experience, then you definitely find it hard here. So we summarize what Multipass has created for us so far,

* An Ubuntu virtual machine running on 192.168.64.5.
* A default user named ubuntu.
* (This is hidden from you) A private key to log in at `/var/root/Library/Application\ Support/multipassd/ssh-keys/id_rsa`.

Therefore, to make it easy to configure in VSCode on macOS, I simply copy that private key to my own macOS account,

``` bash
sudo cp /var/root/Library/Application\ Support/multipassd/ssh-keys/id_rsa ~/.ssh/
```

Make sure you don't have `~/.ssh/id_rsa.pub` there, because that can lead to conflicts.

Launch a terminal to test out the connection,

``` bash
ssh ubuntu@192.168.64.5
```

If succeeded, then time to move on to VSCode (with Remote Development Extension installed).

1. Choose `View | Command Palettes …` menu item.
1. Choose `Remote-SSH: Connect to Host…` from the palette.
1. Choose `+ Add New SSH Host…` from the drop down list.
1. Enter `ssh ubuntu@192.168.64.5`.
1. Select `/Users/lextm/.ssh/config` as lextm is my macOS account.

VSCode informs that the host is added,

![img-description](/images/host-added.png)
_Figure 3: SSH host added._

You can click `Connect now` to open SSH connection, or click `Open Config` if you want to change the host setting (such as its name).

At this moment, your adventure on VSCode/Multipass can start.
