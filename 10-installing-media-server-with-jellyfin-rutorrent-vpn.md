# Installing Jellyfin media server remotely and RSS torrent seedbox integration on Debian

In this tutorial we will be installing a streaming media server (like netflix) but hosted under your control. Then we will install a torrent seedbox configured with RSS to automatically download new shows and then a VPN to protect privacy to the server.

## Get and set up a linux VPS

See guide on setting up a remote debian server

## Setting up jellyfin media server

### Installing Jellyfin

Lets first update the system 

```bash
sudo apt-get update && sudo apt-get upgrade
```

Install HTTPS transport for APT if you haven't already:

```bash
sudo apt install apt-transport-https
```

Import the GPG signing key (signed by the Jellyfin Team):

```bash
wget -O - https://repo.jellyfin.org/debian/jellyfin_team.gpg.key | sudo apt-key add -
```

Add a repository configuration at /etc/apt/sources.list.d/jellyfin.list:

```bash
echo "deb [arch=$( dpkg --print-architecture )] https://repo.jellyfin.org/debian $( lsb_release -c -s ) main" | sudo tee /etc/apt/sources.list.d/jellyfin.list
```

Install Jellyfin

```bash
sudo apt install jellyfin
```

### Managing jellyfin service

```bash
sudo service jellyfin [status/start/stop/restart]
```

check status to see if it's running

### Connecting to and using jellyfin

in your browser go to `http://youripordomain:8096`

You should see something like this

