# Linux-Server-Config
This project enables the user to access the files from any location using a secured source like amazon aws cloud service.

### Prerequisites 

   * Virtual Server Amazon EC2
   * Item Catalog Project, created previously.
   * PostgreSQL

RSA Private Key for grader :
   https://github.com/KSM1598/Linux-Server-Config/blob/master/Linux_server_31_01_2019.pem
Password: ```grader``` 

### 1. Creating New Ubuntu Linux server instance on Amazon EC2

   * Login to ```aws.amazon.com``` and login to default user (ubuntu)
   * Choose ```EC2``` and ```Launch Instance```.
   * Edit Inbound rules and add 3 rules i.e,add port ```ssh 2200```, ```Http port 80```, ```NTP port 123``` and save.
   * Check for instance ```IPv4 public IP - 3.90.89.13``` we can download a ```.pem``` File and connect with following command:
   * ```ssh -i Linux_server_31_01_2019.pem -p 2200 grader@3.91.12.127```
   * 22 is Port by Default, Later we need to Change 2200.
  
### 2: Update and upgrade installed packages

  ``` * sudo apt-get update
   * sudo apt-get upgrade
   * sudo apt-get install unattended-upgrades
   * sudo apt-get dist-upgrade
```
### 3. Change SSH port from 22 to 2200, also remove the comments for all the properties changed.

   * Edit File using: ``` sudo vi /etc/ssh/sshd_config ```
   * Change Port number 22 to 2200
   * PubkeyAuthentication ```yes```
   * PasswordAuthentication ```no```
   * Save and exit using ```esc``` and confirm with ```:wq```
   * Restart: ```sudo service ssh restart```
   * Change inbound rules in Amazon EC2
   * Type : Custom TCP Rule as 2200
   * To check port 2200: Working or Not
   * ```ssh -i linux_31.pem -p 2200 ubuntu@3.90.89.13```

### 4: Configure Firewall (UFW)
   Configure the default firewall allow incoming connections for ```SSH (port 2200)```, ```HTTP (port 80)``` and ```NTP (port 123)```.
   Now Restart ```sudo service ssh restart```
   ```* sudo ufw status
   * sudo ufw default deny incoming
   * sudo ufw default allow outgoing
   * sudo ufw allow 2200/tcp
   * sudo ufw allow www
   * sudo ufw allow 123/udp
   * sudo ufw deny 22
   * sudo ufw enable
     * After Proceed Option Y/n: Y
  ```
   To know the status of port updations: ```sudo ufw status```
        
       ```grader@ip-172-31-42-127:~$ sudo ufw status
          Status: active
          To                         Action      From
          --                         ------      ----
          2200/tcp                   ALLOW       Anywhere
          80/tcp                     ALLOW       Anywhere
          22                         DENY        Anywhere
          123/udp                    ALLOW       Anywhere
          2200/tcp (v6)              ALLOW       Anywhere (v6)
          80/tcp (v6)                ALLOW       Anywhere (v6)
          22 (v6)                    DENY        Anywhere (v6)
          123/udp (v6)               ALLOW       Anywhere (v6)```
  
### 5. Creating grader:
  * Create a user with password: ```sudo adduser grader```
    * password: ```grader```
  * Give the grader premissions ```sudo visudo```.
  * Now below the root add grader as ```grader ALL= (ALL:ALL) ALL```
  * Verify the grader permissions ```su -grader``` and password: ```grader```
  * Configure keypairs for grader Create ```.ssh``` folder by ```mkdir /home/grader/.ssh```
  * Config key pair authentication to grader 
    * ```create .ssh folder```
    * ```mkdir /home/grader/.ssh```
    * ```su grader```
  * Change from ubuntu to grader
    * ```sudo cp /home/ubuntu/.ssh/authorized_keys /home/grader/.ssh/authorized_keys```
  * Change ownership ```chown grader.grader /home/grader/.ssh```
  * Add grader to sudogroup
       ```sudo su
          usermod -aG sudo grader```
  * Change permissions to ```.ssh``` folder
      ```chmod 700 /home/grader/.ssh
         chmod 644 /home/grader/.ssh/authorized_keys
         vi /etc/ssh/sshd_config```
  * Edit authentications as permit root login ```no``` and pubkeyauthentication ```yes```(esc + :wq)
  * Restart the service :
      * ```sudo service ssh restart```
  * Open the server with in the grader:
      * ```ssh -i Linux_server_31_01_2019.pem -p 2200 grader@3.91.12.127```
  * Set the time zone for grader
      * ```sudo dpkg-reconfigure tzdata```

