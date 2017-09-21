# Debian production server

![debian](http://tinyimg.io/i/k9vc7m5.png)

## Updating the system

#### Working with user groups

look at all the groups on the machine

`cat /etc/group`

look at the existing groups for your user

`groups brent`

add a group for users who will be able to rwx in the folder

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
