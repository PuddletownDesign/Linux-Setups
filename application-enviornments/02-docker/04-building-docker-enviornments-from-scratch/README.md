# Building envoirnments in docker from scratch

Here we will cover:

-   Choosing a base image
-   Creating a customized image from the base image
-   Creating a dockerfile
-   Removing everything you don't need (Optional)
-   Creating a docker-compose file
-   Creating persistent storage
-   Networking 2 containers
-   Security best practices for web apps
-   Creating a production docker compose file

## Creating a dockerfile from scratch

In this tutorial I will be creating a mariadb container and a php apache container. 

I know this isn't the coolest combo, but it's classic and it's what I need to do at the moment. This will all work the same way for any other language/ data store combo.

## Initial considerations on architecture

How should we structure these containers? My general philosophy is separate them anytime possible.

-   Grouping all dependencies into a single container is very portable but more expensive. This is how you end up with 17 versions of nginx running on the same server...
-   separating containers encourages reuse and better security but is slightly less portable.

We need to install a LAMP stack. I'm going to

-   Have a container for mariadb
-   Have another container for PHP and apache

mariadb can run fine alone. But PHP can't really serve pages without a webserver so it's a tightly coupled dependency.

This way I can reuse the mariadb container for ruby or python projects.

If it needs to be packaged up and sent to production, it can be automated in the production docker compose file.

### Chosing a base image

I'm going to use Alpine Linux in this tutorial. I strongly recommend taking the time to learn alpine it as it's a slimmed down version of linux that's great for production. You can use a debian distro and follow along all the same replacing `apt` commands with `apk`.

First lets check docker hub for an offical alpine image.

<https://hub.docker.com/_/alpine/>

I'm going to be using `3.11.2` as it's the newest one at this time.

### Create a container to configure from the base image

The syntax is:

```bash
docker create --name [container name] -p [host ports:container ports] [base image location]
```

Here we are naming the container `alpine` and using `alpine:3.11.2` as the base image.

```bash
docker create --name alpine alpine:3.11.2
```

Now run it. The following command starts the `alpine` container `sh` shell. 

> `-it`: Interactive terminal.

```bash
docker run -it alpine /bin/sh
```

_Notice here we are using the `sh` shell. This is because alpine doesn't have bash installed out of the box. ;)_

## Creating a Dockerfile

Now that we are logged into our machine we want to go about setting it up like we would a normal VM. The difference here is that we will take our notes in the Dockerfile. This will be used to build another image on top of this one with the required software installed.

Let's first add our container creation command to the `Dockerfile`. `FROM` describes the base image to use.

```bash
FROM alpine:3.11.2
```

Now, back in the container, let's update the existing software.

```bash
apk update
```

Did this run without error and complete the updated we needed? If so add that command to the dockerfile.

```bash
FROM alpine:3.11.2
RUN apk update
```

Let's install mariadb

```bash
apk install mariadb
```

Did that work? Great! Add it to your dockerfile. Here's what mine looks like so far.

```bash
FROM alpine:3.11.2
RUN apk update && apk install mariadb
```

You get the picture...

Let's build out set up script

```bash
FROM alpine:3.11.2
RUN apk update && apk install mariadb sudo
# add our user and group first to make sure their IDs get assigned consistently, regardless of whatever dependencies get added
RUN adduser maria && addgroup maria mysql
```

Now we've updated the base install and installed mariadb.

## Docker compose
