# Linux Server Configuration

## Project Overview: :cloud:

> You will take a baseline installation of a Linux server and prepare it to host your web applications. You will secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.

## Setup the Project: :nut_and_bolt: :wrench:

1. [Create EC2 Account.](https://signin.aws.amazon.com/)
2. Create EC2 instance.
3. First check for Updates of the Packages then upgrade them.
    * `$ sudo apt-get update`
    * `$ sudo apt-get upgrade`
4. Install package **finger**. :package:
    * `$ sudo apt-get install finger`
5. Restart the Server.
    * `$ sudo reboot`
6. Create new User **grader** (I created password: **grader**).
    * `$ sudo adduser grader`
7. Create file in given directory with name (grader).
    * `$ sudo touch /etc/sudoers.d/grader`
8. Set **sudo** permissons for the new user (grader).
    * `$ sudo nano /etc/sudoers.d/grader`
        * Type: `$ grader ALL=(ALL) NOPASSWD:ALL`
        * Type 'ctrl-o' to save.
        * Type 'ctrl-x' to exit.
9. Now login to user **grader**.
    * `$ sudo su grader`
10. Setup ssh-key based ssh login.
    * Generate SSH Key on local Machine.
        * `$ ssh-keygen -t rsa -b 4096 -C your_email@example.com`
    * Note the filename and file location used (I used the default that was created at ***.ssh/id_rsa***).
    * When prompted, create passphrase for ssh key (I created passphrase: **`grader`** for this instance).
11. Copy public key from local machine to virtual machine.
    * Make new directory after login to the grader.
        * `$ sudo mkdir .ssh`
    * Create file **authorized_keys** in **.shh** directory.
        * `$ sudo touch .ssh/authorized_keys`
    * Edit **authorized_keys**.
        * `$ sudo nano .ssh/authorized_keys`
    * Copy public key from local machine (**.ssh/id_rsa.pub**) and paste into **.ssh/authorized_keys** file on virtual machine.
12. Set file permissions. :scroll:
    * `$ sudo chmod 700 .ssh`
    * `$ sudo chmod 644 .ssh/authorized_keys`
13. Set Owner/Group to user **grader**. :octocat:
    * `$ sudo chown grader .ssh`
    * `$ sudo chgrp grader .ssh`
    * `$ sudo chown grader .ssh/authorized_keys`
    * `$ sudo chgrp grader .ssh/authorized_keys`
14. Restart SSH service.
    * `$ sudo service ssh restart`
15. Login command from Local Machine. :computer:
    * `$ ssh grader@<public-ip> -i .ssh/id_rsa`
16. Forcing Key Based Authentication. :key:
    * `$ sudo nano /etc/ssh/sshd_config`
    * Change: ***PasswordAuthentication*** to ***no***.
    * Type 'ctrl-o' to save.
    * Type 'ctrl-x' to exit.
17. Configure Firewall. :closed_lock_with_key:
    * Enter the following commands to configure defaults:
        * `$ sudo ufw default deny incoming`
        * `$ sudo ufw default allow outgoing`
    * Enter the following to allow only specified ports:
        * `$ sudo ufw allow ssh`
        * `$ sudo ufw allow 2200/tcp`
        * `$ sudo ufw allow 80/tcp`
        * `$ sudo ufw allow 123/udp`
    * Enable Firewall and make sure port 22 is disabled:
        * `sudo nano /etc/ssh/sshd_config`
            to open editor and change port number from **22** to **2200**, set **PermitRootLogin** to **no**.
        * `$ sudo ufw deny 22`
        * `$ sudo ufw status`
        * `$ sudo ufw enable`
        * `$ sudo service ufw restart`
        * **Note:** If using Amazon Lightsail, Amazon also applies a firewall, need to make sure the same ports are enabled in the Amazon console as well.
18. So we can access the server locally by downloading the SSH key pairs provided inside AWS account and then run:
    * `$ ssh ubuntu@<public-ip> -i .ssh/LightsailDefaultPrivateKey-eu-central-1.pem -p 2200`
19. But now login as user **grader** locally run: :computer:
    * `$ ssh grader@<public-ip> -i .ssh/id_rsa -p 2200`
20. Configure Linux timezone to UTC. :clock4:
    * Open linux time zone configuration:
        * `$ sudo dpkg-reconfigure tzdata`
    * Navigate to and Select ***None of the above***
    * Navigate to and Select ***UTC***
21. Install apache2, wsgi, postgresql, git, python and other dependencies: :arrows_clockwise: :package:
    * `$ sudo apt-get install git`
    * `$ sudo apt-get install python-pip`
    * `$ sudo apt-get install apache2`
    * `$ sudo apt-get install libapache2-mod-wsgi-py3`
    * `$ sudo apt-get install postgresql`
    * `$ sudo pip install --upgrade pip`
    * `$ sudo pip install flask`
    * `$ sudo pip install SQLAlchemy`
    * `$ sudo pip install oauth2client`
    * `$ sudo pip install passlib`
    * `$ sudo pip install requests`
    * `$ sudo pip install psycopg2`
22. Clone [Build-an-item-catalog-application](https://github.com/FixEight/udacity-buid-an-item-catalog-application) repository. :cyclone:
    * Change the directory.
        * `$ cd /var/www`
    * Inside that directory run:
        * `$ sudo git clone https://github.com/FixEight/udacity-buid-an-item-catalog-application.git catalog`
    * Get inside the clone repository.
        * `$ cd /var/www/catalog`
23. Create new **project.wsgi** file inside the downloaded repository which will serve my flask application.
    * `$ sudo touch /var/www/catalog/project.wsgi`
    * Edit the file and add the follwing contents:
        * `$ sudo nano /var/www/catalog/project.wsgi`
        * Content:

            ```text
            import sys
            sys.path.insert(0, "/var/www/catalog")

            from project import app as application
            ```

            **"from project"** phrase is actually the name of my main python file.
24. We need to configure Apache to handle requests using the WSGI module. :pushpin:
    * Creating new configuration file.
        * `$ sudo touch /etc/apache2/sites-available/catalog.conf`
    * Open the editor and add the following content.
        * `$ sudo nano /etc/apache2/sites-available/catalog.conf`
        * Content:

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

25. Now, disable the default Apache site, enable your flask app. :white_check_mark: :negative_squared_cross_mark:
    * Disable the default configuration file:
        * `$ sudo a2dissite 000-default.conf`
    * Enable the **catalog.conf** (Our flask app configuration):
        * `$ sudo a2ensite catalog.conf`
    * To active the new configuration we need to run:
        * `$ sudo service apache2 restart`
        * `$ sudo apache2ctl restart`
26. If app was cloned from (**<https://github.com/FixEight/udacity-buid-an-item-catalog-application>**) then all the following (Line numbers: 27, 28, 29) modification are made already in the repository.

27. Modify **app.secret_key** location Move **app.secret_key** so that it becomes available to the app in the new wsgi configuration.
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

28. Also change the **client_secrets.json** directory in project.py according to the linux server.

    ```text
    CLIENT_ID = json.loads(
        open('client_secrets.json', 'r').read())['web']['client_id']
    ```

    -- to this form:

    ```text
    CLIENT_ID = json.loads(
        open('/var/www/catalog/client_secrets.json', 'r').read())['web']['client_id']
    ```

29. Edit project.py, database_setup.py in clone repository to use postgresql database instead of sqlite.

    ```text
    # engine = create_engine('sqlite:///catalog.db')
    engine = create_engine(
        'postgresql+psycopg2://catalog:catalog@localhost/catalog')
    ```

30. Install and Configure PostgreSQL database. :open_file_folder:
    * Create database user 'catalog'
        * `$ sudo -u postgres psql postgres`
        * `postgres=# CREATE DATABASE catalog;`
        * `postgres=# CREATE USER catalog;`
        * `postgres=# ALTER ROLE catalog with PASSWORD 'catalog';`
        * `postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;`
        * `postgres=# \q`

31. To view last few lines of server side error: :x:

    * `$ sudo tail -n 30 /var/log/apache2/catalog_error.log`
    * `$ sudo tail /var/log/apache2/error.log`

## Run the Project: :rocket:

These are the following addresses to run the Website on browser.

* <http://18.194.121.80.xip.io>
* <http://ec2-18-194-121-80.eu-central-1.compute.amazonaws.com>

## Expected Output in Browser: :camel:

![Buid an Item Catalog Application on Configured Linux Server](images/catalog.jpg)

## Server Details

* IP Address: **18.194.121.80**
* SSH Port Configured and Allowed in Firewall: **2200**
* HTTP Port Configured and Allowed in Firewall: **80**
* NTP Port Configured and Allowed in Firewall: **123**

## User Details

* Sudoer User for Testing:
* Username : **grader**
* Password: **grader**
* Private/public key as grader user is attached with it.

## Third-Party Resources: :link:

* <https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps>

## License

Linux Server Configuration is Copyright :copyright: 2018 Kashif Iqbal. It is free, and may be redistributed under the terms specified in the [LICENSE](https://choosealicense.com/licenses/mit/#) file.
