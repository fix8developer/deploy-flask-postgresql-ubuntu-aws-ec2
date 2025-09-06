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

### 🖥 Step 1: Create, Initiate & Connect to the Instance

---

* [Create EC2 Account.](https://signin.aws.amazon.com/ "AWS EC2")
* Create EC2 instance.
* Connect to the instance through the **AWS EC2 console**, or access the default user `ubuntu` locally by using the SSH key (**.pem**) downloaded from your AWS EC2 account.

  ```bash
  ssh ubuntu@<public-ip> -i <downloaded-aws-ec2-key.pem> -p 22
  ```

### 📦 Step 2: First Update Packages

---

```bash
sudo apt update
sudo apt full-upgrade -y 
sudo apt autoremove -y 
```

⚡ Breakdown:

* `apt update` → refresh package index
* `apt full-upgrade -y` → upgrade all packages with dependency handling
* `apt autoremove -y` → remove unneeded packages

### 🔄 Step 3: Restart the Remote Machine Server (EC2)

---

```bash
sudo reboot
```

* `reboot` → restart the system to apply updates (like kernel upgrades)

### 👤 Step 4: Add New User

---

```bash
sudo adduser <user-name>
```

#### Step 4.1: Set **sudo** Permissons For the New User

1. Create a file in the given directory with the **user name**. You can create and edit the file in one go with `nano`:

    ```bash
    sudo nano /etc/sudoers.d/<user-name>
    ```

2. Type:

    ```bash
    <user-name> ALL=(ALL) NOPASSWD:ALL
    ```

    * `ctrl-o` to save.
    * `ctrl-x` to exit.

### 🔑 Step 5: Login as New User

---

```bash
sudo su <user-name>
```

### 🗝 Step 6: Generate an **SSH Key**

---

* Generate an SSH Key on local Machine

  ```bash
  ssh-keygen -t rsa -b 4096 -C <your_email@example.com>
  ```

> Note the filename and file location used (I used the default that was created at ***.ssh/id_rsa***). When prompted, create a secure passphrase for your SSH key (do not share or document your passphrase).

### 📋 Step 7: Copy **Public Key** to Remote Machine Server

---

```bash
mkdir -p ~/.ssh
nano ~/.ssh/authorized_keys
```

⚡ Breakdown:

* `mkdir -p ~/.ssh` → creates the .ssh directory if it doesn’t already exist (no need for sudo if you’re logged in as the new user).
* `nano ~/.ssh/authorized_keys` → directly creates/edits the file.

> Copy public key from local machine (**.ssh/id_rsa.pub**) and paste into **.ssh/authorized_keys** file on remote machine server.

### 📝 Step 8: Set File Permissions

---

```bash
chmod 700 ~/.ssh 
chmod 600 ~/.ssh/authorized_keys
```

### 👥 Step 9: Set **Owner** and **Group** to New User

---

```bash
sudo chown -R <user-name>:<user-name> .ssh
```

⚡ Breakdown:

* `sudo` → run the command as superuser (root), needed if the current user doesn’t own the directory.
* `chown` → change ownership of files or directories.
* `-R` → recursive option, meaning apply the change to the directory (`.ssh`) and everything inside it.
* `<user-name>:<user-name>` →
  * first `<user-name>` = user who should own the files
  * second `<user-name>` = group that should own the files (often same as username)
* `.ssh` → the target directory (inside the current user’s home directory, typically `~/.ssh`).

### ♻️ Step 10: Restart **SSH** Service

---

```bash
sudo service ssh restart
```

### 🌐 Step 11: Access New User on Local Machine Terminal

---

* Login as **new user** on local machine terminal

  ```bash
  ssh <user-name>@<public-ip> -i <generated-private-key-on-local-machine> -p 22  
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
sudo apt install -y git
sudo apt install -y python3-pip
sudo apt install -y apache2
sudo apt install -y libapache2-mod-wsgi-py3
sudo apt install -y postgresql
sudo apt install -y postgresql-contrib
```

### 📂 Step 16: Clone the [Application](https://github.com/fix8developer/udacity-buid-an-item-catalog-application.git) Repository & Install Python dependencies

---

1. Change the directory.

    ```bash
    cd /var/www
    ```

2. Inside that directory run:

    ```bash
    sudo git clone https://github.com/fix8developer/udacity-buid-an-item-catalog-application.git catalog 
    ```

    * `catalog` is the **"project-name"**.

3. Get inside the clone repository.

    ```bash
    cd /var/www/catalog
    ```

4. Set **Owner** and **Group** to New User

    ```bash
    sudo chown -R <user-name>:<user-name> /var/www/catalog
    ```

5. Install Python dependencies

    ```bash
    # Navigate to the project directory
    cd /var/www/catalog

    # Create and activate a Python virtual environment [catalog-venv]
    python3 -m venv catalog-venv
    source catalog-venv/bin/activate

    # Upgrade pip and install required dependencies
    pip install --upgrade pip
    pip install flask SQLAlchemy oauth2client passlib requests psycopg2-binary

    # Deactivate the virtual environment
    deactivate
    ```

    **OPTIONAL**: Remove only pip-installed packages

    ```bash
    # Only show your user-installed packages (doesn’t show system or venv)
    pip3 list --user

    # Remove user-installed packages
    pip3 freeze --user | xargs pip3 uninstall -y
    ```

### ✏️ Step 17: Modify the Cloned Application

---

If the application was cloned from [(build-an-item-catalog-application)](https://github.com/fix8developer/udacity-buid-an-item-catalog-application.git), the following modifications are required:

```bash
nano /var/www/catalog/project.py
```

1. Edit the project.py file and move the app.secret_key out of ...

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

2. Also update the path to **client_secrets.json** in `project.py` to use the absolute file path (e.g., `/var/www/catalog/client_secrets.json`), since the working directory on the remote machine is different from your local machine.

    ```python
    CLIENT_ID = json.loads(
        open('client_secrets.json', 'r').read())['web']['client_id']
    ```

    -- to this form:

    ```python
    CLIENT_ID = json.loads(
        open('/var/www/catalog/client_secrets.json', 'r').read())['web']['client_id']
    ```

### 🐘 Step 18: Use PostgreSQL Instead of SQLite

---

Edit `project.py` and `database_setup.py` in clone repository to use postgresql database instead of sqlite

```python
# engine = create_engine('sqlite:///catalog.db')
engine = create_engine(
    'postgresql+psycopg2://catalog:catalog@localhost/catalog')
```

### 🗄 Step 19: Configure the Database

---

1. Connect to PostgreSQL as the Default Superuser

    ```bash
    sudo -u postgres psql postgres  
    ```

2. Create database user `"catalog"`

    ```pgsql
    CREATE DATABASE catalog;
    CREATE USER catalog;
    ALTER ROLE catalog with PASSWORD 'catalog';
    GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
    \q
    ```

* Create Database Tables and Populate with Sample Data

  ```bash
  # Navigate to the project directory
  cd /var/www/catalog

  # Activate the Python virtual environment
  source catalog-venv/bin/activate

  # Run script to create database tables
  python3 database_setup.py

  # Populate the database with sample dat
  python3 lotsofitems.py

  # Exit the virtual environment
  deactivate
  ```

### 🌀 Step 20: Create The **WSGI** File (`catalog` Is My Project-Name)

---

1. Create new **project-name.wsgi** file inside the downloaded repository which will serve my flask application.

    ```bash
    sudo nano /var/www/catalog/catalog.wsgi
    ```
  
    You can **create** and **edit** the file in one go with `nano`.

2. Add the following content:

    ```python
    import sys
    import logging

    logging.basicConfig(stream=sys.stderr)

    # Add your project directory to sys.path
    sys.path.insert(0, "/var/www/catalog")

    # Import your Flask app
    from project import app as application
    ```

    The phrase `from project` refers to the name of **main Python file**, which in my case is `project.py`.

### 🏗 Step 21: Free DNS Providers

---

DuckDNS gives you a free subdomain (`something.duckdns.org`).

Steps:

1. Go to `DuckDNS.org`
2. Sign in (GitHub, Google, etc.)
3. Create a subdomain → e.g., `pythonflask.duckdns.org`

### 🏗 Step 22: Configure Apache

---

#### 🌍 Step 22.1: Setting Up a New **HTTP** Configuration File for **TESTING**

* Create and open the Apache configuration file

  ```bash
  sudo nano /etc/apache2/sites-available/catalog.conf
  ```

* Add the following content:

  ```apache
  <VirtualHost *:80>
      ServerName pythonflask.duckdns.org
      ServerAlias www.pythonflask.duckdns.org
      ServerAdmin email@address.com

      WSGIDaemonProcess catalog_http python-home=/var/www/catalog/catalog-venv python-path=/var/www/catalog
      WSGIProcessGroup catalog_http
      WSGIScriptAlias / /var/www/catalog/catalog.wsgi

      <Directory /var/www/catalog>
          Options -Indexes +FollowSymLinks
          AllowOverride None
          Require all granted
          WSGIApplicationGroup %{GLOBAL}
      </Directory>

      ErrorLog ${APACHE_LOG_DIR}/catalog_error.log
      CustomLog ${APACHE_LOG_DIR}/catalog_access.log combined
  </VirtualHost>
  ```

  * `ctrl-o` to save.
  * `ctrl-x` to exit.

  ⚡Breakdown:

  * `Options -Indexes +FollowSymLinks` # Security (no directory listing, allow symlinks)
  * `AllowOverride None` # Ignores .htaccess (faster, safer)
  * `Require all granted` # Lets Apache serve your app
  * `WSGIApplicationGroup %{GLOBAL}` # Keeps Python isolated and stable

* ✅ Check configuration syntax errors

  ```bash
  sudo apache2ctl configtest
  ```

#### 🔐 Step 22.2: Setting Up the existing **HTTP** Configuration File for **PUBLISHING**

* Open the existing Apache configuration file

  ```bash
  sudo nano /etc/apache2/sites-available/catalog.conf
  ```

* Add the following content:

  ```apache
  <VirtualHost *:80>
      ServerName pythonflask.duckdns.org
      ServerAlias www.pythonflask.duckdns.org
      Redirect permanent / https://pythonflask.duckdns.org/

      ServerAdmin email@address.com

      WSGIDaemonProcess catalog_http python-home=/var/www/catalog/catalog-venv python-path=/var/www/catalog
      WSGIProcessGroup catalog_http
      WSGIScriptAlias / /var/www/catalog/catalog.wsgi

      <Directory /var/www/catalog>
          Options -Indexes +FollowSymLinks
          AllowOverride None
          Require all granted
          WSGIApplicationGroup %{GLOBAL}
      </Directory>

      ErrorLog ${APACHE_LOG_DIR}/catalog_error.log
      CustomLog ${APACHE_LOG_DIR}/catalog_access.log combined
  </VirtualHost>
  ```

  * `ctrl-o` to save.
  * `ctrl-x` to exit.

* ✅ Check configuration syntax errors

  ```bash
  sudo apache2ctl configtest
  ```

#### Step 22.3: Setting Up new **HTTPS** Configuration File for **PUBLISHING**

```bash
sudo nano /etc/apache2/sites-available/catalog-le-ssl.conf
```

* Add the following content

  ```apache
  <VirtualHost *:443>
      ServerName pythonflask.duckdns.org
      ServerAlias www.pythonflask.duckdns.org
      ServerAdmin email@address.com

      WSGIDaemonProcess catalog_https python-home=/var/www/catalog/catalog-venv python-path=/var/www/catalog
      WSGIProcessGroup catalog_https
      WSGIScriptAlias / /var/www/catalog/catalog.wsgi

      <Directory /var/www/catalog>
          Options -Indexes +FollowSymLinks
          AllowOverride None
          Require all granted
          WSGIApplicationGroup %{GLOBAL}
      </Directory>

      SSLEngine on
      Include /etc/letsencrypt/options-ssl-apache.conf

      ErrorLog ${APACHE_LOG_DIR}/catalog_ssl_error.log
      CustomLog ${APACHE_LOG_DIR}/catalog_ssl_access.log combined

      SSLCertificateFile /etc/letsencrypt/live/pythonflask.duckdns.org/fullchain.pem
      SSLCertificateKeyFile /etc/letsencrypt/live/pythonflask.duckdns.org/privkey.pem
  </VirtualHost>
  ```

  * `ctrl-o` to save.
  * `ctrl-x` to exit.

* ✅ Check configuration syntax errors

  ```bash
  sudo apache2ctl configtest
  ```

### ✅ Step 23: Enable the Application Site

---

Disable the default Apache site and enable your flask app.

1. Disable the default configuration file:

    ```bash
    sudo a2dissite 000-default.conf  
    ```

2. Enable the **catalog.conf** (Flask app configuration for **HTTP**):

    ```bash
    sudo a2enmod wsgi
    sudo a2ensite catalog.conf
    ```

3. Enable the **catalog-ssl.conf** (Flask app configuration for **HTTPS**):

    ```bash
    sudo a2enmod ssl
    sudo a2ensite catalog-le-ssl.conf 
    ```

4. To active the new configuration we need to run:

    ```bash
    sudo systemctl reload apache2
    ```

5. Double-check Apache's current status

    ```bash
    sudo systemctl status apache2
    ```

### Step 24: Secure Your Application with HTTPS (SSL Certificate)

---

#### Step 1: Install Certbot

* Install Certbot to easily obtain and manage SSL certificates.

  ```bash
  sudo apt update
  sudo apt install certbot python3-certbot-apache -y
  ```

#### Step 2: Obtain SSL Certificate

* Run Certbot with the Apache plugin:

  ```bash
  # sudo certbot --apache -d your_domain.com -d www.your_domain.com
  sudo certbot --apache -d pythonflask.duckdns.org -d www.pythonflask.duckdns.org
  ```

  * Follow prompts:
    * Enter email (for renewal notices)
    * Agree to terms
    * Certbot will auto-configure Apache for HTTPS

#### Step 3: Automatic renewal

* Configure Certbot to automatically renew your SSL certificates. Let’s Encrypt certificates are valid for 90 days. Certbot can auto-renew:

  ```bash
  sudo systemctl status certbot.timer
  ```

* Or test renewal manually:

  ```bash
  sudo certbot renew --dry-run
  ```

### 📜 Step 25: Check Server Logs

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
* [https://pythonflask.duckdns.org](https://pythonflask.duckdns.org)

## 🖼 Expected Output

![Build an Item Catalog Application on Configured Linux Server](images/catalog.jpg)

## 📖 Resources

* [https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

## 📜 License

`deploy-flask-postgresql-ubuntu-aws-ec2` is Copyright ©️ 2025 Kashif Iqbal. It is free, and may be redistributed under the terms specified in the [LICENSE](LICENSE) file.
