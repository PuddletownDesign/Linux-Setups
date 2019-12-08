# Setting up a local development server with Virtual box and Debian and SSH access

![debian](http://tinyimg.io/i/k9vc7m5.png)

In this article we will be setting up a headless (no desktop) debian webserver in virtualbox running vanilla Debian with shared folders and connecting via SSH to a static IP address.

## Table of contents

-   [Install via Virtual Box](#install-via-virtual-box)
    -   [Creating a Virtual Box VM](#creating-a-virtual-box-vm)
    -   [Installing Debian](#installing-debian)
-   [Configuring Virtual Box](#configuring-virtual-box)
    -   [Setting the network adapters](#setting-the-network-adapters)
-   [Logging in via SSH](#logging-in-via-ssh)
    -   [Setting up key pairs for passwordless login](#setting-up-key-pairs-for-passwordless-login)
        -   [Generating keypairs](#generating-keypairs-skip-if-you-already-have-them-from-before)
        -   [Copying the SSH keypairs to the guest OS](#copying-the-ssh-keypairs-to-the-guest-os-debian)
-   [Update the system and install critical software](#update-the-system-and-install-some-critical-software)
-   [(Optional) Installing puddletown preferences](#optional-installing-puddletown-preferences)
-   [(Optional) Server Security](#server-security)
    -   [Securing SSH](#securing-ssh)
    -   [Configuring a Basic firewall (UFW)](#configuring-a-basic-firewall-ufw)
    -   [Running a deep port scan with nmap](<#running-a deep-port-scan-with-nmap>)
-   [(Optional) Installing guest additions](#optional-installing-guest-additions)
-   [(Optional) Setting up shared folders](#optional-setting-up-shared-folders)
-   [(Optional) Starting a headless machine from the command line](#optional-starting-a-headless-machine-from-the-command-line-if-you-really-want-to)
-   [Where to go from here](#where-to-go-from-here)
    -   [Take a snapshot](#take-a-snapshot)
    -   [Conclusion](#conclusion)

## Install Virtual Box

Install Virtualbox if you don't have it

**On Mac**

```bash
brew cask install virtualbox
```

**On Linux**

```bash
sudo apt-get install virtualbox
```

**Next download a Debian Net installer torrent**

<https://cdimage.debian.org/debian-cd/current/amd64/bt-cd/>

Form the bottom of this page, it looks like this. **Do not** grab the one labeled mac, if you're using a mac.

`debian-*.*.*-amd64-netinst.iso.torrent`

after it's complete, **¡Keep the torrent seeding for the community!**

Copy the img to a new more permanent location. I put my disk images in `~/Documents/OSs`

### Creating a Virtual Box VM

1.  open virtualbox gui
2.  new machine
3.  name the machine Debian headless
4.  give it 512 ram
5.  Create a new disk (VDI) - Dynamically allocated
6.  Default 8GB - 20GB depending 
7.  Start in normal mode
8.  **Select the Debian ISO you downloaded as a torrent** `debian-9.1.0-amd64-netinst.iso`

### Installing Debian

From this point forward any changes made will only affect the Debian install. No changes will be made to the host OS.

1.  Select `install`. **not graphical install**
2.  Run through your language and regional settings
3.  Set your hostname to whatever you want i chose `debian`
4.  Just hit continue for domain name
5.  Set up a root password
6.  Set up your account name that you will use
7.  Timezone settings..
8.  Select "guided entire disk" for partition
9.  Select only disk available
10. All files in one partition
11. Finish partitioning and "write changes to disk" / Yes
12. Select `no` do not install from the cd
13. Select a network mirror close to you
14. Skip Http Proxy information unless it applies to you
15. Skip package survey question `no`

Let it install... this will take a while

#### Select the packages to install

**In this part use spacebar to select items** DO NOT HIT RETURN 

Only install:

1.  Standard system utilities

wait for more install stuff... this part takes the longest

1.  Install Grub Bootloader/ yes
2.  Select `/dev/sda`

omg finshing even more installations... 

#### Boot into the system

1.  Reboot
2.  Login with your root and the root password you created

then install ssh, an editor and net-tools

```bash
apt-get update
apt-get -y install sudo ssh nano openssh-server
systemctl start ssh.service
systemctl enable ssh.service
```

add your user to sudo and then power off the system

```bash
usermod -aG sudo username

poweroff
```

## Configuring Virtual Box

First give machine 8mb video ram

### Setting the network adapters

We will use 2 network adapters. In Virtual Box preferences -> network -> Host only.

1.  Create a new host only network (make sure if other VMs exist to increment the IP by one). Make sure DHCP server is enabled.

In the machine settings panel go to networks

1.  The first adapter needs to be `Host only`
2.  Select the adapter you just made for it in preferences next to "name"
3.  the second needs to be enabled and set to `NAT`

We will be SSHing into the machine through the host only IP and using the NAT connection for internet access for our guest machine.

Turn the machine back on and log in with your user. (We won't be using root anymore)

Open up the network interfaces file

```bash
sudo cp /etc/network/interfaces /etc/network/interfaces.default
sudo nano /etc/network/interfaces
```

Modify it to the following 

```bash
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface (that's the NAT)
auto enp0s3
iface enp0s3 inet dhcp

# Host-only network
auto enp0s8
iface enp0s8 inet static
  address 192.168.56.101 #increment for each VM on your network
  netmask 255.255.255.0
```

now reboot the machine

```bash
sudo poweroff
```

First let's just **power off the VM** and then **restart it in "Headless mode"**. We don't need that stupid Virtualbox console anymore. We can use our host terminal :)

## Logging in via SSH

By default debian will not allow root login (with password, which you need to set SSH public keys).

You will need to login using the user account that you created during install.

Now with the IP address you just set (address under host only interface) in the last step login with the following. Adjust the line below to fit your username and ip.

```bash
ssh username@ipaddress
```

Boom we should be in! Let's configure SSH to only allow login with Public/ Private key pairs.

### Setting up key pairs for passwordless login

In this step we will be assigning your host OS (Mac) public keys to log you into the Debain VM instead of using passwords. Keys are much more secure and don't require typing stupid passwords.

#### Generating keypairs (skip if you already have them from before)

If you don't already have ssh keys on your host OS (mac) then follow this to generate them. **Otherwise skip this step.**

```bash
ssh-keygen
```

Then save them to the default file.

`/Users/localuser/.ssh/id_rsa`

#### Copying the SSH keypairs to the guest OS (Debian)

```bash
ssh-copy-id username@ipaddress
```

Enter your password for the account.

**Now try logging in!**

```bash
ssh username@ipaddress
```

If everything is working well you should be logged in with no password!

## Update the system and install some critical software

```bash
sudo apt-get update && sudo apt-get upgrade

sudo apt-get install nano net-tools aptitude screen
```

## (Optional) Installing puddletown preferences

The puddletown preferences listed in this guide are custom configurations for `zsh`, `git` and using Atom text editor remotely instead of using a server side editor like Vim or Nano. 

You can skip most of these installations and be ok finishing this guide substituting out some of the commands for your preferred ones. 

I also recommend creating forks of each of these packages to your own github and then replacing `PuddletownDesign` with your own github username.

Please see the guide [Installing Puddletown Preferences for shell and git](https://github.com/PuddletownDesign/Linux-Setups/blob/master/03-installing-puddletown-zsh-git-configs.md)

## (Optional) Server Security

This is only optional beause you are working on a local machine that's not accessible to the internet. I think it's good practice and will closer reseble your production set up by following these steps.

### Securing SSH

We want to configure SSH for two things:

1.  Not allowing root login at all via SSH (Just lock it down)
2.  Not allowing password logins for any user, only SSH keys

Now that you're logged in with your user account via SSH and it has sudo privs, we're going to lock SSH down.

Then log back in with your user account via SSH in your host terminal of choice. Mine's iterm on mac.

Then lets edit the `sshd_config` file.

```bash
sudo ratom /etc/ssh/sshd_config
```

Change the following line First

```bash
#PermitRootLogin prohibit-password
```

to 

```bash
PermitRootLogin no
```

Make sure to uncomment the line it as well. If you're having a hard time finding it, it's located under the authentication header. 

Also change this line:

```bash
#PasswordAuthentication yes
```

to 

```bash
PasswordAuthentication no
```

Now restart the machines SSH servers

```bash
sudo systemctl restart ssh
```

Now, before we log out of the server, we should test our new configuration. We do not want to disconnect until we can confirm that new connections can be established successfully.

Open a new terminal window. In the new window, we need to begin a new connection to our server. This time, instead of using the root account, we want to use the new account that we created.

It should be working here. If not go back and recheck the `ssh_config` file.

### Configuring a Basic firewall (UFW)

Install `ufw`

```bash
sudo apt-get install ufw -y
```

Set up the default rules (deny all incoming)

```bash
sudo ufw default deny incoming
```

and

```bash
sudo ufw default allow outgoing (allow all out going)
```

Now allow connections for SSH (since this is the only service that we have running so far)

```bash
sudo ufw allow ssh
```

or, but not both. _these commands both do the same thing with different syntax_

```bash
sudo ufw allow 22/tcp
```

Now turn on UFW

```bash
sudo ufw enable
```

Once again log into the server from another tab to make sure you can connect before disconnecting your previous ssh session.

Check the status of `UFW`

```bash
sudo ufw status numbered
```

For more information on `ufw` see here

<https://www.digitalocean.com/community/tutorials/how-to-setup-a-firewall-with-ufw-on-an-ubuntu-and-debian-cloud-server>

### Running a deep port scan with nmap

Nmap is one of those cool hacker tools that trinity used in that movie _The Matrix_. It runs a deep probe of the server from the outside revealing any open ports and information that can be extracted from them.

On your **host** machine install `nmap`

**on Mac**

```bash
brew install nmap
```

**on Linux**

```bash
sudo apt-get install nmap
```

Then fire it up against the VM

```bash
sudo nmap -T4 -A -v [ip-address-of-vm]
```

Nmap will return a bunch of information such as: 

```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u1 (protocol 2.0)
```

We can see here that there is only one way into the machine. Port 22 through openSSH.

However we know that:

1.  There is only 1 user
2.  passwords don't work, keys are needed

**All in all this is a pretty secure machine. Of course it doesn't do anything yet, so that's easy ;)**

nmap also shows, public keys, the machines MAC address as well as all this:

```bash
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.10 - 4.11, Linux 3.2 - 4.9
Uptime guess: 41.914 days (since Sun Oct 27 06:54:59 2019)
```

That's a lot of information! It would probably be good to practice tightening up some of this leaked info. People on the outside just don't need to know that the OS is Debian 10 and how long the system has been up and what kernal version it's running.

We will address this more in a later [guide on advanced server security](04-advanced-production-server-security.md).

## (Optional) Installing guest additions

This is step is to provide better integration between the VM and the host. It's required for shared folders.

Install dev tool dependencies

```bash
sudo apt-get install -y build-essential dkms 
```

My Virtual box install is at 5.2.8 we need to grab the iso for the version that we are running.

Find the correct package from this list if below is not current:

<http://download.virtualbox.org/virtualbox/>

I'm going to be using the following file for Debian stretch

<http://download.virtualbox.org/virtualbox/5.2.8/VBoxGuestAdditions_5.2.8.iso> 

Logged into the VM in the `~` directory wget this file. (you can copy url to get the link)

```bash
cd ~

wget http://download.virtualbox.org/virtualbox/5.2.8/VBoxGuestAdditions_5.2.8.iso 
```

Make a directory to mount the guest additions

```bash
sudo mkdir /media/VBoxGuestAdditions
```

Now mount them to above directory

```bash
sudo mount -o loop,ro VBoxGuestAdditions_5.2.8.iso /media/VBoxGuestAdditions
```

Now install them

```bash
sudo sh /media/VBoxGuestAdditions/VBoxLinuxAdditions.run
```

Unmount them

```bash
sudo umount /media/VBoxGuestAdditions
```

Delete the directory you created to hold them

```bash
sudo rmdir /media/VBoxGuestAdditions
```

Delete the iso unless you want to keep it

```bash
rm VBoxGuestAdditions_5.2.8.iso
```

### (Optional) Setting up shared folders

This set up makes working on a project a little easier and links a folder from the host to the guest.

Add user to `vboxsf` user group

```bash
adduser <username> vboxsf 
```

Create a folder on the Host computer that you would like to share, for example `~/Documents/Dev/Sites` to `/var/www` on debian.

1.  Boot the Guest operating system in VirtualBox.
2.  Then in the vm manager Select Settings -> Shared Folders...
3.  Choose the 'Add' button.
    Select `~/Dev/Sites` Optionally select the 'Make permanent' and "auto mount" option

With a shared folder named share, as above, the folder can be mounted as the directory ~/host with the command

first let's trash the default www folder and remake it...

```bash
sudo rm -rf /var/www
sudo mkdir /var/www
```

Then mount the folder from the host

```bash
sudo mount -t vboxsf -o uid=$UID,gid=$(id -g) Sites /var/www
```

I've created a zsh alias for this command. So I only need to type `share` to mount the share point. It would be better to have this automatically happen on boot.

```bash
alias share='sudo mount -t vboxsf -o uid=$UID,gid=$(id -g) Sites /var/www'
```

Your folder should now be mounted and in place of the `/var/www` folder.

You can test this by going to `/var/www` and creating a test file. You will see it pop up on the host side.

## (Optional) Starting a headless machine from the command line if you really want to

> Note: If you close the terminal window after running the boot vm command, it will abort the server... It might be easier just to start it headless from within the virtualbox manager. Powering off from within the machine will end the process. Do not force cancel it.

Make sure to change `"Debian Headless"` to whatever you named the VM in virtualbox.

```bash
VBoxHeadless --startvm "Debian Headless"
```

## Where to go from here

I recommend learning next [how to set up a production Debian server on Digital Ocean or other VPS](https://github.com/PuddletownDesign/Linux-Setups/blob/master/configuring-debian-server-on-digital-ocean.md). It's very similar and actually a bit easier!

### Take a snapshot

In virtual box manager, take a snapshot of the VM at this point. I named mine "Fresh Install".

### Conclusion

Whew! That was quite a ride, but you should now have a really functional development server set up. 
