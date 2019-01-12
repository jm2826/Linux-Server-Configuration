# Joels Linux Server Configuration Project

This is a project I am coding to complete my Full Stack Nanodegree from udacity.com. I will use Amazon Web Services Lightsail to host my Item Catalog Project.
It will show you the steps to:
-Build a secure virtual server
-Deploy a website on this server


## Public IP Address : 
   18.191.225.137.xip.io

## Make an Amazon Lightsail Instance (cloud platform to host a server)

  1. First, log in to [Amazon Lightsail](https://signin.aws.amazon.com). If you don't already have an Amazon Web Services account, you'll be prompted to create one.

  2. Create an instance. Once you're logged in, Lightsail will give you a friendly message with a robot on it, prompting you to create an instance. A Lightsail instance is a Linux server running on a virtual machine inside an Amazon datacenter.

  3. Choose an instance image: First, choose "OS Only" (rather than "Apps + OS"). Second, choose Ubuntu as the operating system.

  4. Choose your instance plan.For this project, the lowest tier of instance is just fine ($3.50 a month, first month free)

  5. Give your instance a hostname. Joels-Linux-Server-2

  6. Log into it with SSH from your browser.

  7. Once logged in update the ubuntu instance packages:
     * `sudo apt-get update`
     * `sudo apt-get upgrade`

## Create a New User "grader"

   1. To create a user called grader, run the following command :

      * `sudo adduser grader`
         Options:
         User: grader
         Password: grader
         Fullname: Udacity Grader

   2. To give grader sudo permission, run the following command :

      * `sudo visudo`
      and then add the following line right below root user:
      * `grader ALL=(ALL:ALL) ALL`

## Get Default Server SSH Key Pair Private and Public Key 

  1. Download your instance Private Key from your profile, which starts with "LightsailDefaultKey" and with ".pem" extension.

  2. On your server, create a new directory and file to store the SSH publick key:
     `mkdir .ssh`
     `sudo touch .ssh/authorized_keys`

  3. In your local machine (git bash) create a keygen pair and save them on your PC as a .pem file:
     `ssh-keygen`
     save to location `/c/Users/Owner/.ssh/linuxcourse.pem` (this save linuxcourse2.pem and creates the linuxcourse.pub file)
     passphrase = joelsserver

  4. Copy public key from local machine to put onto my server:
    `cat .ssh/linuxcourse.pub`

  5. Go backto ssh connection to Lightsail server and open authorized keys file and paste in public key:
     `nano .ssh/authorized_keys`

  6. verify your logged in as grader and change permission:
     `sudo chmod 700 .ssh`
     `sudo chmod 644 .ssh/authorized_keys`
     `sudo service ssh restart`

## Secure the Server SSH Connection and Enable the Firewall with New Rules

  1. Change the SSH port from 22 to 2200. Make sure to configure the Lightsail firewall to allow it.
     *In Lightsail, goto the networking tab and add a custom Port
      'custom TCP 2200' * if this is done before the rest you run the risk of not being able to ssh back into your server.

     * Back in the SSH Connection, Open SSH config , run the following command :
     `sudo nano /etc/ssh/sshd_config`

     * Change the "Port 22" in the file to "Port 2200"
     * Verify PermitRootLogin is no
     * Verify PasswordAuthentication is no

     * Restart SSH service, run the following command :
     `sudo service ssh restart`


  3. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123), run the following commands :
     * 'sudo ufw deny incoming'
     * 'sudo ufw allow outgoing'
     * `sudo ufw allow 2200`
     * `sudo ufw allow 80`
     * `sudo ufw allow 123`
     * If ufw not installed 'sudo apt-get install ufw -y'

     Then enable ufw, run the following command :
     * `sudo ufw enable`

  4. In Amazon Lightsail, go to "Networking" section then "Firewall" :  
     * Disable SSH -TCP - port   22
     * Add Custom - TCP - port   2200
     * Add Cusotm - UDP - port   123
     * LEAVE HTTP - TCP - port   80 

  5. Change file owner, run the following command :
      `sudo chown -R grader:grader /home/grader/.ssh`
      `sudo service ssh restart`
     Now that SSH port has been changed to 2200, Try exiting the SSH connection and re-connecting to your server, run the following command on your local machine:
    	`ssh -i ~/.ssh/linuxcourse2.pem grader@18.191.225.137 -p 2200`    


## Verify TimeZone is UTC

   1. `sudo dpkg-reconfigure tzdata`



