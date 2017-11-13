##### ATTENTION: This project is **OBSOLETE** (NO longer works) as far as everything used (in *this* case) has been shut/closed/deleted now that graduation has come and gone. Feel free to use it as a guide to get to where you wanna be. Or whatever.

------------
## Linux Server Configuration

Udacity Final Project, Full Stack Nanodegree
*by Teraisa*

### The Project Briefly

The hosting of an application through the installation of a secure Linux server and a database server using:

Public IP address: ~~34.213.124.212~~ (now: *yourIPaddress*)
SSH port: 2200
Username:*grader*

### The Project
Using Amazon Lightsail (a new Ubuntu Linux server), secure and set up a Linux server. Ssh into the server, update all current installed packages, change the port to 2200.

Configure the ufw to allow connections ONLY for ssh  *-p 2200* (and open the port firewall first on Lightsail to avoid locking yourself out), http *-p 80*, and ntp *-p 123*.

Next, create a a new user (call it *grader*) and give it sudo permissions. Get an ssh key pair for *grader* by using the ssh-keygen tool. While you're at it, the timezone needs to be UTC (*make it happen*).

Install Apache2 and configure it to serve a Python mod-wsgi app. Install PostgreSQL and configure it with no remote connections. Create a new database user (call it *catalog*). Install Git.

Clone or downoad *your project* into your server so that it operates properly while using the IP address in a browser.

Finally, deploy your application via your secure server using the IP address in a browser (without your .git director being publicly accessible in the broswer).

### The Project:  Step by Painful Step

**Get and Secure Your Server**
After setting up your AWS Lightsail, adding your ECDSA key to .ssh in *known_hosts*, login to terminal as root then:

**Login to root (Unbuntu) using:**
`ssh ubuntu@yourIPaddress`

Answer `yes` if asked if you are sure you want to continue connecting (permanently adds ECDSA fingerprint), permission will be denied due to *publickey*

Login again by SSH'ing into ubuntu with IP and path to your *lightsailkey*:
`ssh ubuntu@yourIPaddress -i ~/.ssh/lightsailKey.pem`

*If* you are denied access with the *WARNING:
UNPROTECTED PRIVATE KEY FILE!* change:
`chmod 400 ~/lightsailKey.pem`
*Then* login:
`ssh ubuntu@yourIPaddress -i ~/lightsailKey.pem`

**Optional** Pull up another terminal window and test your command to *double check*:
`ssh ubuntu@yourIPaddress -i ~/lightsailKey.pem`

