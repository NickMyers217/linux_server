# Linux Server Configuration Project
### How To Connect
In order to SSH into the server you will need the following information:
- The ip address of the server is: **52.24.42.61**
- The SSH port on the server is set at: **2200**
- Access of the root user is disabled, so you will need to log in as the user named **grader**
- Using the correct private key, you can SSH in as shown below:
``` shell
ssh -i /path/to/key.rsa grader@52.24.42.61 -p 2200
```
### Server URL
You can connect to the server at [http://ec2-52-24-42-61.us-west-2.compute.amazonaws.com/catalog](http://ec2-52-24-42-61.us-west-2.compute.amazonaws.com/catalog)
### Configuration Summary
* SSH'ed into the server as root
* Created a new user named grader, and set up SSH for them
``` shell
adduser grader
su grader
cd ~
mkdir .ssh
chmod 700 .ssh
touch .ssh/authorized_keys
chmod 600 .ssh/authorized_keys
exit
cat ~/.ssh/authorized_keys > /home/grader/.ssh/authorized_keys
```
* Give the grader the permission to sudo, disable loggin in as root, then SSH in as the grader
``` shell
touch /etc/sudoers.d/grader
echo "%grader ALL=(ALL:ALL) ALL" > /etc/sudoers.d/grader
vim /etc/ssh/sshd_config # set root access to no
exit
ssh -i ~/.ssh/udacity_key.rsa grader@52.24.42.61
```
* Update all currently installed packages
``` shell
sudo apt-get update
sudo apt-get upgrade
sudo apt-get autoremove
```
* Change the SSH port from 22 to 2200
``` shell
cd /etc/ssh
sudo vim sshd_config
sudo service ssh restart
exit
ssh -i ~/.ssh/udacity_key.rsa grader@52.24.42.61 -p 2200
```
* Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
``` shell
sudo ufw status
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw allow ntp
sudo ufw enable
sudo ufw status
exit
ssh -i ~/.ssh/udacity_key.rsa grader@52.24.42.61 -p 2200
```
* Configure the local timezone to UTC
``` shell
sudo dpkg-reconfigure tzdata # Chose UTC time
```
* Install and configure Apache to serve a Python mod_wsgi application
``` shell
sudo apt-get install apache2 libapache2-mod-wsgi
sudo vim /etc/apache2/sites-enabled/000-default.conf # Modified this configuration to run an app
sudo vim /var/www/myapp.wsgi
sudo apache2ctl restart
```
* Install and configure PostgreSQL
``` shell
sudo apt-get install postgresql postgresql-server-dev-all
sudo -i -u postgres
createuser --interactive # Created a psql role called catalog with limited permissions
createdb catalog
psql
\password catalog
\q
exit
```
* Create a new user named catalog that has limited permissions to your catalog application database
``` shell
sudo adduser catalog # Created the catalog user on the system
```
* Install git, clone and setup your Catalog App project so that it functions correctly when visiting your server’s IP address in a browser.
``` shell
sudo apt-get install git # install git
sudo apt-get install python-pip python-dev libpq-dev # install pip and some python related software
sudo pip install sqlalchemy flask psycopg2 httplib2 oauth2client # install python libraries
sudo git clone https://github.com/nickmyers217/catalog /var/www/ # clone the project
cd /var/www/catalog
sudo vim app.py database_setup.py database.py # edit the code to work with psql and WSGI
python database_setup.py # Create the database tables with the catalog user, role, and db
sudo touch client_secrets.json fb_client_secrets.json
sudo vim client_secrets.json fb_client_secrets.json # Put my client secrets for the app in here
# Go to the google and facebook console and add my server's url as a javascript origin
sudo vim /etc/apache2/sites-enabled/000-default.conf # Point the config to run the catalog app
sudo apache2ctl restart
```

### Third-Party Resources
Here is a list of third-party resources I used to help set this server up:
* [SSH Keys](https://www.digitalocean.com/community/tutorials/how-to-use-ssh-to-connect-to-a-remote-server-in-ubuntu)
* [UTC Time](http://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt)
* [Setting up PostgreSQL](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-14-04)
* [Installing Python software](http://stackoverflow.com/questions/30127224/best-way-to-install-psycopg2-on-ubuntu-14-04)
* [Running a flask app with WSGI](http://flask.pocoo.org/docs/0.10/deploying/mod_wsgi/)
