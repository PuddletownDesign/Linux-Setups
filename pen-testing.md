# Pen Testing Debian on Kali

Enabling more services and finding known exploits

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

## Understanding layer of obfuscation

## Configuring Kali

We need to make Kali be able to speak with the debian vm over the local connection.

1.  Taking a snapshot on debian know the state that you started with. Always. This way you can go back and trace specific changes from clean to compromised.
2.  Searching for specific attacks against wide spread software.
3.  Emabling semi secured servies you think you can exploit
4.  Research strange computer science concepts and find unfortunate consequences at a low level.
5.  Bug bounties don't pay for known bugs, so find specifically interesting lines of research and follow them.
