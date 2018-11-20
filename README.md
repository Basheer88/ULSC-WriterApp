# Udacity Linux Server Configuration Project - WriterApp

This tutorial will guide you through the steps to take a baseline installation of a Linux server and prepare it to host your own Web applications (Flask Application) and secure it from a number of attacks, install and configure a database server.

In this project, I had choose an Ubuntu 18.04 instance on AWB. Below are All technical details of the server as well as the steps that i had done to set it up.

### Server Info

- **Server IP Address:** 18.130.232.94
- **SSH server access port:** 2200
- **SSH login username:** grader
- **Application URL:** http://18.130.232.94.xip.io/

## The Steps

### 1.Start a new Ubuntu Linux server

Register and login into Amazon Light Sail [Here](https://lightsail.aws.amazon.com/). Choose your instance plan and create an instance ( I choose OS Only Ubuntu 18.04 LTS ).

### 2. SSH ( Generating Key Pair )

On your local machine, you need to generate a key pair public and private key. This key pair will be used to authenticate  and securely logging in to the server. The public Key will be stored on your server and The private key will be kept with you in your local machine.

To generate a key pair, run the following command:

   ```console
   $ ssh-keygen
   ```

When it asks to enter a passphrase, you may either leave it empty or enter some passphrase. A passphrase adds an additional layer of security to prevent unauthorized users from logging in.
Log into your server using the following command:
```console
ssh ubuntu@18.130.232.94 -i ~/.ssh/privatekey
```

### 3. Update all currently installed packages

Run the following command to update the Linux virtual server:

```
 # apt update && apt upgrade
```

you might need to reboot the server if its kernel update by running the following command:

```
# reboot
```

### 4. Changing SSH Port to 2200

1. Open the `/etc/ssh/sshd_config` file with your prefered text editor:

   ```
   # nano /etc/ssh/sshd_config
   ```

2. Find `#Port 22` and change it to `Port 2200`

3. Restart the SSH server:
   ```
   # service ssh restart
   ```
4. add port 2200 on Lightsail Firewall.

Now You should be able to log in to the server as `root` on port 2200. 
```
ssh ubuntu@18.130.232.94 -p 2200 -i ~/.ssh/privatekey
```

### 5. Configure The Uncomplicated Firewall UFW

configure the ufw to allow only incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123):

```
# ufw defualt deny incoming
# ufw defualt allow outgoing
# ufw allow 2200/tcp
# ufw allow 80/tcp
# ufw allow 123/udp
```

To activate firewall, run:

```
# ufw enable
```

To check ufw rules, run:

```
# ufw status
```

You should see something like this:

```
To                         Action      From
===================================================
2200/tcp                   ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
123/udp                    ALLOW       Anywhere
2200/tcp (v6)              ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
123/udp (v6)               ALLOW       Anywhere (v6)
```

### 6. Create new user `grader`

use the following command:
```
  # adduser grader
```
Fill the info to complete it. For the UNIX password I have entered `graderu`.

### 7. Give `grader` the permission to `sudo`
Run the following command to add the user `grader` to the `sudo` group.

```
  # usermod -aG sudo grader
```

### 8. Create an SSH for `grader` using `ssh-keygen`

first log into the user account `grader` from your virtual server:

```
# su - grader
```

Then enter the following commands :

```
$ mkdir .ssh
$ chmod 700 .ssh
$ cd .ssh/
$ touch authorized_keys
$ chmod 644 authorized_keys
```

Now go back to your local machine and use `ssh-keygen` to generate new key and then copy the content of the public key file and Paste it in `authorized_keys` file using `nano` or any other text editor.

Now exit and try to login into grader account using your new keys.

```console
ssh grader@18.130.232.94 -p 2200 -i ~/.ssh/grader
```

Now use the following step to disable `root` login.

1. log in as `root` in the server:
   ```
   $ ssh ubuntu@18.130.232.94 -p 2200 -i ~/.ssh/privatekey
   ```

2. open the file `/etc/ssh/sshd_config` with `nano`:
   ```
   # nano /etc/ssh/sshd_config
   ```

3. Find the line `PermitRootLogin yes` and change it to `PermitRootLogin no`.

4. Restart the SSH server:
   ```
   # service ssh restart
   ```

5. Terminate the connection:
   ```
   # exit
   ```

6. when you try to log in as `root` again, you should get the following error:
   ```
   ubuntu@18.130.232.94: Permission denied (publickey).
   ```
   
