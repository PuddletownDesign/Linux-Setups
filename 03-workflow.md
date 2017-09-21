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

now change the group of the `/var/www/html` folder, while leaving the owner the same.

`sudo chgrp sftp-users /var/www/html`

add write permissions for the group

`sudo chmod 775 /var/www/html`

Now set the sticky bit for the group

`sudo chmod g+s /var/www/html`

The 'g+s' sets the sticky bit for the group, which propagates the group AND the permissions down the tree as new files and directories are created.

This allows all users in the group to freely edit those files and create new directories without having to constantly use the command-line to adjust ownership.

Now we log out and back in, because we modified the user during the current session user session. 

`exit`

then log back in and go to the `/var/www/html`

touch test.md

our group has write permissions now. so we don't have to sudo. We can add other members to the group as we without modifying the owner from root.

Now we are going to configure a separate nginx server block to host wordpress.

At first we will basically duplicate the above process for another folder in `var/www`. However we will just set `var/www` to the same permissions as `var/www/html`

```bash
sudo chgrp www /var/www

sudo chmod 775 /var/www

sudo chmod g+s /var/www

exit
```

Great so now users of `sftp-users` can edit `/var/www` directory and everything within.

we now want to pull wordpress from their website. on the website there is a download link to a zip file. copy the link and edit it in.

in `/var/www`

```bash
wget https://wordpress.org/latest.zip
```

now we need to install unzip because we don't yet have it installed

```bash
sudo apt-get install unzip -y
```

unzip it

```bash
unzip latest.zip
```

trash the zip file

```bash
rm latest.zip
```

> **A mega note**: if we had not adjusted `/var/www` for our group we would have had to wget with `sudo`. Pulling remote code with `sudo` is FUCKING SHADY. Even from a trusted source. Note you gotta sudo to apt-get though. This gives whatever you are downloading `sudo` permissions. If there is malicious code, it's now `sudo`. Be VERY CAREFUL pulling remote code with sudo FFS.

Ok now we have to install a bunch of stuff to run wordpress. Mainly PHP and Mysql.

<https://www.digitalocean.com/community/tutorials/how-to-install-wordpress-with-lemp-on-ubuntu-16-04>

I'll cover most of that guide here.

anyway first we gotta install a LEMP (Linux, NGINX, MySQL, PHP) stack. 

<https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-in-ubuntu-16-04>

So we already have nginx installed, let's next install MySQL

```bash
sudo apt-get install php-fpm php-mysql mysql-server -y
```

You can login to mysql by `sudo mysql`

And log out with `exit`

Now we need to adjust a few PHP configurations

```bash
sudo ratom /etc/php/7.0/fpm/php.ini
```

Find the line with 

```php
;cgi.fix_pathinfo=1
```

change it to 

```php
cgi.fix_pathinfo=0
```

> This is an extremely insecure setting because it tells PHP to attempt to execute the closest file it can find if the requested PHP file cannot be found. This basically would allow users to craft PHP requests in a way that would allow them to execute scripts that they shouldn't be allowed to execute.

We will change both of these conditions by uncommenting the line and setting it to "0" like this:

Save and exit

Now restart PHP

```bash
sudo systemctl restart php7.0-fpm
```

## Configure Nginx to Use the PHP Processor

```bash
sudo ratom /etc/nginx/sites-available/default
```

This is the file that you can configure blocks of availble sites. So we can have more than just `/var/www/html`

The top block is being served out of `/var/www/html`

Let's configure another to serve out of `/var/www/wordpress`

Down towards the bottom of the file you will see `# Virtual Host configuration for example.com`

We're going to copy this block and uncomment it. Don't just uncomment and modify the existing block. Leave it for reference.

```nginx
# Wordpress.Debian
server {
	listen 80;
	listen [::]:80;

	server_name wordpress.debian;

	root /var/www/wordpress;
	index index.php;

	location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php7.0-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

save and then test for any syntax errors

```bash
sudo nginx -t
```

if the syntax is messed up go back and check for errors

Now reload nginx 

```bash
sudo systemctl reload nginx
```

Now On your mac, we are going to edit your host file again.

```bash
a /etc/hosts
```

On the very last line you should have a host pointing to your server

```hosts
192.168.1.71 debian.server 
```

add the server name of the wordpress folder to that list. MAKE SURE IT IS STILL THE CORRECT IP.

```hosts
192.168.1.71 debian.server wordpress.debian
```

Hitting the url on the mac side should now serve the wordpress install!!! Don't do anything with it yet.

```bash
http://wordpress.debian
```

## Setting up and securing MYSQL

Now we want to do is very similar to the directory permission except with mysql. 

1.  Secure the root account with a password
2.  Set up a new user for website access
3.  Give that user the least amount of DB permissions needed to run wordpress

```bash
sudo mysql_secure_installation
```

1.  current pass is empty (just hit return)
2.  yes for new password
3.  enter new pass
4.  Remove anonymous users? [Y/n] y
5.  Disallow root login remotely? [Y/n] y
6.  Remove test database and access to it? [Y/n] y
7.  Reload privilege tables now? [Y/n] y 

test logging in

```bash
sudo mysql -uroot -p

exit
```

let's edit the config file

```bash
sudo ratom /etc/mysql/my.cnf
```

add the following above client server

```mysql
bind-address = 127.0.0.1
local-infile=0
log=/var/log/mysql.log
```

1.  we say that we only accept connections from the local machine
2.  then a directive to disable this ability to load local files
3.  then we set a location for the log file

Log back into mysql

CREATE USER 'brent'@'localhost' IDENTIFIED BY '15cakesmashes';

Now let's create the wordpress database

and then give our new user permissions to use it

in mysql

    CREATE DATABASE wordpress;

    show databases;

You can see that wordpress is there now

```mysql
GRANT SELECT , INSERT , UPDATE , DELETE, ALTER, CREATE , DROP , INDEX ON  `wordpress` . * TO  'wordpress'@'localhost';

GRANT ALTER, CREATE , DROP , INDEX ON  `wordpress` . * TO  'wordpress'@'localhost';

SHOW GRANTS FOR 'brent'@'localhost';
```

Now rename the configuration file and edit it

```bash
mv wp-config-sample.php wp-config.php

ratom wp-config.php
```

then fill in `DB_NAME`, `DB_USER` and `DB_PASSWORD`,

Go back to your browser on mac and hit the url for the wordpress server.

```bash
http://wordpress.debian
```
