# Debian production server

![debian](http://tinyimg.io/i/k9vc7m5.png)

## Updating the system

```bash
sudo apt-get update
```

### Working with user groups

look at all the groups on the machine

`cat /etc/group`

look at the existing groups for your user

`groups brent`

add a group for users who will be able to `rwx` in for a folder

`sudo addgroup sftp-users`

add your user to that group

`sudo adduser brent sftp-users`

list all the groups your user is part of

`groups brent`

now change the group of the `/var/www` folder, while leaving the owner the same.

`sudo chgrp sftp-users /var/www`

add write permissions for the group

`sudo chmod 775 /var/www`

Now set the sticky bit for the group

`sudo chmod g+s /var/www`

The 'g+s' sets the sticky bit for the group, which propagates the group AND the permissions down the tree as new files and directories are created.

This allows all users in the group to freely edit those files and create new directories without having to constantly use the command-line to adjust ownership.

Now we log out and back in, because we modified the user during the current session user session. 

then log back in and go to the `/var/www`

`touch test.md`

the file should be created and you should not get a write permissions error.

Our group has write permissions now. so we don't have to sudo. We can add other members to the group as we without modifying the owner from root.

`rm test.md`

## Securing Debian for public use

So if you did change the `Allowpasswordlogin` in the `sshconfig` file the #1 thing you won't be able to login from a different computer. You have to have the key pairs to login with your account at all. Using it from another laptop isn't going to work because we turned off passwords which you would need to create a new keypair. 

What we want to do is get the box ready to be exposed to the public internet (or simulated), to test out different attacks on the box at different settings.

### Upgrading the system

```bash
sudo apt-get install aptitude

sudo aptitude update

sudo aptitude safe-upgrade
```

The system should already be pretty up to date.

### Locking down presets

I'll be following most of this guide for this section

<https://www.digitalocean.com/community/tutorials/an-introduction-to-securing-your-linux-vps>

onto a quick detour to install and configure IPtables

#### Installing IPtables

I will be following this guide for configuring IPtables

<https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-using-iptables-on-ubuntu-14-04>

Iptables should be installed at the latest version already.

Let's first list the current ruleset for ipTables

```bash
sudo iptables -L
```

Let's add our first rules for 

```bash
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT

sudo iptables -I INPUT 1 -i lo -j ACCEPT
```

-   The first rule explicitly will accept incoming SSH connections. 
-   The second and 3rd rules open up ports for SSH (22) and HTTP (80)
-   The last rule allows system services to speak with each other through the local network loopback.

#### Configuring Fail2ban

Now we want to install `fail2ban` which is a tool for detecting and banning IP address that behave maliciously/ suspiciously.

This doesn't stop an attacker from say spoofing a fake IP address with each new connection, but that's more work. 

We will be folling this guide for the fail2ban section.

<https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04>

Please read this entire article carefully.

##### Installing `fail2ban`

```bash
sudo apt-get update
sudo apt-get install fail2ban -y
```

> The fail2ban service keeps its configuration files in the /etc/fail2ban directory. There is a file with defaults called jail.conf.

I don't recommend editing the default jail file. We will be making a copy of it. It's best to let the `jail.conf` file hold all default configs and then make our changes in a `jail.local`

We will copy over that file, with the contents commented out, as the basis for the jail.local file. You can do this by running:

```bash
awk '{ printf "# "; print; }' /etc/fail2ban/jail.conf | sudo tee /etc/fail2ban/jail.local
```

###### Editing the `jail.local` file

```bash
sudo ratom /etc/fail2ban/jail.local
```

#### Configuring TripWire

Settings for banning / logging abusive ips

## Opening up nginx to the public

Open up virtualbox and debians intall of nginx to the public over your home ip.

## Basic Server Logging

Effective methods of logging and flagging in bound connections

## Ongoing adjustment of settings and services

find a comfortable default to run the server on