![Jellyfin in browser](http://tinyimg.io/i/WFOjfUw.png)

### Making a folder to store media

Now we get fancyish with user permissions. We want to create a group that will be able to administer said folder. jellyfish, our torrent client, admin panels all might need access to this folder. 

We will create a group and add users to that.

#### Making a user group and assigning permissions

Jellyfin creates a user `jellyfin` that controls the app. We also have our user to add to the group.

Let's create a folder called `jellyfin` in the `var` directory.

```bash
sudo mkdir /var/jellyfin
```

By default the `jellyfin` folder we made is run by root and we will need `sudo` to be able to make changes in it. We could just change the user to our user, but what if we wanted to have other users be able to edit the contents of the `jellyfin` folder. This is where groups come in. A group can be defined to be allowed read/write access to specific files/ folders.

Let's look at all the groups on the machine

```bash
cat /etc/group
```

look at the existing groups for your user

```bash
groups <username>
```

We can see there is a group already called jellyfin and that the `jellyfin` user is part of it.

add your user to that group

```bash
sudo adduser <username> jellyfin
```

list all the groups your user is part of

```bash
groups <username>
```

now change the group of the `/var/jellyfin` folder, while leaving the owner the same.

```bash
sudo chgrp jellyfin /var/jellyfin
```

add write permissions for the group

```bash
sudo chmod 775 /var/jellyfin
```

Now set the sticky bit for the group

```bash
sudo chmod g+s /var/jellyfin
```

The `g+s` sets the sticky bit for the group, which propagates the group AND the permissions down the tree as new files and directories are created.

This allows all users in the group to freely edit those files and create new directories without having to constantly use the command-line to adjust ownership.

Now we log out and back in, because we modified the user during the current session user session. 

then log back in and go to the `/var/jellyfin`

## Now create 2 folders for TV and Movies

```bash
cd /var/jellyfin/
mkdir Movies
mkdir TV
```

the folders should be created and you should not get a write permissions error.

### Set up the folder you just made in the web UI now

Go back to the web UI and create a library for each of the folders.

Leave all the other settings as default for now and you should be fine.

Once you get through the set up you should have something like this on your home page.

![home page](http://tinyimg.io/i/uFWSsrO.png)

### (Totally optional) Customize the interface

#### Change the jellyfin logo

The logo is stored by default in 

`/usr/lib/jellyfin/bin/jellyfin-web/components/themes/logowhite.png`

replacing this will change the logo in the default (dark) theme only

This is the logo on the login and other pages

`/usr/lib/jellyfin/bin/jellyfin-web/img/logo.png`

#### Changing the Title

Comment out the 2 idential lines in the file that change the title

```bash
sudo nano /usr/lib/jellyfin/bin/jellyfin-web/scripts/librarymenu.js
```

```javascript
// document.title = title || 'Jellyfin'
```

then open up the index.html file and edit the title to your choosing

```bash
sudo nano /usr/lib/jellyfin/bin/jellyfin-web/index.html
```

```html
<title>Jellyfin</title>
```

## Setting up a seedbox (rTorrent)

### Installing rtorrent

#### Install Dependencies:

```bash
sudo apt-get install -y rtorrent zlib1g-dev build-essential subversion autoconf screen g++ gcc ntp curl comerr-dev pkg-config cfv libtool libssl-dev libncurses5-dev ncurses-term libsigc++-2.0-dev libcppunit-dev libcurl3 libcurl4-openssl-dev git
```

#### Install xml-rpc

```bash
svn co -q https://svn.code.sf.net/p/xmlrpc-c/code/stable /tmp/xmlrpc-c

cd /tmp/xmlrpc-c

./configure --

make -j2

sudo make install
```

#### Install LibTorrent

```bash
cd /tmp

curl http://rtorrent.net/downloads/libtorrent-0.13.8.tar.gz | tar xz

cd libtorrent-0.13.8

./autogen.sh

./configure

make -j2

sudo make install
```

#### Configure rTorrent

Create a rtorrent configuraton file

```bash
nano ~/.rtorrent.rc
```

and paste in the following

```bash
# Maximum and minimum number of peers to connect to per torrent.
min_peers = 40
max_peers = 100

# Same as above but for seeding completed torrents (-1 = same as downloading)
min_peers_seed = 25
max_peers_seed = 60

# Maximum number of simultaneous uploads per torrent.
max_uploads = 30

# Global upload and download rate in KiB. "0" for unlimited.
#download_rate = 0
#upload_rate = 0

# Default directory to save the downloaded torrents.
directory = /home/downloads

# Default session directory. Make sure you don't run multiple instance
# of rtorrent using the same session directory. Perhaps using a
# relative path?
session = /home/downloads/.session

# Watch a directory for new torrents, and stop those that have been
# deleted.
schedule = watch_directory,5,5,load_start=/home/downloads/~watch/*.torrent

# Close torrents when diskspace is low.
schedule = low_diskspace,5,60,close_low_diskspace=10240M

# The ip address reported to the tracker.
#ip = 127.0.0.1
#ip = rakshasa.no

# The ip address the listening socket and outgoing connections is
# bound to.
#bind = 127.0.0.1
#bind = rakshasa.no

# Port range to use for listening.
port_range = 55950-56000

# Start opening ports at a random position within the port range.
port_random = yes

# Check hash for finished torrents. Might be usefull until the bug is
# fixed that causes lack of diskspace not to be properly reported.
check_hash = yes

# Set whether the client should try to connect to UDP trackers.
use_udp_trackers = yes

# Alternative calls to bind and ip that should handle dynamic ip's.
#schedule = ip_tick,0,1800,ip=rakshasa
#schedule = bind_tick,0,1800,bind=rakshasa

# Encryption options, set to none (default) or any combination of the following:
# allow_incoming, try_outgoing, require, require_RC4, enable_retry, prefer_plaintext
#
# The example value allows incoming encrypted connections, starts unencrypted
# outgoing connections but retries with encryption if they fail, preferring
# plaintext to RC4 encryption after the encrypted handshake
#
encryption = allow_incoming,enable_retry,prefer_plaintext

# Enable DHT support for trackerless torrents or when all trackers are down.
# May be set to "disable" (completely disable DHT), "off" (do not start DHT),
# "auto" (start and stop DHT as needed), or "on" (start DHT immediately).
# The default is "off". For DHT to work, a session directory must be defined.
#
dht = disable

# UDP port to use for DHT.
#
# dht_port = 6881

# Enable peer exchange (for torrents not marked private)
#
peer_exchange = no

scgi_port = 127.0.0.1:5000
```

#### Using rtorrent

if the above went well let's see if rtorrent starts. If not try to fix the errors.

##### starting a new screen session

```bash
screen -S rtorrent

# hit enter one to get back to shell

rtorrent
```

Then detatach from the screen. `ctrl + a`, `d`

You should now be back at your old shell

#### when you want to shut down rtorrent:

Don't do this yet, this is just for your future reference

```bash
screen -r rtorrent
```

then press `ctrl + q`

then kill the screen

```bash
screen kill rtorrent
```

##### Add rtorrent to start up on boot

```bash
sudo nano /etc/init.d/rtorrent
```

Paste in the follwoing script. Make sure to adjust the username line.

```bash
#!/bin/bash
### BEGIN INIT INFO
# Provides:          rtorrent
# Required-Start:    $local_fs $remote_fs $network $syslog
# Required-Stop:     $local_fs $remote_fs $network $syslog
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Start/stop rtorrent daemon
### END INIT INFO

# ------------------------------------------------------------------------------
# /etc/init.d/rtorrent
#
# This script is an init script to run rtorrent in the background, using a
# screen. The script was designed and tested for Debian systems, but may work on
# other systems. On Debian, enable it by moving the script to
# "/etc/init.d/rtorrent" and issuing the command
# "update-rc.d rtorrent defaults 99"
#    ____                _ _
#   / ___|  ___  ___  __| | |__   _____  __
#   \___ \ / _ \/ _ \/ _` | '_ \ / _ \ \/ /
#    ___) |  __/  __/ (_| | |_) | (_) >  <
#   |____/ \___|\___|\__,_|_.__/ \___/_/\_\
#
# @see http://methvin.net/scripts/rtorrent
# @see http://tldp.org/LDP/abs/html/
# ------------------------------------------------------------------------------

## Username to run rtorrent under, make sure you have a .rtorrent.rc in the
## home directory of this user!
USER="<username>"

## Absolute path to the rtorrent binary.
## run "which rtorrent"
RTORRENT="/usr/local/bin/rtorrent"

## Absolute path to the screen binary.
SCREEN="/usr/bin/screen"

## Name of the screen session, you can then "screen -r rtorrent" to get it back
## to the foreground and work with it on your shell.
SCREEN_NAME="rtorrent"

## Absolute path to rtorrent's PID file.
PIDFILE="/var/run/rtorrent.pid"

## Absolute path to rtorrent's XMLRPC socket.
SOCKET="/var/run/rtorrent/rpc.socket"

## Check if the socket exists and if it exists delete it.
delete_socket() {
if [[ -e $SOCKET ]]; then
rm -f $SOCKET
fi
}

case "$1" in
## Start rtorrent in the background.
start)
echo "Starting rtorrent."
delete_socket
start-stop-daemon --start --background --oknodo \
--pidfile "$PIDFILE" --make-pidfile \
--chuid $USER \
--exec $SCREEN -- -DmUS $SCREEN_NAME $RTORRENT
if [[ $? -ne 0 ]]; then
echo "Error: rtorrent failed to start."
exit 1
fi
echo "rtorrent started successfully."
;;

## Stop rtorrent.
stop)
echo "Stopping rtorrent."
start-stop-daemon --stop --oknodo --pidfile "$PIDFILE"
if [[ $? -ne 0 ]]; then
echo "Error: failed to stop rtorrent process."
exit 1
fi
delete_socket
echo "rtorrent stopped successfully."
;;

