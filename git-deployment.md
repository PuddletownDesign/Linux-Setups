# Configuring git deployment

This will enable us to push our projects to the web using a command like `git push live master`.

## Create a simple new project

This will all take place on the Host side of the dev environment (Mac or on your computer)

### Create a new folder called `html` in your projects folder (shared folder we set up in [Local development server with Virtual box](https://github.com/PuddletownDesign/Linux-Setups/blob/master/local-development-server-with-virtualbox.md))

Here's my fancy raindrops thing if you want to test with that

```bash
<!DOCTYPE html><html lang="en"><head><meta charset="utf-8"><meta name="viewport" content="width=device-width; initial-scale=1.0">
<title>Index</title>
<style type="text/css">
html,body{margin:0;padding:0}html{position:relative;height:100%;background:#000111 url(data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0idXRmLTgiPz48IURPQ1RZUEUgc3ZnIFBVQkxJQyAiLS8vVzNDLy9EVEQgU1ZHIDEuMS8vRU4iICJodHRwOi8vd3d3LnczLm9yZy9HcmFwaGljcy9TVkcvMS4xL0RURC9zdmcxMS5kdGQiPjxzdmcgdmVyc2lvbj0iMS4xIiBpZD0iRHJvcHBsZXQiIG9wYWNpdHk9IjAuNyIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIiB4bWxuczp4bGluaz0iaHR0cDovL3d3dy53My5vcmcvMTk5OS94bGluayIgeD0iMHB4IiB5PSIwcHgiIHdpZHRoPSI1MDBweCIgaGVpZ2h0PSI1MDBweCIgdmlld0JveD0iMCAwIDUwMCA1MDAiIGVuYWJsZS1iYWNrZ3JvdW5kPSJuZXcgMCAwIDUwMCA1MDAiIHhtbDpzcGFjZT0icHJlc2VydmUiPjxwYXRoIGZpbGw9IiMwOEJCREMiIGQ9Ik0yNTYuMjk3LDM4LjQ5OWMwLDEwNC40NzUtMTM5LjI5OCwxNjkuNDgxLTEzOS4yOTgsMjc4LjU5OGMwLDEwOS4xMiw5NC44OTcsMTM5LjI5OSwxMzkuMjk4LDEzOS4yOTljNDQuNDAyLDAsMTM5LjMtMzAuMTc5LDEzOS4zLTEzOS4yOTlDMzk1LjU5NywyMDcuOTgsMjU2LjI5NywxNDIuOTczLDI1Ni4yOTcsMzguNDk5eiIvPjwvc3ZnPg==) center;background-size:10em}body{width:100%;height:100%;background-color:rgba(0,1,17,0.7)}body::before,body::after{position:absolute;content:" ";left:0px;width:100%;height:50%}body::before{top:0;background:linear-gradient(to bottom, #000111 0%,rgba(0,1,17,0.2) 100%)}body::after{bottom:0;background:linear-gradient(to bottom, rgba(0,1,17,0.2) 0%,#000111 100%)}
</style></head><body></body></html>
```

### import it into git and commit it

```bash
git add .
git commit -m "initial import"
```

We are going to push this to `/var/www/html`. This is the page that is served when a user hits the static ip.

## On the server side

First SSH into your server.

First create a folder to hold our production git repos. _Note these will not serve the site, they will track changes that are pushed to the server and deploy the site with whatever scripts are needed._

I've stored all of my repos in `/var/git`. I'm going to create a bare repository here called default.

**On your production server**

```bash
sudo mkdir /var/git/
```

