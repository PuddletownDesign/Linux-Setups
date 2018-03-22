# Installing Puddletown ZSH and Git configs

## Installing the prerequists

```bash
$ sudo apt-get install git zsh curl wget -y
```

## Create a `Config` Directory in your home folder

The `Config` directory will hold all of your presets like `zshrc`, `gitconfig` etc...

```bash
$ mkdir Config && cd Config
```

## Installing Git presets

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

## Installing zsh and oh my zsh presets

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

### Installing Puddletown Oh-my-zsh preferences

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

### If for some reason your prompt is crazy messed up displaying double characters and delete key goes forward

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

sudo apt install python3 python3-pip -y

sudo pip3 install thefuck
```

## Configuring Local atom to read files over SSH

Cause we don't wanna be stuck using nano or vim let's set up atom to be able to edit system files over SSH.

### Install `remote-atom` on the host (Mac)

Open another terminal window and on the host side install `atom-remote`.

```bash
apm install remote-atom
```

### Install `rmate` (aka `ratom`) on guest (Debian)

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

## One last fun task for now! Important Admin stuff - MOTD

We want to add or alter the message that displays when logging into Debian.

![cowpower](http://tinyimg.io/i/rQ7iRQY.png)

Turn off Message of the day or edit the file. This file is only used for static text, no scripts can be put in here. **I removed all the text in mine.**

```bash
sudo ratom /etc/motd
```

Now let's add a multicolored fortune telling cow to your ssh login instead. 

```bash
sudo apt-get install cowsay fortune neofetch -y
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

echo ""
neofetch
echo "$B"
/usr/games/fortune | /usr/games/cowthink
printf "\n$R"
```

save and close the file

This is a big improvement to our VM. 

log out and back in to see the results

There's also a myriad of fun scripts and messages to tinker with if cows don't suit your fancy.