## Restart rtorrent.
restart)
"$0" stop
sleep 1
"$0" start || exit 1
;;

## Print usage information if the user gives an invalid option.
*)
echo "Usage: $0 [start|stop|restart]"
exit 1
;;

esac
```

###### Set permissions and install the init script.

```bash
sudo chmod +x /etc/init.d/rtorrent
sudo update-rc.d rtorrent defaults 99
```

## Setting up a torrent web client using ruTorrent

### Installing Apache and PHP

```bash
sudo apt-get -y install php php-geoip php7.0-cli php7.0-json php7.0-curl php7.0-cgi php7.0-mbstring libapache2-mod-php libapache2-mod-scgi libapache2-mod-xsendfile apache2 unrar unzip libav-tools ffmpeg mediainfo curl screen sqlite3 git net-tools sox
```

### Installing ruTorrent

```bash
sudo git clone https://github.com/Novik/ruTorrent.git /var/www/rutorrent/
sudo chown -R www-data:www-data /var/www/rutorrent
```

#### Configuring ruTorrent

```bash
sudo nano /var/www/rutorrent/conf/config.php
```

edit the `topDirectory` variable to the following

```php
$topDirectory = '/opt/rtorrent/download'; 
```

then make sure your external dependencies are in the right locations

```php
$pathToExternals = array(
    "php"    => '/usr/bin/php',            // Something like /usr/bin/php. If empty, will be found in PATH.
    "curl"    => '/usr/bin/curl',            // Something like /usr/bin/curl. If empty, will be found in PATH.
    "gzip"    => '/bin/gzip',            // Something like /usr/bin/gzip. If empty, will be found in PATH.
    "id"    => '/usr/bin/id',            // Something like /usr/bin/id. If empty, will be found in PATH.
    "stat"    => '/usr/bin/stat',            // Something like /usr/bin/stat. If empty, will be found in PATH.
);
```

The above are where mine are located. It's probably best to check your self with the `whereis` command.

Something like:

```bash
whereis php
whereis gzip
```

You'll get back:

```bash
php: /usr/bin/php /usr/bin/php7.0 /usr/lib/php /etc/php /usr/share/php7.0-curl /usr/share/php7.0-readline /usr/share/php7.0-mbst
```

then `/usr/bin/php` is the correct answer for that array value.

### Secure ruTorrent web UI

```bash
cd /var/www/rutorrent

