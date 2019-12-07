# Installing configuring and using nginx on Linux with SSL

I will be using my debian server for these examples. This guide assumes that you've followed along with the previous guide, Configuring a Debian server on Digital Ocean.

Currently you should have: 

1.  SSH access enabled and configured
2.  UFW configured and only allowing connections on port 22 for SSH.

## Quick note on domain names

_You will need a domain name to complete this guide. Certificate authorities like Let's encrypt will not issue SSL certs for bare IP addresses. Yes this is highly annoying. One possible solution without purchasing a domain name is using a service like <http://xip.io> however I have not tested this personally._

## Add domain name to host panel

Add your domain name if you've purchased one to your control panel. If you're using a service like xip.io you don't need to do this.

You will also need to forward the name servers from the domain registrar.

## Installing nginx

```bash
sudo apt-get update
sudo apt-get install nginx
```

Now nginx is installed and running on the server. However because of the firewall (UFW) we won't be able to access it yet.

## Opening ports in our firewall (UFW)

```bash
sudo ufw status numbered
```

should return something like this

```bash
$  sudo ufw status numbered
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] 22/tcp                     ALLOW IN    Anywhere
[ 2] 22/tcp (v6)                ALLOW IN    Anywhere (v6)
```

We will need to open ports `80` for http and port `443` for https.

Enter the following:

```bash
sudo ufw allow 80
sudo ufw allow 443
```

Now running `sudo ufw status numbered` will result in something like this.

```bash
$  sudo ufw status numbered
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] 22/tcp                     ALLOW IN    Anywhere
[ 2] 80/tcp                     ALLOW IN    Anywhere
[ 3] 443/tcp                    ALLOW IN    Anywhere
[ 4] 22/tcp (v6)                ALLOW IN    Anywhere (v6)
[ 5] 80/tcp (v6)                ALLOW IN    Anywhere (v6)
[ 6] 443/tcp (v6)               ALLOW IN    Anywhere (v6)
```

As you can see we now have ports 22, 80, 443 open for both ipv4 and ipv6.

Enter your IP or domain name into the browser to make sure that the web server is accessible from the outside. You should be greeted with the nginx welcome page.

