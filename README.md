# LINUX SERVER CONFIGURATION PROJECT

## ABOUT

This is guide to take a baseline installation of a linux server and then preparing it to host a Flask Web Application.
And then securing the server from various attacks. We will also configure a database server.
We will use Amazon Web Services - Lightsail to host our web Application.

## BASIC CONFIGURATION INFORMATION

- **Server IP Address:** 13.232.47.168
- **SSH server access port:** 2200
- **SSH login username:** grader
- **Application URL:** http://www.dragfoundation.com, http://13.232.47.168/

## Steps to Set up the SERVER

### 1. Create an instance on Lightsail and login as ubuntu user.

Download default key from instance manage page and save it to desktop. And run following command:

```
   ssh -i <path to private key/*.pem> ubuntu@13.232.47.168
```

### 2. Create a new user Grader and grant the sudo permissions to it.
1. Create User using the following command:
```
  sudo adduser grader
```
2. Adding 'grader' to the 'sudo' group
```
 sudo usermod -aG sudo grader
```
### 3. Adding SSH access to the user `grader`
1. Run the following commands
   ```
    sudo su - grader
    mkdir .ssh
    chmod 700 .ssh
    cd .ssh/
    touch authorized_keys
    chmod 644 authorized_keys

   ```

2. After you have run all the above commands, go back to Lightsail instance and make a new key. Download that key and extract public key by running the following commands
  ```
   ssh-keygen -y
  ```

3. When prompted to enter the file in which the key is, specify the path to your .pem file; for example:  
  ```
    /path_to_key_pair/my-key-pair.pem
  ```

4. The command returns the public key. Copy this key to clipboard.
Again go back to server instance and copy the public key in authorized_keys file using the following command:
 ```
  sudo  cat >> .ssh/authorized_keys
 ```
 Paste the key and press Ctrl+D / Command+D.

### 4. Update all the currently installed packages

   ```
   sudo apt update && apt upgrade
   reboot
   ```
### 5. Change the SSH port from 22 to 2200

1. Open the '/etc/ssh/ssh-config' file
     ```
     sudo nano /etc/ssh/sshd_config
     ```

2. Find the line `Port 22` (would be located around line 13) and change it to `Port 2200`, and save the file.

3. Restart the SSH server to reflect those changes:
     ```
     sudo service ssh restart
     ```
4. Go back to the server instance page and enter Port 2200 in Firewall option under networking
      Now, port should be specified with SSH login command.
      eg: ssh -i <path*.pem> ubuntu@ip -p 2200

### 6. Configure Timezone to Use UTC

To configure the timezone, run the following command:
  ```
  sudo dpkg-reconfigure tzdata
  ```

### 7. Setting up the Firewall

Now we would configure the firewall to allow only incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123):
```
 ufw allow 2200/tcp
 ufw allow 80/tcp
 ufw allow 123/udp
```

To enable the above firewall rules, run:

```
 ufw enable
```
### 8. Disable the Root login

1. Run the following command to switch to root user:
 ```
 sudo su root
 ```

2. After you are logged in, open the file `/etc/ssh/sshd_config` with `nano`:

  ```
   nano /etc/ssh/sshd_config
  ```

3. Find the line `PermitRootLogin yes` and change it to `PermitRootLogin no`.

4. Restart the SSH server:
   ```
   service ssh restart
   ```

### 9. Installing Apache Web Server to serve a Python mod-wsgi Application

1. To install the Apache Web Server, run the following command after logging in as the `grader` user via SSH:

```
$ sudo apt update
$ sudo apt install apache2
```

2. To confirm whether it successfully installed or not, enter the URL `http://<ip address>` in your Web browser:

If the installation has succeeded, you should see the Apache default Webpage at the given url.

3. Install mod_wsgi
```
sudo apt-get install python-setuptools libapache2-mod-wsgi
sudo service apache2 restart
```

### 10. Installing and Configuring PostgreSQL

 Instal PostgreSQL
```
sudo apt install postgresql
```
### 11 Configuring PostgreSQL

1. Log in as the user `postgres` that was automatically created during the installation of PostgreSQL Server:
```
 sudo su - postgres
```
2. Open the `psql` shell:
```
 psql
```
3. This will open the `psql` shell. Now type the following commands one-by-one:

   ```
   sql
   postgres=# CREATE DATABASE catalog;
   postgres=# CREATE USER catalog;
   postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
   postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
   ```
   Then exit from the terminal by running `\q` followed by `exit`.

### 12 Install git, then clone and setup your project

1. Run the following command to install git.
  `sudo apt-get install git`
2. Move into cd /var/www/
3. Create a directory called `FlaskApp` and change the working directory to it:

   ```
   sudo mkdir FlaskApps
   cd FlaskApps/
   ```
4. Clone this repository directory `FlaskApp`:

   ```
   sudo git clone https://github.com/VartikaPahuja/Udacity-catalog-project.git CatalogApp
   ```
4. Move inside the newly created directory:
   ```
   cd FlaskApps/
   ```

### 13 Install pip
    Run the following command:

    ```
    sudo apt-get install python-pip
    ```

### 14. Install required packages:
    Run the following command:

    ```
    sudo pip install --upgrade Flask SQLAlchemy httplib2 oauth2client requests psycopg2 psycopg2-binary
    ```

### 15. Setting Up Virtual Hosts

1. Run the following command in terminal to set up a file called      `CatalogApp.conf` to configure the virtual hosts:

  ```
   sudo nano /etc/apache2/sites-available/CatalogApp.conf
  ```

2. Add the following lines to it:

```
    <VirtualHost *:80>
    ServerName 13.232.47.168
    ServerAlias www.dragfoundation.com
    ServerAdmin vartikapahuja19@gmail.com
    WSGIScriptAlias / /var/www/FlaskApps/FlaskApps.wsgi
    <Directory /var/www/FlaskApps/CatalogApp/>
        Order allow,deny
        Allow from all
    </Directory>
    <Directory /var/www/FlaskApps/CatalogApp/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>
```

3. Enable the virtual host:

   ```
   sudo a2ensite CatalogApp
   ```

4. Restart Apache server:

   ```
   sudo service apache2 restart
   sudo /etc/init.d/apache2 reload
   ```

5. Creating the .wsgi File

   Apache uses the `.wsgi` file to serve the Flask app. Move to the `/var/www/FlaskApps/` directory and create a file named `FlaskApps.wsgi` with following commands:

   ```
   cd /var/www/FlaskApps/
   sudo nano FlaskApps.wsgi
   ```

   Add the following lines to the `FlaskApp.wsgi` file:

   ```
   python
   #!/var/www/FlaskApps
   import sys
   import logging
   logging.basicConfig(stream=sys.stderr)
   sys.path.insert(0, "/var/www/FlaskApps/CatalogApp/")

   from home import app as application
   application.secret_key = 'Add your secret key'
   ```

6. Restart Apache server:

   ```
   sudo service apache2 restart
   sudo /etc/init.d/apache2 reload
   ```

   Now you should be able to run the application at http://www.dragfoundation.com, http://13.232.47.168/

## DEBUGGING

If you are getting an _Internal Server Error_ or any other error(s), make sure to check out Apache's error log for debugging:
```
$ sudo cat /var/log/apache2/error.log
```
## REFERENCES

[1]<https://github.com/shreya28171/Udacity_Linux_Server_Configuration>
[2]<https://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/ec2-key-pairs.html#how-to-generate-your-own-key-and-import-it-to-aws>
