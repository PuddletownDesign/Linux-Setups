# Setting up XMPP Server on a VPS with Prosody

<https://www.youtube.com/watch?v=-0M0NeZ_cU4>

## Installing Prosody Server

```bash
sudo apt-get install prosody mercurial -y
```

check that prosody is running after install

```bash
sudo systemctl status prosody
```

### Configure the firewall

Find out what tcp ports you need to open

```bash
netstat -utlpn
```

In the `tcp` section I get back

```bash
tcp        0      0 0.0.0.0:5269            0.0.0.0:*               LISTEN      -
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -
tcp        0      0 0.0.0.0:443             0.0.0.0:*               LISTEN      -
tcp        0      0 0.0.0.0:5222            0.0.0.0:*               LISTEN      -
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      -
```

and in `ufw` my current open ports are:

`sudo ufw status`

```bash
22/tcp                     ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
443/tcp                    ALLOW       Anywhere
```

So I will need to open `5269`, `5222`

```bash
sudo ufw allow 5222
sudo ufw allow 5269
```

### Downloading required modules

```bash
hg clone https://hg.prosody.im/prosody-modules/ prosody-modules
```

### Configuring the Server

Let's stop the server until we finish configuring it since it's not yet configured or secure.

```bash
sudo systemctl stop prosody
```

Let's open up the configuration file, I'm going to use ratom. If you are using atom too, installing the language-lua plugin will add syntax highlighting.

```bash
sudo ratom /etc/prosody/prosody.cfg.lua
```

_Note: Lua comments are denoted by `--` two hyphens_

#### Configuring the virtual host

look for the following lines in the file:

```lua
VirtualHost "example.com"
	enabled = false -- Remove this line to enable this host
```

and replace it with the following:

```lua
VirtualHost "<yourdomain.com>"
	enabled = true -- Remove this line to enable this host
```

Now let's edit the SSL/TLS path. Right below the virtual host you should see this block.

```bash
ssl = {
	key = "/etc/prosody/certs/example.com.key";
	certificate = "/etc/prosody/certs/example.com.crt";
}
```

Replace it with the following:

```lua
ssl = {
	key = "/var/lib/prosody/chat.<yourdomain.com>.key";
	certificate = "/var/lib/prosody/chat.<yourdomain.com>.crt";
}
```

#### Add an admin user

Search for the string (line 23 for me)

```lua
admins = { }
```

and add in a user

```lua
admins = { "<your-user>@<yourdomain.com>" }
```

#### Add Multi user chat room support

find

```lua
Component "conference.example.com" "muc"
```

and replace with:

```lua
Component "mu.chat.<yourdomain.com>" "muc"
	restrict_room_creation = "admin"
	-- modules_enabled = {}
```

#### Adding BOSH support for xmpp over http

In case our users have connection problems we want to enable BOSH. It's also very useful for sharing files by creating an HTTP server for XMPP.

```lua
--"bosh"; -- Enable BOSH clients, aka "Jabber over HTTP"
```

uncomment this line so it looks like this

```lua
"bosh"; -- Enable BOSH clients, aka "Jabber over HTTP"
```

#### Preventing untrusted users from creating an account

find the following line

```lua
"register"; -- Allow users to register on this server using a client and change passwords
```

...And comment it out:

```lua
-- "register"; -- Allow users to register on this server using a client and change passwords
```

At this point we should be done with the configuration file. Go ahead and save and close it.

### Getting a TLS/SSL certificate

Back on the command line... 

```bash
sudo prosodyctl cert generate <yourdomain.com>
```

Fill in each of the values that it requests.

### Create your first user

Create the admin user first that you specified in the config file.

```bash
sudo prosodyctl adduser <username>@<yourdomain.com>
```

### Start the prosody server

```bash
sudo systemctl start prosody
```

#### Make sure there are no configuration errors

```bash
sudo systemctl status prosody
```

### Configure the DNS for your server

-   Add an `A` record for `chat.<yourdomain.com>`
-   Add a `SRV` record for client
    -   service and protocol: `_xmpp-client._tcp`   
    -   Target: `xmpp.<yourdomain.com>`
    -   Weight: `5`
    -   Priority: `20`
    -   TTL: `43200`
    -   Port: `5222`
-   Add a `SRV` record for server
    -   host: `_xmpp-server._tcp`   
    -   Target: `xmpp.<yourdomain.com>`
    -   Weight: `5`
    -   Priority: `20`
    -   TTL: `43200`
    -   Port: `5269`
-   Add a `TXT` record
    -   host: `_xmppconnect` 
    -   value: `_xmpp-client-xbosh=http://chat.<yourdomain.com>:5280/http-bind`

## Connecting on a client

I recommend starting with pidgin or adium depending on OS to test this out really quick.

Log in with both of the users you made (creating two different accounts in the client) and make sure that you can connect and chat. Try enabling OTR to see if it's working correctly.

If everything is looking good, our xmpp server is configured!

## Conclusion

in this guide you learned how to:

-   Configure an XMPP server using prosody
-   Set up TLS (SSL) encryption
-   add new users
-   connect to the server with a client
