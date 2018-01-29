# DevOps NodeJS using Nginx

### Setup Secure web server for NodeJS Application Configuration Management

##### DevOps Configuration to deploy a secure application.

Steps to follow:
===============
 - Create an ec2 instance if you are using AWS and for DigitalOcean you need to create a Droplet. (Ubunt 16.04)
 - Login to the instance/Droplet and run the following commands:
    - `sudo apt-get upgrade`
    - `sudo apt-get update`
 - Install NodeJS version or your choise. You can easily install the nodejs using [NVM](https://github.com/creationix/nvm). Confirm your nodeJS version using following command.
    - `node -v`
    - `npm -v`
 - Install a process mannager of your choice use one of the followig command:
    - `npm install -g pm2`
    - `npm install -g forever`
 
 - Install Nginx using following command:
    - `sudo apt-get install nginx`
 - Install OpenSSL using followig commands:
    - `mkdir openSSL && cd openSSL`
    - `wget http://www.openssl.org/source/openssl-1.0.1g.tar.gz`
    - `wget http://www.openssl.org/source/openssl-1.0.1g.tar.gz.md5` download MD5 hash to verify the integrity of the downloaded file
    - `md5sum openssl-1.0.1g.tar.gz`
    - `cat openssl-1.0.1g.tar.gz.md5`
    - `tar -xvzf openssl-1.0.1g.tar.gz` extract files from openSSl tar.gz
    - `cd openssl-1.0.1g` navigate to the dir
    - `./config --prefix=/usr/local/openssl --openssldir=/usr/local/openssl` congigure the openSSL
    - `make` compile openSSL
    - `sudo make install` install openSSL. if it thow an error ```POD document had syntax errors at /usr/bin/pod2man line 68.``` then try to install using this command `sudo make install_sw`
    
  - OpenSSL has been successfully installed now verify is it working or not using following command to check version:
    - `/usr/local/openssl/bin/openssl version`
  - Encrypt with letsencrypt. Let's add SSL certificates for free using [letsencrypt](https://github.com/certbot/certbot) tool.
    - `cd ~`
    - `git clone https://github.com/letsencrypt/letsencrypt`
    - `cd letsencrypt`
  - Now Genrate an ssl for our domain i.e digitaldot.io using following command:
    `./letsencrypt-auto certonly --webroot --webroot-path /usr/share/nginx/html --renew-by-default --email asif.javed@digitaldot.io --text --agree-tos -d www.digitaldot.io -d digitaldot.io`. This will ask for sudo permissions and install a bunch of dependencies for the client  and  can take quite a while.
  - Configure Nginx with ssl
     - `sudo mkdir /etc/nginx/ssl`
     - `cd /etc/nginx/ssl`
     - `sudo openssl dhparam -out dhparam.pem 4096`
  - Remove the old default configuration for Nginx and add create our own configuration for our service.
     - `sudo rm /etc/nginx/conf.d/default.conf`
     - `sudo vim /etc/nginx/conf.d/digitaldot.io.conf`
     - Add follwing content to the digitaldot.io.conf file
```
        # Remove server identifiers to help against enumeration
        server_tokens off;

        # Add some protection headers for ClickJacking 
        add_header X-Frame-Options DENY;
        add_header X-Content-Type-Options nosniff;

        # Redirect to https
        server {
            listen 80;
            server_name digitaldot.io;
            return 301 https://$host$request_uri;
        }

        # Redirect to https and remove www
        server {
            listen 80;
            server_name www.digitaldot.io;
            return 301 https://digitaldot.io$request_uri;
        }

        server {
            # Listen for HTTPS connections using http2;
            listen       443 ssl http2;
            server_name  digitaldot.io;

            # Define where to find the certificates
            # These will be under the letsencrypt folder 
            ssl_certificate      /etc/letsencrypt/live/digitaldot.io/fullchain.pem;
            ssl_certificate_key  /etc/letsencrypt/live/digitaldot.io/privkey.pem;

            # Cache SSL handshakes
            ssl_session_cache shared:SSL:50m;
            ssl_session_timeout  5m;

            # Use our new Diffie-Hellman parameter for DHE ciphersuites, recommended 4096 bits
            ssl_dhparam /etc/nginx/ssl/dhparam.pem;

            ssl_prefer_server_ciphers   on;

            # disable SSLv3(enabled by default since nginx 0.8.19) since it's less secure then TLS http://en.wikipedia.org/wiki/Secure_Sockets_Layer#SSL_3.0
            ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

            # ciphers chosen for forward secrecy and compatibility
            # http://blog.ivanristic.com/2013/08/configuring-apache-nginx-and-openssl-for-forward-secrecy.html
            ssl_ciphers "ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA256:ECDHE-RSA-AES256-SHA:ECDHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES128-SHA256:DHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES256-GCM-SHA384:AES128-GCM-SHA256:AES256-SHA256:AES128-SHA256:AES256-SHA:AES128-SHA:DES-CBC3-SHA:HIGH:!aNULL:!eNULL:!EXPORT:!DES:!MD5:!PSK:!RC4";

            # Define our root folder
            root /home/ubuntu/digitaldot/;

            # Use gzip to save on bandwith
            gzip on;
            gzip_comp_level 7;
            gzip_types text/plain text/css text/javascript application/javascript;
            gzip_proxied any;

            # Tell the browser to force HTTPS
            add_header Strict-Transport-Security "max-age=31536000;";

            # Optimise internal TCP connections
            tcp_nopush on;
            tcp_nodelay on;

            # Add static file serving from /home/ubuntu/digitaldot/public folder
            location /public {
                sendfile on;
                include /etc/nginx/mime.types;
                charset utf-8;
                expires 30d;
            }

            # Proxy all other requests to our Node.js application
            location / {
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_pass      http://0.0.0.0:2765;
                proxy_redirect  off;
            }
        }
```
 - `sudo /etc/init.d/nginx restart` Restart your nginx service.
 
 - Test your securtiy at [SSL Labs](https://www.ssllabs.com/ssltest/)
 
 
 ## Enjoy your application is up that is runnning on port 2765