### 6. Deploying the project Steps: 
  * Logged in as grader
  * Install Apache: ```sudo apt-get install apache2```
  * Install the Python 3 mod_wsgi package: ```sudo apt-get install libapache2-mod-wsgi-py3```
  * Enable mod_wsgi using: ```sudo a2enmod wsgi```
  
### 7. Setting up your Google Oauth2
Login to your developer console and select your project and edit OAuth details(Configuration) as following

```  Javascript origin http://ip.xip.io
                  
  redirect URI

  http://ip.xip.io/login

  http://ip.xip.io/gconnect

  http://ip.xip.io/callback
```
  * xip.io is a free DNS which will be the same as using IP address
  * Download the ```client_secrets.json``` file.

### 8. Softwares needed to Install and configure postgresql:
  * ```sudo apt-get install libpq-dev python-dev``` => [Y/n]:Y
  * ```sudo apt-get install postgresql postgresql-contrib``` => [Y/n]:Y
  * Change from grader to postgres: ```sudo su - postgres```
  * Enter "psql"
  * Create role: ```create user catalog with password 'catalog';```
  * Alter role: ```alter user catalog creatdb;```
  * Create DataBase: ```create database catalog with owner catalog;```
  * Connect to the database : ```\c catalog```
  * Revoke public permissions: ```revoke all on schema public from public;```
  * Grant all public permissions:  ```grant all on schema public to catalog;```
  * Quit from psql: ```\q```
  * Exit from postgresql: ```exit```
  * Switch back to grader: ```exit```

Now Logged in as grader and Install git: ```sudo apt-get install git```

### 9. Clone and setup the Item Catalog project from the GitHub repository

```
- After logging in as grader,
- From the /var/www directory, Clone the catalog project:
- sudo git clone `https://github.com/username/catalog.git`
- Change the ownership of the catalog directory to grader using: sudo chown -R grader:grader catalog/.
- Change to the `/var/www/catalog/catalog` directory.
- Rename the mainpage.py file to __init__.py using: mv mainpage.py __init__.py.
- We need to change `sqlite` to `postgresql` create_engine in __init__.py,dbp.py and sample_database.py
  #engine = create_engine("sqlite:///catalog.db") engine = create_engine('postgresql://catalog:catalog@localhost/catalog')
```

### 10. Configure and Enable a New Virtual Host

 ```sudo nano /etc/apache2/sites-available/FlaskApp.conf```

```<VirtualHost *:80>
    ServerName http://3.91.12.127.xip.io
    ServerAlias ec2-3-91-12-127.compute-1.amazonaws.com
    ServerAdmin ubuntu@3.91.12.127
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/catalog/venv3/lib/python3.6/site-packages
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


* Enable the virtual host ```sudo a2ensite catalog```
* Enabling site ```catalog```. 
* To activate the new configuration, you need to run: ```service apache2 reload```

### 11. Set up the Flask application
  * Create ```/var/www/catalog/catalog.wsgi``` file add the following lines:
  ```* import sys
  * import logging
  * logging.basicConfig(stream=sys.stderr)
  * sys.path.insert(0, "/var/www/catalog/")
  * from catalog import app as application
  * application.secret_key = 'supersecretkey'
 ```
Restart Apache: ```sudo service apache2 restart```

From the ```/var/www/catalog/catalog/``` directory

Activate the virtual environment: ```. venv3/bin/activate```

Run: python ```db.py```

Deactivate the virtual environment: ```deactivate```

### 12. Disable the default Apache site
* Disable the default Apache site: ```sudo a2dissite 000-default.conf```
The following prompt will be returned:

```Site 000-default disabled```

* To activate the new configuration, you need to run: ```service apache2 reload```
Reload Apache: ```sudo service apache2 reload```

### Final Step
* Security Updates and package updates Try this commands
```sudo apt-get update
sudo apt-get upgrade
sudo apt-get dist-upgrade
```

### Launch the Web Application

Open your browser to : (http://3.91.12.127.xip.io) Open your browser to : (http://ec2-3-91-12-127.compute-1.amazonaws.com)
