# Headless Debian with SSH on VirtualBox

![debian](http://tinyimg.io/i/k9vc7m5.png)

In this article we will be setting up a headless (no desktop) debian webserver in virtualbox running vanilla Debian.

The goals are:

1.  Log in with SSH only
2.  Load presets onto server
3.  Securing the server
4.  Setting up a web server and various services
5.  Setting up a development environment
6.  Pen testing against the server

## Table of Contents

-   [Install via Virtual Box](#install-via-virtual-box)
    -   [Creating a Virtual Box VM](#creating-a-virtual-box-vm)
-   [Installing Debian](#installing-debian)
    -   [Select the packages to install](#select-the-packages-to-install)
    -   [Boot into the system](#boot-into-the-system)
-   [Take a snapshot](#take-a-snapshot)
-   [Installing SSH](#installing-ssh)
-   [Configuring Virtualbox for SSH access](#configuring-virtualbox-for-ssh-access)
    -   [Finding the IP address of the VM](#finding-the-ip-address-of-the-vm)
    -   [Logging in via SSH](#logging-in-via-ssh)
-   [Setting up Debian](#setting-up-debian)
    -   [Setting up key pairs for passwordless login](#setting-up-key-pairs-for-passwordless-login)
    -   [Copying the SSH keypairs to the guest OS (Debian)](#copying-the-ssh-keypairs-to-the-guest-os-debian)
-   [Configure SSH](#configure-ssh)
-   [Take a snapshot](#take-a-snapshot)
-   [Starting a headless machine from the command line](#starting-a-headless-machine-from-the-command-line)
-   [Setting up basic personal configs like ZSH and git](#setting-up-basic-personal-configs-like-zsh-and-git)
    -   [Installing Git Presets](#installing-git-presets)
    -   [Installing zsh and oh my zsh presets](#installing-zsh-and-oh-my-zsh-presets)
    -   [ZSH Syntax Highlighting](#zsh-syntax-highlighting)
    -   [Install `thefuck`](#install-thefuck)
-   [Take a snapshot](#take-a-snapshot)
-   [Configuring Local atom to read files over SSH](#configuring-local-atom-to-read-files-over-ssh)
    -   [Install `remote-atom` on the host (Mac)](#install-remote-atom-on-the-host-mac)
    -   [Install `rmate` on guest (Debian)](#install-rmate-on-guest-debian)
    -   [Testing out `ratom` on the host](#testing-out-ratom-on-the-host)
-   [Fixing the giant ssh login prompt on the Host side](#fixing-the-giant-ssh-login-prompt-on-the-host-side)
-   [Setting up guest (Debian) keys for github](#setting-up-guest-debian-keys-for-github)
    -   [Backing up changed settings to github](#backing-up-changed-settings-to-github)
-   [Important Admin stuff - MOTD](#important-admin-stuff-motd)
-   [Setting up an nginx web server](#setting-up-an-nginx-web-server)

### _Work in Progress portion of the guide_

-   [Securing Debian for public use](#securing-debian-for-public-use)
    -   [Upgrading the system](#upgrading-the-system)
    -   [Locking down presets](#locking-down-presets)
        -   [Installing IPtables](#installing-iptables)
        -   [Configuring Fail2ban](#configuring-fail2ban)
        -   [Configuring TripWire](#configuring-tripwire)
-   [Opening up nginx to the public](#opening-up-nginx-to-the-public)
-   [Basic Server Logging](#basic-server-logging)
-   [Ongoing adjustment of settings and services](#ongoing-adjustment-of-settings-and-services)

## Install via Virtual Box

Install Virtualbox if you don't have it

`brew cask install virtualbox`

Grab a Debian Net installer torrent

<https://cdimage.debian.org/debian-cd/current/amd64/bt-cd/debian-9.1.0-amd64-netinst.iso.torrent>

**¡Keep the torrent seeding for the community!**

### Creating a Virtual Box VM

1.  open virtualbox gui
2.  new machine
3.  name the machine Debian headless
4.  give it 512mb ram
5.  give it 8mb of video ram
6.  Create a new disk (VDI) - Dynamically allocated
7.  Default 8GB
8.  Start in normal mode
9.  **Select the Debian ISO you downloaded as a torrent** `debian-9.1.0-amd64-netinst.iso`

## Installing Debian

From this point forward any changes made will only affect the Debian install. No changes will be made to the host OS.

1.  Select `install` not graphical install
2.  Run through your language and regional settings
3.  Set your hostname to whatever you want i chose `debian`
4.  Just hit continue for domain name
5.  Set up a root password
6.  Set up your account name that you will use
7.  Timezone settings..
8.  Select guided entire disk for partition
9.  All files in one partition
10. Finish partitioning and write changes to disk / Yes
11. Select `no` do not install from the cd
12. Select a network mirror close to you
13. Skip Http Proxy information unless it applies to you
14. Skip package survey question `no`

Let it install...

### Select the packages to install

**In this part use spacebar to select items** DO NOT HIT RETURN 

Only install:

1.  SSH
2.  Standard Linux Sysem

wait for more install stuff...

1.  Install Grub Bootloader
2.  Select `/dev/sda`

omg finshing even more installations...

### Boot into the system

1.  Reboot
2.  Login with `root` and the root password you created

## Take a snapshot

In virtual box manager, take a snapshot of the VM at this point. I named mine "Fresh Install".

## Installing SSH

Now that we are signed in with root let's install SSH. We'll install `nano` along with it for easy editing.

first update `apt-get`

```bash
apt-get update
```

then install ssh, an editor and net-tools

```bash
apt-get -y install ssh openssh-server nano net-tools
```

## Configuring Virtualbox for SSH access

Make sure the machine is powered off.

In the virtualbox control panel:

1.  Select the machine
2.  Click Settings
3.  Network > Adapter 1 > Attached to "Bridged Adapter"

Now power the machine on and log in

### Finding the IP address of the VM

run the following command to find the machine IP

```bash
ip addr show
```

use the `inet` version without the /24 or whatever number. Mine for example is just `192.168.1.67`

![finding the correct ip](http://tinyimg.io/i/sI0UflX.png)

### Logging in via SSH

By default debian will not allow root login (with password, which you need to set SSH public keys).

You will need to login using the user account that you created during install.

Now with the IP address you found in the last step login with the following. Ajust the line below to fit your username and ip.

```bash
ssh username@ipaddress
```

Boom we should be in! Let's configure SSH to only allow login with Public/ Private key pairs.

## Setting up Debian

We will be folling this guide for initial server set up:

<https://www.digitalocean.com/community/tutorials/initial-server-setup-with-debian-8>

Skip down to **Step 3 in the guide**

1.  In your ssh session login as root `su root`
2.  `apt-get update`
3.  `apt-get install sudo`

Now grant privileges to your normal user account. **Make sure to adjust the following command for your username.**

```bash
usermod -a -G sudo username
```

Now your user account has sudo privileges! We won't really need to use the root account anymore.

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

## Configure SSH

We want to configure SSH for two things:

1.  Not allowing root login at all via SSH (Just lock it down)
2.  Not allowing password logins for any user, only SSH keys

Now that you're logged in with your user account via SSH and it has sudo privs, we're going to lock SSH down.

First let's just **power off the VM** and then **restart it in "Headless mode"**. We don't need that stupid Virtualbox console anymore. We can use our host terminal :)

Then log back in with your user account via SSH.

Then lets edit the `sshd_config` file.

```bash
sudo nano /etc/ssh/sshd_config
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

_Only change this if you don't want to be able to login from a different computer. This is much better for security._

Now restart the machines SSH servers

```bash
sudo systemctl restart ssh
```

Now, before we log out of the server, we should test our new configuration. We do not want to disconnect until we can confirm that new connections can be established successfully.

Open a new terminal window. In the new window, we need to begin a new connection to our server. This time, instead of using the root account, we want to use the new account that we created.

It should be working here. If not go back and recheck the `ssh_config` file.

## Take a snapshot

This is a good place to take a snapshot! I called mine "SSH Configured".

## Starting a headless machine from the command line

> Note: If you close the terminal window after running the boot vm command, it will abort the server... It might be easier just to start it headless from within the virtualbox manager. Powering off from within the machine will end the process. Do not force cancel it.

Make sure to change `"Debian Headless"` to whatever you named the VM in virtualbox.

```bash
VBoxHeadless --startvm "Debian Headless"
```

## Setting up basic personal configs like ZSH and git

First install git and zsh

```bash
sudo apt-get install git zsh -y
```

Now let's install the puddletown settings. Please feel free to fork these repos before actually cloning them, so you can customize and then back them up to github later. 

If you fork make sure to change the github username in the url when cloning.

Let's start with installing some git presets.

### Installing presets

In your user folder we are going to make a folder called `Config`. This folder will hold all of your presets.

#### Installing Git Presets

```bash
mkdir Config && cd Config
```

then

```bash
git clone https://github.com/PuddletownDesign/Git

cd Git

git checkout origin/linux

git checkout linux

./install.sh
```

you might get an error like

```bash
rm: cannot remove '/home/brent/.gitconfig': No such file or directory
rm: cannot remove '/home/brent/.gitignore_global': No such file or directory
```

This is fine. It's telling you it can't delete `gitconfig` and `gitignore_global` because they don't exist yet. Don't worry about it.

however you can test out that the configs are installed by running the `git status` shortcut in the new config by typing `git s`. If you get a message like: `Your branch is up-to-date with 'origin/linux'.` then you are good to go!

#### Installing zsh and oh my zsh presets

go back into the configs directory that you made before and let's install zsh presets.

```bash
cd ~/Config
```

First we have to install `curl` or `wget` first to install oh my ZSH . Then I'm going to use `wget` to download it, because it's a little more snazzy than curl.

```bash
sudo apt-get install curl wget -y

sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```

Boom you should now have oh my zsh installed and zsh as your default shell!!!

Now let's install the puddletown preferences.

```bash
cd ~/Config

git cl https://github.com/PuddletownDesign/ZSH

cd ZSH

git checkout origin/linux

git checkout linux

./install.sh
```

You won't get any feedback back from that command if it works.

**Let's test it out!**

Log out and then back in with SSH.

You should now have a tits fancy prompt :)

![new prompt](http://tinyimg.io/i/Hi1OBix.png)

### ZSH Syntax Highlighting

This will make invalid commands you type red and correct commands green.

```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

And that should be all... There is already a reference to it in the puddletown `zshrc` file.

### Install `thefuck`

Let's install `thefuck`, cause it's handy #1 and will also get rid of the error at the zsh prompt.

> I know, I know... Installing python and a package manager for a terminal auto correct... Whatever, we will be using python later for a number of things. Plus a lot of applications depend on it anyway. Also `thefuck` is really handy. Your call for now.

```bash
sudo apt update

sudo apt install python3-dev python3-pip -y

sudo pip3 install thefuck
```

## Take a snapshot

Since we just did alllll that let's take a snapshot so if we screw something up coming up, we won't have to go through alllll that again.

I called mine "Presets installed"

## Configuring Local atom to read files over SSH

Cause we don't wanna be stuck using nano or vim let's set up atom to be able to edit system files over SSH.

### Install `remote-atom` on the host (Mac)

Open another terminal window and on the host side install `atom-remote`.

```bash
apm install remote-atom
```

### Install `rmate` on guest (Debian)

In the guest side, install `rmate`

```bash
sudo curl -o /usr/local/bin/rmate https://raw.githubusercontent.com/aurora/rmate/master/rmate

sudo chmod +x /usr/local/bin/rmate

sudo mv /usr/local/bin/rmate /usr/local/bin/ratom
```

> Note we renamed `rmate` to `ratom`

### Testing out `ratom` on the host

Open your Atom application, **go to the menu Packages -> Remote Atom, and click Start Server**. Your can also launch the server via command palette. The server can also be configured to be launched at startup in the preference.

**You will have to start a server in atom each time before you login.**

```bash
ssh -R 52698:localhost:52698 username@ip.com
```

Let's also add a zsh alias to only type `a` for `ratom`.

```bash
 ratom ~/Config/ZSH/.zshrc
```

add the following alias

```bash
alias a='ratom'
```

then `reload`.

let's test it out

```bash
cd ~

a test.txt
```

Save and edit the file! You can confirm you just edited the remote file by opening it up in the guest with `nano` and checking to see if the edits were made.

Boom! Host editor configured to edit all dem files!

> One note - is if you currently have a file open on the host and try to logout/exit, the terminal will hang until the open file is closed.

## Fixing the giant ssh login prompt on the Host side

Since the login prompt is pretty out of control now, let's create something easier to work with.

On the host side (mac) open up the `~/ssh/config` file.

```bash
atom ~/.ssh/config
```

Then add the following and change the values to suit you.

```bash
Host debian
    RemoteForward 52698 localhost:52698
    HostName 192.168.2.23
    User brent
    Port 22
```

1.  `host debian` is the name you will use to login. ex `ssh debian`
2.  `HostName is the ip address you use to login`
3.  `User` is your username
4.  `Port 22` should not need to be changed

Now test is out on the host side.

```bash
ssh debian
```

Much easier!

## Setting up guest (Debian) keys for github

Since you've made an edit eariler to your `.zshrc` file. If you forked the puddletown presets, let's back them back up to your repo. So if you use this zshrc file on another linux box it will be there for you already. (Same with any gitconfigs or zshrc edits).

I will be following this as the guide to adding SSH keys to github.

<https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/#platform-linux>

First we will have to generate a new key for your user. Make sure to replace the email with the one you use for github.

In Debian run the following commands:

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

```bash
a ~/.ssh/id_rsa.pub
```

now copy the contents to your clipboard and close the file.

1.  Open up github website on the host.
2.  Login
3.  Settings > SSH & GPG keys
4.  Add new SSH keys
5.  Name the key. I named mine "Debian Headless"
6.  Paste key in and save

Now we will test the key

```bash
ssh -T git@github.com
```

If you get a `Hi Username!` message you are good to go.

### Backing up changed settings to github

So far we've only changed our `.zshrc`. Let's upload the changes to github.

```bash
cd ~/Config/ZSH
 
g aa

g c "added new alias for atom"

g pso linux
```

You will have to enter your github username/ password. But just this time for the first push from the new system. After that it will use your SSH keys!

Done! Setting Synced!

## Important Admin stuff - MOTD

We want to add or alter the message that displays when logging into Debian.

![cowpower](http://tinyimg.io/i/rQ7iRQY.png)

Turn off Message of the day or edit the file. This file is only used for static text, no scripts can be put in here. **I removed all the text in mine.**

```bash
sudo ratom /etc/motd
```

Now let's add a multicolored fortune telling cow to your ssh login instead. 

```bash
sudo apt-get install cowsay fortune -y
```

now replace the contents of the  `/etc/update-motd.d/10-uname` file. This file is what prints out the string with os and last login.

```bash
sudo ratom /etc/update-motd.d/10-uname
```

Add the following:

```bash
W="\033[01;37m"
B="\033[01;34m"
R="\033[01;31m" 
G="\033[01;32m"
X="\033[00;37m"

echo "$G"
uname -snrvm
echo "$B"
/usr/games/fortune | /usr/games/cowthink
printf "\n$R"
```

This is a big improvement to our VM. 

There's also a myriad of fun scripts and messages to tinker with if cows don't suit your fancy.

### If minimalism is more your thing

The following turns off displaying the date you last logged in. This works on mac too...

```bash
touch ~/.hushlogin
```

## Setting up an nginx web server

I'll be following this guide for the most part.

<https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-debian-8>

Here we will set up an nginx webserver that will be available over the local network. We will have a single testing page that is viewable over the local network.

### Installing Nginx

```bash
sudo apt-get install nginx -y
```

Test the webserver from a host browser. Enter the IP address of the VM that you use for SSH or grab the `inet` from running `ip addr show`.

You should see the nginx default page.

![nginx is running](http://tinyimg.io/i/XVnjidD.png)

Let's go a step further and make the page look more interesting. 

```bash
cd /var/www/html

sudo cp index.nginx-debian.html index.html
```

Now open up the copied `index.html` file in atom.

    sudo ratom index.html

I'm going to use my default place holder template. You can do whatever you want. Copy and paste this low bandwidth mess right into the index.html file.

```html
<!DOCTYPE html><html lang="en"><head><meta charset="utf-8"><meta name="viewport" content="width=device-width; initial-scale=1.0"><title>Index</title><style type="text/css">html,body{margin:0;padding:0}html{position:relative;height:100%;background:#000111 url(data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0idXRmLTgiPz48IURPQ1RZUEUgc3ZnIFBVQkxJQyAiLS8vVzNDLy9EVEQgU1ZHIDEuMS8vRU4iICJodHRwOi8vd3d3LnczLm9yZy9HcmFwaGljcy9TVkcvMS4xL0RURC9zdmcxMS5kdGQiPjxzdmcgdmVyc2lvbj0iMS4xIiBpZD0iRHJvcHBsZXQiIG9wYWNpdHk9IjAuNyIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIiB4bWxuczp4bGluaz0iaHR0cDovL3d3dy53My5vcmcvMTk5OS94bGluayIgeD0iMHB4IiB5PSIwcHgiIHdpZHRoPSI1MDBweCIgaGVpZ2h0PSI1MDBweCIgdmlld0JveD0iMCAwIDUwMCA1MDAiIGVuYWJsZS1iYWNrZ3JvdW5kPSJuZXcgMCAwIDUwMCA1MDAiIHhtbDpzcGFjZT0icHJlc2VydmUiPjxwYXRoIGZpbGw9IiMwOEJCREMiIGQ9Ik0yNTYuMjk3LDM4LjQ5OWMwLDEwNC40NzUtMTM5LjI5OCwxNjkuNDgxLTEzOS4yOTgsMjc4LjU5OGMwLDEwOS4xMiw5NC44OTcsMTM5LjI5OSwxMzkuMjk4LDEzOS4yOTljNDQuNDAyLDAsMTM5LjMtMzAuMTc5LDEzOS4zLTEzOS4yOTlDMzk1LjU5NywyMDcuOTgsMjU2LjI5NywxNDIuOTczLDI1Ni4yOTcsMzguNDk5eiIvPjwvc3ZnPg==) center;background-size:10em}body{width:100%;height:100%;background-color:rgba(0,1,17,0.7)}body::before,body::after{position:absolute;content:" ";left:0px;width:100%;height:50%}body::before{top:0;background:linear-gradient(to bottom, #000111 0%,rgba(0,1,17,0.2) 100%)}body::after{bottom:0;background:linear-gradient(to bottom, rgba(0,1,17,0.2) 0%,#000111 100%)}</style></head><body></body></html>
```

Going back and refreshing the page, you should now see a bunch of tear drops. 

Our server is up and running and we have configured a default `index.html` page.

![teardrops](http://tinyimg.io/i/FssCw6b.png)
