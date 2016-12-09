Find/Replace references:
- IP: 35.160.63.155
- Filename: item-catalog
- Virtual Environment Name: item-catalog-venv



# AWS Linux Webserver

## Prep-Work
### Create authorized key
- [ ] Open terminal to local machine and run:
```
ssh-keygen -f ~/.ssh/authorized_keys
```
- [ ] Enter and make note of passphrase  
  ________________________________________
- [ ] Open public key file for later step
```
vi ~/.ssh/authorized_keys.pub
```
- [ ] *Leave this terminal open for use later*

### Create AWS Instance on Udacity site
- Go to `https://www.udacity.com/account#!/development_environment`
- [ ] Click 'Create Development Environment'
- [ ] Make note of IP address (paste here)
- [ ] Download `udacity_key.rsa` to `Downloads` folder
- [ ] Move `udacity_key.rsa` to local machine
	```
	mv ~/Downloads/udacity_key.rsa ~/.ssh/
	chmod 600 ~/.ssh/udacity_key.rsa
	ssh -i ~/.ssh/udacity_key.rsa root@35.160.63.155
	```

## Initial Set-Up

- [ ] Open new terminal window
- [ ] Connect using the connection command provided by Udacity
IP Address: ________________________________
- [ ] Run `echo "127.0.0.1 $(hostname)"` >> /etc/hosts` to clean up legacy AWS bug

### Add `grader`

```
sudo adduser grader
```
- [ ] Answer questions and enter password
```
usermod -aG sudo grader
```

#### Provide permissions to connect externally
```
sudo su - grader
mkdir .ssh
chmod 700 .ssh
touch .ssh/authorized_keys
chmod 600 .ssh/authorized_keys
nano ~/.ssh/authorized_keys
```
- [ ] Copy contents of local terminal screen (from Prepwork) into nano screen
	[ ] Save and Exit


#### Ensure grader has external access
- [ ] Open new terminal and log in as `grader`
	```
	ssh -i ~/.ssh/authorized_keys grader@35.160.63.155
	```
- [ ] If able to sign in, reconfirm sudo privileges
- [ ] **If either of these conditions are not met you cannot advance**


#### Force key-based Authentication/Remove root remote access
- [ ] Open sshd_config for editing: `sudo nano /etc/ssh/sshd_config`
- [ ] Find `Password Authentication`
	- [ ] Change 'yes' to 'no' if not already set to 'no.'
- [ ] Find `#PermitRootLogin`
	- [ ] Change `without-password` to `no`
- [ ] Save and Exit
- [ ] Restart SSH service with `sudo service ssh restart`
- [ ] Remove root's authorized keys
	```
	sudo su - root
	sudo rm -f ~/.ssh/authorized_keys
	```
- Attempt log in as root
	- [ ] Open new terminal and use code below, substituting the IP Address.
		```
		ssh -i ~/.ssh/udacity_key.rsa root@35.160.63.155
		```
	- [ ] Attempt should fail
- [ ] **If you ARE able to sign in, do not advance - fix this.**
- Reference:
	- http://www.tecmint.com/disable-or-enable-ssh-root-login-and-limit-ssh-access-in-linux/

### Upgrade packages, clean residual files
```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get autoremove
```

### Change local time
sudo dpkg-reconfigure tzdata
```
- User Prompt
	- Scroll to the bottom of Continents list
	- Select `None of the above`; in the second list
	- Select `UTC`


## SSH Port change
```
sudo nano /etc/ssh/sshd_config
```
- [ ] Find `#port 22` and change to `port 2200`
	- [ ] ensure there is no `#` on the line
	- [ ] Save and Close

## Firewall
### Confirm current status
```
sudo ufw status
```
- [ ] Confirm status - initial state should show `Status: inactive`


### Configure 
```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow 2200/tcp
sudo ufw allow ntp
sudo ufw allow www
sudo ufw enable
sudo ufw status
```
- [ ] Confirm status, should now be enabled with details that align with configuration
	- [ ] SSH; port 2200
	- [ ] HTTP; port 80
	- [ ] NTP; port 123
	- Example below:
	```
		To                         Action      From
	--                         ------      ----
	22                         ALLOW       Anywhere
	2200/tcp                   ALLOW       Anywhere
	123                        ALLOW       Anywhere
	80/tcp                     ALLOW       Anywhere
	22 (v6)                    ALLOW       Anywhere (v6)
	2200/tcp (v6)              ALLOW       Anywhere (v6)
	123 (v6)                   ALLOW       Anywhere (v6)
	80/tcp (v6)                ALLOW       Anywhere (v6)
	```