>**See Also:**
[Get started on Lightsail,](https://classroom.udacity.com/nanodegrees/nd004-connect/parts/63d60eee-becf-43eb-8cfe-bbad19a4cd43/modules/357367901175462/lessons/3573679011239847/concepts/c4cbd3f2-9adb-45d4-8eaf-b5fc89cc606e)
[After setting up your AWS Lightsail,](https://discussions.udacity.com/t/setting-up-lightsail-ports/273747)
[Issues deploying application to Lightsail](https://discussions.udacity.com/t/issues-deploying-application-to-lightsail/403522)

**As *ubuntu@34.213.124.212* Create New User, *grader***
`sudo adduser grader`
*program asks for password* `xxx`

Confirm *grader*'s' Existence (see bottom of output)
`sudo cat /etc/passwd` 
*and/or*
`finger grader`

Give *grader* Sudo Permission
Create a new file in *sudoers* directory
`sudo usermod -a -G sudo grader`

##FYI
>When you want to check *sudo privileges*
`sudo -l`
If you forget to use *sudo*
`sudo !!` means to *repeat last command*
OPTIONAL:
to ad insults when you mess up your password add
................
Defaults       insult
................
`sudo visudo`

>**See Also:** 
[How to add and delete users on Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps)

**Update Package Source List**
`sudo apt-get update`

**Upgrade Software**
`sudo apt-get upgrade`
`sudo apt-get dist-upgrade`

**Install Finger (for debugging)**
`sudo apt-get install finger`

**Remove Unnecessary Packages**
`sudo apt-get autoremove`

***ON LOCAL MACHINE* Generate Key-based Authentication for *grader***
`ssh-keygen`
When asked about file path and passphrase:
`yes` *or* press enter to */Users/teraisa/.ssh/id_rsa*; `no` to passphrase (the command line will print your saved key's path and it's fingerprint)

>**See Also:* 
[Generate a New SSH Key,](https://www.ssh.com/ssh/keygen/) 
[SSH Keys,](https://wiki.archlinux.org/index.php/SSH_keys) 
[How to Configure SSH Key-Based Authentication,](https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-freebsd-server) 
[Generating a new SSH key and adding it to the ssh-agent,](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/) 
[SSH Essentials: Working with SSH Servers, Clients, and Keys](https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys)

**Continue *ON LOCAL MACHINE***
Copy Key and Intall it as an Authorized User
`ssh-copy-id ubuntu@yourIPaddress`
`cat ~/.ssh/id_rsa.pub ssh-rsa`

>**See Also:**
[SSH Copy ID](https://www.ssh.com/ssh/copy-id)

**Logged in as *Ubuntu***
Temporarily switch to new user *grader*
`su grader`
`cd`

Make *.ssh* directory
`mkdir ~/.ssh`

Add your public key *ssh-rsa* from ssh/id_rsa.pub ssh-rsa file
`nano ~/.ssh/authorized_keys`

Restrict permissions
`chmod 700 ~/.ssh`
`chmod 600 ~/.ssh/authorized_keys`

>**See Also:**
[Connect to a Server by Using SSH on Linux or Mac OS X](https://support.rackspace.com/how-to/connecting-to-a-server-using-ssh-on-linux-or-mac-os/)
[Initial Server Setup with Ubuntu](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-16-04)

**Optional** Pull up another terminal window (**local machine**) and test: `ssh -i ~/.ssh/id_rsa grader@yourIPaddress`

**Logged in as *grader***
Disable Password Authentication
`sudo nano /etc/ssh/sshd_config`
Verify and/or UNCOMMENT: 
PasswordAuthentication *no* 
PubkeyAuthentication *yes*
ChallengeResponseAuthentication *no*

Reload SSH Daemon 
`sudo systemctl reload sshd`
or
`sudo service ssh restart`

**Optional** In a new window, make sure you can login as *grader* (I've been doing this all along; *a stitch in time saves nine*)
`ssh grader@yourIPaddress`

**Configure Uncomplicated Firewall (UFW)**
**Logged in as *grader***
See available applications
`sudo ufw app list` (asks for *grader* password)

Make sure the firewall allows SSH connections (be wary, you CAN make it so you can't re-login next time)
`sudo ufw allow OpenSSH`

Change *port 22* to *port 2200*
`sudo nano /etc/ssh/sshd_config`

Reload
`sudo systemctl reload sshd`

Disable root's SSH login
`sudo nano /etc/ssh/sshd_config`

Reload
`sudo systemctl reload sshd`
`sudo service ssh restart`

Allow incoming connections for SSH
`sudo ufw allow 2200/tcp`
`sudo ufw allow 80/tcp`
`sudo ufw allow 123/udp`

Enable the firewall
`sudo ufw enable`
Prints that command may disrupt existing ssh connections, choose `y`
Prints `Firewall is active and enabled on system startup`

Disable Port 22
`sudo ufw deny 22`

**Can Now Login as**
ssh grader@yourIPaddress -p 2200

**Optional**
Check the current UFW status
`sudo ufw status`
Prints *active* and shows the allowed ports
and
Using a new terminal, verify you can login on port 2200
`ssh grader@yourIPaddress -p 2200`

>**See Also:**
[Understanding public network ports and firewall settings in Amazon Lightsail](https://lightsail.aws.amazon.com/ls/docs/overview/article/understanding-firewall-and-port-mappings-in-amazon-lightsail)

**Change Timezone to UTC**
`sudo dpkg-reconfigure tzdata`
choose *other*; *UTC*
confirm by
`date`

**Install Apache2 and Install Git and Do Some Configuring**
Apache2 install
`sudo apt-get install apache2`

Install mod-wsgi
`sudo apt-get install libapache2-mod-wsgi python-dev`

Restart
`sudo service apache2 start`

Verify Module wsgi is enabled
`sudo a2enmod wsg`

Start Apache2
`sudo service apache2 start`

>**FYI**
Apache's error logs
`/var/log/apache2/error.log`
Apache watches the end of your file
`tail -f /var/log/apache2/error.log`

**Install Git**
`sudo apt-get install git`

Configure Git
`git config --global user.name <username>`
`git config --global user.email <userEmail>`

Create Directory, Clone Proj4 App (Github), Create wsgi file, and Then Some 
`cd /var/www`

Create *catalog* directory
`sudo mkdir catalog`

Change owner for *catalog*
`sudo chown -R grader:grader catalog`

Cd into newly created directory
`cd catalog`

Clone Proj4
`git clone https://github.com/Teraisa/Proj4.git catalog`
Shows the cloning process as it happens
>**FYI**
If you do need to make changes to Proj4, edit on Github (my preference at this point in this particular project), then 
`git pull (catalog file name)`

Create file
`touch catalog.wsgi`

Open file for editing
`sudo nano catalog.wsgi`
add:
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog")

from catalog import app as application
application.secret_key = 'There_is_no_way_out'
```
>**See Also:**
[How To Install Linux, Apache, MySQL, PHP -LAMP- stack on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu-14-04)
[Start, Restart, and Stop Apache web server on Linux](http://www.learn4master.com/programming-language/shell/start-restart-and-stop-apache-on-linux)
[André Sabino and Posts Tagged ‘mod_wsgi’](http://andresabino.com/tag/mod_wsgi/)

**Install Pip, Venv, Flask, Dependencies, Deal with Permissions**
Pip install
`sudo apt-get install python-pip`
 
 Venv install
`sudo pip install virtualenv`
(Will ask if you want to upgrade `pip install --upgrade pip`, I did NOT because a few of my upgrades on past *This Project* disastors broke things up)

Go into catalog
`cd /var/www/catalog`
and create a virtual environment
`sudo virtualenv venv`
Asks for *grader* password
Prints `New python executable in /var/www/venv/bin/python`

Activate venv
`source venv/bin/activate`
Takes you to *vnev*
(looks like: *(venv) grader@ipAddress*)

Change permissions
`sudo chmod -R 777 venv`

**Install Flask and dependencies**
`pip install Flask`
`pip install bleach httplib2 request oauth2client sqlalchemy psycopg2`

Verify virtual host is enabled
`ls -alh /etc/apache2/sites-enabled/`

Edit file
`sudo nano /etc/apache2/sites-available/catalog.conf`
with
```
<VirtualHost *:80>
    ServerName 34.213.124.212
    ServerAlias http://ec2-34-213-124-212.us-west-2.compute.amazonaws.$
    ServerAdmin admin@34.213.124.212
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/ca$
    WSGIProcessGroup catalog
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/catalog/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/catalog/catalog/static
    <Directory /var/www/catalog/catalog/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
>**See Also:**
[Configuring a webserver](https://symfony.com/doc/current/setup/web_server_configuration.html)
[How to Serve Flask Applications](https://www.digitalocean.com/community/tutorials/how-to-serve-flask-applications-with-uwsgi-and-nginx-on-ubuntu-16-04)

Enable new virtual host
`sudo a2ensite catalog`

Reload
`service apache2 reload`

Authenticate 
Choose identity
`2`
Asks for password to complete Authentication
(*2* is *grader*; *1* is *Ubuntu*, I guessed *2* and was **Authenticated!**)

**Install PostgreSQL (no remote connections)and Create a New Database User (call it *catalog*)**

Read and Install Packages
`sudo apt-get install libpq-dev python-dev`
`sudo apt-get install postgresql postgresql-contrib`

Login as *postgres*
`sudo su - postgres`
(looks like: *postgres@ipAddress*)

`psql`
*$* is exchanged for *postgres=#*

Create Role
`CREATE USER catalog WITH PASSWORD 'postgres';`

Alter Role
`ALTER USER catalog CREATEDB;`

Create Database
`CREATE DATABASE catalog WITH OWNER catalog;`

Connect as *catalog*
`\c catalog`
(looks like: catalog=#)

Permissions
`REVOKE ALL ON SCHEMA public FROM public;`
`GRANT ALL ON SCHEMA public TO catalog;`

**Exit to *(venv) grader@ipAddress***
`\q`
and
`exit`

>**See Also:**
[How to Secure PostgreSQL on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)
[How to Install and Use PostgreSQL](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-14-04)

**Change Flask Application in Previous Project**
(For me, three separate files *create_engine* changed)
```
python
engine = create_engine('postgresql://catalog:postgres@localhost/catalog')
```
Change Absolute Path in Your wsgi File
*mine was sys.path.insert(0, '/var/www/catalog/catalog')*

**Restart Apache2**
`sudo service apache2 restart`

**Secure Your Server with Fail2Ban**
Open another terminal, **login as *root***
Ensure system is up to date:
`sudo apt-get update && apt-get upgrade -y`

Install Fail2ban
`sudo apt-get install fail2ban`

**Allow SSH access through UFW**
`sudo ufw allow ssh`

Enable firewall
`ufw enable`

>**See Also:**
[Use Fail2ban to Secure Your Server](https://www.linode.com/docs/security/using-fail2ban-for-security)

##Launch the app!
Go to your browser, type in 
http://34.213.124.212


###Additional Research and Resources

[Ask Ubuntu](https://askubuntu.com/questions/24006/how-do-i-reset-a-lost-administrative-password)

[aviaryan/flask-postgres-apache-server](https://github.com/aviaryan/flask-postgres-apache-server)

[AWS Articles and Tutorials](https://aws.amazon.com/articles/)

[Changing default SSH port in OpenSSH](https://www.knownhost.com/wiki/security/misc/how-can-i-change-my-ssh-port)

[Flask Quickstart](http://flask.pocoo.org/docs/0.12/quickstart/)

[Launch a Linux Virtual Machine](https://aws.amazon.com/getting-started/tutorials/launch-a-virtual-machine/)

[Python virtual environment issues](https://discussions.udacity.com/t/problems-with-the-digital-ocean-tutorial/336376)

[SSH Essentials: Working with SSH Servers, Clients, and Keys](https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys)

##Living Resources
Steve Wooding,Trish, and Greg: true assets to Udacity; their commitment to OUR success is amazing.

Thanks to Chris and Josh, my fellow students for not worrying about "competition" and sharing their mistakes and best resources so I could finish in time and to Renee, who had the [most beautiful README file](https://github.com/rtiinuma/udacity_server_config) for this project, and Serenity for her insight, wisdom, and patience.


-->

--->

---->

----->


###Still Here?<br>
My work is always written in as *plain English* and I can make it, please use freely, but keep it easy to read. Oh, and by the way, have an amazing day!
