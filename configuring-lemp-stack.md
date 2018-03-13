# COnfiguring a LEMP stack

here we will go through creating a LEMP stack. We've already installed nginx, now we will install & configure PHP and MYSQL

We will use wordpress as an example for this lesson.

## Installing PHP and Mysql

```bash
sudo apt-get install php-fpm php-mysql mysql-server -y
```

## Configuring PHP

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

Save and exit

Now restart PHP

```bash
sudo systemctl restart php7.0-fpm
```

## Configuring MYSQL

### Setting up and securing MYSQL

#### Goals

1.  Secure the root account with a password
2.  Set up a new user for website access
3.  Give that user the least amount of DB permissions needed to run wordpress

Run secure installation

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

add the top of the configuration file

```mysql
# The MariaDB configuration file

[mysqld]
bind-address=127.0.0.1
local-infile=0
log=/var/log/mysql.log
```

save and close

1.  we said that we only accept connections from the local machine
2.  then a directive to disable this ability to load local files
3.  then we set a location for the log file

Log back into mysql

CREATE USER 'brent'@'localhost' IDENTIFIED BY '15cakesmashes';

## Configuring vhosts in nginx

We are going to start installing wordpress as an example here.

We want to add other folders that will work as a server root for a specific domain name.

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

and restart nginx

```bash
sudo service nginx restart
```

## Installing Wordpress

Download wordpress to your local share folder on host

rename the `wp-config-sample.php` to `wp-config.php`

### Set up the Mysql Database

Log back into mysql

Now let's create the wordpress database

and then give our new user permissions to use it

in mysql

```mysql
CREATE DATABASE wordpress;

show databases;
```

You can see that wordpress is there now

```mysql
GRANT SELECT , INSERT , UPDATE , DELETE, ALTER, CREATE , DROP , INDEX ON  `wordpress` . * TO  'brent'@'localhost';

GRANT ALTER, CREATE , DROP , INDEX ON  `wordpress` . * TO  'brent'@'localhost';

SHOW GRANTS FOR 'brent'@'localhost';
```

Now rename the configuration file and edit it

```bash
mv wp-config-sample.php wp-config.php

ratom wp-config.php
```

### Installing wordpress

You should now have a copy of wordpress in the `/var/www` that you downloaded on the host side.

### Configure the wordpress config

in the wordpress/config.php file edit in your database user details

### Give it a domain in the Host OSs host file

in your local (host side) hosts file, add a domain name for wordpress

```bash
sudo atom /etc/hosts
```

mine looks something like this (fix it for your ip and whatever you want the fake domain name to be)

```hosts
192.168.56.107 debian.server wordpress.debian
```

Then open up <http://wordpress.debian> in your browser.

## A quick review

To create a new site as a vhost you need to:

1.  Create a new directory for it in the shared folder
2.  edit the sites-available/default file to make an nginx block
3.  Modify your Host OSs /etc/hosts file with the new name
