---
author: Cody
published: 2018-03-12T00:00:00+00:00
title: Run Linux GUI Applications on Windows 10
_hash: e03b72110649ad12bede229a4381feeaadb175cca2e311cb43d5ebd929c28bc6
updated: 2026-03-19T11:38:21.464691+00:00
...

Linux is my main development environment but I also need to test things on Windows. My computer boots into Windows 10 and then I run a Linux VM.
Today I was trying to setup a new remote Windows 7 VM to build an installer. I was connecting to it with `virt-manager` through the local Linux VM.

For some reason it was making my local Linux VM very sluggish and it was taking minutes to update the mouse position on the remote VM with `virt-manager`.

So I decided to see if I could run `virt-manager` on my Windows 10 host but there is no Windows installer for it.

I did find this ServerFault answer https://serverfault.com/a/840962 about using the Windows Subsystem for Linux and I was able to get it working.

Here is what I had to do.

Following this https://docs.microsoft.com/en-us/windows/wsl/install-win10

Open PowerShell as Administrator and run:

```PowerShell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
```

Don't restart just yet.

Install your distro by going to https://aka.ms/wslstore

You have to install it through the Windows Store but don't worry you don't need an account.
When it asks you to sign in just say **No, Thanks** as it is already installing.

![Windows 10 store dialog asking if you would like to sign in.](/static/img/windows10store.png)

It will be available in the Start Menu under the name of the distribution you chose, Ubuntu for me.

First I made sure it was up to date and was able to fetch programs. This might take a while to download and install the updates.

```shell
sudo apt update
sudo apt upgrade
```

Then I installed the program I want to launch.

```shell
sudo apt install virt-manager
```

## Install Xming on Windows 10

Download and install it from here https://sourceforge.net/projects/xming/.

In order to run Linux GUI Applications Xming needs to be running.

### Run Xming on boot

To run Xming on startup, you need to add a shortcut to the Startup directory.

- Search for Xming in the Start Menu
- Right click on the shortcut and select **Open file location**
- Copy the selected Xming shortcut
- Press `Ctrl-l` to move the cursor to the Explorer location input
- Type `shell:startup` and then Enter
- Paste the Xming shortcut in the Startup directory.

Now Xming will start automatically when Windows 10 starts.

## Launch Linux GUI Applications

In Linux set the display to use Xming.

```shell
export DISPLAY=localhost:0.0
```

To have it be the default add that line to `~/.bashrc` which will run it on login.

```shell
echo 'export DISPLAY=localhost:0.0' >> ~/.bashrc
```

Now you can just run the Linux GUI application in the terminal with a `&`.

```shell
virt-manager &
```

And if it works, Yay!

Since `virt-manager` needs to connect to a remote computer I had to install `ssh-askpass`.


```shell
sudo apt install ssh-askpass
```

I also created a keypair and uploaded my public key to the remote computer.

```shell
ssh-keygen -t rsa -C "email@example.com"
ssh-copy-id $user@$host
```

And now I can use `virt-manager` from Windows 10 and it works great. Well sometimes
the bottom of the VM display is white and only works on my main monitor. When this
happens I restart Xming and launch `virt-manager` again.
