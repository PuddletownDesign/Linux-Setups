# Installing Jellyfin a remote media server using docker with a torrent seedbox, RSS automatic downloads and VPN on Debian

## Set up a server

You can set up a local testing machine (Free) with virtualbox or set up a production box on a VPS ($$$).

-   [Set up a local testing server on a VM](01-local-development-server-with-virtualbox)
-   [Set up a remote production VPS](02-configuring-debian-server-on-digital-ocean)

## Installing Docker

New screen for installing

```bash
screen -S install
```

Once you have hello world running you can proceed... 

The second half of the guide goes over basic usage and the commands. If you're not familiar with them please read the whole guide.

-   <https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-debian-10>

### Install `docker-compose`

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.25.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

You may need to reload your shell to get the command to pop up.

Now that you're done kill the screen

```bash
screen kill install
```

## Installing Jellyfin

### configuring docker persistent directories

Make a Docker directory to hold all persistent data

```bash
sudo mkdir /var/Docker

sudo chgrp docker /var/Docker

sudo chmod 775 /var/Docker

sudo chmod g+s /var/Docker
```

Now create the folders we will use for Jellyfin

```bash
take /var/Docker/Jellyfin/

mkdir {config,cache,media,ui}
```

### Create a `docker-compose.yml` file

```bash
touch docker-compose.yml

nano docker-compose.yml
```

paste the following into the file. This will be our template for creating futher instances of this container.

```docker
version: "3"  
services:  
    jellyfin:  
      image: jellyfin/jellyfin  
      network_mode: "host"  
      volumes:  
        - /var/Docker/Jellyfin/config:/config  
        - /var/Docker/Jellyfin/cache:/cache  
        - /var/Docker/Jellyfin/media:/media 
```

Start a new screen to run the daemon.

```bash
screen -S jellyfin
```

Now run `docker-compose`

```bash
# dcu
docker-compose up
```

or with [puddletown ZSH aliases]((https://github.com/PuddletownDesign/Linux-Setups/blob/master/03-installing-puddletown-zsh-git-configs.md))

```bash
dcu
```

If you install these, take a look in the `~/zshrc` file for other docker aliases and change them to whatever you want.

Detach from the screen with: `ctrl+a`, then `d`

Now that you have a prompt back, check that the folders you made are populated with files.

```bash
tree /var/Docker/Jellyfin
```

_Note: you may not have `tree` installed, but it can be installed with `apt-get`_

## Configuring nginx reverse proxy

A reverse proxy takes incoming requests and routes them to a specific place. It's just like a proxy, except backwards.

### Installing nginx as a docker container

Let's make a new screen for nginx

```bash
# ss nginx
screen -S nginx
```

First download the nginx docker image.

```bash
# dkpl
docker pull nginx
```

Then start a container listening to port 80

```bash
# dkr --name
docker run --name nginx -p 80:80 nginx
```

Here we are running the nginx image with the name of nginx and setting port 80 on `[host]:[container]`

Now that the container is running test it with the servers IP on port 80 in the browser.

You should be greated with the default nginx page

![nginx welcome page](http://tinyimg.io/i/znsXpV9.png)

```bash
# List all containers
docker container ls -a

# Stop the container
docker container stop [container-id]

# Start a stopped container
docker container start [container-id]

# delete a  container (dkcrm)
docker container rm [container-id]
```

**Now stop and delete the container**

#### Configuring nginx

You may have noticed that with jellyfin that the host had a bunch of shared folders with the container. 

How would we configure nginx with out a configuration file and no shell access?

Create a local folder for configs or html in `/var/Docker`

```bash
take /var/Docker/nginx
mkdir {sites-enabled,sites-available,www}
```

We've created three folders to start:

-   `sites-available` - to hold each site and services config info
-   `site-enabled` - to activate configs stored in `sites-available`
-   `www` - to hold any web pages/applications in the future

##### What to use it for?

_We have the option of using a new nginx container for each app or site we create on the server, or we can store basic web pages or content, just resusing the same container. This often makes more sense to me to reuse a container in more simple circumstances._

> For example:
> 1\.  I would host static web pages on the same container making new domains.
> 2\.  An application with a database I would create a new container for and bundle it all together.

I base it on portability mostly, but also try to keep resources managable.

It's up to you how you want to set this up for yourself or how your apps are contructed.

### nginx `docker-compose.yml` file

Create a yaml file in the /var/Docker/nginx directory.

```bash
cd /var/Docker/nginx

nano docker-compose.yml
```

```yml
nginx:
  image: nginx
  ports:
   - "80:80"
  environment:
   - NGINX_PORT=80
  command: /bin/bash -c exec nginx -g 'daemon off;'
```
