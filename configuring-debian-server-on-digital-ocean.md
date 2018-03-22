# Initial Set up for Debian server on Digital Ocean with Puddletown Settings

This guide will cover creating and basic configuration of a Debian server on digital ocean. In future guides we will cover, more advanced server security, setting up several websites on nginx with SSL, git deployment, running crypto trading bots, and a bunch of other fun things.

The puddletown preferences listed in this guide are custom configurations for `zsh`, `git` and using Atom text editor remotely instead of using a server side editor like Vim or Nano. 

If you find this guide useful, you can use my referral to get $25 in free credit. They will kick me down $10 after you've been with them for 5 months.

<https://m.do.co/c/3cd924449532>

## Creating the server

After signing up with digital ocean and entering payment data:

1.  Click create new droplet
2.  Select Debian (most recent version, in this case it's 9.4) ![Server type](http://tinyimg.io/i/twr4eXW.png)
3.  Select your droplet size (I'm choosing the smallest, simlest droplet for myself, you can upgrade in a few clicks later if needed) ![Droplet Type](http://tinyimg.io/i/1fCFduC.png)
4.  I'm not choosing to add opimizations or flexible droplets to my plan.
5.  Not adding block storage.
6.  Choose the region your server will be hosted in (I'm choosing SF)
7.  I add no additional options
8.  **Do not add SSH keys now**
9.  Create a good host name. Pick a good name for the machine based on what it will be doing. Mine will be hosting puddletown.design so I will name the machine `puddletown`.
10. Click _Create the droplet_

## Logging into the Server

After a few minutes you will get a email confirmation that your droplet has been created. The email will contain an IP address, a username and a password.

Open up your terminal and enter the following 

Replace the characters contained in the brackets with your own information. Make sure not to leave the brackets in the command

```bash
$ ssh <username>@<ip>
```

1.  Type `yes` when prompted
2.  Enter the `password` when prompted

### Create a new root password

1.  Enter the same password again
2.  Create a new password. Create a long random string of numbers and letters. Make note of it but you will never need to use it again. Something like `4b22fb94cb4c3b594a0048ba2e` is good. 
3.  type (paste) the same password again.

You are now logged in as `root`

## Create a new user for yourself

`<username>` is the user you will create for yourself

```bash
useradd -m <username>
```

### Create a password for your useradd

Make a password that you will remember. You will need this for a lot of commands when running `sudo` in the future.

```bash
passwd <username>
```

### Give your user `sudo` power!

```bash
$ usermod -aG sudo <username>
```

## Create SSH keys for login

I'm going to assume you've already created SSH keys on your local machine. Like for using github or something. If you haven't Please check [this guide about how to do it](erver-setup-with-ubuntu-16-04#step-four-—-add-public-key-authentication-(recommended))

On your local machine in my case on my mac. Open a new terminal tab and copy your SSH keys to the server.

```bash
ssh-copy-id <username>@<server_ip_address> 
```

1.  Enter the password for your user.

## Log in with your new user

In the same local machine (mac) side terminal tab try logging in with the new user.

```bash
ssh <username>@<server_ip_address> 
```

If you can log in without being propted for a password, go ahead and `exit` and close the `root` terminal tab from before. 

## Updating the system

```bash
$ sudo apt-get update && sudo apt-get upgrade
```

## Installing puddletown preferences

The puddletown preferences listed in this guide are custom configurations for `zsh`, `git` and using Atom text editor remotely instead of using a server side editor like Vim or Nano. 

You can skip most of these installations and be ok finishing this guide substituting out some of the commands for your preferred ones. 

I also recommend creating forks of each of these packages to your own github and then replacing `PuddletownDesign` with your own github username.

Please see the guide [Installing Puddletown Preferences](https://github.com/PuddletownDesign/Debian-Server-Setup/blob/master/installing-puddletown-zsh-git-configs.md)

Once the guide is complete come back and finish the remainder of this guide.

## Basic Server security

### Configure SSH for security

We want to configure SSH for two things:

1.  Not allowing root login at all via SSH (Just lock it down)
2.  Not allowing password logins for any user, only SSH keys

Now that you're logged in with your user account via SSH and it has sudo privs, we're going to lock SSH down.

First let's just **power off the VM** and then **restart it in "Headless mode"**. We don't need that stupid Virtualbox console anymore. We can use our host terminal :)

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

## Configuring a Basic firewall

Install `ufw`

```bash
sudo apt-get install ufw -y
```

### Set up the defaults

```bash
sudo ufw default deny incoming
```

and

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

Check the status of `UFW`

```bash
sudo ufw status numbered
```

For more information on `ufw` see here

<https://www.digitalocean.com/community/tutorials/how-to-setup-a-firewall-with-ufw-on-an-ubuntu-and-debian-cloud-server>

## Where to go from here

I recommend moving on to [Installing, configuring and using nginx on with SSL](https://github.com/PuddletownDesign/Linux-Setups/blob/master/installing-configuring-and-using-nginx-on-linux.md)
