# install ejabberd

sudo apt-get install ejabberd

dpkg-reconfigure ejabberd
	\-	hostname: <domainnname>
	\-	username: <username> pricaj
	\-	password: <password> p0ts&p4ns

This username and password is the admin username and password you can use to admin the server
through a web interface - as described in the dpkg gui.

# open ports

    iptables -A INPUT -p tcp --dport 5280 -j ACCEPT
    iptables -A INPUT -p tcp --dport 5222 -j ACCEPT
    iptables -A INPUT -p tcp --dport 5269 -j ACCEPT

# setup and install certbot

    sudo apt-get install certbot

    iptables -A INPUT -p tcp --dport 443 -j ACCEPT

    certbot certonly -d <domain-name> --standalone

    cat /etc/letsencrypt/live/<domain name>/privkey.pem /etc/letsencrypt/live/<domain-name>/fullchain.pem > ejabberd.pem

edit /etc/ejabberd/ejabberd.yaml

Indentation matters.

Pay special attention to make sure you don't an extra space.

Uncomment:

    start_ttlsrequired: true
    fqnd: "<domain-name>"

create a new use

    ejabberdctl register <username> <domain> <password>

restart the service

    systemctl restart ejabberd.service
    tail -f /var/log/ejabberd/ejabberd.log

When your client connects you should see messages in the log that show you have connected.

When your client disconnects you should see messages in teh log that your client has disconnected

# Test your install using your favorite xmpp client

I am going to use pidgin. Add new account in pidgin.

Under the basic tab:

    -   Protocol: XMPP
    -   Username: <user you added in previous step>
    -   Domain:  <domain you configured for ejabberd>
    -   Password: <password from previous step for user>

Under advanced tab:

-   Connection security: require encryption
-   clear file transfer proxy
