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
* `sudo apt-get install finger`  // [finger](http://manpages.ubuntu.com/manpages/precise/man1/finger.1.html)
* `sudo apt-get install ntp`  // [NTP](http://www.ntp.org/) (network time protocol)
* `sudo apt-get install apache2`  // [Apache](https://httpd.apache.org/) http web server
* `sudo apt-get install libapache2-mod-wsgi`  // [mod_wsgi](https://modwsgi.readthedocs.io/en/develop/) - Apache module for WSGI compliant hosting python web apps
* `sudo apt-get install postgresql`  // [PostgreSQL](https://www.postgresql.org/) for db management
* `sudo apt-get install python-pip`  // [`pip`](https://pypi.python.org/pypi/pip) command for python packages
* `sudo pip install virtualenv`  // [virtualenv](https://virtualenv.pypa.io/en/stable/) for isolated python environments
* `sudo pip install Flask`  // [Flask](http://flask.pocoo.org/) python web framework
* `sudo pip install sqlalchemy`  // [SQLAlchemy](http://www.sqlalchemy.org/) ORM
* `sudo pip install psycopg2`  // [PostgreSQL](http://initd.org/psycopg/) ORM
* `sudo apt-get install libpq-dev`  // [Contains binaries and headers required for building 3rd party apps with PostgreSQL](https://pypi.python.org/pypi/libpq-dev/9.0.0)

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
    
## Project set up
* `sudo a2enmod wsgi`  // enables wsgi apache module.  Viewable in /etc/apache2/mods-enabled.
* Created project location:
  * /var/www/Catalog/Catalog
* Branched [item catalog project repo](https://github.com/cardvark/item-catalog-project/tree/ec2-postgres-server) on local machine to make changes.
  * Replaced SQLite db with PostgreSQL, log in with catalog role ([Line 10](https://github.com/cardvark/item-catalog-project/blob/ec2-postgres-server/database_setup_catalog.py))
  * Renamed main python file to [\_\_init\_\_.py](https://github.com/cardvark/item-catalog-project/blob/ec2-postgres-server/__init__.py)
  * Set absolute file paths for 'client_secrets.json' and 'fb_client_secrets.json' [Lines 29, 245, 334](https://github.com/cardvark/item-catalog-project/blob/ec2-postgres-server/__init__.py)
* Set up virtual environment
  * `sudo virtualenv venv`  // name for virtual environment, stored as directory in ../Catalog/Catalog
  * `source venv/bin/activate`  // activated venv virtual environment
  * `sudo pip install Flask`
  * `sudo pip install sqlalchemy`
  * `sudo pip install psycopg2`
  * `deactivate` // to leave virtual environment
* Configure new virtual host
  * `sudo vim /etc/apache2/sites-available/Catalog.conf`  // Set up ServerName, WSGIScriptAlias path, Alias paths for /static and /templates
  * `sudo a2ensite Catalog`  // enable Catalog site; viewable in /etc/apache2/sites-available, linked in ../sites-enabled
  * `sudo a2dissite 000-default.conf`  // disabled default apache2 page.  Removed symlink in ../sites-enabled
* Create .wsgi file
  * `sudo vim /var/www/Catalog/CatalogApp.wsgi`  // set path, import app (from \_\_init\_\_.py in Catalog), moved application secret here.
* `sudo service apache2 restart`  // restarted apache server

## Resources
* [Deploying a Flask app on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
* [Configuring time to UTC](https://help.ubuntu.com/community/UbuntuTime)
* [Disabling root remote login](https://mediatemple.net/community/products/dv/204643810/how-do-i-disable-ssh-login-for-the-root-user)
* [Creating new role in PostgreSQL](https://www.postgresql.org/docs/9.2/static/app-createuser.html)
* [Altering database properties](https://www.postgresql.org/docs/9.1/static/sql-alterdatabase.html)
* [Stack Overflow](http://stackoverflow.com/ "All night long.")
