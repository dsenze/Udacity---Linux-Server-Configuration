# live url: http://18.194.182.23

# Udacity---Linux-Server-Configuration

You will take a baseline installation of a Linux server and prepare it to host your web applications. You will secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.


## Instructions
#### Get your server ready
    1. Start a new Ubuntu Linux server instance on Amazon Lightsail. There are full details on setting up your Lightsail instance on the next page.
    2. Follow the instructions provided to SSH into your server.
#### Configuration
    3. Update all currently instapalled packages.
    4. Change the SSH port from 22 to 2200. Make sure to configure the Lightsail firewall to allow it.
    5. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP
    (port 123).

    6. Create a new user account named grader.
    7. Give grader the permission to sudo.
    8. Create an SSH key pair for grader using the ssh-skeygen tool.

    9. Configure the local timezone to UTC.
    10. Install and configure Apache to serve a Python mod_wsgi application.
        If you built your project with Python 3, you will need to install the Python 3 mod_wsgi package on your server:
        sudo apt-get install libapache2-mod-wsgi-py3 
    11. Install and configure PostgreSQL:
        *   Do not allow remote connections
        *   Create a new database user named catalog that has limited permissions to your catalog application database.
    12. Install git.
    13. Clone and setup your Item Catalog project from the Github repository you created earlier in this Nanodegree program.
    14. Set it up in your server so that it functions correctly when visiting your server’s IP address in a browser. 
        Make sure that your .git directory is not publicly accessible via a browser!

## Howto

#### 1-2. Get a  server.
	 # start VM in Amazon lightsail. https://lightsail.aws.amazon.com
	 # Open ssh console
#### 3-4. Get updates and lock down firewall and connect to VM with Putty.
	
	# ** Install updates **
	sudo apt-get update
	sudo apt-get upgrade #select 'Keep the local version currently installed' if prompted
	
	# ** change ssh port from 22 to 2200, add WWW and port 123 and secure firewall **
	sudo ufw default deny incoming
	sudo ufw default allow outgoing
	sudo ufw allow 2200
	sudo ufw allow www
	sudo ufw allow ntp
	sudo ufw enable
	#!Important: open ports on Amazon VM. Otherwise its an deny from public internet. 
![Image of firewall](https://github.com/dsenze/Udacity---Linux-Server-Configuration/blob/master/firewall.PNG)
	
	sudo vi /etc/ssh/sshd_config #(change port 22 to 2200) and save
	sudo service ssh stop
	sudo service ssh start
	
	# Download SSH key from Amazon Account Page
	# Open putty and add SSH key and connect to public IP on port 2200.
	# Login with  user : ubuntu
	
#### 6-8. Create user, configure SSH Key and sudo permissions
	sudo adduser grader --disabled-password
	sudo su grader
	cd ~
	mkdir .ssh
	chmod 700 .ssh
	touch .ssh/authorized_keys
	chmod 600 .ssh/authorized_keys
	
	# logon with ubuntu user (through putty session)
	sudo vi .ssh/authorized_keys (paste ssh public generated key, used puttygen)
	# Save!
	sudo service ssh start

	# Disable root login
	sudo vi /etc/ssh/sshd_config
	PermitRootLogin no
	sudo service ssh start

#### 9 configure time zone
	sudo dpkg-reconfigure tzdata 
	# select europé / stockholm.

#### 10. Install and configure Apache to serve a Python mod_wsgi application
	$ sudo apt-get install apache2
	$ sudo apt-get install libapache2-mod-wsgi
	$ sudo a2enmod wsgi
	$ sudo service apache2 restart
#### 11. Install and configure PostgreSQL, add DB and User
	
	sudo apt-get install postgresql
	# Check pg_hba.conf file, only localhost allowed. No remote connections allowed.
	sudo vi /etc/postgresql/9.5/main/pg_hba.conf #no action needed
	sudo su - postgres
	psql
	    CREATE DATABASE catalog;
        CREATE USER catalog;
	    ALTER ROLE catalog WITH PASSWORD 'password';
	    GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
	    \q
	
#### 12 - 13. deploy from git, add content, set permissions, install dependencies
	sudo apt-get install git
	cd /var/www
	sudo mkdir itemcatalog
	cd itemcatalog
	sudo git clone -b apache_version https://github.com/dsenze/udacity-FSND-item_catalog.git itemcatalog
	# have done some modifcation in project at apache_version branch. 
    # for example: PSQL connection string, fullname path to files etc.
    # IMPORTANT! be sure to download from apache branch and not from master bransch.
		
	sudo apt-get install python-pip
	sudo apt-get -qqy install postgresql python-psycopg2
	
	# install dependencies for the project
	sudo pip install Flask
	sudo pip install requests
	sudo pip install httplib2
	sudo pip install Flask-SQLAlchemy
	
	# Add some fake data to DB.
	cd /var/www/itemcatalog/itemcatalog
	sudo python model.py
	sudo python add_data.py

    # Give www-data permissions to images folders (to support upload / delete)
    sudo chown -R www-data;www-data uploads
    cd static
    sudo chown -R www-data:www-data images/*
	
#### 14. Set it up in your server so that it functions correctly when visiting your server’s IP address in a browser. 
    sudo vi /etc/apache2/sites-available/item_catalog.conf
    
    <VirtualHost *:80>
            ServerName 18.194.182.23
            ServerAdmin tommy.vadman@dsenze.com
            WSGIScriptAlias / /var/www/itemcatalog/itemcatalog.wsgi
            <Directory /var/www/ItemCatalog/ItemCatalog/>
                    Order allow,deny
                    Allow from all
            </Directory>
            Alias /static /var/www/itemcatalog/itemcatalog/static
            <Directory /var/www/itemcatalog/itemcatalog/static/>
                    Order allow,deny
                    Allow from all
            </Directory>
            ErrorLog ${APACHE_LOG_DIR}/error.log
            LogLevel warn
            CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
    
    # enable site
    sudo a2ensite item_catalog
    
    cd /var/www/itemcatalog
    sudo vi itemcatalog.wsgi
    # add text below
        import sys
        import logging
        logging.basicConfig(stream=sys.stderr)
        sys.path.insert(0,"/var/www/itemctalog/")
        from itemcatalog import app as application
        application.secret_key = 'secretkey'
        sudo service apache2 restart

### Oauth Facebook URL Blocked
#### There where problems with Facebook login.

![Image of error](https://github.com/dsenze/Udacity---Linux-Server-Configuration/blob/master/facebook_error.png)

#### Log into developer account and Server IP for valid oauth redirect URI
![Image of error](https://github.com/dsenze/Udacity---Linux-Server-Configuration/blob/master/facebook_resolution.png)

## Resources used


