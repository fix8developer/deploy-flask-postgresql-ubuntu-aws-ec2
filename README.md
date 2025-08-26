# 🐧 How to Deploy a Flask (Python) Application with PostgreSQL on an Ubuntu VPS (AWS-EC2)

## ⚡ Overview

This project provides a **complete guide and configuration scripts** for setting up a  **secure Linux server on AWS EC2** . It walks through every step required to transform a fresh EC2 instance into a production-ready environment capable of hosting web applications.

The configuration covers:

* 🔐 **Security Hardening** → User management, SSH key authentication, firewall (UFW) setup, and disabling root login.
* ⚙️ **Server Setup** → Package updates, timezone configuration, Apache2 + WSGI setup, PostgreSQL installation, and Python dependencies.
* 🌐 **Web Deployment** → Cloning a Flask application, configuring Apache Virtual Hosts for HTTP/HTTPS, enabling SSL, and setting up PostgreSQL for persistence.
* 🛠 **Automation & Monitoring** → Essential commands, log checks, and configurations to keep the server stable and secure.

This repository is ideal for **students, developers, and system administrators** who want a ready-made reference for deploying Flask or other Python web apps on a  **hardened Linux environment with AWS EC2** .

## 🛠 Project Setup

### 🖥 Step 1: Create & Initiate the Instance

---

