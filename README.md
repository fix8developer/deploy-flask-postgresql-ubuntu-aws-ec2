# Linux Server Configuration

## ⚡ Project Overview

This project provides a **complete guide and configuration scripts** for setting up a  **secure Linux server on AWS EC2** . It walks through every step required to transform a fresh EC2 instance into a production-ready environment capable of hosting web applications.

The configuration covers:

* 🔐 **Security Hardening** → User management, SSH key authentication, firewall (UFW) setup, and disabling root login.
* ⚙️ **Server Setup** → Package updates, timezone configuration, Apache2 + WSGI setup, PostgreSQL installation, and Python dependencies.
* 🌐 **Web Deployment** → Cloning a Flask application, configuring Apache Virtual Hosts for HTTP/HTTPS, enabling SSL, and setting up PostgreSQL for persistence.
* 🛠 **Automation & Monitoring** → Essential commands, log checks, and configurations to keep the server stable and secure.

This repository is ideal for **students, developers, and system administrators** who want a ready-made reference for deploying Flask or other Python web apps on a  **hardened Linux environment with AWS EC2** .

## 🔧 Setup the Project

### Step 1: Create & Initiate the Instance

* [Create EC2 Account.](https://signin.aws.amazon.com/ "AWS EC2")
* Create EC2 instance.

### Step 2: First Check for Updates of the Packages

```bash
sudo apt-get update
sudo apt-get upgrade
```

### Step 3: Install package **finger**. 📦

```bash
sudo apt-get install finger
```

### Step 4: Restart the Server

```bash
sudo reboot
```

### Step 5: Create new User **grader** (I created password: **grader**)

```bash
sudo adduser grader
```

#### Step 5.1: Set **sudo** permissons for the new user (**grader**)

Create file in given directory with name (grader)

```bash
sudo touch /etc/sudoers.d/grader
sudo nano /etc/sudoers.d/grader
```

* Type: `grader ALL=(ALL) NOPASSWD:ALL`
* `ctrl-o` to save.
* `ctrl-x` to exit.

### Step 6: Now login to user **grader**

```bash
sudo su grader
```

### Step 7: Setup `ssh-key` based login

* Generate SSH Key on local Machine.

    ```bash
    ssh-keygen -t rsa -b 4096 -C <your_email@example.com>
    ```

* Note the filename and file location used (I used the default that was created at ***.ssh/id_rsa***).
* When prompted, create passphrase for ssh key (I created passphrase: **`grader`** for this instance).

### Step 8: Copy public key from local machine to virtual machine

* Make new directory after login to the grader.

    ```bash
    sudo mkdir .ssh
    ```

* Create file **authorized_keys** in **.shh** directory.

    ```bash
    sudo touch .ssh/authorized_keys
    ```

* Edit **authorized_keys**.

    ```bash
    sudo nano .ssh/authorized_keys
    ```

* Copy public key from local machine (**.ssh/id_rsa.pub**) and paste into **.ssh/authorized_keys** file on virtual machine.

### Step 9: 📜 Set file permissions

```bash
sudo chmod 700 .ssh
sudo chmod 644 .ssh/authorized_keys
```

### Step 10: Set Owner/Group to user **grader**

```bash
sudo chown grader .ssh
sudo chgrp grader .ssh
sudo chown grader .ssh/authorized_keys
sudo chgrp grader .ssh/authorized_keys
```

### Step 11: Restart SSH service

```bash
sudo service ssh restart
```

### Step 12: 💻 Login command from Local Machine

```bash
ssh grader@<public-ip> -i .ssh/id_rsa
```

### Step 13: 🔑 Forcing Key Based Authentication

```bash
sudo nano /etc/ssh/sshd_config
```

* Change: `PasswordAuthentication` to `no`.
* `ctrl-o` to save.
* `ctrl-x` to exit.

### Step 14: 🔐 Configure Firewall

* Enter the following commands to configure defaults:

    ```bash
    sudo ufw default deny incoming
    sudo ufw default allow outgoing
    ```

* Enter the following to allow only specified ports:

    ```bash
    sudo ufw allow ssh
    sudo ufw allow 2200/tcp
    sudo ufw allow 80/tcp
    sudo ufw allow 123/udp
    sudo ufw allow 443/tcp
    ```

* Enable Firewall and make sure port 22 is disabled:

    ```bash
    sudo nano /etc/ssh/sshd_config
    ```

* Open editor and change port number from `22` to `2200`, set `PermitRootLogin` to `no`.

    ```bash
    sudo ufw deny 22/tcp
    sudo ufw enable
    sudo ufw status
    sudo service ufw restart    
    ```

**Note:** If using Amazon EC2, Amazon also applies a firewall, need to make sure the same ports are enabled in the Amazon console as well.

### Step 15: Access the Server Locally

So we can access the server locally by downloading the SSH key pairs provided inside AWS account and then run

```bash
ssh ubuntu@<public-ip> -i <key.pem> -p 2200
```

#### Step 15.1: 💻 But now login as user **grader** locally run

```bash
ssh grader@<public-ip> -i .ssh/id_rsa -p 2200    
```

### Step 16: 🕓 Configure Linux timezone to UTC

* Open linux time zone configuration:

    ```bash
    sudo dpkg-reconfigure tzdata   
    ```

* Navigate to and Select `None of the above`
* Navigate to and Select `UTC`

### Step 17: 📦 Install packages and other dependencies

```bash
sudo apt-get install git
sudo apt-get install python-pip
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi-py3
sudo apt-get install postgresql
sudo pip install --upgrade pip
sudo pip install flask
sudo pip install SQLAlchemy
sudo pip install oauth2client
sudo pip install passlib
sudo pip install requests
sudo pip install psycopg2    
```

### Step 18: 🌀 Clone [Build-an-item-catalog-application](https://github.com/FixEight/udacity-buid-an-item-catalog-application) repository

* Change the directory.

    ```bash
    cd /var/www
    ```

* Inside that directory run:

    ```bash
    sudo git clone https://github.com/FixEight/udacity-buid-an-item-catalog-application.git catalog    
    ```

* Get inside the clone repository.

    ```bash
    cd /var/www/catalog    
    ```

### Step 19: Create **WSGI** file

Create new **project.wsgi** file inside the downloaded repository which will serve my flask application.

```bash
sudo touch /var/www/catalog/project.wsgi    
sudo nano /var/www/catalog/project.wsgi    
```

Add the follwing content

```python
import sys
sys.path.insert(0, "/var/www/catalog")

from project import app as application
```

**"from project"** phrase is actually the name of my main python file.

### Step 20: 📌 Configure Apache to handle requests using the WSGI module

#### Step 20.1: Creating new configuration file for **HTTP**

```bash
sudo touch /etc/apache2/sites-available/catalog.conf
sudo nano /etc/apache2/sites-available/catalog.conf    
```

* Add the following content:

    ```apache
    <VirtualHost *:80>
        ServerName <public-ip/localhost>
        ServerAdmin kashifiqbal23@gmail.com

        WSGIScriptAlias / /var/www/catalog/project.wsgi

        <Directory /var/www/catalog>
            Require all granted
            WSGIApplicationGroup %{GLOBAL}
            WSGIScriptReloading On
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/catalog_error.log
        CustomLog ${APACHE_LOG_DIR}/catalog_access.log combined
    </VirtualHost>
    ```

#### Step 20.2: Creating new configuration file **HTTPS**

```bash
sudo touch /etc/apache2/sites-available/catalog-ssl.conf
sudo nano /etc/apache2/sites-available/catalog-ssl.conf    
```

* Add the following content

    ```apache
    <IfModule mod_ssl.c>
    <VirtualHost *:443>
        ServerName <public-ip/localhost>
        ServerAdmin fix8developer@gmail.com

        WSGIScriptAlias / /var/www/catalog/project.wsgi

        <Directory /var/www/catalog>
            Require all granted
            WSGIApplicationGroup %{GLOBAL}
            WSGIScriptReloading On
        </Directory>

        SSLEngine on
        SSLCertificateFile /etc/ssl/certs/selfsigned.crt
        SSLCertificateKeyFile /etc/ssl/private/selfsigned.key

        ErrorLog ${APACHE_LOG_DIR}/catalog_ssl_error.log
        CustomLog ${APACHE_LOG_DIR}/catalog_ssl_access.log combined
    </VirtualHost>
    </IfModule>
    ```

#### Step 20.3: **Optional:** Redirect HTTP to HTTPS

```bash
sudo nano /etc/apache2/sites-available/catalog.conf    
```

Content

```apache
<VirtualHost *:80>
    ServerName <public-ip/localhost>
    Redirect permanent / https://<public-ip/localhost>/
</VirtualHost>
```

### Step 21: Disable the default Apache site, enable your flask app

* Disable the default configuration file:

    ```bash
    sudo a2dissite 000-default.conf    
    ```

* Enable the **catalog.conf** (Our flask app configuration):

    ```bash
    sudo a2ensite catalog.conf    
    ```

* Enable the **catalog-ssl.conf** (Our flask app configuration):

    ```bash
    sudo a2enmod ssl
    sudo a2ensite catalog-ssl.conf    
    ```

* To active the new configuration we need to run:

    ```bash
    sudo service apache2 restart
    sudo apache2ctl restart
    sudo systemctl reload apache2
    ```

### Step 22: If app was cloned from (**[https://github.com/FixEight/udacity-buid-an-item-catalog-application](https://github.com/FixEight/udacity-buid-an-item-catalog-application)**) then all the following modification are required

* Modify **app.secret_key** location Move **app.secret_key** so that it becomes available to the app in the new wsgi configuration.

* Edit the project.py file and move the app.secret_key out of ...

    ```python
    if __name__ == '__main__':
        app.secret_key = 'super_secret_key'
        app.run()
    ```

    -- by moving it to the following line:

    ```python
    app = Flask(__name__)
    app.secret_key = 'super_secret_key'
    ```

* Also change the **client_secrets.json** directory in project.py according to the linux server.

    ```python
    CLIENT_ID = json.loads(
        open('client_secrets.json', 'r').read())['web']['client_id']
    ```

    -- to this form:

    ```python
    CLIENT_ID = json.loads(
        open('/var/www/catalog/client_secrets.json', 'r').read())['web']['client_id']
    ```

### Step 23: PostgreSQL instead of SQLite

Edit project.py, database_setup.py in clone repository to use postgresql database instead of sqlite

```python
# engine = create_engine('sqlite:///catalog.db')
engine = create_engine(
    'postgresql+psycopg2://catalog:catalog@localhost/catalog')
```

### Step 24: 📂 Install and Configure PostgreSQL database

* Create database user `catalog`

    ```bash
    sudo -u postgres psql postgres    
    ```

    ```pgsql
    CREATE DATABASE catalog;
    CREATE USER catalog;
    ALTER ROLE catalog with PASSWORD 'catalog';
    GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
    \q
    ```

### Step 25: ❌ To view last few lines of server side error

```bash
sudo tail -n 30 /var/log/apache2/catalog_error.log
sudo tail /var/log/apache2/error.log    
```

## 🚀 Run the Project

These are the following addresses to run the Website on browser.

* [http://EC2 Public IP](http://EC2PublicIP)

## 🐫 Expected Output in Browser

![Buid an Item Catalog Application on Configured Linux Server](images/catalog.jpg)

## 🖥️ Server Details

* IP Address: **EC2 Public IP**
* SSH Port Configured and Allowed in Firewall: **2200**
* HTTP Port Configured and Allowed in Firewall: **80**
* HTTPS Port Configured and Allowed in Firewall: **443**
* NTP Port Configured and Allowed in Firewall: **123**

## 👤 User Details

* Username : **grader**
* Password: **grader**
* Private/public key as grader user is attached with it.

## 📖 Resources

* [https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

## 📜 License

Linux Server Configuration is Copyright ©️ 2025 Kashif Iqbal. It is free, and may be redistributed under the terms specified in the [LICENSE](https://choosealicense.com/licenses/mit/#) file.
