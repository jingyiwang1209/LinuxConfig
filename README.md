#Linux Configuration

### The Linux configuration project is about migrating WE-LOVE-CODING-APP project from local virtual environment to the Amazon remote virtual server.

### The URL is: http://ec2-52-37-81-230.us-west-2.compute.amazonaws.com/
### The IP address is: 52.37.81.230
### SSH port range: 22

Procedures:
1. Creating an Amazon Lightsail instance and set up the firewall.
Update all the packages:
sudo apt-get upgrade
Set and change rules:
sudo ufw status
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw allow ntp
sudo ufw enable
sudo ufw status
Also add an custom rule on GUI: custom TCP 2200

2. Download the LightsailDefaultPrivateKey file from Lightsail site into a local environment.

3. Cd the directory where the LightsailDefaultPrivateKey file is located and ssh to the remote ubuntu system:
    ssh ubuntu@52.37.81.230 -i LightsailDefaultPrivateKey-us-west-2.pem

4. Add a new user called grader and give grader permission to sudo:
    sudo adduser grader
    sudo adduser grader sudo

5. Create a SSH key pair for grader using the ssh-keygen tool:
    on the local side:
     mkdir linux
     cd linux
     ssh-keygen
    on the server side:
     su - grader
     mkdir .ssh
     touch .ssh/authorized_keys
     nano .ssh/authorized_keys
    copy the public key from linux.pub(cat  /Users/username/.ssh/linux.pub) into the above file.
    now it is supposed to login using ssh grader:
     ssh grader@52.37.81.230 -p 2200 -i /Users/username/.ssh/linux



6. Configure the local timezone to UTC:
    go to server side:
     sudo dpkg-reconfigure tzdata
    when the shell GUI comes up, select None of the above and then UTC.

7. Install Apache and mod_wsgi to serve a Python mod_wsgi application:
    sudo apt-get install apache2
    sudo apt-get install libapache2-mod-wsgi

8. Install postgresql:
    sudo apt-get install postgresql

9. Make the structure of WE-LOVE-CODING-APP:
    cd /var/www
    sudo mkdir catalog
    cd catalog
    git clone GitHub repo address(a special one without vagrant directory) catalog

10. Create and edit wsgi file:
     sudo touch catalog.wsgi
     sudo nano catalog.wsgi
     inside the file:
      ** #!/usr/bin/python
        import sys
        import logging
        logging.basicConfig(stream=sys.stderr)
        sys.path.insert(0,"/var/www/catalog/")
        from catalog import app as application
        application.secret_key = ‘supersecretkey'**

11. Update conf file:
     sudo touch /etc/apache2/sites-available/catalog.conf
     sudo nano /etc/apache2/sites-available/catalog.conf
     inside the file:
      **<VirtualHost *:80>
        ServerName example.com
        ServerAdmin useremail
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
      </VirtualHost>**

12. Enable catalog.conf and disable 000-default.conf:
     a2ensite catalog
     a2dissite 000-default

13. Update the engine in all the app files:
     engine = create_engine('postgresql://catalog:catalog@localhost/catalog')


14. Create a user called catalog and grant the ownership to the user to CRUD tables in. the database:
     sudo su - postgres
     createuser catalog
     psql
     postgres=# \password catalog
     Enter new password: catalog
     Enter it again: catalog
     alter database catalog OWNER TO catalog;

15. Install all the libraries and packages needed for the app:
     cd /var/www/catalog/catalog
     sudo -H pip install Flask passlib flask-httpauth Flask-WTF sqlalchemy