* [Create EC2 Account.](https://signin.aws.amazon.com/ "AWS EC2")
* Create EC2 instance.

### 📦 Step 2: First Update Packages

---

```bash
sudo apt-get update
sudo apt-get upgrade
```

### 🔄 Step 3: Restart the Server

---

```bash
sudo reboot
```

### 👤 Step 4: Add User **"grader"**

---

```bash
sudo adduser grader
```

#### Step 4.1: Set **sudo** Permissons For the New User **"grader"**

* Create file in given directory with name **"grader"**

    ```bash
    sudo touch /etc/sudoers.d/grader
    ```

    ```bash
    sudo nano /etc/sudoers.d/grader
    ```

* Type:
  
  ```bash
  grader ALL=(ALL) NOPASSWD:ALL
  ```

* `ctrl-o` to save.
* `ctrl-x` to exit.

### 🔑 Step 5: Login as User **"grader"**

---

```bash
sudo su grader
```

### 🗝 Step 6: Generate an **SSH Key**

---

* Generate an SSH Key on local Machine.

    ```bash
    ssh-keygen -t rsa -b 4096 -C <your_email@example.com>
    ```

    > Note the filename and file location used (I used the default that was created at ***.ssh/id_rsa***). When prompted, create a secure passphrase for your SSH key (do not share or document your passphrase).

### 📋 Step 7: Copy **Public Key**

---

* Make new directory after login to the "grader".

    ```bash
    sudo mkdir .ssh
    ```

* Create file **authorized_keys** in **.ssh** directory.

    ```bash
    sudo touch .ssh/authorized_keys
    ```

* Edit **authorized_keys**

    ```bash
    sudo nano .ssh/authorized_keys
    ```

> Copy public key from local machine (**.ssh/id_rsa.pub**) and paste into **.ssh/authorized_keys** file on remote machine.

### 📝 Step 8: Set File Permissions

---

```bash
sudo chmod 700 .ssh
sudo chmod 644 .ssh/authorized_keys
```

### 👥 Step 9: Set **Owner** and **Group** to User "grader"

---

```bash
sudo chown grader .ssh
sudo chgrp grader .ssh
sudo chown grader .ssh/authorized_keys
sudo chgrp grader .ssh/authorized_keys
```

### ♻️ Step 10: Restart **SSH** Service

---

```bash
sudo service ssh restart
```

### 🌐 Step 11: Access the Server Locally

---

* So we can access the server locally by downloading the SSH key pairs provided inside AWS account

    ```bash
    ssh ubuntu@<public-ip> -i <key.pem> -p 2200
    ```

* But now login as user **"grader"** locally

    ```bash
    ssh grader@<public-ip> -i .ssh/id_rsa -p 2200    
    ```

### 🚫 Step 12: Enforce Key-Based Authentication

---

```bash
sudo nano /etc/ssh/sshd_config
```

* Change: `PasswordAuthentication` to `no`.
* `ctrl-o` to save.
* `ctrl-x` to exit.

### 🔒 Step 13: Configure Firewall

---

* Enter the following commands to configure defaults:

    ```bash
    sudo ufw default deny incoming
    sudo ufw default allow outgoing
    ```

* Enter the following to allow/deny only specified ports:

    ```bash
    sudo ufw allow ssh
    sudo ufw allow 2200/tcp
    sudo ufw allow 80/tcp
    sudo ufw allow 123/udp
    sudo ufw allow 443/tcp
    sudo ufw deny 22/tcp
    ```

* Before enable Firewall make sure port `22` is disabled:

    ```bash
    sudo nano /etc/ssh/sshd_config    
    ```

* Open editor and change port number from `22` to `2200`, set `PermitRootLogin` to `no`.

    ```bash
    sudo ufw enable
    sudo service ufw restart
    sudo ufw status
    ```

> **Note:** If using Amazon EC2, you must configure both the AWS Security Groups (in the AWS console) and the server's UFW firewall to allow the same ports (e.g., 2200, 80, 443, 123) for proper connectivity.

### 🕓 Step 14: Configure Linux Timezone to UTC

---

* Open linux time zone configuration:

    ```bash
    sudo dpkg-reconfigure tzdata   
    ```

* Navigate and Select `None of the above`
* Navigate and Select `UTC`

### ⚙️ Step 15: Install Packages and Dependencies

---

```bash
sudo apt-get install git
sudo apt-get install python3-pip
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi-py3
sudo apt-get install postgresql
```

```python
# (Recommended) Create and activate a Python virtual environment for your project:
python3 -m venv <venv-name>
source <venv-name>/bin/activate

# Upgrade pip and install Python dependencies inside the virtual environment:
pip install --upgrade pip
pip install flask
pip install SQLAlchemy
pip install oauth2client
pip install passlib
pip install requests
pip install psycopg2
```

> When you are finished working on your project, you can deactivate the virtual environment:

```python
deactivate
```

### 📂 Step 16: Clone the [Application](https://github.com/fix8developer/udacity-buid-an-item-catalog-application.git) Repository

---

* Change the directory.

    ```bash
    cd /var/www
    ```

* Inside that directory run:

    ```bash
    sudo git clone https://github.com/fix8developer/udacity-buid-an-item-catalog-application.git catalog    
    ```

* Get inside the clone repository.

    ```bash
    cd /var/www/catalog    
    ```

### 🌀 Step 17: Create the **WSGI** File

---

* Create new **project.wsgi** file inside the downloaded repository which will serve my flask application.

    ```bash
    sudo touch /var/www/catalog/project.wsgi
    ```

    ```bash
    sudo nano /var/www/catalog/project.wsgi   
    ```

* Add the following content

    ```python
    import sys
    sys.path.insert(0, "/var/www/catalog")

    from project import app as application
    ```

    > **"from project"** phrase is actually the name of my main python file.

### 🏗 Step 18: Configure Apache

---

#### 🌍 Step 18.1: Creating New **HTTP** Configuration File

```bash
sudo touch /etc/apache2/sites-available/catalog.conf
```

```bash
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

#### 🔐 Step 18.2: Creating New **HTTPS** Configuration File

```bash
sudo touch /etc/apache2/sites-available/catalog-ssl.conf
```

```bash
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

#### ↪️ Step 18.3: Redirect HTTP to HTTPS **(Optional)**

```bash
sudo nano /etc/apache2/sites-available/catalog.conf    
```

* Content

    ```apache
    <VirtualHost *:80>
        ServerName <public-ip/localhost>
        Redirect permanent / https://<public-ip/localhost>/
    </VirtualHost>
    ```

### 🔐 Step 19: Create Your Own Self-Signed SSL Certificate

---

#### Step 19.1: Install `OpenSSL`

```bash
sudo apt-get update
sudo apt-get install -y openssl
```

#### Step 19.2: Create a Private Key

```bash
openssl genrsa -out selfsigned.key 2048
```

#### Step 19.3: Generate a Certificate Signing Request (CSR)

```bash
openssl req -new -key selfsigned.key -out selfsigned.csr
```

> Common Name should be your domain (or public IP if no domain).

#### Step 19.4: Generate a Self-Signed SSL Certificate

```bash
openssl x509 -req -days 365 -in selfsigned.csr -signkey selfsigned.key -out selfsigned.crt
```

> This creates `selfsigned.crt` (certificate) valid for 1 year.

#### Step 19.5: Copy SSL Files to Secure Location

```bash
sudo cp selfsigned.crt /etc/ssl/certs/
sudo cp selfsigned.key /etc/ssl/private/
```

Set proper permissions:

```bash
sudo chmod 600 /etc/ssl/private/selfsigned.key
```

### ✅ Step 20: Enable the Application Site

---

Disable the default Apache site and enable your flask app.

* Disable the default configuration file:

    ```bash
    sudo a2dissite 000-default.conf    
    ```

* Enable the **catalog.conf** (Flask app configuration for **HTTP**):

    ```bash
    sudo a2ensite catalog.conf
    sudo a2enmod wsgi
    ```

* Enable the **catalog-ssl.conf** (Flask app configuration for **HTTPS**):

    ```bash
    sudo a2enmod ssl
    sudo a2ensite catalog-ssl.conf    
    ```

* To active the new configuration we need to run:

    ```bash
    sudo systemctl reload apache2
    ```

### ✏️ Step 21: Modify the Cloned Application

---

If the application was cloned from [(build-an-item-catalog-application)](https://github.com/fix8developer/udacity-buid-an-item-catalog-application.git), the following modifications are required:

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

* Also update the path to **client_secrets.json** in `project.py` to use the absolute file path (e.g., `/var/www/catalog/client_secrets.json`), since the working directory on the remote machine is different from your local machine.

    ```python
    CLIENT_ID = json.loads(
        open('client_secrets.json', 'r').read())['web']['client_id']
    ```

    -- to this form:

    ```python
    CLIENT_ID = json.loads(
        open('/var/www/catalog/client_secrets.json', 'r').read())['web']['client_id']
    ```

### 🐘 Step 22: Use PostgreSQL Instead of SQLite

---

Edit project.py, database_setup.py in clone repository to use postgresql database instead of sqlite

```python
# engine = create_engine('sqlite:///catalog.db')
engine = create_engine(
    'postgresql+psycopg2://catalog:catalog@localhost/catalog')
```

### 🗄 Step 23: Configure the Database

---

Create database user `"catalog"`

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

### 📜 Step 24: Check Server Logs

---

To view server side error for **HTTP**:

```bash
sudo tail -n 30 /var/log/apache2/catalog_error.log
```

To view server side error for **HTTPS**:

```bash
sudo tail -n 30 /var/log/apache2/catalog_ssl_error.log
```

To view server side error for **ALL**:

```bash
sudo tail /var/log/apache2/error.log    
```

## 🚀 Run the Project

* [http://EC2 Public IP](http://EC2PublicIP)
* [https://EC2 Public IP](https://EC2PublicIP)

## 🖼 Expected Output

![Build an Item Catalog Application on Configured Linux Server](images/catalog.jpg)

## 📖 Resources

* [https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

## 📜 License

Linux Server Configuration is Copyright ©️ 2025 Kashif Iqbal. It is free, and may be redistributed under the terms specified in the [LICENSE](https://choosealicense.com/licenses/mit/#) file.
