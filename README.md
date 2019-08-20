# Linux Server Configuration

This is the final project for Udacity's [Full Stack Web Developer Nanodegree](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004).

This Readme will detail the steps I followed to set up a Linux distribution on an Amazon Lightsail server. This document will also show the steps followed to secure the server from multiple attack vendors, create a new user with Super User access and finally configure the server to host a Python Flask web application with a PostgreSQL database.

- The Linux distribution is [Ubuntu](https://ubuntu.com/download/server) 16.04 LTS.
- The virtual private server is [Amazon Lighsail](https://aws.amazon.com/lightsail/).
- The hosted web application is my [Item Catalogue Project](https://github.com/conorbailey90/item-catalogue) created earlier in the course.
- The database server is [PostgreSQL](https://www.postgresql.org/).

You can visit [here](http://ec2-35-178-196-68.eu-west-2.compute.amazonaws.com/catalogue/) for deployed web application.

Public IP Address: 35.178.196.68

username: `grader`

# Create A Server

## Step 1: Create a new Ubuntu Linux server instance on Amazon Lightsail

- Login into [Amazon Lightsail](https://aws.amazon.com/lightsail/) using an Amazon Web Services account.
- Once logged in, click `Create instance`.
- Select `Linux/Unix` platform, `OS Only` and `Ubuntu 16.04 LTS`.
- Choose an instance plan. For this project I selected the cheapest plan.
- Choose a name for your new instance or keep the default name provided by AWS.
- Click `Create` and wait for the instance to start up.

### Reference
- ServerPilot, [How to Create a Server on Amazon Lightsail](https://serverpilot.io/docs/how-to-create-a-server-on-amazon-lightsail).

## Step 2: SSH into the server

- Go to the "SSH Keys" tab under your Lightsail Account page. Select the `Default` option under your region and download the key pair file. This will be a .pem file called `LightsailDefaultPrivateKey-us-west-2.pem`.
- On your local machine, open up your terminal and navigate to the directory where the above file was downloaded to. Move the `LightsailDefaultPrivateKey-us-west-2.pem` to your local ssh directory `~/.ssh` with the following command: `mv LightsailDefaultPrivateKey-us-west-2.pem ~/.ssh`. 
- Navigate to your local ssh directory with `cd ~/.ssh`. Next run `chmod 600 LightsailDefaultPrivateKey-us-west-2.pem` at the command line to restrict file permission so only you can read it.
- Run `ssh -i LightsailDefaultPrivateKey-us-west-2.pem ubuntu@35.178.196.68` to establish the ssh connection to your Lightsail
server.

# Secure Your Server

## Step 3: Update and upgrade the installed packages 

Run the following commands when you first log in to your server:

```
sudo apt-get update

sudo apt-get upgrade

sudo apt-get autoremove
```

## Step 4: Change the SSH port from 22 to 2200

- Edit the /etc/ssh/sshd_config file with the following command: `sudo nano /etc/ssh/sshd_config`
- Change the port number on line 5 from 22 to 2200.
- Save and exit using CTRL+X and confirm changes with Y.
- Restart SSH: `sudo service ssh restart`.

## Step 5: Configure the Uncomplicated Firewall (UFW)

- Configure the default firewall for Ubuntu to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).

```
sudo ufw status
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw allow 123/udp
sudo ufw deny 22
sudo ufw enable
```

- Enable the UFW with the following command: `sudo ufw enable`. You will see the following message:

```
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup
```

- Exit the SSH connection: `exit`

- Click on the `Manage` option of the Amazon Lightsail Instance, then the Networking tab, and then change the firewall configuration to match the UFW settings above. 

- From your local terminal, run: `ssh -i LightsailDefaultPrivateKey-us-west-2.pem -P 2200 ubuntu@35.178.196.68`, where 35.178.196.68 is the public IP address of the server instance.


# Add new Super User Grader

## Step 6: Create a new user account with the name `grader`

- SSH into your server as user: `ubuntu`
- Once logged in create a new user called `grader` with the following command: `sudo adduser grader`
- Enter a password for the user and proceed to complete the information fields for this user.


## Step 7: Provide super user access to grader:

- Create a new sudoers file for `grader` with the following command: `sudo nano /etc/sudoers.d/grader`

- Add the following line to the empty file:
```
grader ALL=(ALL) ALL
```
- Save and exit using CTRL+X and confirm changes with Y.

- Check grader is listed under the following command: `sudo ls /etc/sudoers.d`

## Step 8: Create an SSH key pair for `grader` using the ssh-keygen tool

- On your local machine, open your terminal and navigate to the `~/.ssh` directory.
- run the command: `ssh-keygen`
- Choose the file name (I chose `lightsailserver`)
- Enter an optional passphrase. I left this blank. Two files will be generated (`lightsailserver` and `lightsailserver.pub`)
- Run `cat ~/.ssh/lightsailserver.pub` and copy the contents of the file.
- Now SSH into the Amazon Lightsail server as the `grader` user with the following command: `ssh grader@35.178.196.68`.
- Once logged in, create a new directory called `~/.ssh` with the following command: `mkdir .ssh`
- Run the command: `sudo nano ~/.ssh/authorized_keys` and paste the above contents into this file. Save and exit using CTRL+X and confirm changes with Y.
- Set up specific file permissions on the `ssh` directory and `authorized_keys` file. This is a security measure that ssh enforces to ensure other users cannot gain access to your account. We'll set the permission using the following commands

```
chmod 700 .ssh
chmod 644 .ssh/authorized_keys
```

- Disable password based logings with the following command: `sudo nano /etc/ssh/sshd_config`. Change `PasswordAuthetication` from `yes` to `no`

- Restart SSH: `sudo service ssh restart`

- Back on your local machine, run: `ssh -i ~/.ssh/lightsailserver -p 2200 grader@35.178.196.68`. to log in as the grader with the generated key.

## Step 9: Configure the local timezone to UTC.

- Open the timezone selection dialog with the following command: `sudo dpkg-reconfigure tzdata`. Then chose `None of the above`, then `UTC`.

# Install and configure Apache2 to serve a Python mod_wsgi application

## Step 10: Installing Apache2

- Whilst logged in to Ubunutu as `grader`, enter the following command to install the `Apache2` web server: `sudo apt-get install apache2`

- Visit the servers public ip address in your webs browser (35.178.196.68). The Apache2 Ubuntu default page should be displayed which shows the installation was successfull.

- Next, configure Apache to hand-off certain requests to an application handler - mod_wsgi. The first step in this process is to install mod_wsgi. My project was built with Python 3 so the following command was used to install the Python 3 mod_wsgi package: 

`sudo apt-get install libapache2-mod-wsgi-py3`

- Enable mod_wsgi with the following command: `sudo a2enmod wsgi`


# Install and configure PostgreSQL

## Step 11: Installing PostgreSQL

- My Item Catalogue project was initially built using an SQLite3. We will be reconfiguring this to a PostgreSQL database for this Linux VPS porject. Whilst logged in to ubuntu as `grader`, install PostgreSQL with the following command: `sudo apt-get install postgresql`.

- PostgreSQL should not allow remote connections. In the /etc/postgresql/9.5/main/pg_hba.conf file, you should see:

```
local   all             postgres                                peer
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```

- Switch to the postgres user with the following command: `sudo su - postgres`
- Open the PostgreSQL interactive terminal with the following command: `psql`
- Create a user called `catalogue` with a password and give them the ability to create databases with the following commands:

```
postgres=# CREATE ROLE catalogue WITH LOGIN PASSWORD 'catalog';
postgres=# ALTER ROLE catalogue CREATEDB;
```
- List the existing database roles with the following command: `/du`. The output should be as follows:
```
                                   List of roles
 Role name |                         Attributes                         | Member of 
-----------+------------------------------------------------------------+-----------
 catalogue   | Create DB                                                  | {}
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}

 ```
 - Exit psql: `\q`. Switch back to the grader user: `exit`.

 - Create a new Linux user called `catalogue`with the following command: `sudo adduser catalogue`. Enter password and fill out the details.

 - Give the `catalogue` user the permission to run `sudo` commands. Create a new sudoers file for `catalogue` with the following command: `sudo nano /etc/sudoers.d/catalogue`

- Add the following line to the empty file:
```
catalogue ALL=(ALL) ALL
```
- Save and exit using CTRL+X and confirm changes with Y.

- Check `catalogue` is listed under the following command: `sudo ls /etc/sudoers.d`

- Log into your Linux server as the `catalogue` user.

- Create a new database with the following command: `createdb catalogue`
- Run `psql` and then run `\l` to see that the new database has been created. The output should be as follows:

```
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
-----------+----------+----------+-------------+-------------+-----------------------
 catalog   | catalog  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
(4 rows)
```
- Exit psql with the following command: `/l` 

## Step 12: Install GIT

- Whilst logged into the server as `grader`, install `git` with the following command: `sudo apt-get install git`

# Deploying the Item Catalogue Project

- Whilst logged in as `grader` cd into the `/var/www/` directory with the following command: `cd /var/www`
- Create a new directory called `catalogue` with the following command: `mkdir catalogue`
- Change to the `catalogue` directory and clone the Item Catalogue Project with the following command: `sudo git clone https://github.com/conorbailey90/item-catalogue.git catalogue`
- From the `/var/www` directory, change the ownership of the `catalogue` directory to `grader` using: `sudo chown -R grader:grader catalogue/`
-  Change to the `/var/www/catalogue/catalogue` directory.
- Rename the `application.py` file to `__init__.py` using the following command: `mv application.py __init__.py`
- Open/Edit the `__init__.py` with the following command: `sudo nano __init__.py`
- Navigate to the bottom of the `__init__.py` file and replace the line `app.run(host="0.0.0.0", port=8000, debug=True)` with `app.run()`
- In the files `application.py`, `database_setup.py` and `dummy_data.py`, replace the line `engine = create_engine("sqlite:///item-catalogue.db")` with the following (Be sure to replace 'PASSWORD' with the password you created for the `catalogue` user):

```
engine = create_engine('postgresql://catalogue:PASSWORD@localhost/catalogue')
```

##  Install the virtual environment and dependencies
- While logged in as grader, install pip: `sudo apt-get install python3-pip`
- Install the virtual environment: `sudo apt-get install python-virtualenv`
- Navigate (cd) to the `/var/www/catalogue/catalogue/` directory.
- Create a new virtual environment: `sudo virtualenv -p python3 venv3`
- Change the ownership to `grader` with the following command: `sudo chown -R grader:grader venv3/`
- Activate the new virtual environment: `source venv3/bin/activate`
- Install the following dependencies:
```
pip install httplib2
pip install requests
pip install --upgrade oauth2client
pip install sqlalchemy
pip install flask
sudo apt-get install libpq-dev
pip install psycopg2
```
- Initiate the application with: `python3 __init__.py`. You should see:

```
* Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```
- Press `CTRL+C` and deactivate the virtual environment with: `deactivate` 

## Set up and enable a virtual host

- Add the following line in `/etc/apache2/mods-enabled/wsgi.conf` file to use Python 3:
```
#WSGIPythonPath directory|directory-1:directory-2:...
WSGIPythonPath /var/www/catalog/catalog/venv3/lib/python3.5/site-packages
```
- Save the file and exit.
- Next, run the command `sudo nano /etc/apache2/sites-available/catalogue.conf` and add the following lines to configure the virtual host:
```
<VirtualHost *:80>
                ServerName 35.178.196.68
                ServerAdmin admin@35.178.196.68
                ServerAlias ec2-35-178-196-68.eu-west-2.compute.amazonaws.com
                WSGIScriptAlias / /var/www/catalogue/catalogue.wsgi
                <Directory /var/www/catalogue/catalogue/>
                        Order allow,deny
                        Allow from all
                </Directory>
                Alias /static /var/www/catalogue/catalogue/static
                <Directory /var/www/catalogue/catalogue/static/>
                        Order allow,deny
                        Allow from all
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
- Enable virtual host: `sudo a2ensite catalogue.` The following prompt will be returned:
```
Enabling site catalog.
To activate the new configuration, you need to run:
  service apache2 reload
```
- Reload Apache: `sudo service apache2 reload`

## Set up the Flask application

- Run the command ` sudo nano /var/www/catalogue/catalogue.wsgi` and add the following code to the file: 

```
#activate_this = '/var/www/catalogue/catalogue/venv3/bin/activate_this.py'
#with open(activate_this) as file_:
    #exec(file_.read(), dict(__file__=activate_this))


#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalogue/catalogue")
sys.path.insert(1, "/var/www/catalogue/")

from catalogue import app as application
application.secret_key = 'Add your secret key'
```
- Restart Apache: `sudo service apache2 restart`


## Update Google Authentication

- Find the hostname of your server using the [hcidata](https://www.hcidata.info/host2ip.cgi). Copy your servers public IP address and click `Find Host Name`. The hostname for my Item Catalog project is `http://ec2-35-178-196-68.eu-west-2.compute.amazonaws.com`.
- Log in to the [Google Developers Console](https://console.developers.google.com/), select `Credentials` from the left menu and open the relevant application which is linked to the `Item Catalogue` application.
- Add the above hostname (http://ec2-35-178-196-68.eu-west-2.compute.amazonaws.com) and the servers public IP address to the list of Authorised JavaScript origins.
- Add `http://35.178.196.68.xip.io/gconnect` to the Authorized redirect URIs list. Click `Save`
- In the Google Developers Console, click back into the application and download the corresponding JSON file. Open the file and copy the contents.
- Log back into the server as `grader` and update the `client_secrets JSON` file with the following command: `sudo nano /var/www/catalogue/catalogue/client_secrets.json`. Replace the entire contents with the updated JSON contents copied from Google.
- CTRL+X and y to save and exit
- Update the `CLIENT_ID` path in the `__init__.py` file to the following:
```
CLIENT_ID = json.loads(open('/var/www/catalogue/catalogue/client_secrets.json', 'r').read())['web']['client_id']
```
- Save the file and exit.

# Helpful Resources

## Websites

1. DigitalOcean [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

## Github Repos

1. [kamalneel178/Linux_Server_Configuration](https://github.com/kamalneel178/Linux_Server_Configuration)
2. [CPinnkathok/Linux-Server-Configuration-Project](https://github.com/CPinnkathok/Linux-Server-Configuration-Project)