_Note: We had to use `sudo` here because the `var` directory is owned by root. We need to fix that now by adding a group similar to what we did in [using nginx on Linux with SSL](https://github.com/PuddletownDesign/Linux-Setups/blob/master/installing-configuring-and-using-nginx-on-linux.md) guide._

### Correcting user permissions for our new git folder

By default the `git` folder is owned by root and we will need `sudo` to be able to make changes in it. We could just change the user to our user, but what if we wanted to have other users be able to edit the contents of the `git` folder. This is where groups come in. A group can be defined to be allowed read/write access to specific files/ folders.

Add a group for users who will be able to `rwx` in for a folder. I'm just going to call this group `www-users`. for users who can edit the entire `www` directory. We can add more specific controls for different sites/folders later.

If you haven't already added this group `www-users` do so now.

`sudo addgroup www-users`

add your user to that group

`sudo adduser <username> www-users`

list all the groups your user is part of

`groups <username>`

now change the group of the `/var/git` folder, while leaving the owner the same.

`sudo chgrp www-users /var/git`

add write permissions for the group

`sudo chmod 775 /var/git`

Now set the sticky bit for the group

`sudo chmod g+s /var/git`

The `g+s` sets the sticky bit for the group, which propagates the group AND the permissions down the tree as new files and directories are created.

This allows all users in the group to freely edit those files and create new directories without having to constantly use the command-line to adjust ownership.

Now we log out and back in, because we modified the user during the current session user session. 

then log back in and go to the `/var/git`

```bash
cd /var/git
touch test.md
```

the file should be created and you should not get a write permissions error.

Our group has write permissions now. so we don't have to `sudo`. We can add other members to the group without modifying the owner from root.

Now just delete the test file.

```bash
rm test.md
```

### Creating a repo for our default html folder (served from our IP address)

```bash
mkdir ip.git
cd ip.git
git init --bare
```

Git repositories have a folder called 'hooks'. This folder contains some sample files for possible actions that you can hook and perform custom actions set by you.

Git documentation define three possible server hooks: 'pre-receive', 'post-receive' and 'update'. 'Pre-receive' is executed as soon as the server receives a 'push', 'update' is similar but it executes once for each branch, and 'post-receive' is executed when a 'push' is completely finished and it's the one we are interested in.

```bash
a hooks/post-receive
```

I will add the following to the blank file. You might need to adjust some values....

```bash
#!/bin/sh
git --work-tree=/var/www/html --git-dir=/var/git/ip.git checkout -f
```

save and close and then make the file executable.

```bash
chmod +x hooks/post-receive
```

## Add the new remote repo we just created to your project

**Back on your local machine (remember on the host side)**

add it as a remote to the git repo

```bash
git remote add live ssh://<username>@<ip-address-or-domain>/var/git/ip.git
```

running `g rv` for `git remote -v` should show you the remote you've added

## Pushing changes to the live repo

lastly push it to the live server for deployment

```bash
git push live master
```

That's it! As long as you don't get any errors you should be able to look at the page on the web as see the updates were applied. 

## How to add another site (`<yourdomain.com>`)

Now that we've set everything up and added one site, let's go ahead and add another for `<yourdomain.com>` so you can see the exact, basic process for each future site.

_This assumes that you've [created a server block for `<yourdomain.com>` and have nginx configured properly](https://github.com/PuddletownDesign/Linux-Setups/blob/master/installing-configuring-and-using-nginx-on-linux.md)_.

### Creating a new live repo

First, **SSH into your server**.

```bash
mkdir /var/git/<yourdomain.com>.git
cd /var/git/<yourdomain.com>.git
git init --bare
a hooks/post-receive
```

enter the following into the file:

```bash
#!/bin/sh
git --work-tree=/var/www/<yourdomain.com>/html --git-dir=/var/git/<yourdomain.com>.git checkout -f
```

Save and close the file, then make it executable.

```bash
chmod +x hooks/post-receive
```

### Creating a local repo if you don't already have one

In your projects folder

```bash
mkdir <yourdomain.com>
cd <yourdomain.com>
touch index.html
```

open it and enter a message into the file.

Then we will make it a git repo and add the remote.

```bash
git init
git add .
git commit -m "initial import"
git remote add live ssh://<username>@<ip-address-or-domain>/var/git/<yourdomain.com>.git
git push live master
```

## Conclusion

In this tutorial we learned how to:

1.  Create a remote bare git repo
2.  Fix permissions for git
3.  Edit the post hooks file to manage deployment
4.  add a new remote to a local repos
5.  deploy the local repo to the live server
6.  set up an additional site

## Where to go from here

I recommend building on this lesson and the last three by setting up a static site generator and deploying it to a new server block with git.
