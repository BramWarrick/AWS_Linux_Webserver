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
	ssh -i ~/.ssh/udacity_key.rsa root@35.165.146.88
	```

## File prep
- [ ] Copy and Paste entire file to new, local file
- [ ] Find and Replace all below with your values:
	- [ ] IP: 35.165.146.88
	- [ ] Virtual Environment Name: item-catalog-venv
	- [ ] Filename: item-catalog
	- [ ] Authorized Key: [authorized_key]


## Initial Set-Up
### Add `grader` with permissions
```
sudo adduser grader
usermod -aG sudo grader
sudo su - grader
mkdir .ssh
chmod 700 .ssh
cat > ~/.ssh/authorized_keys <<- "EOF"
[authorized_key]
EOF
```
- [ ] Save and Exit
```
chmod 600 .ssh/authorized_keys
sudo ls -al
```

### Ensure grader has external access
- [ ] Open new terminal and log in as `grader`
	```
	ssh -i ~/.ssh/authorized_keys grader@35.165.146.88
	```
- [ ] If able to sign in, reconfirm sudo privileges
- [ ] **If either of these conditions are not met you cannot advance**


### Secure Server
####Force key-based Authentication & Remove root remote access
- [ ] Open sshd_config for editing: 
	```
	sudo nano /etc/ssh/sshd_config
	```
	- [ ] Find `#PermitRootLogin`
		- [ ] Change `without-password` to `no`
	- [ ] Find `PasswordAuthentication`
		- [ ] Change 'yes' to 'no' if not already set to 'no.'
- [ ] Save and Exit
- [ ] Restart SSH service (for above) & remove root's authorized keys with:
	```
	sudo service ssh restart
	sudo su - root
	sudo rm -f ~/.ssh/authorized_keys
	```
- Attempt log in as root
	- [ ] Open new terminal and use code below, substituting the IP Address.
		```
		ssh -i ~/.ssh/udacity_key.rsa root@35.165.146.88
		```
	- [ ] Attempt should fail
- [ ] **If you ARE able to sign in, do not advance - fix this.**
- Reference:
	- http://www.tecmint.com/disable-or-enable-ssh-root-login-and-limit-ssh-access-in-linux/

- [ ] Sign in using `grader` user
	```
	ssh -i ~/.ssh/authorized_keys grader@35.165.146.88
	```


#### SSH Port change
```
sudo nano /etc/ssh/sshd_config
```
- [ ] Find `#port 22` and change to `port 2200`
	- [ ] ensure there is no `#` on the line
	- [ ] Save and Close
- [ ] `sudo service ssh restart`
- [ ] Validate by signing in (addtl terminal) with the below command:
```
ssh -i ~/.ssh/authorized_keys grader@35.165.146.88 -p 2200
```

 
#### Firewall
##### Confirm current status
```
sudo ufw status
```
- [ ] Confirm status - initial state should show `Status: inactive`


##### Configure 
```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw delete allow 22
sudo ufw allow 2200/tcp
sudo ufw allow ntp
sudo ufw allow www
sudo ufw --force enable
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
	2200/tcp                   ALLOW       Anywhere
	123                        ALLOW       Anywhere
	80/tcp                     ALLOW       Anywhere
	2200/tcp (v6)              ALLOW       Anywhere (v6)
	123 (v6)                   ALLOW       Anywhere (v6)
	80/tcp (v6)                ALLOW       Anywhere (v6)
	```
- [ ] Confirm successful log-in
	```
	ssh -i ~/.ssh/authorized_keys grader@35.165.146.88 -p 2200
	```

### Upgrade packages, clean residual files, change local time zone
```
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get autoremove
sudo timedatectl set-timezone UTC
```


## Script (continuous) for subsequent steps packages
### Install packages, project, initial wsgi config
```
# Initial installs, prep virtualenv
sudo apt-get update
sudo apt-get -y install apache2
sudo apt-get -y install libapache2-mod-wsgi python-dev
sudo a2enmod wsgi
sudo apache2ctl restart
sudo apt-get -y install git
sudo apt-get -y install python-pip
yes | sudo pip install virtualenv
# Install Item Catalog Repository
sudo mkdir -p /var/apps/item-catalog/staged
sudo mkdir -p /var/apps/item-catalog/version/0001
sudo mkdir -p /var/apps/item-catalog/version/archive
sudo mkdir -p /var/apps/item-catalog/recent
sudo ln -s /var/apps/item-catalog/version/0001 /var/apps/item-catalog/live
sudo ln -s /var/apps/item-catalog/live /var/www/html/item-catalog
sudo git clone https://github.com/BramWarrick/Item_Catalog_PostGreSQL.git /var/apps/item-catalog/staged
sudo virtualenv /var/apps/item-catalog/staged/item-catalog-venv
# Install missing packages on virtual environment (item-catalog-venv)
source /var/apps/item-catalog/staged/item-catalog-venv/bin/activate
yes | sudo pip install Flask
sudo apt-get -y install postgresql postgresql-contrib
sudo apt-get -y install python-psycopg2
yes | sudo pip install SQLAlchemy
sudo apt-get -y install python-oauth2client
deactivate
# Install completed project
sudo cp -r /var/apps/item-catalog/staged/* /var/apps/item-catalog/recent
sudo cp -r /var/apps/item-catalog/staged/* /var/apps/item-catalog/version/0001/
```