### 9. Configure The local timezone to UTC

To configure the timezone to use UTC, run the following command:

```
# sudo timedatectl set-timezone UTC
```

### 10. Install and Configure Apache

To install python3, run the following command:

```
apt-get install python3
apt-get install python3-setuptools
apt install python3-pip
```

To install the Apache Web Server, run the following command:

```
apt-get install apache2
```

To install mod_wsgi, run the following command:
```
sudo apt-get install libapache2-mod-wsgi-py3
```

### 11. Install and Configure PostgreSQL

To install PostgreSQL, run the follwoing command:

   ```
   $ sudo apt install postgresql-10
   ```

To prevent remote connection, Use the following command to Check ( By default there is no remote connection )

```
nano /etc/postgresql/10/main/pg_hba.conf
```

Configuring PostgreSQL

1. Log in as the user `postgres` that was automatically created during the installation of PostgreSQL Server:

   ```
   $ sudo su - postgres
   ```

2. Open the `psql` shell:

   ```
   $ psql
   ```

3. This will open the `psql` shell. Now type the following commands one-by-one:

   ```sql
   postgres=# CREATE DATABASE catalog;
   postgres=# CREATE USER catalog;
   postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
   postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
   ```

4. exit by running `\q` followed by `exit`.

### 12. Installing Git

In Ubuntu 18.04, `git` might already be pre-installed. If it isn't, run the following commands:

```
$ sudo apt-get install git
```

### 13. Deploy The WriterApp

1. Change the current working directory to `/var/www/`:

   ```
   cd /var/www/
   ```

2. Clone your GitHub repository (Mine is : WriterApp) for example:

   ```
   git clone https://github.com/Basheer88/writerapp.git writerapp
   ```

3. Change the current working directory to the newly created directory:

   ```
   $ cd writerapp/
   ```

4. Install required packages:

   ```
   pip3 install --upgrade Flask SQLAlchemy httplib2 oauth2client requests psycopg2 psycopg2-binary
   ```
   
   Or 
   ```
   pip3 install -r requirments.txt
   ```

#### 14 Setting Up the VirtualHost Configuration

1. Run the following command in terminal to set up a file called `writerapp.conf` to configure the virtual hosts:

   ```
   nano /etc/apache2/sites-available/writerapp.conf
   ```

2. Add the following lines to it:

   ```
   <VirtualHost *:80>
      ServerName 18.130.232.94
      ServerAlias 18.130.232.94.xip.io
      ServerAdmin 18.130.232.94@domain.com
      WSGIScriptAlias / /var/www/writerapp/writerapp.wsgi
      <Directory /var/www/writerapp/>
          Require all granted
      </Directory>
      Alias /static /var/www/writerapp/static
      <Directory /var/www/writerapp/static/>
          Require all granted
      </Directory>
      ErrorLog ${APACHE_LOG_DIR}/error.log
      LogLevel warn
      CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>

   ```

3. Enable the virtual host:

   ```
   sudo a2ensite FlaskApp
   ```

4. Restart Apache server:

   ```
   sudo service apache2 restart
   ```

5. Creating the .wsgi File

   Apache uses the `.wsgi` file to serve the Flask app. now craete a file named `writerapp.wsgi` inside `/var/www/witerapp/` directory with following commands:

   ```
   $ cd /var/www/writerapp/
   $ sudo nano writerapp.wsgi
   ```

   Add the following lines to the `flaskapp.wsgi` file:

   ```python
   import sys
   import logging
   logging.basicConfig(stream=sys.stderr)
   sys.path.insert(0, "/var/www/writerapp/")

   from writerapp import app as application
   ```
   
6. Restart Apache server:

   ```
   $ sudo service apache2 restart
   ```
   
   Now you should be able to run the application at <http://18.130.232.94.xip.io/>.

## Debugging

If you get any error check out Apache's error log for debugging:

```
$ sudo cat /var/log/apache2/error.log
```

## References

[1] <https://www.digitalocean.com/community/tutorials>

[2] <http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/>

[3] <https://www.topikettunen.com/2018/02/01/h3-linux-palvelimet-ict4tn021-8.html>

[4] <http://terokarvinen.com/2016/deploy-flask-python3-on-apache2-ubuntu>

[5] udacity class.

[5] found a solution for several problems in various websites such as ( Stackoverflow ) and ( udacity knowleadge ).
