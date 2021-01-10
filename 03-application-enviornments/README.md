# Installing Docker

## Have a server set up

You can set up a local testing machine (Free) with virtualbox or set up a production box on a VPS ($$$).

-   [Set up a local testing server on a VM](01-local-development-server-with-virtualbox)
-   [Set up a remote production VPS](02-configuring-debian-server-on-digital-ocean)

## Installing Docker

```bash
sudo apt-get update

sudo apt install -y apt-transport-https ca-certificates curl gnupg2 software-properties-common

curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"

sudo apt update

apt-cache policy docker-ce

sudo apt-get install -y docker-ce
```

Test it:

```bash
sudo systemctl status docker
```

Once you have hello world running you can proceed... 

The second half of the guide goes over basic usage and the commands. If you're not familiar with them please read the whole guide.

### Run docker without sudo

```bash
sudo usermod -aG docker ${USER}
```

## Install `docker-compose`

Find the latest release here:

<https://github.com/docker/compose/releases>

Right now it is `1.26.0`

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.26.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

You may need to reload your shell to get the command to pop up.

Now that you're done kill the screen

```bash
screen kill install
```