![nginx welcome page](http://tinyimg.io/i/znsXpV9.png)

## Creating a user group to be able to edit the `www` folder

By default the `www` folder is run by root and we will need `sudo` to be able to make changes in it. We could just change the user to our user, but what if we wanted to have other users be able to edit the contents of the `www` folder. This is where groups come in. A group can be defined to be allowed read/write access to specific files/ folders.

Let's look at all the groups on the machine

`cat /etc/group`

look at the existing groups for your user

`groups <username>`

Add a group for users who will be able to `rwx` in for a folder. I'm just going to call this group `www-users`. for users who can edit the entire `www` directory. We can add more specific controls for different sites/folders later.

`sudo addgroup www-users`

add your user to that group

`sudo adduser <username> www-users`

list all the groups your user is part of

`groups <username>`

now change the group of the `/var/www` folder, while leaving the owner the same.

`sudo chgrp www-users /var/www`

add write permissions for the group

`sudo chmod 775 /var/www`

Now set the sticky bit for the group

`sudo chmod g+s /var/www`

The `g+s` sets the sticky bit for the group, which propagates the group AND the permissions down the tree as new files and directories are created.

This allows all users in the group to freely edit those files and create new directories without having to constantly use the command-line to adjust ownership.

Now we log out and back in, because we modified the user during the current session user session. 

then log back in and go to the `/var/www`

```bash
cd /var/www
touch test.md
```

the file should be created and you should not get a write permissions error.

Our group has write permissions now. so we don't have to `sudo`. We can add other members to the group without modifying the owner from root.

Now just delete the test file.

```bash
rm test.md
```

## Creating and enabling more than one site

So just for kicks let's leave our IP address serving the default nginx welcome page and set up our domain on a different vHost. This way we will be serving two different pages from two different areas.

in our `/var/www` we will want to have two directories.

-   `html` for serving to the default ip address
-   `<yourdomain.com>/html` - for serving up the new domain name on port `443`

```bash
cd /var/www
mkdir <yourdomain.com>  
mkdir <yourdomain.com>/html 
touch <yourdomain.com>/html/index.html
nano <yourdomain.com>/html/index.html
```

Add a message into your index.html that you just created so we won't just get a blank page.

### Setting up a server block (vhost) and enabling the site for `<yourdomain.com>`

First copy over a default configuration file for `<yourdomain.com>`. This will be served out of the `<yourdomain.com>/html` folder.

```bash
sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/<yourdomain.com>
```

Now open up the the `/etc/nginx/sites-available/<yourdomain.com>` file you just created.

```bash
sudo nano /etc/nginx/sites-available/<yourdomain.com>
```

Scroll down past the default server block to the area that says:

`# Virtual Host configuration for example.com`. Delete the default server block above this and clean up the file. So the whole file looks something like this.

```bash
# Virtual Host configuration for example.com
#
# You can move that to a different file under sites-available/ and symlink that
# to sites-enabled/ to enable it.
#
server {
	listen 80;
	listen [::]:80;

	server_name example.com;

	root /var/www/example.com;
	index index.html;

	location / {
		try_files $uri $uri/ =404;
	}
}
```

1.  Find and replace all `example.com` values with `<yourdomain.com>` value.
2.  Lastly, change the value:

```bash
root /var/www/<yourdomain.com>;
```

to 

```bash
root /var/www/<yourdomain.com>/html;
```

Save and close the file.

### Enabling the site in nginx

By default with this method each additional server block will need to be enabled before they can be used. You will do this by creating a symlink to the `sites-enabled` folder.

```bash
sudo ln -s /etc/nginx/sites-available/<yourdomain.com> /etc/nginx/sites-enabled/
```

In order to avoid a possible hash bucket memory problem that can arise from adding additional server names, we will go ahead and adjust a single value within our /etc/nginx/nginx.conf file. Open the file now:

```bash
sudo nano /etc/nginx/nginx.conf
```

Within the file, find the server_names_hash_bucket_size directive. Remove the # symbol to uncomment the line:

```bash
http {
    . . .

    server_names_hash_bucket_size 64;

    . . .
}
```

Save and close the file.

#### Verify that all of the nginx files have no syntax errors

```bash
sudo nginx -t
```

If no problems are found and everything is looking good. Go ahead and restart nginx.

```bash
sudo systemctl restart nginx
```

### Testing the server blocks out

First try to connect to `<yourdomain.com>` in the browser. You should see the index.html you created.

Now try connecting to the ip address directly. You should still see the default nginx welcome page.

## Adding a SSL cerificate to enable https in the browser.

We want to add https to `<yourdomain.com>` server block so users will connect through port 443 instead of 80. 

_We will not be able to use https to reach the ip address directly in the `/var/www/html` directory. A FQDN (Fully qualified domain name) is required to get a SSL certificate._

We want to direct all port 80 traffic to port 443 to be encrypted for `<yourdomain.com>`

_UPDATE: Guide will soon be updated to configure wildcard(\*) domains to cover, chat.domain.com, mail.domain.com, domain.com etc..._

### Installing certbot

Update apt and install certbot

```bash
sudo apt-get install python-certbot-nginx -y
```

### Obtaining a certificate for your domain

We will for now just obtain a certificate for `<yourdomain.com>`and not `<www.yourdomain.com>`.

```bash
sudo certbot --authenticator webroot --installer nginx
```

1.  Enter your email
2.  Agree to Terms of Service
3.  Enter all applicable domains
4.  Enter your web root. `/var/www/<yourdomain.com>/html/`
5.  Secure - Make all requests redirect to secure HTTPS access

This will create your certificate in `/etc/letsencrypt/live/<yourdomain.com>/fullchain.pem`

and save a symlink in `/var/www/<yourdomain.com>/html/.well-known/`

Now go back to the browser and test it out. enter `<yourdomain.com>`. You should be automatically forwarded to the https version of the site on port 443. There will be a little lock in the address bar. Note if you select the address and try to connect with just `http://<yourdomain.com>` you will once again be forwarded to `https://<yourdomain.com>`.

Your site is now secured with SSL. This is good. However these certificates expire every 3 months. Let's set up auto renewal.

## Cron Job to Auto renewing SSL certificates

Now we will create a "cron job" to auto renew our certificate. Cron is a command to automate tasks by time. It looks a little weird at first but is actually very simple to set up. 

Learn more about cron here:

<http://mediatemple.net/blog/news/complete-beginners-guide-cron-part-1/>

After reading through how it works, let's set up a cron job to try to renew the certificate every week. _Note: the reason I'm selecting weekly is because the certs will not renew until they are close to expiring. If we set this for once a month, we might miss the expiration date and try to renew after it's already expired._

We will need the cronjob to run as root, since using `certbot` requires `sudo`.

```bash
su
```

enter your root password.

### Dry run test

Let's do a dry run test as root to make sure everything's set up correctly.

```bash
certbot renew --dry-run
```

Hopefully you get a long message containing:

```bash
Congratulations, all renewals succeeded. The following certs have been renewed:
  /etc/letsencrypt/live/<yourdomain.com>/fullchain.pem (success)
** DRY RUN: simulating 'certbot renew' close to cert expiry
**          (The test certificates above have not been saved.)
```

### Setting up cron for the script

```bash
crontab -e
```

Now enter the following into the bottom of the file below the comments. _Note that you have to include the full path to certbot. To find out where a program is located you can run `which certbot`_

```bash
0 0 * * 0 /usr/bin/certbot renew --quiet
```

Now save and close the file.

While still root you can see the cronjob you just added you can run:

```bash
crontab -l
```

We are all done with root account for now so `exit` back to your main user account.

## Conclusion

And there we have it!

In this tutorial you learned how to:

-   Install nginx
-   Open firewall ports 80, 443 in UFW 
-   Adjust user/group permissions to edit the `www` folder
-   Configure server blocks to host more than one site
-   Obtain an SSL certificate to use https
-   Auto renew the certificate with a cron job

## Where to go from here

I recommend [learning how to set up git deployment for both the static IP and the new site](https://github.com/PuddletownDesign/Linux-Setups/blob/master/git-deployment.md).

Good luck!
