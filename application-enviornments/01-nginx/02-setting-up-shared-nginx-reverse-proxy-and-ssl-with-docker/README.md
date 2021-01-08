# Setting up separate ngninx block and SSL with docker

In this guide we will be setting up a different installation of nginx for static sites. We will set up one of the sites to remove the `.html` extension and leave the other site as is.

We will have a total of 3 nginx installations for 2 sites.

1.  Will host site #1 (<https://puddletowndesign.com>)
2.  Will host site #2 that will have the `.html` extension removed and forward www to non-www (<https://www.gunstocks.com> redirects to <https://gunstocks.com>).
3.  Will be a reverse proxy that routes each domain to each site's nginx install.

## Prerequisites

Make sure docker and docker compose are installed on your server. 

## Creating the reverse proxy

First we will create the reverse proxy that will automatically generate SSL certs from LetsEncrypt.

Create a `docker-compose.yml` file.

```docker
---
version: "2"
services:
  letsencrypt:
    image: linuxserver/letsencrypt
    container_name: letsencrypt
    cap_add:
      - NET_ADMIN
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Los_Angeles
      - URL=gunstocks.com
      - SUBDOMAINS=www
      - VALIDATION=http
    volumes:
      - </path/to/appdata/config>:/config
    ports:
      - 443:443
      - 80:80 #optional
    restart: unless-stopped
```

server
    reverse proxy and static host
        domain (SSL)
            server block
            sub domains
        domain (SSL)
            server block
            sub domains

Lets first modify the timezone to the correct value for the servers timezone location.

You can get a [list of tz codes here](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones).

## Now let's create the server to host the site

For the first example we will be using nginx-alpine, since we're only going to host a static page.