- [ ] Confirm successful log-in
	```
	ssh -i ~/.ssh/authorized_keys grader@35.160.63.155
	```

## Apache
### Install
```
sudo apt-get update
sudo apt-get install apache2
```

- [ ] Confirm page works - default apache page
	- Navigate to server's IP address in a browser
	```35.160.63.155```

## mod_wsgi - Install and Configure
### Install 
```
sudo apt-get install libapache2-mod-wsgi python-dev
```

### Enable
```
sudo a2enmod wsgi
sudo apache2ctl restart
```

### Verification
- [ ] Confirm page works - no errors
	- Navigate to server's IP address in a browser
	```35.160.63.155```


## Git - Install
```
sudo apt-get install git
```

## Pip - install
- [ ] Ensure pip is installed locally
```
sudo apt-get install python-pip

```

## Virtualenv - install
- [ ] Ensure virtualenv is installed locally
```
sudo pip install virtualenv
```


## Flask
### Create Flask App
Reference: https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps  
#### Create Flask Directory Structure
- [ ] Create the directories as specified below, substituting your application's name
```
cd /var/www
sudo mkdir item-catalog
cd item-catalog
git clone https://github.com/BramWarrick/Item_Catalog.git
sudo mv Item_Catalog item-catalog
```

#### Create __init__.py file
```
sudo nano /var/www/item-catalog/item-catalog/__init__.py
```
- [ ] Add code below to file, save and exit
```
from flask import Flask
app = Flask(__name__)
@app.route("/")
def hello():
    return "Hello, World!!!"
if __name__ == "__main__":
    app.run()
```

### Install Flask
- [ ] Create virtual environment
```
cd /var/www/item-catalog/item-catalog
sudo virtualenv item-catalog-venv
```

- [ ] Install Flask on virtual environment
```
source /var/www/item-catalog/item-catalog/item-catalog-venv/bin/activate
sudo pip install Flask
```

- [ ] Confirm install
 ```
 sudo python /var/www/item-catalog/item-catalog/__init__.py
 ```
 - Should show “Running on http://localhost:5000/” or "Running on http://127.0.0.1:5000/"
 - Ctrl+C to exit afterward
- To leave virtual environment, enter `deactivate`.

## Virtual Host
### Create Config File
```
sudo nano /etc/apache2/sites-available/item-catalog.conf
```
- [ ] Add these lines of code to file
```
<VirtualHost *:80>
	ServerName localhost
	ServerAdmin grader@localhost
	WSGIScriptAlias / /var/www/item-catalog/item-catalog.wsgi
	<Directory /var/www/item-catalog/item-catalog/>
		Order allow,deny
		Allow from all
	</Directory>
	Alias /static /var/www/item-catalog/item-catalog/static
	<Directory /var/www/item-catalog/item-catalog/static/>
		Order allow,deny
		Allow from all
	</Directory>
	ErrorLog ${APACHE_LOG_DIR}/error.log
	LogLevel warn
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
- [ ] Save and Exit

### Enable virtual host
```
sudo a2ensite item-catalog
```

### Create .wsgi File
```
sudo nano /var/www/item-catalog/item-catalog.wsgi  
```
- [ ] Modify lines below to reflect
	- [ ] Secret Key
```
activate_this = '/var/www/item-catalog/item-catalog/item-catalog-venv/bin/activate_this.py'
execfile(activate_this, dict(__file__=activate_this))

#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/item-catalog/")

from item-catalog import app as application
application.secret_key = '[Secret Key]'
```
- [ ] Copy into nano window, save and exit

## Apache restart
```
sudo service apache2 restart
```

## Python modules
### PostGreSQL
```
source /var/www/item-catalog/item-catalog/item-catalog-venv/bin/activate
sudo apt-get update
sudo apt-get install postgresql postgresql-contrib
```
- [ ] Confirm page works - no errors

- [To Do] Add configuration instructions
	- Do not allow remote connections
	```
	sudo nano /etc/postgresql/9.1/main/pg_hba.conf
	```
	- Create new user 'catalog'
		- Has limited permissions on application database
- Reference: https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps

### SQLAlchemy
```
source /var/www/item-catalog/item-catalog/item-catalog-venv/bin/activate
sudo pip install SQLAlchemy
```
### Flask
- Installed earlier in process

### oauth2client (google)
```
source /var/www/item-catalog/item-catalog/item-catalog-venv/bin/activate
sudo apt-get update
sudo apt-get install python-oauth2client
```
