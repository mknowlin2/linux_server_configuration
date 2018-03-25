# linux_server_configuration
Udacity project to configure a Linux server

## Project Overview:
Take a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

## Project Details:
Leveraged Amazon Lightsail to create an Ubuntu 16.04 LTS server. Configure server to only support ssh(2200), http(80), and ntp(123). Deploy the Flask [Catalog](https://github.com/mknowlin2/catalog_app) application to the server. Create a Grader user account with sudo access.

### Server Information:
#### Lightsail Ubuntu Server Setup:
1. Go to Amazon [Lightsail](https://lightsail.aws.amazon.com)
2. Select Linux/Unix for Platform
3. Select OS only and Ubuntu for Blueprint
4. Select $5 Instance Plan
5. Change Default Name (optional)
6. Create Instance
7. Download default key pair
8. Rename key pair file <filename>.pem
  * ```mv <filename>.pem <filename>.pub```
9. Change permissions on key pair file
  * ```chmod 600 <filename>.pub```
10. ssh into server
  * ```ssh ubuntu@<server ip address> -p 22 -i <filename>.pub```

#### Server Configuration:
1. Update baseline server
  * Execute ```sudo apt-get update```
  * Execute ```sudo apt-get upgrade```
  * Execute ```sudo apt-get autoremove```
2. Change local time to UTC
  * Execute ```sudo dpkg-reconfigure tzdata```
  * Select "None of the above"
  * Select "UTC"
  * Confirm UTC date by executing ```date```
    * Example result: Fri Mar 23 21:13:36 UTC 2018
3. Change ssh port to 2200 from 22
  * Execute ```sudo nano /etc/ssh/sshd_config```
  * Change "Port 22" to "Port 2200"
  * Restart ssh ```sudo service ssh restart```
4. Open up port 2200 on Lightsail
  * Go to Amazon [Lightsail](https://lightsail.aws.amazon.com)
  * Select the server instance
  * Select the "Networking" tab
  * Click "Add another" under the Firewall table
  * Input Application (Custom), Protocol (TCP) Port range (2200)
  * Click "Save"
5. Configure UFW
  * Check status ```sudo ufw status```
    * Expected result: "Status: inactive"
  * Set incoming default to deny
    * ```sudo ufw default deny incoming```
  * Set outgoing default to allow
    * ```sudo ufw default allow outgoing```
  * Setup ssh firewall rules
    * ```sudo ufw allow ssh```
    * ```sudo ufw allow 2200/tcp```
  * Setup http firewall rules
    * ```sudo ufw allow http```
  * Setup ntp firewall rules
    * ```sudo ufw allow ntp```
    * ```sudo ufw allow 123/udp```
  * Enable firewall ```sudo ufw enable```
  * Check status ```sudo ufw status```
    * Expected result: "Status: active"
6. Open up port 123 on Lightsail
  * Go to Amazon [Lightsail](https://lightsail.aws.amazon.com)
  * Select the server instance
  * Select the "Networking" tab
  * Click "Add another" under the Firewall table
  * Input Application (Custom), Protocol (UDP) Port range (123)
  * Click "Save"
7. Install finger package
  * Execute ```sudo apt-get install finger```

#### Create Grader user account
1. Execute ```sudo adduser grader```
2. Input Unix password
3. Input Unix password again to confirm
4. Input full name: Udacity Grader
5. Input Room Number: (optional)
6. Input Work Phone: (optional)
7. Input Home Phone: (optional)
8. Input Other: (optional)
9. Confirm input by pressing "Y"
10. Execute ```finger grader```

#### Give grader sudo access
1. Execute ```sudo ls /etc/sudoers.d```
2. Copy the 90-cloud-init-users file
  * ```sudo cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/grader```
3. Execute ```sudo nano /etc/sudoers.d/grader```
4. Change "ubuntu ALL=(ALL) NOPASSWD:ALL" to "grader ALL=(ALL:ALL) ALL"
5. Verify sudo access
  * Execute ```su - grader```
  * Input password
  * Execute ```sudo -l```
    * Expected result: "Matching Defaults entries for grader on ..."

#### Generate ssh key pair for grader
1. Sudo into grader account
  * Execute ```su - grader```
  * Input password
2. Execute ```pwd```
  * Expected result: /home/grader
3. Execute ```mkdir .ssh```
4. Execute ```chmod 700 .ssh```
5. Execute ```touch .ssh/authorized_keys```
6. Execute ```chmod 600 .ssh/authorized_keys```
7. Generate keypair on Lightsail
  * Go to Amazon [Lightsail](https://lightsail.aws.amazon.com)
  * Select the server instance
  * Click link to the "Account page"
  * Click "Create New"
  * Select the region
  * Input name for keypair
  * Download the keypair
8. Obtain public key string for keypair on local machine
  * Execute ```mv <keypair path>/<keypair>.pem ~/<local path>/.ssh/<keypair>.pem```
  * Execute ```chmod 400 <keypair>.pem```
  * Execute ```ssh-keygen -y```
  * Input <local path>/.ssh/<keypair>.pem
  * Copy output
9. Execute ```vi .ssh/authorized_keys```
10. Paste public key string from clipboard and save file
11. Test SSH key from local machine
  * Execute ```ssh grader@<server ip address> -p 2200 -i ~/.ssh/<keypair>.pem```
     * Expected result: A new SSH session with ```grader@<ip_address>```
  * Execute ```id```
     * Expected result: ```uid=1001(grader) gid=1001(grader) groups=1001(grader)```

#### Software Installed:
Installed packages needed for the Catalog application.

1. Install Apache server
  * Execute ```sudo apt-get install apache2```
  * Test installation
    * Go to ```http://<server ip address>:80```
    * The "Apache2 Ubuntu Default Page" successfully retrieved
2. Install Python 3 mod_wsgi
  * Execute ```sudo apt-get install libapache2-mod-wsgi-py3```
3. Install PostgreSQL
  * Execute ```sudo apt-get install postgresql```
4. Install Git (This should already be installed.)
  * Execute ```sudo apt-get install git```
5. Install Python3-Pip
  * Execute ```sudo apt-get install -y python3-pip```
  * Execute ```sudo pip3 install --upgrade pip```
6. Install Flask
  * Execute ```sudo pip3 install Flask```
7. Install Python3 virtual environment
  * Execute ```sudo apt-get install -y python3-venv```

### Application Deployment:
#### Configure Apache to run Flask Python3
1. Enable mod_wsgi
  * Execute ```sudo a2enmod wsgi```
2. Execute ```cd /var/www```
3. Make FlaskApp directory
  * Execute ```sudo mkdir FlaskApp```
4. Make FlaskApp directory in FlaskApp directory
  * Execute ```sudo mkdir FlaskApp/FlaskApp```
5. Make static directory in FlaskApp/FlaskApp directory
  * Execute ```sudo mkdir FlaskApp/FlaskApp/static```
6. Make template directory in FlaskApp/FlaskApp directory
  * Execute ```sudo mkdir FlaskApp/FlaskApp/templates```
7. Create Python3 app
  * Execute ```sudo touch FlaskApp/FlaskApp/__init__.py```
  * Update "\__init__.py" file with the following:
  ```
  from flask import Flask
app = Flask(__name__)
@app.route("/")
def hello():
    return "Please, please work!"
if __name__ == "__main__":
    app.run()
  ```
8. Test Python3 app
  * Execute ```python3 FlaskApp/FlaskApp/__init__.py```
    * Expected result: ```Running on http://127.0.0.1:5000/```
9. Setup virtual environment
  * Execute ```python3 -m venv <environment name>```
  * Execute ```source <environment name>/bin/activate```
  * Execute ```pip install Flask```
10. Configure and Enable site
  * Execute ```sudo touch /etc/apache2/sites-available/FlaskApp.conf```
  * Update "FlaskApp.conf" file with the following:
  ```
  <VirtualHost *:80>
        ServerName <server_ip_address>
        ServerAdmin admin@mywebsite.com
        WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
        <Directory /var/www/FlaskApp/FlaskApp/>
            Order allow,deny
            Allow from all
        </Directory>
        Alias /static /var/www/FlaskApp/FlaskApp/static
        <Directory /var/www/FlaskApp/FlaskApp/static/>
            Order allow,deny
            Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
        WSGIDaemonProcess pylons python-home=/var/www/FlaskApp/yourenvironment
</VirtualHost>
  ```
  * Execute ```a2ensite FlaskApp```
  * Execute ```sudo touch /var/www/FlaskApp/flaskapp.wsgi```
  * Update "flaskapp.wsgi" file with the following:
  ```
  #!/usr/bin/python
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0,"/var/www/FlaskApp/")
  from FlaskApp import app as application
  application.secret_key = 'Add your secret key'
  ```
11. Execute ```sudo service apache2 restart```




1. Go to Google APIs Console — https://console.developers.google.com/apis
2. Create a project
3. Choose Credentials from the menu on the left.
4. Create an OAuth Client ID and configure consent screen.
5. Download OAuth Client Id json file to (cloned repository)/src directory
4. Rename file to "client_secret.json"  

## Resources Referenced
* https://lightsail.aws.amazon.com/ls/docs/all
* https://aws.amazon.com/premiumsupport/knowledge-center/new-user-accounts-linux-instance/
* http://www.islandtechph.com/2017/10/23/how-to-deploy-a-flask-python-3-5-application-on-a-live-ubuntu-16-04-linux-server-running-apache2/
