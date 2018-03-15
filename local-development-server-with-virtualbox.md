# Headless Debian with SSH on VirtualBox

![debian](http://tinyimg.io/i/k9vc7m5.png)

In this article we will be setting up a headless (no desktop) debian webserver in virtualbox running vanilla Debian with shared folders.

## Install via Virtual Box

Install Virtualbox if you don't have it

`brew cask install virtualbox`

Grab a Debian Net installer torrent

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

## Installing Debian

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

### Select the packages to install

**In this part use spacebar to select items** DO NOT HIT RETURN 

Only install:

1.  Standard system utilities

wait for more install stuff... this part takes the longest

1.  Install Grub Bootloader/ yes
2.  Select `/dev/sda`

omg finshing even more installations... 

### Boot into the system

1.  Reboot
2.  Login with your root and the root password you created

then install ssh, an editor and net-tools

```bash
apt-get update
apt-get -y install sudo ssh openssh-server nano net-tools aptitude
systemctl start ssh.service
systemctl enable ssh.service
```

add your user to sudo and then power off the system

```bash
usermod -aG sudo username

poweroff
```

## Take a snapshot

In virtual box manager, take a snapshot of the VM at this point. I named mine "Fresh Install".

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

# Host only network interface
auto enp0s3
iface enp0s3 inet static
address 192.168.56.107
network 192.168.56.0

# NAT network interface
auto enp0s8
iface enp0s8 inet dhcp
dns-nameservers 8.8.8.8 8.8.4.4
netmask 255.255.255.0
broadcast 192.168.56.255
```

now reboot the machine

```bash
sudo reboot
```

### Logging in via SSH

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

### Copying the SSH keypairs to the guest OS (Debian)

```bash
ssh-copy-id username@ipaddress
```

Enter your password for the account.

**Now try logging in!**

```bash
ssh username@ipaddress
```

If everything is working well you should be logged in with no password!

## Installing puddletown preferences

The puddletown preferences listed in this guide are custom configurations for `zsh`, `git` and using Atom text editor remotely instead of using a server side editor like Vim or Nano. 

You can skip most of these installations and be ok finishing this guide substituting out some of the commands for your preferred ones. 

I also recommend creating forks of each of these packages to your own github and then replacing `PuddletownDesign` with your own github username.

Please see the guide [Installing Puddletown Preferences](https://github.com/PuddletownDesign/Debian-Server-Setup/blob/master/installing-puddletown-zsh-git-configs.md)

## Starting a headless machine from the command line if you really want to

> Note: If you close the terminal window after running the boot vm command, it will abort the server... It might be easier just to start it headless from within the virtualbox manager. Powering off from within the machine will end the process. Do not force cancel it.

Make sure to change `"Debian Headless"` to whatever you named the VM in virtualbox.

```bash
VBoxHeadless --startvm "Debian Headless"
```

Once the guide is complete come back and finish the remainder of this guide.

## Creating shared folders in virtual box

### Downloading the correct Guest additions

Install dev tool dependencies

```bash
sudo apt-get install linux-headers-$(uname -r) build-essential dkms -y
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

#### Installing guest additions

##### Make a directory to mount the guest additions

`sudo mkdir /media/VBoxGuestAdditions`

##### Now mount them to above directory

`sudo mount -o loop,ro VBoxGuestAdditions_5.2.8.iso /media/VBoxGuestAdditions`

##### Now install them

`sudo sh /media/VBoxGuestAdditions/VBoxLinuxAdditions.run`

##### Unmount them

`sudo umount /media/VBoxGuestAdditions`

##### Delete the directory you created to hold them

`sudo rmdir /media/VBoxGuestAdditions`

##### Delete the iso unless you want to keep it

`rm VBoxGuestAdditions_5.2.8.iso`

### Setting up shared folders

Let's first install nginx and link our `www` folder to our local machine.

#### Add user to `vboxsf` user group

```bash
adduser <username> vboxsf 
```

#### Setting up the local share folder

Let's first install nginx if you don't have it and then link the www folder to be shared.

```bash
sudo apt-get install nginx
```

Create a folder on the Host computer that you would like to share, for example `~/Documents/Dev/Sites` to `/var/www` on debian.

1.  Boot the Guest operating system in VirtualBox.
2.  Then in the vm manager Select Settings -> Shared Folders...
3.  Choose the 'Add' button.
    Select `~/Dev/Sites` Optionally select the 'Make permanent' and "auto mount" option

With a shared folder named share, as above, the folder can be mounted as the directory ~/host with the command

first let's trash the default nginx folder and remake it...

```bash
sudo rm -rf /var/www
sudo mkdir /var/www
```

`sudo mount -t vboxsf -o uid=$UID,gid=$(id -g) Sites /var/www`

I've created a zsh alias for this command. So I only need to type `share` to mount the share point. It would be better to have this automatically happen on boot.

```bash
alias share='sudo mount -t vboxsf -o uid=$UID,gid=$(id -g) Sites /var/www'
```

You folder should now be mounted and in place of the `/var/www` folder.

You can test this by going to `/var/www` and creating a test file. You will see it pop up on the host side.
