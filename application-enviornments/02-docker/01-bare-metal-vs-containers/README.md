# Thoughts on working bare metal or with containers or with a VM

IMHO there's a time and place to work each of these ways. 

I think VMs or containers really shine when it's someone elses library with specific requirements and you would end up with conflicting packages. For instance, you web server needs a different version of PHP than wordpress does for another project.

## Bare metal approach

Bare metal is great for very small and simple things but leads to a lot of redundant work deploying them to production. 

This approach can also lead to conflicting libraries and packages that then need to be sorted out and managed.

## Virtal machines

Virtual machines are great to "containerize" your applications if your business or project already uses them. However spinning up a virtual box dev server only to then spin up a virtualbox server within it, is a bit wasteful with resources and creates additional complexity. I recommend taking this approach only if you need to. Like if your work or projects already require it. Otherwise, it's kind of outdated and silly.

## Containers

Over all these days I really prefer containers. I only need the container software installed on the machine and I don't need to manage packages for each application.

You will notice that a lot of these guides are written for people using this approach.

Containers keep things nicely... "containerized" this also makes deployment a lot easier since it's all wrapped up and ready to go.

Containers are not without their own drawbacks though. There are a lot of security implications to using them. Surprisingly though, performance doesn't take a huge hit.

This does lead to extra complexity such as container management, partitioning, networking and security but with a little knowledge and practice it's mostly mitigated.

### Example decisions/ complexities when working with containers

So the hip thing for the last few years has been microservices. This is where each aspect of an application runs in it's own container and is as small as possible. So each app would have exactly the tools it needed to function.

#### Let's look at an example

1.  You make websites and have a handful of clients.
2.  Each of these clients has slightly different needs, but for the most part it's pretty simple. A lot of static sites, a wordpress install here and there and a couple custom apps.
3.  Say, 3 of these clients require a database. two of them are wordpress and need MYSQL, but generally you prefer postgres for most things.

##### Example scenario

Let's say we have 8 simple apps to manage

1.  A 2 page static website with no bells or whistles.
2.  A static website that has an email form and requires sendmail
3.  A static website that includes redundant resources via PHP includes and has an email form.
4.  A dynamic website that runs PHP5 and has a sqlite database
5.  A node API application that uses postgres
6.  A wordpress website that uses PHP7 and mysql.
7.  A plex/jellyfin mediaserver that will use a bunch of different tools to make the whole thing work.
8.  a ruby on rails application that uses postgres.

##### 9 instances of nginx

If we just gave each container it's own isolated needs you'd have like 8 instanaces of nginx plus, an extra one as a reverse proxy to route them all. 

It would be awesomely easy to deploy each should a client here or there want to change hosts or they just give up on your work and moves on.

But in the meantime you'd have 9 instnaces of ngnix running. 

###### How to weigh this out

On one hand it makes the most sense to KISS and jsut give each project it's own needs and keep it all separated and highly deployable. 

On the otherhand it's incredibly noobish and wasteful with your resources. Linux already provides many tools to do this and you don't want to end up in some universe turtle quandry on your own servers where it's Linux servers running in Linux servers all the way down with your 9 nginxs for 8 projects.

Let's look at the needs a little old school as if we were just going to install all this bare metal.

1.  we need nginx
2.  2 of the sites needs sendmail
3.  2 sites need PHP5 and 1 needs PHP7 
4.  we need ruby and a rails envoirnent
5.  postgres database
6.  mysql database
7.  a plex or jellyfin install
8.  sqlite support

### One possible solution

1.  nginx with PHP7 installed that to act as a reverse proxy and host for static or mostly static websites and has sendmail configured. 
    -   this alone takes care of items 1-3
2.  our dymanic PHP apps requires PHP5 and not 7, so we would create a separate container with another nginx and PHP5 with sqlite support. 
3.  A node container that will run our node app
4.  A postgres container for ur postgres databases
5.  Wordpress is honestly a bit of a shithouse and should never be used, but since we're stuck with it, I would create unique containers for each install and each of it's needs. Probably via a compose script.
6.  Jellyfn or plex app. These are large apps that require a lot of different things to work to their potential. I would handle this by containerizing each aspect of these and not having them overlap with my own work.
7.  For the ruby on rails app, a new container for ruby/rails and then put the data in a fresh DB within the postgres container.

so we end up with the following containers:

1.  nginx - w/ php7 support and as a reverse proxy.
2.  A PHP5 container with nginx
3.  A ruby /rails container running on nginx
4.  A plex and friends container
5.  A wordpress container

Cool! that's not 9 instances of nginx. However now we have to secure this and then network it all together. Bummer, it's complicated again. But at least your not that asshole running 9 versions of the same thing for no good reason and we don't need to figure out how to install 2 different versions of PHP like we would if we installed it bare metal.

I'm not sure there's really a correct answer (with anything really) but I try to take a more balanced approach and keep things resource efficient, seperated and easy to work with.
