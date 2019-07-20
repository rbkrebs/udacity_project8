# Linux Server Configuration

This is the eighth project of Full Stack Web Developer Nanodegree Program from `Udacity`.

Access the link [here](http://34.201.100.48.xip.io/)

## About

The aim of this project is to create and configure a web server to run the Catalog Project, which you can check [here](https://github.com/rbkrebs/udacity_projeto4)

## Configuration

### AWS Account

The first step is to create a Linux server on Amazon Lightsail. To do this, you must have a [AWS account](https://aws.amazon.com).
Then, search for Lightsail service. Click on Create Instance button.

### Installation!

Before install anything, it is important to make the server updated. So, type the next commands:
```
sudo apt-get update
sudo apt-get upgrade
```
Then, install Apache, PostgreSql, WSGI and Git(this probably is already installed)
```
sudo apt-get install apache2
sudo apt-get install postgresql
sudo apt-get install libapache2-mod-wsgi
sudo apt-get install ntp
sudo apt-get install git
```
### Firewall configuration

To configure the firewall according to the project (SSH = port 2200; HTTP = port 80; NTP = port 123), you have to change some specific files.

1.  SSH
Access the sshd_config file typing:
```
sudo nano /etc/ssh/sshd_config
```
and change Port 22 to 2200
```
Port 2200
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::
```
then, restart the service
```
service ssh restart
```
Perhaps it will be necessary adjust the instance on AWS to allow just on 2200 port. To do this click on Networking->Firewall->Add another. Create a curtom application with TCP protocol and Port Range 2200 then delete the default SSH aplication.
2.  HTTP
Access the 000-default.conf file typing:
```
sudo nano /etc/apache2/ports.conf
```
and change Port 80
```
# If you just change the port or add more ports here, you will likely also
# have to change the VirtualHost statement in
# /etc/apache2/sites-enabled/000-default.conf

Listen 80

```
Also add this information `WSGIScriptAlias / /var/www/html/catalog/myapp.wsgi` in the `000-default.conf` file, just before `</VirtualHost>`, and the next lines too.

```
# For most configuration files from conf-available/, which are
        # enabled or disabled at a global level, it is possible to
        # include a line for only one particular virtual host. For example the
        # following line enables the CGI configuration for this host only
        # after it has been globally disabled with "a2disconf".
        #Include conf-available/serve-cgi-bin.conf

        WSGIScriptAlias / /var/www/html/catalog/myapp.wsgi

        <Directory /var/www/catalog/>
                Order allow,deny
                Allow from all
        </Directory>
        Alias /static /var/www/catalog/static
        <Directory /var/www/catalog/static/>
                Order allow,deny
                Allow from all
        </Directory>

</VirtualHost>

```
then, restart the apache
```
sudo apache2ctl restart
```

Last but not least:
```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw allow from any to any port 123 proto udp
sudo ufw deny 22
sudo ufw enable
```

### User Grader

To create a new user, write the following command:
```
sudo adduser grader --disabled-password
```
It will ask for a password and other information, as shown below:
```
Adding user `grader' ...
Adding new group `grader' (1001) ...
Adding new user `grader' (1001) with group `grader' ...
Creating home directory `/home/grader' ...
Copying files from `/etc/skel' ...
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
Changing the user information for grader
Enter the new value, or press ENTER for the default
	Full Name []:
	Room Number []:
	Work Phone []:
	Home Phone []:
	Other []:
Is the information correct? [Y/n] y
```
To give sudo access to grader user, just type:
```
sudo cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/grader
sudo nano /etc/sudoers.d/grader
```
Inside the file, change the name ubuntu to grader and save.
```
grader ALL=(ALL) NOPASSWD:ALL
```
### SSH Keygen

To create an SSH key pair for grader using the ssh-keygen tool:
1. type
```
ssh-keygen
```
2. choose a path to save the files
```
/path/to/folder/.ssh/linuxCourse
```
3. choose a passphrase and then you gonna see a message like this:
```

Your identification has been saved in /path/to/folder/.ssh/linuxCourse
Your public key has been saved in /path/to/folder/.ssh/linuxCourse.pub
```
4. on the server side, login as grader user:
```
sudo su grader
```
5. create a .ssh folder and a authorized_keys file
```
sudo mkdir .ssh
sudo touch .ssh/authorized_keys
```
6. on client side, run
```
cat ~/.ssh/linuxcourse.pub
```
7. copy all the key that appears on terminal and, on server side, paste inside of the .ssh/authorized_keys running the following command:
```
nano .ssh/authorized_keys
```
8. then, protect the folder and file with chmod permissions
```

chmod 700 .ssh
chmod 644 .ssh/authorized_keys
```
9. to avoid the need for using password, on server side:
```
sudo nano /etc/ssh/sshd_config
```
10. look for `PasswordAuthenthication` and change for `no`

Now it is possible to access the serve using ssh just typing:
```
ssh grader@<public-ip> -p 2200 -i  ~/.ssh/authorized_keys
```
## Configure TimeZone

Just run
```
sudo dpkg-reconfigure tzdata
```
and choose you UTC!

## Python

Create a virtual enviroment and install all necessary libs
```
sudo apt-get install python-pip
sudo apt install libpq-dev python-dev
sudo pip install virtualenv
sudo /usr/lib/python2.7 virtualenv env
source /usr/lib/python2.7/env/bin/activate
sudo pip install flask
sudo pip install sqlalchemy
sudo pip install oauth2client
sudo pip install httplib2
sudo pip install requests
sudo pip install psycopg2
```

## The database - PostgreSql

Check if there is any remote connection to postgreSql
```
sudo nano /etc/postgresql/9.3/main/pg_hba.conf
```
You gonna see something like that
```
# Database administrative login by Unix domain socket
local   all             postgres                                peer

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
host    all             all             ::1/128                 md5
# Allow replication connections from localhost, by a user with the
# replication privilege.
local   replication     all                                     peer
host    replication     all             127.0.0.1/32            md5
host    replication     all             ::1/128                 md5
```
Access postgreSql using
```
sudo -u postgres psql
```
then, run the follow commands to create the catalog database and the password and user
```
CREATE DATABASE catalog;
CREATE USER grader WITH PASSWORD 'grader';
```
and give all privileges to user grader
```
GRANT ALL PRIVILEGES ON DATABASE catalog TO grader;
```

## Clone Catalog Aplication

As grader user, move to ` cd /var/www/html`, create a folder `sudo mkdir catalog`, enter and clone [Catalog](https://github.com/rbkrebs/udacity_projeto4) project.
Then create a wsgi file inside `sudo touch /var/www/html/catalog/myapp.wsgi`.
In both app.py and database_setup.py, change `'sqlite:///catalog.db',
    connect_args={'check_same_thread': False}` to `postgresql://grader:grader@localhost/catalog`
Change the name of the file app.py to `__init__.py` using
```
sudo mv app.py __init__.py
```
Inside myapp.wsgi, paste the follow code:
```
activate_this = '/usr/lib/python2.7/env/bin/activate_this.py'
execfile(activate_this, dict(__file__=activate_this))
import sys
import logging

logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, '/var/www/html/catalog/')
from __init__ import app as application
```
Attention!

I had to create a variable to save the fullpath of client_secrets.json file to solve some problems that occurred during execution
```
client_secrets = '/var/www/html/catalog/client_secrets.json'
```

## References

To finish this project, beyond the instructions in Udacity course I also followed this link:

* https://docs.aws.amazon.com/pt_br/AWSEC2/latest/UserGuide/managing-users.html
* https://docs.bitnami.com/aws/faq/get-started/connect-ssh/
* https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
* https://github.com/boisalai/udacity-linux-server-configuration
* https://github.com/VasudhaLalit/Linux-Server-Configuration
* https://github.com/kongling893/Linux-Server-Configuration-UDACITY
* https://www.digitalocean.com/community/tutorials/ufw-essentials-common-firewall-rules-and-commands
* http://cs.sru.edu/~zhou/linux/howto/sshpasswd.html
* https://docs.sqlalchemy.org/en/13/dialects/postgresql.html#module-sqlalchemy.dialects.postgresql.psycopg2
* https://flask.palletsprojects.com/en/1.1.x/deploying/mod_wsgi/
