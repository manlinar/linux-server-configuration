# Linux Server Configuration

Linux Server Configuration,part of the Udacity [Full Stack Web Developer Nanodegree](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004). 

This page explains how to secure and set up a Linux server on a virtual machine, install and configure a database server to host a web application. 
- The Linux distribution is [Ubuntu](https://www.ubuntu.com/download/server) 16.04.5 LTS.
- IP address: http://18.185.188.214
- PORT: 2200
- Application URL: http://ec2-18-185-188-214.eu-central-1.compute.amazonaws.com/

##### Issue

- This web application is not reachable under the [AWS-server](http://ec2-18-185-188-214.eu-central-1.compute.amazonaws.com/) anymore, since I have now graduated.

### Step 1: Start a new Ubuntu Linux server instance on Amazon Lightsail 

- Login into [Amazon Lightsail](https://lightsail.aws.amazon.com/ls/webapp/home/resources) using an Amazon Web Services account.
- Once you are login into the site, click `Create instance`. 
- Choose `Linux/Unix` platform, `OS Only` and  `Ubuntu 16.04 LTS`.
- Choose a instance plan (e.g $3.5/month).
- Keep the default name provided by AWS or rename your instance.
- Click the `Create` button to create the instance.
- Wait for the instance to start up.

**Reference**
- Follow the instructions provided in Udacity or [How to create a server on Amazon lightsail](https://serverpilot.io/docs/how-to-create-a-server-on-amazon-lightsail)


### Step 2: SSH into the server

- From the `Account` menu on Amazon Lightsail, click on `SSH keys` tab and download the Default Private Key.
- Move this private key file named `LightsailDefaultPrivateKey-*.pem` into the local folder `~/.ssh` and rename it `Lightsail_Key.rsa`.
- You need to change permission of the downloaded lightsail private key: `chmod 600 ~/.ssh/Lightsail_Key.rsa`  or  `chmod 400 ~/.ssh/Lightsail_Key.rsa`.
- In order to connect to your instance via the terminal: `ssh -i ~/.ssh/Lightsail_Key.rsa ubuntu@18.185.188.214`, 
  where `18.185.188.214` is the public IP address of the instance

### Step 3: Update and upgrade installed packages
- `sudo apt-get update`
- `sudo apt-get upgrade`
 

### Step 4: Create a new user grader and give it permission to sudo

- While logged in as `ubuntu` type:  `sudo adduser grader`. It will ask for a password.Type it twice. 
- After that give `grader` sudo privilege. Type `sudo visudo`.
Go to section user privilege specification and after `root    ALL=(ALL:ALL) ALL`   type `grader    ALL=(ALL:ALL) ALL`.
- To see if the new user is created type `sudo login grader` then type the password you gave earlier and you are logged in as `grader`


### Step 5: Change SSH port from 22 to 2200
- Go to LightSail page where your instance is and go to the Networking tab.On the firewall click `+ Add another` and `Custom    TCP    2200`.The other two options should be ` HTTP    TCP    80` and `Custom    UDP    123` and we do these changes in order to deny port 22.
- While logged in as `Ubuntu` type: `sudo nano /etc/ssh/sshd_config` .Then change the following options.
    ```
    # What ports, IPs and protocols we listen for
    # Port 22
    Port 2200

    # Authentication:
    LoginGraceTime 120
    #PermitRootLogin prohibit-password
    PermitRootLogin no
    StrictModes yes

    # Change to no to disable tunnelled clear text passwords
    PasswordAuthentication No
   ```
- Save and exit.Restart the server:`sudo service ssh restart`
- You can now login by typing: `ssh -i ~/.ssh/Lightsail_Key.rsa -p 2200 ubuntu@18.185.188.214`.

### Step 6: Configure the Uncomplicated Firewall (UFW)

- While logged in as `ubuntu` type the following commands:
     ```
  sudo ufw status                  # The UFW should be inactive.
  sudo ufw default deny incoming   # Deny any incoming traffic.
  sudo ufw default allow outgoing  # Enable outgoing traffic.
  sudo ufw allow 2200/tcp          # Allow incoming tcp packets on port 2200.
  sudo ufw allow www               # Allow HTTP traffic in.
  sudo ufw allow 123/udp           # Allow incoming udp packets on port 123.
  sudo ufw deny 22                 # Deny tcp and udp packets on port 53.
  ```

- Turn UFW on: `sudo ufw enable`. The output should be like this:
  ```
  Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
  Firewall is active and enabled on system startup
  ```

- Check the status of UFW to list current roles: `sudo ufw status`. The output should be like this:

  ```
  Status: active
  
  To                         Action      From
  --                         ------      ----
  2200/tcp                   ALLOW       Anywhere                  
  80/tcp                     ALLOW       Anywhere                  
  123/udp                    ALLOW       Anywhere                  
  22                         DENY        Anywhere                  
  2200/tcp (v6)              ALLOW       Anywhere (v6)             
  80/tcp (v6)                ALLOW       Anywhere (v6)             
  123/udp (v6)               ALLOW       Anywhere (v6)             
  22 (v6)                    DENY        Anywhere (v6)
  ```


### Step 7: Configure firewall to monitor for repeated unsuccessful login attempts and ban attackers

Install `Fail2Ban` to protect computer servers from brute-force attacks.
- Update first: `sudo apt-get update`
- Install Fail2Ban: `sudo apt-get install fail2ban`.
- We need the sendmail package to send the alerts to the admin user: `sudo apt-get install sendmail iptables-persistent`.
- Create a file to safely customize the fail2ban functionality: `sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`.
- Open the jail.local and edit it: `sudo nano /etc/fail2ban/jail.local`
  ```
  set bantime = 600
  maxretry = 3
  destemail = youremail@domain
  action = %(action_mwl)s 
  ```
- Under `[sshd]` change `port = ssh` by `port = 2200`.
- Save and exit.
- Restart the service: `sudo service fail2ban restart`.


### Step 8: Create SSH keys for grader
- On your local machine: `ssh-keygen -f ~/.ssh/grader_key.rsa`
- Then `cat ~/.ssh/grader_key.rsa.pub` and copy the key.
- While logged in as `grader` type the following commands: 
- `mkdir ~/.ssh`
- `chmod 700 ~/.ssh`
- `touch ~/.ssh/authorized_keys`
- `chmod 600 ~/.ssh/authorized_keys`
- `sudo nano ~/.ssh/authorized_keys`
- Then paste the key, save and exit.
- Check if PasswordAuthentication is set to No by typing `sudo nano /etc/ssh/sshd_config`.
- Restart: `sudo service ssh restart`
- On your local machine when you type 
`ssh -i ~/.ssh/grader_key.rsa -p 2200 grader@18.185.188.214` 
you will login as `grader`.

### Step 9: Configuring timezone for server
- While logged in as `grader`, type:`sudo dpkg-reconfigure tzdata`.
Choose none of the above and then choose your local time zone.
You will see something like:
    ```
    Current default time zone: 'Etc/GMT-2'
    Local time is now:      Thu Nov  1 14:32:06 +02 2018.
    Universal Time is now:  Thu Nov  1 12:32:06 UTC 2018.
    ```

### Step 10: Install and configure Apache, mod_wsgi
- While logged in as `grader` type: `sudo apt-get install apache2`
- Open a tab and type http://ec2-18-185-188-214.eu-central-1.compute.amazonaws.com/ and it show a message "It works!"
- Install mod_wsgi: `sudo apt-get install libapache2-mod-wsgi python-dev`.
- Enable mod_wsgi: `sudo a2enmod wsgi`
- `sudo service apache2 start`


### Step 11: Install and configure PostgreSQL
- While logged in as `grader`, install PostgreSQL:
 `sudo apt-get update`
 `sudo apt-get install postgresql postgresql-contrib`
- PostgreSQL should not allow remote connections.Type:
 `sudo nano /etc/postgresql/9.5/main/pg_hba.conf` and you should see:
  ```
  local   all             postgres                                peer
  local   all             all                                     peer
  host    all             all             127.0.0.1/32            md5
  host    all             all             ::1/128                 md5
  ```
   Remove comments if neccesary.
-  Peer Authentication: Upon installation PostGres creates a Linux user called `postgres` which can be used to access the system.We change to this user by typing
`sudo su -postgres`.Then type:`psql`.Exit psql:`\q`.Type `exit` and back to `grader`.
- Create a user named `catalog` that has limited permissions to my catalog database.
`sudo -u postgres createuser -P catalog`.Type grader's password and type twice your new user's password.I typed `password`.
- Now create an empty database named `catalog` with the command:
`sudo -u postgres createdb -O catalog catalog`


### Step 12: Install git
- While logged in as `grader`, type:  `sudo apt-get install git`.


### Step 13: Deploy the Item Catalog Project 

While logged in as `grader` :
 - Create a directory in order to clone the catalog project:
 `sudo mkdir /var/www/catalog/`
 `cd /var/www/catalog`
 `sudo git clone https://github.com/manlinar/item-catalog catalog`
- Go to `/var/www` directory to change the ownership of catalog directory to grader using the command: `sudo chown -R grader:grader catalog/`
- Change directory: `cd /var/www/catalog/catalog`
- Rename `application.py` to `__init__.py` by typing:
`mv application.py __init__.py`
- `sudo nano __init__.py` and at the end of the file comment out
    ```
    # app.debug = True
    # app.run(host='0.0.0.0',port=8000)
    ```
- Save and exit.
- Go to files `database_setup.py`, `fakehotelinfo.py` and `__init__.py` with `sudo nano` command, comment out 
`# engine = create_engine('sqlite:///hotelcatalogwithusers.db')`
and type:
`engine = create_engine('postgresql://catalog:password@localhost/catalog')`.
Do this to each of these files.

##### Authenticate login with Google
- Go to [Google Cloud Platform](https://console.cloud.google.com/).
- Click `APIs & services` on left menu.
- Click `Credentials`.
- Create an OAuth Client ID (under the Credentials tab), choose Web application and add http://18.185.188.214 and 
http://ec2-18-185-188-214.eu-central-1.compute.amazonaws.com/ as authorized JavaScript 
origins.
- Add http://ec2-18-185-188-214.eu-central-1.compute.amazonaws.com/oauth2callback ,
http://ec2-18-185-188-214.eu-central-1.compute.amazonaws.com/oauth2callback/login,
http://ec2-18-185-188-214.eu-central-1.compute.amazonaws.com/oauth2callback/gconnect
as authorized redirect URIs.
- Download the corresponding JSON file, open it and copy the contents.
- Open ` sudo nano /var/www/catalog/catalog/client_secret.json` and paste the previous contents into the this file.
- Replace the client ID to line 25 of the `templates/login.html` file in the project directory.Save and exit.
- Update the path of clientID: `sudo nano __init__.py` and change in lines 20 and 71
and write `/var/www/catalog/catalog/client_secret.json`.Save and exit.


### Step 14: Install virtual enviroment
- While logged in as `grader`, install pip: `sudo apt-get install python-pip`.
- Install the virtual environment: `sudo apt-get install python-virtualenv`.
- `cd /var/www/catalog/catalog/`
- Create the virtual environment: `sudo virtualenv venv`.
- Activate the virtual environment: `source venv/bin/activate`.
- Change permissions to the virtual environment folder: `sudo chmod -R 777 venv`.
- Install the following dependancies:
    ```
    pip install Flask
    pip install bleach httplib2 oauth2client psycopg2 requests sqlalchemy
    pip install psycopg2-binary
    ```
- Run `python __init__.py` and you should see:
    ```
    * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
    ```
- Deactivate the virtual environment: `deactivate`.

##### Setup and enable a virtual host
- Add the following line in /etc/apache2/mods-enabled/wsgi.conf file
to use Python 2.7.
    ```
    #WSGIPythonPath directory|directory-1:directory-2:...
    WSGIPythonPath /var/www/catalog/catalog/venv/lib/python2.7/site-packages
    ```
- `sudo touch /etc/apache2/sites-available/catalog.conf` and add the following:
    ```
    <VirtualHost *:80>
        ServerName 18.185.188.214
      ServerAlias ec2-18-185-188-214.eu-central-1.compute.amazo$
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
 - Enable virtual host: `sudo a2ensite catalog`. The following prompt will be returned:`Enabling site catalog`.
 - Reload Apache: `sudo service apache2 reload`.
 
#### Setup Flask App
- `sudo touch /var/www/catalog/catalog.wsgi` and add:
    ```
    activate_this = '/var/www/catalog/catalog/venv/bin/activate$
    with open(activate_this) as file_:
    exec(file_.read(), dict(__file__=activate_this))

    #!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0, "/var/www/catalog/catalog/")
    sys.path.insert(1, "/var/www/catalog/")

    from catalog import app as application
    application.secret_key = "super_secret_key"
    ```
#### Disable the default Apache site
- `sudo a2dissite 000-default.conf`
- Reload Apache: `sudo service apache2 reload`.

#### Launch the Web Application

- Restart Apache again: `sudo service apache2 restart`.
- Open your browser to http://18.185.188.214 or http://ec2-18-185-188-214.eu-central-1.compute.amazonaws.com/.

#### Make additional updates
 - In order to update some packages I ran the following commands:
 - `sudo apt-get update`
 - `sudo apt-get dist-upgrade`
 - `sudo shutdown -r now`

## Helpful Resources
- [Udacity Forum](https://study-hall.udacity.com/sg-617415-1968/rooms/community:nd004:617415-cohort-1968-project-2007?contextType=room)
- DigitalOcean, [How To Add and Delete Users on an Ubuntu 14.04 VPS](https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps)
- DigitalOcean, [How To Protect SSH with Fail2Ban on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04).
- DigitalOcean [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
- Flask documentation, [virtualenv](http://flask.pocoo.org/docs/0.12/installation/)
- GitHub Repositories 
  - [iliketomatoes/linux_server_configuration](https://github.com/iliketomatoes/linux_server_configuration)
  - [SteveWooding/fullstack-nanodegree-linux-server-config](https://github.com/SteveWooding/fullstack-nanodegree-linux-server-config)
  - [rrjoson/udacity-linux-server-configuration](https://github.com/rrjoson/udacity-linux-server-configuration/blob/master/README.md)
  - [harushimo/linux-server-configuration](https://github.com/harushimo/linux-server-configuration)
###### I would like to thank very much   [boisalai](https://github.com/boisalai) who wrote a great README in his Github [Repo](https://github.com/boisalai/udacity-linux-server-configuration) and helped me understand lots of things about Linux Server Confiruration.