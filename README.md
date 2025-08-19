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

1. [Create EC2 Account.](https://signin.aws.amazon.com/ "AWS EC2")
2. Create EC2 instance.
3. First check for Updates of the Packages then upgrade them.

   ```bash
   sudo apt-get update
   sudo apt-get upgrade
   ```

4. Install package **finger**. 📦

   ```bash
   sudo apt-get install finger
   ```

5. Restart the Server.

   ```bash
   sudo reboot
   ```

6. Create new User **grader** (I created password: **grader**).

   ```bash
   sudo adduser grader
   ```

7. Create file in given directory with name (grader).

   ```bash
   sudo touch /etc/sudoers.d/grader
   ```

8. Set **sudo** permissons for the new user (grader).

   ```bash
   sudo nano /etc/sudoers.d/grader
   ```

   * Type: `grader ALL=(ALL) NOPASSWD:ALL`
   * `ctrl-o` to save.
   * `ctrl-x` to exit.
9. Now login to user **grader**.

   ```bash
   sudo su grader
   ```

10. Setup `ssh-key` based login.

    * Generate SSH Key on local Machine.

        ```bash
        ssh-keygen -t rsa -b 4096 -C <your_email@example.com>
        ```

    * Note the filename and file location used (I used the default that was created at ***.ssh/id_rsa***).
    * When prompted, create passphrase for ssh key (I created passphrase: **`grader`** for this instance).
11. Copy public key from local machine to virtual machine.

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
12. 📜 Set file permissions

    ```bash
    sudo chmod 700 .ssh
    sudo chmod 644 .ssh/authorized_keys
    ```

13. Set Owner/Group to user **grader**

    ```bash
    sudo chown grader .ssh
    sudo chgrp grader .ssh
    sudo chown grader .ssh/authorized_keys
    sudo chgrp grader .ssh/authorized_keys
    ```

14. Restart SSH service.

    ```bash
    sudo service ssh restart
    ```

15. 💻 Login command from Local Machine

    ```bash
    ssh grader@<public-ip> -i .ssh/id_rsa
    ```

16. 🔑 Forcing Key Based Authentication

    ```bash
    sudo nano /etc/ssh/sshd_config
    ```

    * Change: `PasswordAuthentication` to `no`.
    * `ctrl-o` to save.
    * `ctrl-x` to exit.
17. 🔐 Configure Firewall
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
18. So we can access the server locally by downloading the SSH key pairs provided inside AWS account and then run:

    ```bash
    ssh ubuntu@<public-ip> -i <key.pem> -p 2200
    ```

19. 💻 But now login as user **grader** locally run

    ```bash
    ssh grader@<public-ip> -i .ssh/id_rsa -p 2200    
    ```

20. Configure Linux timezone to UTC. 🕓

    * Open linux time zone configuration:

        ```bash
        sudo dpkg-reconfigure tzdata   
        ```

    * Navigate to and Select `None of the above`
    * Navigate to and Select `UTC`
21. 📦 Install packages and other dependencies

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

22. 🌀 Clone [Build-an-item-catalog-application](https://github.com/FixEight/udacity-buid-an-item-catalog-application) repository

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

23. Create new **project.wsgi** file inside the downloaded repository which will serve my flask application.

    ```bash
    sudo touch /var/www/catalog/project.wsgi    
    ```

    Edit the file

    ```bash
    sudo nano /var/www/catalog/project.wsgi    
    ```

    Add the follwing content

    ```text
    import sys
    sys.path.insert(0, "/var/www/catalog")

    from project import app as application
    ```

    **"from project"** phrase is actually the name of my main python file.
24. We need to configure Apache to handle requests using the WSGI module. 📌

    * Creating new configuration file for **HTTP**.

        ```bash
        sudo touch /etc/apache2/sites-available/catalog.conf
        ```

        Open the editor

        ```bash
        sudo nano /etc/apache2/sites-available/catalog.conf    
        ```

        Add the following content:

        ```text
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

    * Creating new configuration file **HTTPS**.

        ```bash
        sudo touch /etc/apache2/sites-available/catalog-ssl.conf
        ```

        Open the editor

        ```bash
        sudo nano /etc/apache2/sites-available/catalog-ssl.conf    
        ```

        Add the following content

        ```text
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

    * **Optional:** Redirect HTTP to HTTPS.

        ```bash
        sudo nano /etc/apache2/sites-available/catalog.conf    
        ```

        Content

        ```text
        <VirtualHost *:80>
            ServerName <public-ip/localhost>
            Redirect permanent / https://<public-ip/localhost>/
        </VirtualHost>
        ```

25. ✅ ❎ Now, disable the default Apache site, enable your flask app

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

26. If app was cloned from (**[https://github.com/FixEight/udacity-buid-an-item-catalog-application](https://github.com/FixEight/udacity-buid-an-item-catalog-application)**) then all the following modification are required.
    * Modify **app.secret_key** location Move **app.secret_key** so that it becomes available to the app in the new wsgi configuration.

        * Edit the project.py file and move the app.secret_key out of ...

            ```text
            if __name__ == '__main__':
                app.secret_key = 'super_secret_key'
                app.run()
            ```

            -- by moving it to the following line:

            ```text
            app = Flask(__name__)
            app.secret_key = 'super_secret_key'
            ```

    * Also change the **client_secrets.json** directory in project.py according to the linux server.

        ```text
        CLIENT_ID = json.loads(
            open('client_secrets.json', 'r').read())['web']['client_id']
        ```

        -- to this form:

        ```text
        CLIENT_ID = json.loads(
            open('/var/www/catalog/client_secrets.json', 'r').read())['web']['client_id']
        ```

27. Edit project.py, database_setup.py in clone repository to use postgresql database instead of sqlite.

    ```text
    # engine = create_engine('sqlite:///catalog.db')
    engine = create_engine(
        'postgresql+psycopg2://catalog:catalog@localhost/catalog')
    ```

28. 📂 Install and Configure PostgreSQL database

    * Create database user `catalog`

        ```bash
        sudo -u postgres psql postgres    
        ```

        ```sql
        CREATE DATABASE catalog;
        CREATE USER catalog;
        ALTER ROLE catalog with PASSWORD 'catalog';
        GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
        \q
        ```

29. ❌ To view last few lines of server side error

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
