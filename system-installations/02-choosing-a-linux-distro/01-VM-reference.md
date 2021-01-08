# Setting up a local development server running the latest version of Debian in Parallels on macOS

![debian](http://tinyimg.io/i/k9vc7m5.png)

In this article we will be setting up a headless (no desktop) debian webserver in Virtualbox or Parallels running the latest version of   Debia with shared folders and connecting via SSH to a static IP address.

## Table of contents

## Installing Parallels Trial

You can decide if you want to buy it, pirate it, or just use Virtualbox

you can download a free trial

```bash
brew cask install parallels
```

## [Parallels Only] Creating a Debian VM

![Install from DVD](images/01-parallels-install1.png)

1.  Open Parallels
2.  Select New
3.  Click on "Install Windows or another OS from a DVD or image file"
4.  If you don't see Debian in the list on the next page click choose manually and select the Debian iso you downloaded.
5.  Give it a name. I name my first one "Template - Debian Server", since I will be creating more Debian VMs off of this install.
6.  Choose where to save it. Mine all go in my `System` or `VM` folder.
7.  I uncheck "create alias on mac desktop" and check "Customize settings before install"

### Configuring the Parallels VM

#### [Parallels Only] General Tab

Give it a name and a descripton

![General tab](images/01-parallels-install2.png)

#### Under Options

##### Start up and shut down

![start and stop](images/01-parallels-install3.png)

##### Optimizataion

I leave my development VM running all the time in the background so, I leave it very little. Feel free to adjust this later.

![resources](images/01-parallels-install4.png)

##### Sharing

I set **shared folders to none** and then define custom folders as I need them. Since this is the template you want to come back later and define it for your first development VM.

I also make sure that those 3 checkboxes are all unchecked. I like to keep my VMs pretty well isolated from the host except for certain predefined points.

**The rest of the side tabs under `options`**

I basically turn everything off in the remaining tabs for parallels

![sharing](images/01-parallels-install5.png)

Then Go ahead and start the installer.

##### [Parallels Only] Hardware

For hardware specs I give the template:

1.  1 CPU
2.  1GB RAM
3.  64MB Video Memory
4.  Turn off printers
5.  64 GB Hard Disk (Minimum) & Split into 2gb files - expanding disk checked
6.  Shared Network (Default)
7.  USB & Bluetooth - ALl Checked (Default)

I like to allocate basically the least amount of virtual hard ware at

### [Parallels users can skip ahead to here](#Installing-Debian)

## [Virtual Box Only] Install Virtual Box

Install Virtualbox if you don't have it

**On Mac**

```bash
brew cask install virtualbox`
```

**On Linux**

```bash
sudo apt install virtualbox
```

### [Virtual Box Only] Creating a Virtual Box VM

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
apt update
apt -y install sudo ssh nano openssh-server
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
sudo apt update && sudo apt upgrade

sudo apt -y install net-tools aptitude screen
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
sudo apt install ufw -y
```

Set up the default rules (deny all incoming)

```bash
sudo ufw default deny incoming
```

and allow all out going connections

```bash
sudo ufw default allow outgoing
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

Check the status of `ufw`

```bash
sudo ufw status numbered
```

For more information on `ufw` see here

<https://www.digitalocean.com/community/tutorials/how-to-setup-a-firewall-with-ufw-on-an-ubuntu-and-debian-cloud-server>

### Running a deep port scan with nmap

Nmap is one of those cool hacker tools that Trinity used in _The Matrix_. It runs a deep probe of the server from the outside revealing any open ports and information that can be extracted from them.

On your **host** machine install `nmap`

**on Mac**

```bash
brew install nmap
```

**on Linux**

```bash
sudo apt install nmap
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

Follow this guide for now:

Start your VM

Install dev tool dependencies

```bash
sudo apt update
sudo apt install -y build-essential linux-headers-`uname -r`
```

Find the correct package from this list if below is not current:

<http://download.virtualbox.org/virtualbox/>

I'm going to be using the following file for Debian stretch

<https://download.virtualbox.org/virtualbox/6.10.0/VBoxGuestAdditions_6.10.0.iso>

Logged into the VM in the `~` directory wget this file. (you can copy url to get the link)

```bash
cd ~

wget https://download.virtualbox.org/virtualbox/6.1.10/VBoxGuestAdditions_6.1.10.iso
```

Make a directory to mount the guest additions

```bash
sudo mkdir /media/VBoxGuestAdditions
```

Now mount them to above directory

```bash
sudo mount -o loop,ro VBoxGuestAdditions_6.1.10.iso /media/VBoxGuestAdditions
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
rm VBoxGuestAdditions_6.1.10.iso
```

### (Optional) Setting up shared folders

> _Guest additions must be installed for this to work_

This set up makes working on a project a little easier and links a folder from the host to the guest.

Create a folder on the Host computer that you would like to share, for example `~/Documents/Dev/Sites`.

Open VirtualBox

1.  Right-click your VM, then click Settings
2.  Go to Shared Folders section
3.  Add a new shared folder
4.  On Add Share prompt, select the Folder Path in your host that you want to be accessible inside your VM.
5.  In the Folder Name field, type `/var/www`
6.  Set mount point to the folder in the guest OS
7.  Uncheck Read-only and Auto-mount

_Be aware that whatever guest folder you chose to share will be replaced with content from the host side. So if you choose your `~` home directory you'll likely erase all your ssh keys and then be locked out._

Folder Path in your host that you want to be accessible inside your VM. Type shared for the Folder Name. Make sure that Read-only and Auto-mount are unchecked.

With a shared folder named share, as above, the folder can be mounted from the host

In the VM side run the following commands.

Check the permissions of the shared folder on the guest (virtual box side). If the group is set to `vboxsf`, simply set your user to have that group attached to it so that we will be able to make edits from the guest side. (note that write access is reuqired for docker to work.)

`sudo usermod -aG vboxsf your-user-name`

Then Reboot.

```bash
sudo reboot
```

If the group was not set to `vboxsf` try the steps below.

```bash
sudo nano /etc/fstab
```

Now modify and append this string to the bottom of the file.

```bash
[name of share]	[guest folder path]	vboxsf	defaults	0	0

# so for instance the above would be
Docker /home/brent/Docker vboxsf	defaults	0	0
```

Then modify the modules

```bash
sudo nano /etc/modules
```

add the following line

```bash
vboxsf
```

then reboot

```bash
sudo reboot
```

Once you reboot the machine the selected folders should be synced

### (Optional) Take a snapshot

In virtual box manager, take a snapshot of the VM at this point. I named mine "Fresh Install". Then you can clone this machine in the snap shot pane.

In the future to create new VMs based on this one at this point (should take 2-3 minutes total):

1.  Open up virtualbox GUI
2.  Clone machine. Everyting. Leave it all the exact same.
3.  Boot machine and `sudo nano /etc/network/interfaces`
4.  adjust the ip to one that's not taken
5.  `poweroff` the VM and start headless
6.  connect via ssh to the IP that you set in the `interfaces` file
7.  Link the new machine name and ip in the host's `~/.ssh/config` file
8.  do any customizations to distinguish it from the others (hostname, shell theme)

Now you have a brand new VM based on this point!

### Conclusion

Whew! That was quite a ride, but you should now have a really functional development server set up.
