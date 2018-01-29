# DevOps NodeJS using Nginx

### Setup Secure web server for NodeJS Application Configuration Management

##### DevOps Configuration to deploy a secure application.

###### Steps to follow:

Step 1:
=====
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
