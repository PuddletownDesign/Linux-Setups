# Configuring git deployment

This will all take place on the Host side of the dev environment.

## Create a simple new project

1.  create a new folder called `html` in your shared folder
2.  create a simple index file (use the above raindrops thing)
3.  import it into git and commit it

We are going to replace the default `/var/www/html` folder that we deleted before when configuring shared folders.

_This time we won't need to modify the nginx file since it's in there my default. Right now just hitting <http://you-ip-address> will result in a forbidden error._ 

## Creating a bare repo on your live server

I've stored all of my repos in `/var/git`. I'm going to create a bare repository here called default.

**On your production server**

```bash
cd /var/git/

take default.git

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
git --work-tree=/var/www/html --git-dir=/var/git/default.git checkout -f
```

save and close and then make the file executable.

```bash
chmod +x hooks/post-receive
```

**Back on your local machine (remember on the host side)**

add it as a remote to the git repo

```bash
git remote add live ssh://brent@example.com/var/git/default.git
```

running `g rv` for git remote -v should show you the remotes you've added

lastly push it to the live server for deployment

```bash
git push live master
```