## Application Deployment
   Setting up my application will require a Python (Flask) virtual enviornment, Apache2 Server and mod_wsgi to run between them.      

   1. Install Apache, run the following command at the gobal level:
 
      * `sudo apt-get install apache2`
      * `sudo apt-get install libapache2-mod-wsgi python-dev`
      * `sudo apt-get install python-setuptools libapache2-mod-wsgi`
      * `sudo apt-get install git`
      * `sudo apt-get install python`

   2. Enable mod_wsgi. Any change to server requires a restart to oad in changes.

      * `sudo a2enmod wsgi`
      * `sudo service apache2 restart`     

   3. Beginning to buid the struture of the application enviornment.
      * 'cd /var/www'
      * `sudo mkdir FlaskApp`
      * `sudo chown -R grader:grader FlaskApp`
      * `cd FlaskApp`
      * `git clone [github repository]` FlaskApp
        (https://github.com/jm2826/Item-Catalog.git)

   4. Create my wsgi file in /var/www/FlaskApp and edit the secret key to super_secret_key:
     
      * `sudo nano flaskapp.wsgi`, then copy and paste in code below

      'activate_this = '/var/www/FlaskApp/FlaskApp/venv/bin/activate_this.py'
       execfile(activate_this, dict(__file__=activate_this))

       #!/usr/bin python
       import sys
       import logging
       logging.basicConfig(stream=sys.stderr)
       sys.path.insert(0, "/var/www/FlaskApp")

       from FlaskApp import app as application
       application.secret_key = 'super_secret_key''

   5. Verify your in the directory where your python files are (/var/www/FlaskApp/FlaskApp)
      and rename your application to __init__.py. My file is called projectflask.py.
      * `mv projectflask.py __init__.py`

   6. Your directory structure should now look like this:

	|----FlaskApp
        |---------flaskapp.py
	|---------FlaskApp
	|--------------static
	|--------------templates
	|--------------__init__.py
        |--------------venv(coming in next steps)

## Install virtual machine to seperate future application package installs from the current app.
   
   1.Install Virtual Machine (Verify you are in /var/www/FlaskApp/FlaskApp first)
     * `sudo pip install virtualenv`
     * `sudo virtualenv venv`
     * `source venv/bin/activate`
     * `sudo chmod -R 777 venv`
 
   2. In the virtual machine, install all the required packages mostly using pip:
     * `sudo apt-get install python.pip`
     * `sudo pip install Flask`
     * `sudo pip install sqlalchemy
     * `pip install psycopg2-binary
     * `pip install requests
     * `pip install oauth2client`
     * `pip install passlib`

## Edit Python files and create Virtual Host in Apache2

   1. in the __init__.py file, change the client secrets path for oauth.
     * `sudo nano /var/www/FlaskApp/FlaskApp/__init__.py` Edit this line:
      'CLIENT_ID = json.loads(
      open('/var/www/FlaskApp/FlaskApp/client_secrets.json', 'r').read())['web'][$
       APPLICATION_NAME = "Catalog App" '

     * Since I'm here also edit your database create engine to postgresql 
	***you also want to edit the create databse in my setup_database.py file**:

       `engine = create_engine('postgresql://catalog:password@localhost/catalog')
        Base.metadata.bind = engine'`

     * change `app.run(host='0.0.0.0', port=5000)` to `app.run()`

   2. Create a file called FlaskApp.conf and put it in apache2. 
      ***You can you whatismyipaddress.com to find you ServerAlias***
      * `sudo nano /etc/apache2/sites-available/FlaskApp.conf`
      * Copy and paste the code below:
      * `<VirtualHost *:80>
                ServerName 18.191.225.137
                ServerAdmin jm2826@att.com
                ServerAlias ec2-18-191-225-137.us-east-2.compute.amazonaws.com
                WSGIDaemonProcess FlaskApp user=grader group=grader threads=5
                WSGIProcessroup FlaskApp
                WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi

                <Directory /var/www/FlaskApp/FlaskApp/>
                        Order allow,deny
                        Allow from all
                WSGIProcessGroup FlaskApp
                WSGIApplicationGroup %{Global}
                </Directory>
                Alias /static /var/www/FlaskApp/FlaskApp/static
                <Directory /var/www/FlaskApp/FlaskApp/static/>
                        Order allow,deny
                        Allow from all
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
         </VirtualHost>`

    3. Now we have to enable the virtual host had not have apache use the default virtual host.
       * `sudo a2ensite FlaskApp.conf`
       * `a2dissite 00-default.conf`

## Setup Goole oauth (Edit existing oauth client from my Item Catalog)
    1. For DNS purposes, I had to add `xip.io` to the authorized domain
    2. Now in authorized javascript origins add this: `http://18.191.225.137.xip.io`
    3. In authorized redirect URIs add 3 urls:
       `http://18.191.225.137.xip.io/login`
       `http://18.191.225.137.xip.io/gconnect`
       `http://18.191.225.137.xip.io/oauth2callback`
    4. Save your additions and download the JSON file.
    5. Copy the contents of the JSON file and paste them into you client_secrets.json file on your server:
      * `sudo nano /var/www/FlaskApp/FlaskApp/client_secrets.json`

## Create Database and give user `grader` limited access.
    * `sudo apt-get install libq-dev python-dev`
    * `sudo apt-get install postgresql postgresql-contrib`
    * `sudo su - postgres`
    * `psql`

    * `CREATE USER catalog WITH PASSWORD 'password';`
    * `ALTER USER catalog CREATEDB;`
    * `CREATE DATABASE catalog WITH OWNER catalog;`      
    * `\c catalog`
    * `REVOKE ALL ON SCHEMA public FROM public;`
    * `GRANT ALL ON SCHEMA public TO catalog;`
    * `\q`
    * `exit`

## Finally Restart APACHE2
   * `sudo service apache2 restart`     
   * Now you can access your web app open up http://18.191.225.137.xip.io in the browser.
   
## Third Party Resources Used
* Udacity Full-Stack Nanodegree
* http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/
* https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
* https://whatismyipaddress.com/ip/18.191.225.137
* https://console.developers.google.com/apis/dashboard?project=catalog-project-218501

# Contributing
Please see CONTRIBUTING.md.
All files were originally written by udacity.com and modified by Joel Magee