sudo htpasswd -c /var/www/rutorrent/.htpasswd your_username_here
```

Then type out the password for your user twice.

```bash
sudo nano /var/www/rutorrent/.htaccess
```

and add the follwing to the file

```bash
AuthUserFile /var/www/rutorrent/.htpasswd
AuthName "ruTorrent_login"
AuthType Basic
require valid-user
```

#### Cofigure access to rutorrent

```bash
sudo nano /etc/apache2/sites-available/rutorrent.conf
```

and add the following

```xml
# ruTorrent
#=====================================================================
<IfModule alias_module>

Alias /rutorrent        /var/www/rutorrent/ 

        <Directory /var/www/rutorrent/>
                Options +Indexes +Includes +FollowSymLinks +MultiViews
                AllowOverride all
                Require all granted
        </Directory>

</IfModule>
#======================================================================
```

Enable ruTorrent WebUI.

```bash
sudo ln -s /etc/apache2/sites-available/rutorrent.conf /etc/apache2/sites-enabled/rutorrent.conf
```

##### Activate Apache2 modules.

```bash
sudo a2enmod auth_digest

sudo a2enmod authn_file

sudo a2enmod xsendfile

sudo a2enmod scgi
```

##### Secure SCGI (/RPC2) directory.

```bash
sudo mkdir -p /etc/apache2/passwords-{available,enabled}
```

**make sure to change [username] and remove brackets below**

```bash
cd /etc/apache2/passwords-available
sudo htpasswd -c rtorrentscgi [username]
```

then enable the rtorrent password with apache

```bash
sudo ln -s /etc/apache2/passwords-available/rtorrentscgi /etc/apache2/passwords-enabled/rtorrentscgi
```

Edit file /etc/apache2/sites-available/rtorrentscgi and add location of SCGI secured by password.

```bash
sudo nano /etc/apache2/sites-available/rtorrentscgi.conf
```

```bash
#rTorrent SCGI Password Location
#===========================================================================
        <LocationMatch "/RPC2">
                AuthType        Basic
                AuthName        "rtorrentscgi"
                AuthUserFile    /etc/apache2/passwords-enabled/rtorrentscgi
                Require         valid-user
                BrowserMatch    "MSIE"  AuthDigestEnableQueryStringHack=On
                Require ip 127.0.0.1
        </LocationMatch>
#===========================================================================

# SCGI PORT
#===========================================================================

#LoadModule scgi_module /usr/lib/apache2/modules/mod_scgi.so
SCGIMount /RPC2 127.0.0.1:5000

#===========================================================================
```

Now enable rtorrentscgi config.

```bash
sudo ln -s /etc/apache2/sites-available/rtorrentscgi.conf /etc/apache2/sites-enabled/rtorrentscgi.conf
```

Restart Apache2 server

```bash
sudo service apache2 restart
```

##### create rtorrent start up scripts

```bash
sudo nano /etc/systemd/system/rtorrent.service
```

```bash
[Unit]
Description=rTorrent
After=network.target

[Service]
UMask=002
Type=forking
RemainAfterExit=yes
KillMode=none
User=rtorrent
ExecStartPre=-/bin/rm -f /opt/rtorrent/session/rtorrent.lock
ExecStart=/usr/bin/screen -d -m -fa -S rtorrent /usr/bin/rtorrent
ExecStop=/usr/bin/killall -w -s 2 /usr/bin/rtorrent
WorkingDirectory=/opt/rtorrent/

[Install]
WantedBy=multi-user.target
```

Enable rTorrent autostart

```bash
sudo service rtorrent start 
```

Now let's test it

```bash
sudo netstat -npl | grep rtorrent
```

### Now let's test it!

in your browser enter `http://your-ip/rtorrent`

### Setting up a domain name

### setting up an nginx reverse proxy

```bash
sudo apt-get install nginx
```

```bash

```

## Setting up RSS feeds

Here we will set up direct access to utorrent web panel and the jellyfin service

### setting up an SSL certificate on both domains

## Setting up a VPN for extra protection

## Getting on private trackers

Now is the time that you go and start talking to people on reddit and other sites about getting invitations to private torrent trackers. Explain during interviews that you have a home rolled seedbox/ media server with loads of bandwidth.

Private trackers will provide you with endless amounts of specialized material released as soon as it's available.

This may take a while to get interviews or meet the right people. But it's worth it once you get there, I promise.
