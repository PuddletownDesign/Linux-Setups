# Initial Set up for Debian server on Digital Ocean with Puddletown Settings

This guide will cover creating and basic configuration of a Debian server on digital ocean. In future guides we will cover, more advanced server security, setting up several websites on nginx, running crypto trading bots, and a host of other fun things.

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
$ sudo apt-get update
```

## Installing puddletown preferences

The puddletown preferences listed in this guide are custom configurations for `zsh`, `git` and using Atom text editor remotely instead of using a server side editor like Vim or Nano. 

You can skip most of these installations and be ok finishing this guide substituting out some of the commands for your preferred ones. 

I also recommend creating forks of each of these packages to your own github and then replacing `PuddletownDesign` with your own github username.

### Installing the prerequists

```bash
$ sudo apt-get install git zsh curl wget -y
```

### Create a `Config` Directory in your home folder

The `Config` directory will hold all of your presets like `zshrc`, `gitconfig` etc...

```bash
$ mkdir Config && cd Config
```

### Installing Git presets

You can check out the puddletown git presets here, there's a bunch of aliases for super quick gitting as well as a well crafted general purpose `gitignore` file.

```bash
git clone https://github.com/PuddletownDesign/Git
```

```bash
cd Git
```

```bash
git checkout origin/linux
```

```bash
git checkout linux
```

```bash
./install.sh
```

you might get an error like:

```bash
rm: cannot remove '/home/brent/.gitconfig': No such file or directory
rm: cannot remove '/home/brent/.gitignore_global': No such file or directory
```

This is fine. It's telling you it can't delete gitconfig and gitignore_global because they don't exist yet. Don't worry about it.

### Installing zsh and oh my zsh presets

```bash
$ cd ~/Config
```

Let's use wget to download oh-my-zsh, because it's a little more snazzy than using curl ;)

```bash
$ sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```

Now change the default shell for your user

```bash
$ sudo chsh -s /usr/bin/zsh <username>
```

Note you will not yet see a change.

#### Installing Puddletown Oh-my-zsh preferences

```bash
git clone https://github.com/PuddletownDesign/ZSH
```

```bash
cd ZSH
```

```bash
git checkout origin/linux
```

```bash
git checkout linux
```

```bash
./install.sh
```

You won't get any feedback back from that command if it works.

Let's test it out!

Log out and then back in with SSH.

You should now have a tits fancy prompt :)

![New Prompt](https://camo.githubusercontent.com/9c1563c047551662645cb7d74c34ef0ea1fb7587/687474703a2f2f74696e79696d672e696f2f692f4869314f4269782e706e67)

#### If for some reason your prompt is crazy messed up displaying double characters and delete key goes forward

simply add the following to your zshrc

```bash
TERM=xterm # fixes bizarre character prompt issues
```

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

### Configuring Local atom to read files over SSH

Cause we don't wanna be stuck using nano or vim let's set up atom to be able to edit system files over SSH.

#### Install `remote-atom` on the host (Mac)

Open another terminal window and on the host side install `atom-remote`.

```bash
apm install remote-atom
```

#### Install `rmate` (aka `ratom`) on guest (Debian)

In the guest side, install `rmate`

```bash
sudo curl -o /usr/local/bin/rmate https://raw.githubusercontent.com/aurora/rmate/master/rmate

sudo chmod +x /usr/local/bin/rmate

sudo mv /usr/local/bin/rmate /usr/local/bin/ratom
```

> Note we renamed `rmate` to `ratom`

#### Testing out `ratom` on the host

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

### Fixing the giant ssh login prompt on the Host side

Since the login prompt is pretty out of control now, let's create something easier to work with.

On the host side (mac) open up the `~/ssh/config` file.

```bash
atom ~/.ssh/config
```

Then add the following and change the values to suit you.

```bash
Host debian
    RemoteForward 52698 localhost:52698
    HostName <guest-ip-here>
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

## Basic Server security

## Configure SSH for security

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
sudo apt-get install ufw
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

For more information on `ufw` see here

<https://www.digitalocean.com/community/tutorials/how-to-setup-a-firewall-with-ufw-on-an-ubuntu-and-debian-cloud-server>

## One last fun task for now! Important Admin stuff - MOTD

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
#!/bin/sh
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

save and close the file

This is a big improvement to our VM. 

log out and back in to see the results

There's also a myriad of fun scripts and messages to tinker with if cows don't suit your fancy.

## Where to go from here

I suggest reading up on advanced server security and learning about ufw, fail2ban and tripwire before moving onto any of the other guides. It's a wild world out there and it's best not to roll without protection. If rolling bareback is more your thing, feel free to skip to installing nginx and setting up a web server.
