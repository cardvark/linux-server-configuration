# linux-server-configuration
Ubuntu linux server hosted on AWS EC2 with apache2 web server, flask app, postgresql DB

## Overview
* Address: http://ec2-52-42-219-117.us-west-2.compute.amazonaws.com/
* Hosting the [Item Catalog udacity project](https://github.com/cardvark/item-catalog-project/tree/ec2-postgres-server)

## Accessing the server:
* IP: 52.42.219.117
* Port: 2200
* Key-based authentication is enforced.
* Login via: `ssh grader@52.42.219.117 -i ~/.ssh/<key_file> -p 2200`

## Packages installed
* `sudo apt-get install finger`  // finger
* `sudo apt-get install ntp`  // NTP (network time protocol)
* `sudo apt-get install apache2`  // apache http web server
* `sudo apt-get install libapache2-mod-wsgi`  // mod_wsgi - Apache module for WSGI compliant hosting python web apps
* `sudo apt-get install postgresql`  // postgresql for db management
* `sudo apt-get install python-pip`  // `pip` command for python packages
* `sudo pip install virtualenv`  // virtualenv for isolated python environments
* `sudo pip install Flask`  // Flask python web framework

## Configurations
* Update and upgrade all packages
  * `sudo apt-get update`
  * `sudo apt-get upgrade`
* Added user grader
  * `sudo adduser grader`
  * Made super user
    * `sudo vim /etc/sudoers.d/grader`
    * `grader ALL=(ALL) ALL`
  * Set up key-based authentication
    * On local machine:
      * `ssh key-gen`  // Generate key
      * Copied contents of .pub file
    * On remote server:
      * `sudo vim ~/.ssh/authorized_keys`  // Pasted contents of pub key to (in grader home dir)
      * Set permissions for folder and file:
        * `sudo chmod 700 ~/.ssh`
        * `sudo chmod 644 ~/.ssh/authorized_keys`
* Updated sshd_config file:
  * `sudo vim /etc/ssh/sshd_config`
  * Listening on Port 22 to '2200'
  * Set PasswordAuthentication to 'no'
  * Set PermitRootLogin to 'no' (after confirming ability to log in as grader and use sudo)
* Set up firewall ports:
  * `sudo ufw default deny incoming`
  * `sudo ufw default allow outgoing`
  * `sudo ufw allow 2200/tcp`  // for SSH
  * `sudo ufw allow www`  // default 80
  * `sudo ufw allow ntp`  // default 123
* Change timezone to UTC:
  * `sudo dpkg-reconfigure tzdata`
* Add catalog user:
  * `sudo adduser catalog`
* Postgresql db management:
  * `sudo -i -u postgres`  // switch to postgres db super user
  * `sudo createuser -D -R -S catalog`  // create new user 'catalog' -No database creation, -No role creation, -Not superuser.
  * `createdb catalog`  // create new catalog db
  * `psql`  // enter postgresql interactive shell
  * `ALTER USER catalog PASSWORD '<password>';`  // Gave catalog role a password
  * `ALTER DATABASE catalog OWNER TO catalog;`  // Change catalog DB owner to catalog role.
  * Note - 3 'catalog' names in use:
    * linux user 'catalog'.  No sudo privileges
    * postgresql role 'catalog'.  No create db, no create role, not super user.
    * postgresql DB 'catalog'.  Database for storing item catalog app data.