### Apache2 Configuration
#### Modify `conf` file
```
sudo nano /etc/apache2/sites-enabled/000-default.conf
```
- [ ] Paste the code (below) into the section immediately following the `DocumentRoot` line
```
		#WSGIDaemonProcess item-catalog threads=5
        WSGIScriptAlias / /var/www/html/item-catalog/main.wsgi

        <Directory item-catalog>
                WSGIProcessGroup item-catalog
                WSGIApplicationGroup %{GLOBAL}
                Order deny,allow
                Allow from all
        </Directory>
```
- [ ] Save and Close
- [ ] `sudo apache2ctl restart`

### Validation
Navigate to `35.165.146.88`

### Create and Secure PostGres DB
```
sudo su - postgres
psql
```

- [ ] Paste the code below into the psql window
```
CREATE ROLE login_role WITH login;
CREATE ROLE access_role;
CREATE DATABASE catalog WITH OWNER access_role;
\c catalog
REVOKE ALL ON SCHEMA public FROM public;
GRANT ALL ON SCHEMA public TO access_role;
RESET ROLE;
GRANT access_role TO login_role;
CREATE USER catalog PASSWORD '4tunat3lySaf31sh';
GRANT login_role TO catalog;
```
- [ ] Quit


__________________________________________________________

## Manual Processing - useful as resource

### Apache
#### Install
```
sudo apt-get update
sudo apt-get install apache2
```

- [ ] Confirm page works - default apache page
	- Navigate to server's IP address in a browser
	```35.165.146.88```

### mod_wsgi - Install and Configure
#### Install 
```
sudo apt-get -y install libapache2-mod-wsgi python-dev
sudo a2enmod wsgi
sudo apache2ctl restart
```

#### Verification
- [ ] Confirm page works - no errors
	- Navigate to server's IP address in a browser
	```35.165.146.88```


### Git - Install
```
sudo apt-get install git
```

### Pip - install
- [ ] Ensure pip is installed locally
```
sudo apt-get install python-pip
```

### Virtualenv - install
- [ ] Ensure virtualenv is installed locally
```
sudo pip install virtualenv
```


### File Structure
#### Create Flask App
Reference: https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps  
Reference: http://www.datasciencebytes.com/bytes/2015/02/24/running-a-flask-app-on-aws-ec2/  
#### Create Flask Directory Structure
- [ ] Create the directories as specified below, substituting your application's name
```
cd /var/www/html
sudo git clone https://github.com/BramWarrick/Item_Catalog_PostGreSQL.git
sudo mv Item_Catalog_PostGreSQL item-catalog
```

#### Create virtual environment
```
cd /var/www/html/item-catalog
sudo virtualenv item-catalog-venv
```


### Python modules
#### Install Flask
- [ ] Install Flask on virtual environment
```
source /var/www/html/item-catalog/item-catalog-venv/bin/activate
sudo pip install Flask
deactivate
```


#### PostGreSQL
```
source /var/www/item-catalog/item-catalog-venv/bin/activate
sudo apt-get update
sudo apt-get install postgresql postgresql-contrib
```
- Reference: https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps

#### Psycog2
```
source /var/www/html/item-catalog/item-catalog-venv/bin/activate
sudo apt-get -y install python-psycopg2
```

#### SQLAlchemy
```
source /var/www/html/item-catalog/item-catalog-venv/bin/activate
sudo pip install SQLAlchemy
```

#### oauth2client (google)
```
source /var/www/html/item-catalog/item-catalog-venv/bin/activate
sudo apt-get update
sudo apt-get install python-oauth2client
```

## Secure PostGreSQL database
```
sudo su - postgres
psql
```

```
CREATE ROLE login_role WITH login;
CREATE ROLE access_role;
CREATE DATABASE catalog WITH OWNER access_role;
\c catalog
REVOKE ALL ON SCHEMA public FROM public;
GRANT ALL ON SCHEMA public TO access_role;
RESET ROLE;
GRANT access_role TO login_role;
CREATE USER catalog;
GRANT login_role TO catalog;
```
